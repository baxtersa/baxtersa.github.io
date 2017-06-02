---
layout: page
title: About
permalink: /about/
---

![me](/images/me.jpg)

<br />

{{ site.motto }}

<br />

Research
===
---

# [Stopify](https://github.com/plasma-umass/Stopify)
Web-based programming environments lack many basic features that programmers
expect to find in desktop-based IDEs. For example, given a non-terminating
program, most environments simply crash the browser. Some environments
implement "infinite loop detectors", but these sacrifice the ability to execute
long-running, interactive programs. The few exceptions that handle this
correctly, such as Pyret and WeScheme, have required tremendous effort to build
and are tightly coupled to specific programming languages.

We present Stopify, a new approach to building web-based programming
environments, that supports any language that compiles to JavaScript and
generates source maps. Stopify transforms the JavaScript produced by an
ordinary compiler and implements a runtime scheduler that adds support for
pausing, resuming, stepping, and break-pointing at the source language level.

Following this approach, we present a web-based debugger that supports three
major languages.

<br />

Education
===
---
 - **University of Massachusetts** - M.S./Ph.D. in Computer Science. In Progress.
 - **Boston College** - B.A. in Computer Science and B.S. in Mathematics. May 2014.

<br />

Experience
===
---
 - **Research Assistant** - University of Massachusetts Amherst. June 2016 - Present.  
   - with Prof. Arjun Guha
 - **Software Engineer** - iZotope Inc. June 2014 - June 2016.
 - **Undergraduate Research Assistant** - Boston College. May - September 2013.
   - with Prof. Hao Jiang.

<br />

Contributions
===
---

# Frenetic
[![frenetic](/images/frenetic.png)](http://www.frenetic-lang.org) is a family of
languages and tools for network programming, providing a semantic foundation for
trusting network programs. I hack on frenetic's compiler internals and
extensions to the NetKAT semantics, playing with expressions of network
behaviors and constraints so that network administrators don't have to hate
themselves.

# nanomsg
As a software engineer at iZotope, I prototyped the inter-plugin communication
seen in their [Neutron](https://www.izotope.com/en/products/mix/neutron.html)
product using the socket library [nanomsg](http://nanomsg.org).
I [identified](https://github.com/nanomsg/nanomsg/issues/411)
and [fixed](https://github.com/nanomsg/nanomsg/pull/413) race conditions in
their in-process and pipe-based transports causing crashes on rebinding and
closing sockets.

<br />

Awards
===
---
 - **Sudha and Rajesh Jha Scholarship** - University of Massachusetts Amherst.
  2016.

<br />

Activities & Service
===
---
 - **P4 Developer Day**. Attendee. November 2016.
 - **Women in Engineering and Computing Day**. Volunteer. October 2016.
 - **Oregon Programming Languages Summer School**. Attendee. June - July 2016.
 - **Programming Languages Mentoring Workshop**. Attendee. POPL 2013.
