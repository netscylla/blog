---
layout: post
title:  "Offensive Implants Part 2"
date:   2018-01-30 13:49:33 +0000
tags: [hardware, pentest, redteam, blueteam]
---
![](/blog/assets/croc.png)

In the previous post Part 1, we looked at some hardware that was very similar to the Raspberry Pi, we lightly touched on how to prepare the linux kernel, the commands to compile a custom kernel, and how to install a base Linux image. We also wrote about ConfigFS and how we planned to turn the small disposable platform into a customisable BadUSB device for the purposes of offensive security testing. Part 2 looks at some of the scripts jig-sawed/hacked together to provide the features and functionalities we are after from a BadUSB based device.

# Disclaimer
The information contained in this website is for general information purposes only. The information is provided by Netscylla and whilst we endeavour to keep it up-to-date and correct, we make no representations or warranties of any kind, express or implied, about the completeness, accuracy, reliability, suitability or availability with respect to the website or the information, products, services, or related graphics contained on the website for any purpose. Any reliance you place on such information is therefore strictly at your own risk.

In no event will we be liable for any loss or damage including without limitation, indirect or consequential loss or damage, or any loss or damage whatsoever arising from loss of data or profits arising out of, or in connection with, the use of this website.

# The birth of the Crazy-Croc

## The ConfigFS Program — AttackMode

This is the program (or Bash script) that features everything we mentioned in the previous post, by choosing an attack-mode, you can choose what kind of composite USB device you wish to emulate.

