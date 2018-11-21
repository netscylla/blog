---
layout: post
title:  "PDF Analysis"
description: "A walkthrough on analysing a PDF received within a potenital phishing email."
date:   2018-11-19 13:49:33 +0000
tags: [blue-team, malware, incident response]
---
![Screenshot](/blog/assets/malware.png)

# Introduction

With automation systems systematically analysing and cleaning your mailbox from phishing and malware; thesedays
you would think it would be on the rare occasion that malware still manages to creep through.

Attackers have started to use encrypted pdf's, so that these automated systems fail to correctly identify 
the malware or phish contained within. In this blog post we look at utilising Didier Stevens toolkit to
analyse the suspicious pdf's and extract any URL's for further analysis.

# The Sample

 * Name:  Project#1542292355.pdf
 * Size:  204586 Bytes 
 * MD5:   a11aa6cee2f5db1591444f256c89a924
 * SHA1:  5c898848ec3e30fd3a3642dc0785395d262b81f4
 * SSDEEP:3072:ooOFxQacnglBhX667Aj8EqBYjNYWyn/vBeeuGVvWTUjrDE43mmAKwLzKKeZolN18:0lcSBhXKAE5KBeKZJo43mTHKlZ4vp8dN
 
 The attacker has also given us the password within the email body
  * Password=123456
 
 This is what the pdf looks like after you've entered the password:
 
![Screenshot](/blog/assets/pdf_malware_1.png)
 
 And the URL link is: 
  * https://clarksharongdcoonedriveodocsa.appspot.com/vvewrs/?6c98974873faa1919c2a787a67f83c37=jeJKSC156
  
  But how can we automate extraction of this URL for automated systems....
  
# Using Didier Stevens toolkit

