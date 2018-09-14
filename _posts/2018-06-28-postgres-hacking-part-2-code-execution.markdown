---
layout: post
title:  "Postgres Hacking Part 2 — Code Execution"
date:   2018-06-28 13:49:33 +0000
tags: [Postgres, Databases, pentest, redteam, blueteam]
---
In Part 1 we gave you the basics to PostgreSQL hacking. Now we broach the different kinds of remote code execution both local and remote. Most of the functionality here is extremely similar to exploitation on the MySQL platform, although the Postgres syntax is slightly different.

## Local Command Execution
Manually PostgreSQL databases can interact with the underlying operating by allowing the database administrator to execute various database commands and retrieve output from the system. You can also perform the following (again similar to MySQL’s !mode):
```
postgres=# select pg_ls_dir('./');
```
![](/assets/postgres_2.png)

It is also possible to create a database table in order to store and view contents of a file that exist in the host.
```
postgres-# CREATE TABLE temp(t TEXT);
postgres-# COPY temp FROM '/etc/passwd';
postgres-# SELECT * FROM temp limit 1 offset 0;
```
![](/assets/postgres_3.png)

## Remote Command Execution
Similar to the MySQL UDF exploit we have in our arsenal, another Metasploit module for the win: https://github.com/rapid7/metasploit-framework/blob/master/modules/exploits/linux/postgres/postgres_payload.rb

This exploit was made from the initial research of Nico Leidecker and PGShell: http://www.leidecker.info/pgshell/Having_Fun_With_PostgreSQL.txt

Execution of arbitrary code is also possible if the postgres service account has permissions to write into the /tmp directory through user defined functions (UDF).
```
msf > exploit/linux/postgres/postgres_payload
```
![](/assets/postgres_4.png)

## Conclusion
You should now have the basic knowledge required for attacking a Postgres database!

There is usually a vulnerable Postgres install on the Metasploitable CTF ISOs. They are easily found on Bit-Torrent trackers as a legal download. Or download the Open Source package, and install it yourself in a VM, docker container, or your own system.

Remember, to only practise Postgres attacks on your own legally owned systems!
