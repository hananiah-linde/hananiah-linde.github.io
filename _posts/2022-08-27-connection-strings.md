---
title: Reference - Connection Strings
date: 2022-08-27 07:30:00 -500
categories: [software]
tags: [database,reference,.net]
---

# Connection Strings Reference

## Postgre
``` terminal
"Server=localhost; Port=5432; Database=dbname; User Id=postgres; Password=password"
```

## SQL Server

### Local DB - Trusted Connection:
``` terminal
"Server=(localdb)\\mssqllocaldb;Database=EfCoreAppSqlServer;Trusted_Connection=True;MultipleActiveResultSets=true"
```
