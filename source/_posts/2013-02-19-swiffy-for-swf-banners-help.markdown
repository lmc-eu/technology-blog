---
layout: post
title: "Swiffy For SWF Banners Help"
date: 2013-02-19 10:00
author: Marek Raida
comments: true
categories: web
---

I've prepared this presentation for purposes of our company and it describes benefits of utilizing [Swiffy](https://www.google.com/doubleclick/studio/swiffy/) conversion tool for our/client banner ads business.
Our creative guys are great when using *Adobe Flash Professional* authoring tool and our customers love it, but __growing iOS devices market share__ is already forcing us to change our current technology approach. This presentation should help us with the transformation process.

New [DZSlides](https://github.com/paulrouget/dzslides) version, which I've used as the presentation engine, is a little bit less MSIE 9 friendly then its previous releases and I had to hack it a bit. So now the presentation works well in MSIE 10, Firefox, Chrome, Safari, Opera and acceptably also on MSIE 9 as well, but sorry, MSIE 8- ...
In the presentation slides is purposefully __embedded SWF file__ and to have it *visible in Gecko browsers*, there is a trick, which must be used - add __wmode="opaque"__ to the Flash object tag declaration; maybe will this hint save you some time, well, it cost me a lot to discover this, to be honest...

* [Presentation Itself](/assets/swiffy.html) which contains a lot of information for you to know how *(and if)* start to think differently, about how to serve/use your *Adobe-Fixed-Creatives* banner content
* [Simple Banner Fallback Example](/assets/banner.htm) shows easiest possible markup for you to test proposed fallback scenario *(yes. MSIE 8 users, this is the only sensible link for you there...)*
* [Sample Conversion Gallery](http://svg.kvalitne.cz/swiffy/gallery.html) is collection of *(mostly)* LMC banners with positive results of Swiffy conversion process, with some comments, file size comparisons etc. *(those demos do not have proposed fallback applied to them so SVG aware browser is a must!)* Gallery is temporarily hosted on different server, because of some special mime type handling.


