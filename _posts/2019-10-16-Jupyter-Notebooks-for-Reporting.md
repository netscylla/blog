---
layout: post
title:  "Jupyter Notebooks for Reporting & Collaboration"
description: "Demonstrating the advantages of Jupyter notebooks for reporting, collaboration and code-sharing"
date:   2019-10-16 14:49:33 +0000
tags: [Blue Team, Red Team, Knowledge sharing, Reporting, Python, DevSecOps]
---

![jupyter logo](/assets/Jupyter.png)

## Jupyter

 * [https://jupyter.org/about](https://jupyter.org/)
 
Project Jupyter is a non-profit, open-source project, born out of the IPython Project in 2014 as it evolved to support interactive data science and scientific computing across all programming languages. Jupyter will always be 100% open-source software, free for all to use and released under the liberal terms of the modified BSD license.

### Why Jupyter

As a web application it enables users to create and share documents that contain live code, equations, visualizations as well as text, the Jupyter Notebook is one of the ideal tools to help you to gain the data science skills you need.

## Using Jupyter in Penetration Testing

By generating mock issues in this way, they are easy to copy and paste into new and future notebooks. Can be stored
in either a document or even code or version control platform. The format can be easily distributed, to addition team members.
Well written issues with validated and documented code can be shared within the team.

The code would need to be modified on a case by case basis, but it enables code sharing within teams, and should help with the up-skilling (or teaching a new programming language) to juniors or even the wider team.

### Issue Reporting
Below is a sample notebook for demonstration purposes:
![Sample Notebook](/assets/Report-sample-notebook.png)

Jupyter uses a number of cells, these cells can be used for
 * Markdown
 * Python scripts/programs & any associated output
 * Raw code
 * Nearly anything else... depending on the plugins you can install and enable
 
Below is an example where we can generate a report about web application cookies:

![Sample Cookies](/assets/Report-cookies.png)

Another example, parsing HTTP headers:

![Sample HTTP Headers](/assets/Report-http-headers.png)

For the two examples above you can see how we have used a Markdown cell to deliver a smooth looking issue report (description, impact and remediation).
Then secondly used active python cells: to demonstrate in code the technical proof of the issues. 

### Active Content

The advantages of Jupyter and active content means that when we write issues. We can insert code that deliveries the proof of work.
For example we can create code to fetch a specific webpage or content, in the example below we demonstrate directory listings:

![Sample HTTP Headers](/assets/Report-active-content-1.png)

![Sample HTTP Headers](/assets/Report-active-content-2.png)

Other examples:

#### Validating Cracked Password Hashes

![Sample password 1](/assets/Report-password1.png)

#### Demonstrating Weak Cryptography

![Sample password 2](/assets/Report-password2.png)

#### Demonstrating Portscanning

![Sample portscan](/assets/Report-portscan.png)

### Markdown enables movies & animated gifs

Another advantage of these notebooks, is that it utilises Markdown (just like Github/Gitlab), it enabled
users to embed active content such as animated gifs or short movies. The advantage here is this can be used to generate
active content, thats not only useful to the Team, but also acts as proof of work to the client.

![Embedded Content 1](/assets/Report-embed-animated-gifs.png)

Here is the animated gif from the sample issue (apologies for the slow speed):

![animated gif](/assets/password-in-source.gif)

## Notebooks for training
You can use notebooks for training purposes. Use Markdown cells to include background information, challenges (description and questions), code samples. 
The user can see demo code in action, or where required alter the code in some fashion to achieve a result.

Due to the nature of markdown and embedded content, you could also include youtube tutorials that may be useful to junior or other members of staff.

An example of python code on how to embed youtube movies (more for training than reporting) that may be beneficial
to other team members.

![Embedded Content 2](/assets/Report-youtube.png)

## Converting Jupyter Notebooks
Now a Jupyter Notebook is not necessarily the best format of a report to pass to a client. However, converting the notebook into other text/mardown/formats is
 relatively easy, and makes transferring the data from Jupyter to another reporting format rather trivial.
 
The simplest way to use nbconvert is:
```
 > jupyter nbconvert mynotebook.ipynb
```
which will convert mynotebook.ipynb to the default format (probably HTML). You can specify the export format with `--to`.
```
 Options include ['asciidoc', 'custom', 'hide_code_html', 'hide_code_latex', 'hide_code_pdf', 'hide_code_slides', 'html', 'html_ch', 'html_embed', 'html_toc', 'html_with_lenvs', 'html_with_toclenvs', 'latex', 'latex_with_lenvs', 'markdown', 'notebook', 'pdf', 'python', 'rst', 'script', 'selectLanguage', 'slides', 'slides_with_lenvs']

 > jupyter nbconvert --to latex mynotebook.ipynb
```
Both HTML and LaTeX support multiple output templates. LaTeX includes 'base', 'article' and 'report'.  HTML includes 'basic' and 'full'. You can specify the flavor of the format used.
```
 > jupyter nbconvert --to html --template basic mynotebook.ipynb
```
You can also pipe the output to stdout, rather than a file
```
 > jupyter nbconvert mynotebook.ipynb --stdout
```
PDF is generated via latex
```
 > jupyter nbconvert mynotebook.ipynb --to pdf
```
You can get (and serve) a Reveal.js-powered slideshow
```
 > jupyter nbconvert myslides.ipynb --to slides --post serve
```

## Conclusion
Jupyter notebooks is a very interesting tool in terms of data science and the ability to dynamically create graphs and charts. But we can adapt
this tool not necessarily directly for customer reporting, but specifically internal reporting and collaboration.  The unique feature of mixing
Markdown cells with Python cells, permits users to create dynamic and active content, that should help demonstrate issues and understanding of 
software vulnerabilities and exploits to the wider team.  It also challenges people to work more in python (or other supported languages), thus
through code sharing and collaboration it should be easier to teach or improve other members technical programming.

Jupyter notebooks should be useful for:
 * internal knowledge sharing
 * code sharing/collaboration
 * ctf reporting
 * (possible) Offensive security reporting
 * generating presentation slides