Didier Stevens has lead the research in analysing pdf's, you can refer to his website pages here:
 * [didierstevens pdf-tools](https://blog.didierstevens.com/programs/pdf-tools/)
 * [didierstevens pdf category](https://blog.didierstevens.com/category/pdf/)
 * [didierstevens cracking-encrypted-pdfs-part-1](https://blog.didierstevens.com/2017/12/26/cracking-encrypted-pdfs-part-1/)

But in summary, you can read the below:

## PDFID

Pdfid will give us a summary of the type of objects within a pdf.

In this sample, we are interested in:
 * its encrypted
 * it contains URLs
```
$ python pdfid.py Project#1542292355.pdf
PDFiD 0.2.5 Project#1542292355.pdf
 PDF Header: %PDF-1.4
 obj                   18
 endobj                18
 stream                 6
 endstream              6
 xref                   1
 trailer                1
 startxref              1
 /Page                  1
 /Encrypt               1
 /ObjStm                0
 /JS                    0
 /JavaScript            0
 /AA                    0
 /OpenAction            1
 /AcroForm              0
 /JBIG2Decode           0
 /RichMedia             0
 /Launch                0
 /EmbeddedFile          0
 /XFA                   0
 /URI                   2
 /Colors > 2^24         0
 ```
 
## PDF-Parser

We can use the -s flag to search for strings.
 
In the example below we want to locate the encrypted object:
 
```
 $ python pdf-parser.py -s /Encrypt Project#1542292355.pdf
trailer
  <<
    /Size 19
    /Root 18 0 R
    /Info 17 0 R
    /Encrypt 16 0 R
    /ID [<8679b726ea493460d3f8798bce7a7b1a><8679b726ea493460d3f8798bce7a7b1a>]
  >>
```

### Extracting the encryption key

If we wanted to extract the hash for cracking we can use pdf-parser n this way:

```
$ python pdf-parser.py -o 16 Project#1542292355.pdf
obj 16 0
 Type: 
 Referencing: 

  <<
    /Filter /Standard
    /V 1
    /R 2
    /O '(\xb1\xdbV\xa8\x83\xca\xb5\xa2-\xd5\xfc9\x06\x18\xa0\xf8\xe1l\xab\x8a\xf1Ng\xcc\xba_\x90\x83z\xac\x89\x8b)'
    /U '(\xf6\x17>\xe3\xafq\x17\xec\xfb\xf5\xdb_\xe5l\xe0\x89\x02\xb1\x0e}\xbb^\xbeH\x90\xf3\xed\xcd0\x9b\xbbG)'
    /P 4294963392
  >>
```

The hashes are:
 * /O - Owner hash (usually used to export the unencrypted form)
 * /U - User hash (used to view the document)

or simply use [pdf2john](https://github.com/magnumripper/JohnTheRipper/blob/bleeding-jumbo/run/pdf2john.pl), which is part of JohnTheRipper?

```
$ ~/JohnTheRipper/run/pdf2john.pl Project#1542292355.pdf
Project#1542292355.pdf:$pdf$1*2*40*4294963392*1*16*8679b726ea493460d3f8798bce7a7b1a*32*f6173ee3af7117ecfbf5db5fe56ce08902b10e7dbb5ebe4890f3edcd309bbb47*32*b1db56a883cab5a22dd5fc390618a0f8e16cab8af14e67ccba5f90837aac898b
```

In the next example, we attempt to observe any URL's but they are encrypted:

```
$ python pdf-parser.py -s /URI Project#1542292355.pdf
obj 5 0
 Type: /Annot
 Referencing: 

  <<
    /Type /Annot
    /Subtype /Link
    /Rect [327.195 393.321 494.351 371.496]
    /NM '(&\xd6\x00\xdbl\xf1\xe5\x83\xdb)'
    /M '(R\xdc\x02\xdap\xf9\xe4\x82\xda\xc7;\x11v\x83\x1a\xd9)'
    /Border [0 0 0]
    /A
      <<
        /S /URI
        /URI '(~\x92D\x9a2\xfb\xfa\x9c\x88\x9ekW.\xc2A\x8dq\xae\xf8\xd3:\xc1\x14\x81Eh\x97\xc0\xbe\xaeo\xa3\x08pd\xd4SY\xa2\x18\x8c\xa7\xdc\x18\x98O\x15\xf1\x1e\x94k\xb5\xdb\x90=t\x06Ml\xe5PV\xacMGCH\xcb\x1de<&\xe9\xf6U\xa9J\xad\xd2,3\xb7\xdeg\x01\xdc\x1f\xfc+U\xfccZ\xf6\xa2E\xbc\x99f\x05)'
      >>
  >>
```
  
## QPDF
As the potenital attacker has given us the password decryption is easy with the help of [qpdf](http://qpdf.sourceforge.net/) (should be available via your package manager)

```
  qpdf --decrypt --password=123456 Project#1542292355.pdf Project#1542292355.pdf_2
```

We save the unencrypted version as **Project#1542292355.pdf_2**, we can confirm this with pdfid
  
```
$ python pdfid.py Project#1542292355.pdf_2
PDFiD 0.2.5 Project#1542292355.pdf_2
 PDF Header: %PDF-1.4
 ...
 xref                   1
 trailer                1
 startxref              1
 /Page                  1
 /Encrypt               0
 /URI                   2
```

### Extracting URLs

Now we can easily extract the unencrypted URLs like so:
 
```
$ python pdf-parser.py -s /URI Project#1542292355.pdf_2
obj 5 0
 Type: /Annot
 Referencing: 

  <<
    /A
      <<
        /S /URI
        /URI (https://clarksharongdcoonedriveodocsa.appspot.com/vvewrs/?6c98974873faa1919c2a787a67f83c37=jeJKSC156)
      >>
    /Border [ 0 0 0 ]
    /M (D:20181115143235)
    /NM (0001-0000)
    /Rect [ 327.195 393.321 494.351 371.496 ]
    /Subtype /Link
    /Type /Annot
  >>
```  

## Phishing URL

We can then perform additional reconnaissance and research on the URL and confirm it is indeed a phishing website

![phishing website](/blog/assets/pdf_malware_2.png)

Third Party scans can be found here:
 * [VirusTotal 198233e52782c6470c99110379e908c5817d63d0d4640b5115305ee4dad34626](https://www.virustotal.com/#/url/198233e52782c6470c99110379e908c5817d63d0d4640b5115305ee4dad34626/detection)
 * [any.run d2bf867c-ac3e-4e01-9661-96d2c1ea1df4](https://app.any.run/tasks/d2bf867c-ac3e-4e01-9661-96d2c1ea1df4)
 * [phishtank 5848201 ](https://www.phishtank.com/phish_detail.php?phish_id=5848201)
 
 
# Automation with docker

We even built a docker container with these tools, to aid other analysts in extracting URLs from encrypted pdfs:

 * [pdf-analysis Docker](http://www.github.com/netscylla/pdf-analysis) 
 
Our **run.sh** script accepts two arguments

1. pdf
1. password (optional)
 
If the pdf is encrypted, and you dont supply a password, it will attempt a simple and small dictionary attack
 
```
$ docker run -v /tmp:/tmp -it pdftools:latest /opt/pdftools/run.sh /tmp/Project#1542292355.pdf
No password found, if encrypted may not be able to proceed!
PDF is encrypted...
trying password...
Trying default passwords
/tmp/Project#1542292355.pdf: invalid password
/tmp/Project#1542292355.pdf: invalid password
Valid Password, continuing...
extracting URIs:
https://clarksharongdcoonedriveodocsa.appspot.com/vvewrs/?6c98974873faa1919c2a787a67f83c37=jeJKSC156
```
