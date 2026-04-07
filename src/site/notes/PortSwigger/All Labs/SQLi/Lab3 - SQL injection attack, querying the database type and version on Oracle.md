---
{"dg-publish":true,"permalink":"/port-swigger/all-labs/sq-li/lab3-sql-injection-attack-querying-the-database-type-and-version-on-oracle/","created":"2026-03-26T09:42:14.935+01:00","updated":"2026-04-07T10:23:15.613+02:00","dg-note-properties":{}}
---


## Description

This lab contains a SQL injection vulnerability in the product category filter. You can use a UNION attack to retrieve the results from an injected query.

To solve the lab, display the database version string.

## Payload Crafting

This payload identifies that there are 2 columns. Increasing to 3 will result in an error and confirm that only 2 columns exist in the DB.
`"'ORDER BY 2--"`

This will display the database version. Since this is an Oracle DB the `"v$version"` is used. 2 columns are selected since it was identified that only 2 columns exist.
`"' UNION SELECT BANNER, NULL FROM v$version--"`

![lab3_pwned.png](/img/user/Pentesting/PortSwigger_Labs/SQLi/lab3_pwned.png)