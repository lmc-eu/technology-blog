---
layout: post
title: "JAXB bug in java 6"
date: 2013-01-20 11:41
author: Kamil Podlesak
comments: true
categories: java
---

# Short version #

**JAXB implementation embedded in Java 6 is broken, don't use it.**
Java 7 is ok, if you're lucky enough to use latest java version.

To check if your implementation is correct, you can use tool at [github](https://github.com/podlesh/jaxb-bug)

# Long story version #

I discovered this bug when I wrote some utility classes for marshalling and unmarshalling JAXB objects (lots of boilerplate code, but that's different story).
As a conscientious and responsible developer, I wrote unit test. This unit test contains also test case that repeatedly deserializes and serializes xml field, checking
that the result is the same. Unit tests were green, so commit and push followed. And in a few minutes, Jenkins reported broken test.

WTF? There must be some difference in environment... and it certainly is: Jenkins job uses jdk 1.6 while I use jdk 1.7 by default (on my macbook, this is the native apple one).
Indeed, after downloading jdk 1.6 from Sun/Oracle and running tests with this version, the deserialization/serialization test case failed.

The failure is quite peculiar one: some elements that are not *nil* in XML source are marked `xsi:nil="true"` in the result. But not all of them. The result is correct
until the first `xsi:nil="true"` in the source. All following *nil*-able elements (ie wrapped by `JAXBElement`) are marked as *nil*. Apparently, the JAXB unmarshalling code fails to reset
some internal state flag (and debugging the code confirms this observation).

To check different implementations, I've made [testing tool](https://github.com/podlesh/jaxb-bug). Just to be sure, it checks all four variations of nil and non-nil elements. In reality, it is always the last list that displays either OK or ERROR.

Result on jdk 1.6:
	resource `incorrect1.xml` unmarshalled to class eu.lmc.bug.jaxb.ElementBug
	  A is nil    , expected nil     (OK)
	  B is nil    , expected element ERROR!

Result on jdk 1.7:
	resource `incorrect1.xml` unmarshalled to class eu.lmc.bug.jaxb.ElementBug
	  A is nil    , expected nil     (OK)
	  B is element, expected element (OK)

# Solution #

There is no real workaround, but there are two ways how to use correct implementation:

*  upgrade to jdk 1.7
*  use correct `jaxb-impl-X.jar` (it overrides the embedded version)

When using JAXB-RI, don't forget to check it for this bug. According to [jaxb.java.net](http://jaxb.java.net/guide/Which_JAXB_RI_is_included_in_which_JDK_.html), it looks like this bug is limited to versions 2.1.x

