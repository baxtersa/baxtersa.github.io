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

# [Stopify](http://stopify.org)
Stopify is a JavaScript-to-JavaScript compiler that makes it possible for other
compilers and Web-based IDEs to gracefully run long-running programs, stop
non-terminating programs, support blocking I/O, set breakpoints, and step
through code entirely in the browser. These features are enabled by Stopify's
first-class continuation support for JavaScript. There are some interesting
insights necessary to implement efficient continuations, get in touch if you
want to discuss this!

This work appeared at PLDI 2018.

<a href="https://arxiv.org/pdf/1802.02974.pdf">Paper</a> | <a
href="https://github.com/plasma-umass/Stopify">Source Code</a> | <a
href="https://www.youtube.com/watch?v=M8PEWKQh2k4">Talk</a>

<br />

Conferences & Workshops
===
---

| -------------------------------- |
| Year | Venue             | Title |
| :--- | :---------------- | :---- |
| 2018 | [PLDI 2018](https://pldi18.sigplan.org/event/pldi-2018-papers-putting-in-all-the-stops-execution-control-for-javascript) | **Putting in All the Stops: Execution Control for JavaScript** |
| 2017 | [IBM PL Day](http://researcher.watson.ibm.com/researcher/view_group_subpage.php?id=8106) | **Wrestling Control from the Browser: Compiling to JavaScript with Fewer Compromises** |
| 2017 | [NEPLS](http://www.nepls.org/Events/31/abstracts.html#sbaxter) | **Stopify: Web-based Debugging For Free** |
| -------------------------------- |

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
 - **Research Intern** - IBM. May - August 2018.
   - with Olivier Tardieu
 - **Software Engineer** - iZotope Inc. June 2014 - June 2016.
 - **Undergraduate Research Assistant** - Boston College. May - September 2013.
   - with Prof. Hao Jiang.

<br />

Contributions
===
---

# Adapton
[Adapton](https://github.com/cuplv/adapton.rust) is a library of incremental
data structures for performant reuse of sub-computations. I ported probabilistic
tries from the original OCaml implementation to Rust and wrote some preliminary
benchmarks using tries to implement incremental computation over sets, finite
maps, and graphs.

# Frenetic
[![frenetic](/images/frenetic.png)](http://www.frenetic-lang.org) is a family of
languages and tools for network programming, providing a semantic foundation for
trusting network programs. I hacked on frenetic's compiler internals and
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
 - **PLDI Student Volunteer**. Volunteer. June 2017.
 - **ECOOP Summer School**. Attendee. June 2017.
 - **UMass CICS New Student Committee**. Member. 2016 - Present.
 - **P4 Developer Day**. Attendee. November 2016.
 - **Women in Engineering and Computing Day**. Volunteer. October 2016.
 - **Oregon Programming Languages Summer School**. Attendee. June - July 2016.
 - **Programming Languages Mentoring Workshop**. Attendee. POPL 2013.