```
#!/bin/bash
# Max number of attack modes is 3. E.g.
#   HID + STORAGE + RNDIS_ETHERNET
# Input parameters:
# $1:           HID
# $2: optional: STORAGE
# $3: optional: RNDIS_ETHERNET or CDC_ETHERNET
# $4,$5: optional: VID_0x04b3 PID_0x4010

usage() {
	echo "Usage: ATTACKMODE <mode1> [mode2] [mode3]"
	echo
	echo "List of supported combinations of attack modes are:"
	echo "  ATTACKMODE SERIAL STORAGE               (this is setup mode)"
	echo "                                          (vid/pid: 0xF000/0xFFF0)"
	echo
	echo "  ATTACKMODE HID                          (vid/pid: 0xF000/0xFF01)"
	echo "  ATTACKMODE STORAGE						(vid/pid: 0xF000/0xFF10)"
	echo "  ATTACKMODE SERIAL						(vid/pid: 0xF000/0xFF11)"
	echo "  ATTACKMODE RNDIS_ETHERNET               (For Windows)"
	echo "                                          (vid/pid: 0xF000/0xFF12)"
	echo "  ATTACKMODE ECM_ETHERNET                 (for Mac and Linux)"
	echo "                                          (vid/pid: 0xF000/0xFF13)"
	echo "  ATTACKMODE HID SERIAL                   (vid/pid: 0xF000/0xFF14)"
	echo "  ATTACKMODE HID STORAGE                  (vid/pid: 0xF000/0xFF02)"
	echo "  ATTACKMODE HID RNDIS_ETHERNET           (For Windows)"
	echo "                                          (vid/pid: 0xF000/0xFF03)"
	echo "  ATTACKMODE HID ECM_ETHERNET             (for Mac and Linux)"
	echo "                                          (vid/pid: 0xF000/0xFF04)"
	echo "  ATTACKMODE HID STORAGE RNDIS_ETHERNET   (For Windows)"
	echo "                                          (vid/pid: 0xF000/0xFF05)"
	echo "  ATTACKMODE HID STORAGE ECM_ETHERNET     (for Mac and Linux)"
	echo "                                          (vid/pid: 0xF000/0xFF06)"
	echo "  ATTACKMODE SERIAL RNDIS_ETHERNET        (For Windows)"
	echo "                                          (vid/pid: 0xF000/0xFF07)"
	echo "  ATTACKMODE SERIAL ECM_ETHERNET          (for Mac and Linux)"
	echo "                                          (vid/pid: 0xF000/0xFF08)"
	echo "  ATTACKMODE STORAGE RNDIS_ETHERNET       (For Windows)"
	echo "                                          (vid/pid: 0xF000/0xFF20)"
	echo "  ATTACKMODE STORAGE ECM_ETHERNET         (for Mac and Linux)"
	echo "                                          (vid/pid: 0xF000/0xFF21)"
	echo "  un-supported                            (vid/pid: 0xF000/0xFEFF)"
}

# If there is no argument, print usage and exit.
if [ "x$1" = "x" ] ; then
	usage
	exit 1
fi


# If ATTACKMODE is already called ...
if [ "x$status" != "x" ] ; then
	# If dhcp service is started, then stop it.
	dhcp_service=$(systemctl list-units -t service | grep isc-dhcp-server.service)
	if [ "x$dhcp_service" != "x" ]; then
		/etc/init.d/isc-dhcp-server stop
	fi

	# If agetty service of USB serial is started, stop it.
	serial_service=$(systemctl list-units -t service | grep "serial-getty@ttyGS0.service")
	if [ "x$serial_service" != "x" ]; then
		systemctl stop serial-getty@ttyGS0.service
	fi

	# Give USB host some time for software way of "unplug / replug USB".
	sleep 5
fi


####################################################################
# Below are default parameters for configfs
####################################################################

e_hid=0
e_storage=0
e_ethernet=0
e_serial=0

# WARNING: Re-install of RNDIS driver on Windows is mandatory if serialnumber is changed!
serialnumber="ch000001"
manufacturer="BadUSB Demo"
host_addr="00:11:22:33:44:55"
dev_addr="5a:00:00:5a:5a:00"

vid_default="0xF000"
pid_default="0xFEFF"
pid_setup_mode="0xFFF0"
pid_hid_only="0xFF01"
pid_hid_storage="0xFF02"
pid_hid_rndis="0xFF03"
pid_hid_ecm="0xFF04"
pid_hid_storage_rndis="0xFF05"
pid_hid_storage_ecm="0xFF06"
pid_serial_rndis="0xFF07"
pid_serial_ecm="0xFF08"
pid_storage_only="0xFF10"
pid_serial_only="0xFF11"
vid_rndis_only="0x04b3"
pid_rndis_only="0x4010"
pid_ecm_only="0xFF13"
pid_hid_serial="0xFF14"
pid_storage_rndis="0xFF20"
pid_storage_ecm="0xFF21"

####################################################################
# Function to start hid_reader
####################################################################
hid_reader() {
	kill $(cat /var/run/hid_reader.pid 2>/dev/null) &>/dev/null
	touch /tmp/hid_out
	python /usr/local/bunny/hid_reader.py &
	echo $! > /var/run/hid_reader.pid
}

####################################################################
# Parse and run attack modes.
####################################################################

parse_mode() {
	if [ "x$1" = "x" ] ; then
		return
	fi

	case $1 in
	HID)
		has_hid=1
		;;
	STORAGE)
		has_storage=1
		;;
	RO_STORAGE)
		has_storage=1
		has_rostorage=1
		;;
	RNDIS_ETHERNET)
		has_net=1
		is_win=1
		;;
	ECM_ETHERNET)
		has_net=1
		is_win=
		;;
	AUTO_ETHERNET)
		has_net=1
		is_auto=1
		is_win=
		;;
	SERIAL)
		has_serial=1
		;;
	VID_*)
		str_vid_custom=$1
		str_vid_custom=${str_vid_custom#VID_}
		;;
	PID_*)
		str_pid_custom=$1
		str_pid_custom=${str_pid_custom#PID_}
		;;
	SN_*)
		str_sn_custom=$1
		str_sn_custom=${str_sn_custom#SN_}
		;;
	MAN_*)
		str_man_custom=$1
		str_man_custom=${str_man_custom#MAN_}
		;;
	RNDIS_SPEED_*)
		str_speed_custom=$1
		str_speed_custom=${str_speed_custom#RNDIS_SPEED_}
		;;
	ETHERNET_TIMEOUT_*)
		str_timeout_custom=$1
		str_timeout_custom=${str_timeout_custom#ETHERNET_TIMEOUT_}
		;;
	OFF)
		exit 0
		;;
	esac
}

attack_mode_params() {

	if [ "x$has_hid" = "x1" ]; then
		#mod_params="$mod_params is_hid=1"
		e_hid=1
		hid_reader
        else
		e_hid=0
	fi

	if [ "x$has_storage" = "x1" ]; then
		#mod_params="$mod_params is_storage=1 $str_mass_storage"
		e_storage=1
	else
		e_storage=0
	fi

	if [ "x$has_rostorage" = "x1" ]; then
		#mod_params="$mod_params ro=1"
		echo "readonly storage"
	fi

	if [ "x$has_net" = "x1" ]; then
		e_ethernet=1
	else
		e_ethernet=0
	fi

	if [ "x$has_serial" = "x1" ]; then
		#mod_params="$mod_params is_cdc_serial=1"
		e_serial=1
	else
		e_serial=0
	fi

	#mod_params="$mod_params idVendor=$vid_default"
	cd /sys/kernel/config/usb_gadget/
	mkdir -p croc
	cd croc
	echo $vid_default > idVendor # Linux Foundation

	if [ "x$has_hid" != "x1" ] && [ "x$has_storage" = "x1" ] && [ "x$has_net" != "x1" ] && [ "x$has_serial" = "x1" ]; then
		# setup mode, storage + serial
		echo 0xFFF0 > idProduct # Multifunction Composite Gadget

	elif [ "x$has_hid" = "x1" ] && [ "x$has_storage" != "x1" ] && [ "x$has_net" != "x1" ] && [ "x$has_serial" != "x1" ]; then
		# hid only
		#mod_params="$mod_params idProduct=$pid_hid_only"
		echo $pid_hid_only > idProduct

	elif [ "x$has_hid" != "x1" ] && [ "x$has_storage" = "x1" ] && [ "x$has_net" != "x1" ] && [ "x$has_serial" != "x1" ]; then
		# storage only
		#mod_params="$mod_params idProduct=$pid_storage_only"
		echo $pid_storage_only > idProduct

	elif [ "x$has_hid" != "x1" ] && [ "x$has_storage" != "x1" ] && [ "x$has_net" != "x1" ] && [ "x$has_serial" = "x1" ]; then
		# serial only
		#mod_params="$mod_params idProduct=$pid_serial_only"
		echo $pid_serial_only > idProduct

	elif [ "x$has_hid" != "x1" ] && [ "x$has_storage" != "x1" ] && [ "x$has_net" = "x1" ] && [ "x$has_serial" != "x1" ]; then
		# ethernet only, rndis or ecm
		if [ "x$is_win" = "x1" ]; then
			#mod_params="$mod_params idVendor=$vid_rndis_only idProduct=$pid_rndis_only"
			echo $vid_rndis_only > idVendor
			echo $pid_rndis_only > idProduct
		else
			#mod_params="$mod_params idProduct=$pid_ecm_only"
			echo $pid_ecm_only > idProduct
		fi
	elif [ "x$has_hid" = "x1" ] && [ "x$has_storage" = "x1" ] && [ "x$has_net" != "x1" ] && [ "x$has_serial" != "x1" ]; then
		# hid + storage
		#mod_params="$mod_params idProduct=$pid_hid_storage"
		echo �$pid_hid_storage > idProduct

	elif [ "x$has_hid" = "x1" ] && [ "x$has_storage" != "x1" ] && [ "x$has_net" != "x1" ] && [ "x$has_serial" = "x1" ]; then
		# hid + serial
		#mod_params="$mod_params idProduct=$pid_hid_serial"
		echo $pid_hid_serial > idProduct

	elif [ "x$has_hid" = "x1" ] && [ "x$has_storage" != "x1" ] && [ "x$has_net" = "x1" ] && [ "x$has_serial" != "x1" ]; then
		# hid + ethernet
		if [ "x$is_win" = "x1" ]; then
			#mod_params="$mod_params idProduct=$pid_hid_rndis"
			echo $pid_hid_rndis > idProduct
		else
			#mod_params="$mod_params idProduct=$pid_hid_ecm"
			echo $pid_hid_ecm > idProduct
		fi
	elif [ "x$has_hid" = "x1" ] && [ "x$has_storage" = "x1" ] && [ "x$has_net" = "x1" ] && [ "x$has_serial" != "x1" ]; then
		# hid + storage + ethernet
		if [ "x$is_win" = "x1" ]; then
			#mod_params="$mod_params idProduct=$pid_hid_storage_rndis"
			echo $pid_hid_storage_rndis > idProduct
		else
			#mod_params="$mod_params idProduct=$pid_hid_storage_ecm"
			echo $pid_hid_storage_ecm > idProduct
		fi
	elif [ "x$has_hid" != "x1" ] && [ "x$has_storage" != "x1" ] && [ "x$has_net" = "x1" ] && [ "x$has_serial" = "x1" ]; then
		# serial + ethernet
		if [ "x$is_win" = "x1" ]; then
			#mod_params="$mod_params idProduct=$pid_serial_rndis"
			echo $pid_serial_rndis > idProduct
		else
			#mod_params="$mod_params idProduct=$pid_serial_ecm"
			echo $pid_serial_ecm > idProduct
		fi
	elif [ "x$has_hid" != "x1" ] && [ "x$has_storage" = "x1" ] && [ "x$has_net" = "x1" ] && [ "x$has_serial" != "x1" ]; then
		# storage + ethernet
		if [ "x$is_win" = "x1" ]; then
			#mod_params="$mod_params idProduct=$pid_storage_rndis"
			echo $pid_storage_rndis > idProduct
		else
			#mod_params="$mod_params idProduct=$pid_storage_ecm"
			echo $pid_storage_ecm > idProduct
		fi
	else
		# unsupported, use default
		#mod_params=" $mod_params idProduct=$pid_default"
                echo $pid_default > idProduct
	fi

	# override vid/pid
	if [ "x$str_vid_custom" != "x" ] && [ "x$str_pid_custom" != "x" ]; then
		#mod_params="$mod_params idVendor=$str_vid_custom idProduct=$str_pid_custom"
		echo $str_vid_custom > idVendor
		echo $str_pid_custom > idProduct
	fi

	#setup descriptors
	echo 0x0100 > /sys/kernel/config/usb_gadget/croc/bcdDevice # v1.0.0
	echo 0x0200 > /sys/kernel/config/usb_gadget/croc/bcdUSB # USB2
	mkdir -p /sys/kernel/config/usb_gadget/croc/strings/0x409 #LANG=US/ENGLISH
	mkdir -p /sys/kernel/config/usb_gadget/croc/configs/c.1/strings/0x409

	# override manufacturer
	if [ "x$str_man_custom" != "x" ]; then
		#mod_params="$mod_params iManufacturer=$str_man_custom"
		echo $str_man_custom > strings/0x409/manufacturer
	else
		echo $manufacturer > strings/0x409/manufacturer
	fi

	# override RNDIS speed
	if [[ "x$str_speed_custom" != "x" && $str_speed_custom =~ ^-?[0-9]+$ && "$str_speed_custom" -gt "0" ]]; then
		[[ "$str_speed_custom" -gt "4294967" ]] && str_speed_custom=4294967
		speed=$((str_speed_custom*1000))
		#mod_params="$mod_params rndis_speed=$speed"
	fi

	# override serial
	if [ "x$str_sn_custom" != "x" ]; then
		#mod_params="$mod_params iSerialNumber=$str_sn_custom"
		echo $str_sn_custom > strings/0x409/serialnumber
	else
		echo $serialnumber > strings/0x409/serialnumber
	fi


}

load_module() {
	cd /sys/kernel/config/usb_gadget/croc/
	echo "Crazy Croc USB Device" > strings/0x409/product
	echo "Config 1: Linux" > configs/c.1/strings/0x409/configuration
	echo 250 > configs/c.1/MaxPower
        mkdir -p /sys/kernel/config/usb_gadget/croc/configs/c.2/strings/0x409
        echo "Config 2: WinXX" > configs/c.2/strings/0x409/configuration
        echo 250 > configs/c.2/MaxPower

	# Add functions here

if [ "$e_serial" -eq 1 ]; then
    echo "create serial function"
    mkdir -p functions/acm.usb0
    ln -s functions/acm.usb0 configs/c.1/
    ln -s functions/acm.usb0 configs/c.2/
fi

if [ "$e_ethernet" -eq 1 ]; then
    echo "create ecm ethernet function"
    mkdir -p /sys/kernel/config/usb_gadget/croc/configs/c.2/strings/0x409
    echo "Config 2: WinXX" > configs/c.2/strings/0x409/configuration
    echo 250 > configs/c.2/MaxPower
    
    mkdir -p functions/ecm.usb0
    mkdir -p functions/rndis.usb0
    
    echo $host_addr > functions/ecm.usb0/host_addr
    echo $dev_addr > functions/ecm.usb0/dev_addr

    echo "0x02" > /sys/kernel/config/usb_gadget/croc/c.2/bDeviceClass
    echo "0x00" > /sys/kernel/config/usb_gadget/croc/c.2/bDeviceSubClass
    #echo "0x3066" > /sys/kernel/config/usb_gadget/croc/bcdDevice
    mkdir /sys/kernel/config/usb_gadget/croc/c.2/os_desc
    echo "1" > /sys/kernel/config/usb_gadget/croc/c.2/os_desc/use
    echo "0xcd" > /sys/kernel/config/usb_gadget/croc/c.2/os_desc/b_vendor_code
    echo "MSFT100" > /sys/kernel/config/usb_gadget/croc/c.2/os_desc/qw_sign
    ln -s configs/c.2 /sys/kernel/config/usb_gadget/croc/os_desc

    echo $host_addr > functions/rndis.usb0/host_addr
    echo $dev_addr > functions/rndis.usb0/dev_addr

    ln -s functions/ecm.usb0 configs/c.1/
    ln -s functions/rndis.usb0 configs/c.2/
fi

if [ "$e_hid" -eq 1 ]; then
    echo "create hid function"
    mkdir -p functions/hid.usb0
    echo 1 > functions/hid.usb0/protocol
    echo 1 > functions/hid.usb0/subclass
    echo 8 > functions/hid.usb0/report_length
    echo -ne \\x05\\x01\\x09\\x06\\xa1\\x01\\x05\\x07\\x19\\xe0\\x29\\xe7\\x15\\x00\\x25\\x01\\x75\\x01\\x95\\x08\\x81\\x02\\x95\\x01\\x75\\x08\\x81\\x03\\x95\\x05\\x75\\x01\\x05\\x08\\x19\\x01\\x29\\x05\\x91\\x02\\x95\\x01\\x75\\x03\\x91\\x03\\x95\\x06\\x75\\x08\\x15\\x00\\x25\\x65\\x05\\x07\\x19\\x00\\x29\\x65\\x81\\x00\\xc0 > functions/hid.usb0/report_desc
    ln -s functions/hid.usb0 configs/c.1/
    ln -s functions/hid.usb0 configs/c.2/
fi

if [ "$e_storage" -eq 1 ]; then
    echo "create mass storage function"
    FILE=/mass_storage
    mkdir -p /root/udisk
    mount -o loop,ro, -t vfat $FILE /root/udisk # FOR IMAGE CREATED WITH DD
    mkdir -p functions/mass_storage.usb0
    echo 1 > functions/mass_storage.usb0/stall
    echo 0 > functions/mass_storage.usb0/lun.0/cdrom
    if [ has_rostorage -eq 1 ]; then
	echo 1 > functions/mass_storage.usb0/lun.0/ro
    else
        echo 0 > functions/mass_storage.usb0/lun.0/ro
    fi
    echo 0 > functions/mass_storage.usb0/lun.0/nofua
    echo $FILE > functions/mass_storage.usb0/lun.0/file
    ln -s functions/mass_storage.usb0 configs/c.1/
    ln -s functions/mass_storage.usb0 configs/c.2/
fi

# see gadget configurations below
# End functions
ls /sys/class/udc > UDC
}

####################################################################
# Parse /var/lib/dhcp/dhcpd.leases for TARGET_IP
####################################################################

output_folder=/tmp
leasefile="/var/lib/dhcp/dhcpd.leases"

check_dhcp_target_ip() {
	target_ip=$(cat $leasefile | grep ^lease | awk '{ print $2 }' | sort | uniq)
	target_hostname=$(cat $leasefile | grep hostname | awk '{print $2 }' \
			| sort | uniq | tail -n1 | sed "s/^[ \t]*//" | sed 's/\"//g' | sed 's/;//')
}

# timeout = 20 seconds
loop_dhcp_lease() {
	rm -rf $output_folder/dhcp.time.txt

	timeout=20
	[[ "x$str_timeout_custom" != "x" ]] && [[ "x$is_auto" != "x" ]] && {
		timeout=$str_timeout_custom
	}

	for i in `seq $timeout`;
	do
		check_dhcp_target_ip
		if [ "x$target_ip" != "x" ]; then
			TARGET_IP=$target_ip
			TARGET_HOSTNAME=$target_hostname
			HOST_IP=$(cat /etc/network/interfaces.d/usb0 | grep address | awk {'print $2'})
			echo "got dhcp ip address after $i seconds"
			echo "got dhcp ip address after $i seconds" > $output_folder/dhcp.time.txt
			return 0
		fi

		sleep 1
	done

	return 1
}


post_load_module() {
	# delete dhcp lease file, so 'source bunny_functions.sh' won't get valid TARGET_IP
	# if target ip is not successfully negotiated, e.g. in 2 cases:
	# a) attack mode does not include ETHERNET
	# b) target is configured as manual ip address
	rm -rf /var/lib/dhcp/dhcpd.leases

	if [ "x$has_net" = "x1" ]; then
		sleep 1
		# start dhcp server
		/etc/init.d/isc-dhcp-server start

		loop_dhcp_lease
		ret=$?

		[[ "x$is_auto" != "x" ]] && {
			[[ "$ret" == "1" ]] && {
				$(echo $CMD | sed 's/AUTO_ETHERNET/RNDIS_ETHERNET/gI')
				exit
			}
		}

		echo TARGET_IP = $TARGET_IP, TARGET_HOSTNAME = $TARGET_HOSTNAME, HOST_IP = $HOST_IP
	fi

	if [ "x$has_serial" = "x1" ]; then
		systemctl start getty.target
		systemctl start serial-getty@ttyGS0.service
	fi

	# Host is quicker to load hid only driver.
	# But, host takes time to load rndis, cdc ecm or mass stoage driver.

	# We wait dhcp above, there is no need to further delay here.
	if [ "x$has_net" = "x1" ]; then
		return
	fi

	if [ "x$is_win" = "x1" ]; then
		if [ "x$has_storage" = "x1" ] || [ "x$has_serial" = "x1" ]; then
			sleep 2
		else
			sleep 2
		fi
	else
		if [ "x$has_storage" = "x1" ] || [ "x$has_serial" = "x1" ]; then
			sleep 4
		else
			sleep 2
		fi
	fi
}


CMD="$0 $@"

for arg in "$@"
do
	parse_mode $(echo $arg | awk '{print toupper($0)}')
done


attack_mode_params
load_module
post_load_module

exit 0
```

