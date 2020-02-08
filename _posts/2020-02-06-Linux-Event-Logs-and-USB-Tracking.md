---
layout: post
title:  "Linux Events & USB Tracking"
description: "A brief post on tracking USB devices natively with the Linux's Syslog"
date:   2020-02-06 13:00:01 +0000
tags: [Blue-Team, SOC, Monitoring, Logging]
---

![USBs](/blog/assets/USBs.jpg)

## Inspiration
Our previous post covered USB tracking via native Windows event logs. 
 * [Windows Event Logs and USB Tracking](https://www.netscylla.com/blog/2020/02/03/Windows-Event-Logs-and-USB-Tracking.html)

In this post we thought we would add some balance and write about Linux native USB tracking.

### Using Syslog to Track USBs

Not many people may realise this but Syslog already logs USB events.  Events can appear in either/or both the following locations depending on your distribution:
```
sudo cat /var/log/kern.log | grep usb

May 25 07:38:51 mycomputer kernel: [  607.296847] scsi7 : usb-storage 3-1:1.0
May 25 07:38:54 mycomputer kernel: [  609.790892] usb 3-2: new high-speed USB device number 3 using xhci_hcd
May 25 07:38:54 mycomputer kernel: [  609.817462] usb 3-2: ep 0x81 - rounding interval to 32768 microframes, ep desc says 0 microframes
May 25 07:38:54 mycomputer kernel: [  609.817474] usb 3-2: ep 0x2 - rounding interval to 32768 microframes, ep desc says 0 microframes
May 25 07:38:54 mycomputer kernel: [  609.818399] usb-storage 3-2:1.0: Quirks match for vid 13fe pid 3600: 4000
May 25 07:38:54 mycomputer kernel: [  609.818529] scsi8 : usb-storage 3-2:1.0
```
You can also search the syslog as follows:
```
sudo cat /var/log/syslog.1 | grep usb

May 25 07:31:25 tardis-w520 kernel: [  161.469096] usb 1-1.5.5: new high-speed USB device number 8 using ehci_hcd
May 25 07:31:25 tardis-w520 mtp-probe: checking bus 1, device 8: "/sys/devices/pci0000:00/0000:00:1a.0/usb1/1-1/1-1.5/1-1.5.5"
May 25 07:31:25 tardis-w520 kernel: [  161.658587] scsi6 : usb-storage 1-1.5.5:1.0
May 25 07:31:25 tardis-w520 kernel: [  161.658685] usbcore: registered new interface driver usb-storage
May 25 07:31:25 tardis-w520 kernel: [  161.795563] usbcore: registered new interface driver uas
May 25 07:38:51 tardis-w520 kernel: [  607.268280] usb 3-1: new high-speed USB device number 2 using xhci_hcd
May 25 07:38:51 tardis-w520 kernel: [  607.293280] usb 3-1: ep 0x81 - rounding interval to 32768 microframes, ep desc says 0 microframes
May 25 07:38:51 tardis-w520 kernel: [  607.293292] usb 3-1: ep 0x2 - rounding interval to 32768 microframes, ep desc says 0 microframes
```

### The problem
Syslog also outputs many other different kinds of system and kernel alerts. Hopefully, you are also collecting the syslog logs on a centralised platformas part of your SIEM solution, or for log retention purposes.

The problem is often how do you filter the interesting targets from the noise?


### Possible solution with USBrip 

USBrip is is an Opensource tool written in Python, its primary function is a forensics tool with CLI interface that lets you keep track of USB device artifacts (i.e., USB event history) on Linux machines.
 * [https://github.com/snovvcrash/usbrip](https://github.com/snovvcrash/usbrip)  

USBrip is a Python3 program so we recommend installing it with the use of virtual-env in the following way:

```
~/usbrip$ python3 -m venv venv && source venv/bin/activate
(venv) ~/usbrip$ python setup.py install
```
USBrip help information:
```
(venv) ~/usbrip$ usbrip -h
# ---------- BANNER ----------

$ usbrip banner
Get usbrip banner.

# ---------- EVENTS ----------

$ sudo usbrip events history [-t | -l] [-e] [-n <NUMBER_OF_EVENTS>] [-d <DATE> [<DATE> ...]] [--host <HOST> [<HOST> ...]] [--vid <VID> [<VID> ...]] [--pid <PID> [<PID> ...]] [--prod <PROD> [<PROD> ...]] [--manufact <MANUFACT> [<MANUFACT> ...]] [--serial <SERIAL> [<SERIAL> ...]] [--port <PORT> [<PORT> ...]] [-c <COLUMN> [<COLUMN> ...]] [-f <FILE> [<FILE> ...]] [-q] [--debug]
Get USB event history.

$ sudo usbrip events open <DUMP.JSON> [-t | -l] [-e] [-n <NUMBER_OF_EVENTS>] [-d <DATE> [<DATE> ...]] [--host <HOST> [<HOST> ...]] [--vid <VID> [<VID> ...]] [--pid <PID> [<PID> ...]] [--prod <PROD> [<PROD> ...]] [--manufact <MANUFACT> [<MANUFACT> ...]] [--serial <SERIAL> [<SERIAL> ...]] [--port <PORT> [<PORT> ...]] [-c <COLUMN> [<COLUMN> ...]] [-f <FILE> [<FILE> ...]] [-q] [--debug]
Open USB event dump.

$ sudo usbrip events genauth <OUT_AUTH.JSON> [-a <ATTRIBUTE> [<ATTRIBUTE> ...]] [-e] [-n <NUMBER_OF_EVENTS>] [-d <DATE> [<DATE> ...]] [--host <HOST> [<HOST> ...]] [--vid <VID> [<VID> ...]] [--pid <PID> [<PID> ...]] [--prod <PROD> [<PROD> ...]] [--manufact <MANUFACT> [<MANUFACT> ...]] [--serial <SERIAL> [<SERIAL> ...]] [--port <PORT> [<PORT> ...]] [-f <FILE> [<FILE> ...]] [-q] [--debug]
Generate a list of trusted (authorized) USB devices.

$ sudo usbrip events violations <IN_AUTH.JSON> [-a <ATTRIBUTE> [<ATTRIBUTE> ...]] [-t | -l] [-e] [-n <NUMBER_OF_EVENTS>] [-d <DATE> [<DATE> ...]] [--host <HOST> [<HOST> ...]] [--vid <VID> [<VID> ...]] [--pid <PID> [<PID> ...]] [--prod <PROD> [<PROD> ...]] [--manufact <MANUFACT> [<MANUFACT> ...]] [--serial <SERIAL> [<SERIAL> ...]] [--port <PORT> [<PORT> ...]] [-c <COLUMN> [<COLUMN> ...]] [-f <FILE> [<FILE> ...]] [-q] [--debug]
Get USB violation events based on the list of trusted devices.

# ---------- STORAGE ----------

$ sudo usbrip storage list <STORAGE_TYPE> [-q] [--debug]
List contents of the selected storage. STORAGE_TYPE is "history" or "violations".

$ sudo usbrip storage open <STORAGE_TYPE> [-t | -l] [-e] [-n <NUMBER_OF_EVENTS>] [-d <DATE> [<DATE> ...]] [--host <HOST> [<HOST> ...]] [--vid <VID> [<VID> ...]] [--pid <PID> [<PID> ...]] [--prod <PROD> [<PROD> ...]] [--manufact <MANUFACT> [<MANUFACT> ...]] [--serial <SERIAL> [<SERIAL> ...]] [--port <PORT> [<PORT> ...]] [-c <COLUMN> [<COLUMN> ...]] [-q] [--debug]
Open selected storage. Behaves similary to the EVENTS OPEN submodule.

$ sudo usbrip storage update <STORAGE_TYPE> [IN_AUTH.JSON] [-a <ATTRIBUTE> [<ATTRIBUTE> ...]] [-e] [-n <NUMBER_OF_EVENTS>] [-d <DATE> [<DATE> ...]] [--host <HOST> [<HOST> ...]] [--vid <VID> [<VID> ...]] [--pid <PID> [<PID> ...]] [--prod <PROD> [<PROD> ...]] [--manufact <MANUFACT> [<MANUFACT> ...]] [--serial <SERIAL> [<SERIAL> ...]] [--port <PORT> [<PORT> ...]] [--lvl <COMPRESSION_LEVEL>] [-q] [--debug]
Update storage -- add USB events to the existing storage. COMPRESSION_LEVEL is a number in [0..9].

$ sudo usbrip storage create <STORAGE_TYPE> [IN_AUTH.JSON] [-a <ATTRIBUTE> [<ATTRIBUTE> ...]] [-e] [-n <NUMBER_OF_EVENTS>] [-d <DATE> [<DATE> ...]] [--host <HOST> [<HOST> ...]] [--vid <VID> [<VID> ...]] [--pid <PID> [<PID> ...]] [--prod <PROD> [<PROD> ...]] [--manufact <MANUFACT> [<MANUFACT> ...]] [--serial <SERIAL> [<SERIAL> ...]] [--port <PORT> [<PORT> ...]] [--lvl <COMPRESSION_LEVEL>] [-q] [--debug]
Create storage -- create 7zip archive and add USB events to it according to the selected options.

$ sudo usbrip storage passwd <STORAGE_TYPE> [--lvl <COMPRESSION_LEVEL>] [-q] [--debug]
Change password of the existing storage.

# ---------- IDs ----------

$ usbrip ids search [--vid <VID>] [--pid <PID>] [--offline] [-q] [--debug]
Get extra details about a specific USB device by its <VID> and/or <PID> from the USB ID database.

$ usbrip ids download [-q] [--debug]
Update (download) the USB ID database.
```
There is a lot of output to digest, so we will try to simplify its operation a simple example.

### Searching for policy violations
Here we demonstrate parsing a local syslog for violations against a file containing data of permitted USB devices (auth.json)

Where auth.json is a JSON file containing identification material for permitted devices in your organisation.  

In our example we use manufacturer IDs:
```

{
    "manufact": [
        "100012DAF2A2121E6AF21CE1D8",
        "100032F42B683A7F5ADFF7",
        "10008BB9C6907E25C1AFEDA7250B4E",
        "1000A11AB0CF5B878FF71AA343231871",
        "1000D0FC9CD5BE188996FC8AE7",
        "1000EC1FF40AB796767AB24C0681F",
...
}
```
Searching for violations to our corporate policy is as simple as **usbrip events violations**

We can root out anomalous behaviours and non-permitted USB devices like so:
```
(.venv) usbrip events violations ./auth.json --file /var/log/syslog

         _     {{4}}    {v2.1.4-9}
 _ _ ___| |_ ___[E]___
| | |_ -| . |  _[n] . |
|___|___|___|_| [5]  _|
               x[I]_|   https://github.com/snovvcrash/usbrip


[*] Started at 2020-01-20 14:12:22
[14:12:26] [INFO] Reading "/var/log/syslog"
100%|██████████████████████████████| 900000/900000 [00:38<00:00, 23467.33line/s]
[14:13:05] [INFO] Opening authorized device list: "./auth.json"
[14:13:06] [INFO] Searching for violations
 54%|██████████████████▍               | 54311/100000 [03:42<04:28, 170.25dev/s]
```
Eventually, we are presented with an unknown suspect device:
```
[14:22:28] [INFO] Preparing gathered events
[14:22:28] [INFO] Representation: table

┌USB-Violation-Events─┬──────┬──────┬──────┬───────────────────────────┬──────────────────────────┬──────────────────────────────────┬──────┬─────────────────────┐
│           Connected │ Host │  VID │  PID │                   Product │             Manufacturer │                    Serial Number │ Port │        Disconnected │
├─────────────────────┼──────┼──────┼──────┼───────────────────────────┼──────────────────────────┼──────────────────────────────────┼──────┼─────────────────────┤
│ ????-08-03 •••••••• │ −−−− │ −−−− │ −−−− │ −−−−−−−−−−−−−−−−−−−−−−−−− │ −−−−−−−−−−−−−−−−−−−−−−−− │ −−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−− │ −−−− │ −−−−−−−−−−−−−−−−−−− │
│ ????-08-03 07:18:01 │ kali │ 3993 │ 9324 │ 1F8ADAEE73D993944FC7C7783 │ 884CCC9A3DF08F49C621373E │ 71DF5A33EFFDEA5B1882C9FBDC1240C6 │  1-1 │ ????-08-03 07:18:10 │
└─────────────────────┴──────┴──────┴──────┴───────────────────────────┴──────────────────────────┴──────────────────────────────────┴──────┴─────────────────────┘
[*] Shut down at 2020-01-20 14:22:28
[*] Time taken: 0:10:06.574319
```
At this point we would report our findings and start the incident response process; Next steps would involve identifying the user of the computer at the time of the incident, interviewing the likely suspect,
and locating and physical un-approved device for forensic analysis.
