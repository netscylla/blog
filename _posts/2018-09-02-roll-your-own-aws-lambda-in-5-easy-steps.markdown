---
layout: post
title:  "Roll your own AWS Lambda in 5 easy steps"
date:   2018-09-02 13:49:33 +0000
tags: [Cloud, AWS, pentest, redteam, blueteam]
---
Below we walk though a very simple example of writing your own serverless code / Lambda functions from a Linux workstation.

## Lambda Languages
AWS Lambda currently supports the following languages:
* C# versions 1,2, & 2.1
* Go version 1
* Java version 8
* Node versions 4, 6 & 8
* Python versions 2 & 3
It is advisable that you at least know one of these programming scripting languages before you begin with any Lamba code.

**For demonstration purposes we will continue with python3.**

## Lambda Creation
### Step 1 - Create a workspace
```
mkdir my_lambda_module
```
### Step 2 - Create lambda_function.py
Assuming we are creating a python-based lambda function:
```
touch my_lamba_module/lambda_function.py
```
### Step 3 - Install any missing/prerequisite modules:
```
pip3 install <module> -t ./my_lamba_module
```

### Step 4 - Code
Code away using your favourite IDE or text editor, or import a pre-programmed lambda from github. Don’t forget to install any missing modules.

### Step 5 - Package it:
```
cd my_lambda_module;zip -r my_lambda.zip *
```
Goto the AWS console and open your lambda function, click ‘Edit Code Inline’, and change to ‘Upload a .ZIP file’

![](/assets/lambda_3.png)

You code should appear after clicking ‘Save’

### Finally
It is always a good idea to ‘Test’ you code before going straight into production; It may help to have a few print() statements to help you debug any issues through Cloudwatch logs.

Any missing modules can be rectified by repeating the install module above, simply re-zip, and re-upload until your happy your lambda code is functioning correctly.

