---
layout: post
title:  "Hacking MongoDB"
date:   2018-09-13 13:49:33 +0000
tags: [Databases, pentest, redteam, blueteam]
---
![](/blog/assets/mongodb.png)

Another day, another breach. But this time an unsecured mongodb instance in the cloud. https://www.scmagazineuk.com/veeam-mongodb-left-unsecured-440-million-records-exposed/article/1492715

## What is Mongodb
MongoDB stores data in flexible, JSON-like documents, meaning fields can vary from document to document and data structure can be changed over time. Enabling users to perform ad-hoc queries, indexing and data aggregation in real-time.

## Finding Mongo
By default Mongodb listens on TCP/27017, but it can depending on configuration also be found on 27018–27019.

In the fore mentioned breach Shodan was used to discover the vulnerable instance, and from our screenshot today there are approximately 59,503 accessible instances:

![](/blog/assets/shodan_mongo.png)

## Security and Authentication Nightmare
By default mongodb has no authentication and all users have full read privileges.

If you create an admin user, this privileged account has full write access to all databases.

Also by default there is no encryption, and all communication is in clear-text.

## Installing Mongo Client
On debian based systems this is as easy as:
```
sudo apt install mongodb-clients
```
You can then use mongo to connect to a mongodb service locally or across the internet.

## List of Mongodb Operations
```
help                         ;show help
show dbs                     ;show database names
show collections             ;show collections in current database
show users                   ;show users in current database
show profile                 ;show most recent system.profile entries with time >= 1ms
use <db name>                ;set curent database to <db name>

db.addUser (username, password)
db.removeUser(username)

db.cloneDatabase(fromhost)
db.copyDatabase(fromdb, todb, fromhost)
db.createCollection(name, { size : ..., capped : ..., max : ... } )

db.getName()
db.printCollectionStats()

db.currentOp() ;displays the current operation in the db

db.getProfilingLevel()
db.setProfilingLevel(level) ;0=off 1=slow 2=all

db.getReplicationInfo()
db.printReplicationInfo()
db.printSlaveReplicationInfo()
db.repairDatabase()

db.version() ;current version of the server
```

## Adding Security
Authentication is stored in each database’s system.users collection. For example, on a database my_project, my_project.system.users will contain user information.

We should first configure an administrator user for the entire db server process. This user is stored under the special admin database.

If no users are configured in admin.system.users, one may access the database from the localhost interface without authenticating. Thus, from the server running the database (and thus on localhost), run the database shell and configure an administrative user:
```
$ ./mongo
> use admin
> db.addUser("theadmin", "4dm1np4ssw0rd")
```
We now have a user created for database admin. Note that if we have not previously authenticated, we now must if we wish to perform further operations, as there is a user in admin.system.users.
```
> db.auth("theadmin", "4dm1np4ssw0rd")
```
We can view existing users for the database with the command:
```
> db.system.users.find()
```
Now, let’s configure a “regular” user for another database.
```
> use my_project
> db.addUser("joe", "passwordForJoe")
```
Finally, let’s add a readonly user.
```
> use my_project
> db.addUser("guest", "passwordForGuest", true)
```

## Adding Encryption through SSL
This has already been covered in Stampery’s blog post here, so we won't repeat his work here:
* https://medium.com/mongoaudit/how-to-enable-tls-ssl-on-mongodb-d973a92cefa6

## Mongodb Audit
Stampery has created a useful program to audit your Mongodb’s for known vulnerabilities and mis-configurations:
* https://github.com/stampery/mongoaudit
or
```
curl -s https://mongoaud.it/install | bash
```

## Conclusion
No company should want their Mongodb services accessible to the Internet. Obviously, there are more issues than Mongo’s default configuration of lack of security! Ideally Mongodb should not be publically accessible, but if necessary IP white-listing should be applied to firewall policies to only permit those hosts that require access to these potentially sensitive services.

Also visit Stampery’s blogs and Github repository for more information and support on hardening Mongodb.

## References
* https://docs.mongodb.com/manual/reference/default-mongodb-port/
