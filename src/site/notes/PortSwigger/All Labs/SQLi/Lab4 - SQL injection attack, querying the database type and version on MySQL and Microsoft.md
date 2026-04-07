---
{"dg-publish":true,"permalink":"/port-swigger/all-labs/sq-li/lab4-sql-injection-attack-querying-the-database-type-and-version-on-my-sql-and-microsoft/","created":"2026-03-26T17:14:30.295+01:00","updated":"2026-04-07T10:23:21.803+02:00","dg-note-properties":{}}
---


## Description
This lab contains a SQL injection vulnerability in the product category filter. You can use a UNION attack to retrieve the results from an injected query.

To solve the lab, display the database version string.

## Objective

Make the database retrieve the string: '8.0.42-0ubuntu0.20.04.1
## Payload Crafting

`"UNION SELECT @@version, NULL#"`

