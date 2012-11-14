---
layout: post
title: "Funny column names"
date: 2012-11-14 17:32
author: Kamil Podlesak
comments: true
categories: database
---

# JPA, HSQLDB and funny column names

There is probably no need to introduce [hsqldb](http://hsqldb.org), let's just say that it's invaluable tool for automated (integration) tests of java/sql code. Especially when combined with Hibernate/JPA and its automatic ddl.

But there are, of course, pitfalls. One of them looks like this:

```
[2012-11-08 18:08:46,257] [main] (ERROR) hibernate.tool.hbm2ddl.SchemaExport - HHH000389: Unsuccessful: create table PUBLIC.ADDRESS_CNDP (ADDRESS_ID bigint not null, $SEC_DOMAIN_G2ID bigint, DETAIL varchar(100), GIS_ID varchar(255), NUMBER varchar(16), ORIENTATION_NUMBER varchar(16), STATUS integer, STR_CITY varchar(100), STR_CITYPART varchar(100), STR_COUNTRY varchar(100), STR_DISTRICT varchar(100), STR_REGION varchar(100), STR_STREET varchar(100), STR_TERRITORY varchar(100), TYPE integer, ZIP varchar(255), CITY_ID integer, CITYPART_ID integer, COUNTRY_ID integer, DISTRICT_ID integer, REGION_ID integer, STREET_ID integer, TERRITORY_ID integer, primary key (ADDRESS_ID))
[2012-11-08 18:08:46,257] [main] (ERROR) hibernate.tool.hbm2ddl.SchemaExport - unknown token: 
```
Yes, that's all - the error message ends by colon. Not really helpful...

After some digging, the culprit has been found: the "dollar" sign **$** in column name. Of course, using such character ~~stupid~~ *controversial*, but it's too late now as the main db2 database is eight year old. The same apply to other *controversial* column name choices, like `ORDER`, `COUNT` or `PACKAGE`.

Fortunately, both hibernate and JPA hsqldb can handle such names and the solution is actually really simple: just quote them.

```
	//ORDER is reserved word
    @Column(name = "\"ORDER\"", nullable = false)
    private Integer order;
    
    //$ is special character!
    @Column(name = "\"$SEC_DOMAIN_G2ID\"", updatable = false)
    private Long $secDomain;
    
    //no quoting is needed here
    @Column(name = "PARAM", length = 255)
    private String param;
```
Hibernate internally converts quotes (they are actually part of JPA2.0 standard) to back-ticks (that's hibernate-specific way to escape names) and then back to quotes (in `org.hibernate.dialect.HSQLDialect`).
