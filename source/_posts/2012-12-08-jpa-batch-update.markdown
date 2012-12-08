---
layout: post
title: "JPA batch update"
date: 2012-12-08 12:34
author: Kamil Podlesak
comments: true
categories: java
---

I've just spent few hours hunting some very weird bug in our new JPA code and found out that hibernate 4 (I don't know about other JPA implementations) does not clear cache on HQL batch updates.

This code fill some initial values to previously `null` columns:
{% codeblock lang:java %}
    public int fillOilife() {
        Query query = em.createQuery("UPDATE PdjdOrderItem poi SET poi.oilife = :oilife " +
                "WHERE poi.pdjdGeneration>0 AND poi.oistate = :oistate AND poi.oilife is NULL");
        query.setParameter("oilife", INITIAL_OILIFE);
        query.setParameter("oistate", VALID_OISTATE);
        return query.executeUpdate();
    }
{% endcodeblock %}
The update is correctly processed into SQL query and executed, but the session cache remains as it is. I've expected hibernate to detect that PdjdOrderItem is being changed and to clear all cached entities (or just the changed ones, but that would be unreasonably complicated). Hibernate, however, does nothing and happily server stale PdjdOrderItem entities with `null` values.

Solution is quite simple, if a little annoying: clear the cache by explicit `clear()` call:
{% codeblock lang:java %}
    public int fillOilife() {
        em.flush();
        em.clear();
        Query query = em.createQuery("UPDATE PdjdOrderItem poi SET poi.oilife = :oilife " +
                "WHERE poi.pdjdGeneration>0 AND poi.oistate = :oistate AND poi.oilife is NULL");
        query.setParameter("oilife", INITIAL_OILIFE);
        query.setParameter("oistate", VALID_OISTATE);
        return query.executeUpdate();
    }
{% endcodeblock %}
But I still feel disappointed.