# Croc_Framework

The initialising start-up script; this script is called by systemd at boot, allowing your device to auto setup and begin its attack-sequence:
```
#!/bin/bash

export PATH=$PATH:/usr/local/croc/bin:/tools

function init_croc() {
	dmesg -c > /dev/null
}

function update_languages() {
	mkdir -p /root/udisk/languages
	cp /root/udisk/languages/*.json /usr/local/croc/lib/languages/
}

function install_tools() {
	mkdir -p /tools
	mkdir -p /root/udisk/tools

	dpkg -i /root/udisk/tools/*.deb &> /tmp/dpkg_log
	rm -rf /root/udisk/tools/*.deb

	cp -rf /root/udisk/tools/* /tools/ &> /dev/null
	rm -rf /root/udisk/tools/*

	[[ -f /tools/install.sh ]] && {
		bash /tools/install.sh
		rm /tools/install.sh
	}

	sync
}
#demo purposes gpio - subject to change later? these gpio's will vary from device platform to device platform
function check_switch() {
	local SWITCH1=$(cat /sys/class/gpio/gpio172/value)
	local SWITCH2=$(cat /sys/class/gpio/gpio173/value)

	[[ "$SWITCH1" == "0" ]] && echo "switch1" && return 0
	[[ "$SWITCH2" == "0" ]] && echo "switch2" && return 0
	echo "invalid" && return 0
}

function mount_udisk() {
	echo "mount udisk"
	mount -o loop -t vfat /mass_storage /root/udisk
}

function unmount_udisk(){
	echo "umount udisk"
	if mount | grep /mnt > /dev/null;then
		umount /mnt
	fi
}

function install_payload() {
	local install_file="/root/udisk/payloads/${1}/install.sh"
	[[ -f $install_file ]] && {
		LED SETUP
		cp $install_file /tmp/install_payload.sh
		sed -i 's/\r//g' /tmp/install_payload.sh
		bash -c "${install_file}"
		[[ "$?" == "0" ]] && {
			mv "$install_file" "${install_file}.INSTALLED"
			sync
		}
		LED OFF
		rm /tmp/install_payload.sh
	}
}

function run_payload() {
	local payload="/root/udisk/payloads/${1}/payload.txt"
	[[ -f $payload ]] && {
		cp $payload /tmp/payload.sh
		sed -i 's/\r//g' /tmp/payload.sh

		export SWITCH_POSITION=$1
		for f in /root/udisk/payloads/extensions/*; do source $f; done
		[[ -f /root/udisk/config.txt ]] && source <(sed 's/\r//g' /root/udisk/config.txt)
		PATH=$PATH bash -c '/tmp/payload.sh'
	} || return 1
}

function snap() {
	init_croc
	mount_udisk

	local switch=$(check_switch)
	case $switch in
		"switch2" | "invalid")
			echo "Preparing Arming Mode"

			LED SETUP

			update_languages
			install_tools

			LED B SLOW

			sleep 1
                        echo "Arming Mode Enabled"
			/bin/bash -c "/usr/local/croc/bin/ATTACKMODE SERIAL ECM_ETHERNET"
			echo "done"
			;;
		*)
			install_payload $switch
			run_payload $switch
			;;
	esac

	unmount_udisk
}

snap &
exit 0
```

# Notes about Croc_Framework
LED is a Bash Script that sets up the GPIO for control of the PiZero’s RGB LED.

check_switch function is linked to the GPIO’s we use to switch payloads using standard 2.54 headers and a standard motherboard Jumper, you could alternatively use a micro switch?

* /mass_storage is a volume created like so (to appear as mass storage on the client/host computer)

## Creating /mass_storage
Create a 1GB file for emulated storage:
<pre>
dd if=/dev/zero of=/mass_storage bs=1024 count=1048576
</pre>
## Create a partition / filesystem
<pre>
mkdosfs /mass_storage
</pre>
## Similarities to a popular similar device?
We agree that there are a lot of similarities to the another device already on the market. The principal behind this design, is that there are a large number of payloads already publicly available for this other device. To avoid the community changing and modifying existing payloads, we have intensionally made our loader-scripts backward compatible with these existing payloads. Thus making the CrazyCroc easier to use.

## Comparison of other devices on the Market?

![](/blog/assets/croc_2.png)

We plan to release everything as OpenSource (the Kernel and rootfs is already available :) See the previous post!)… watch this space!

## Future updates / ToDo
* Theres a bit of automation, and user friendliness to finish
* Decide on how to implement the HID injection mode, as there are several different methods available from various authors?
* Windows OS’s can behaviour unexpectedly?
* Finalise the GPIO settings.
* Think about a suitable case?
* Check compatibility with Raspberry Pi Zero.

# Media
* Croc Image from: https://www.flickr.com/photos/jdhancock/4999813881
