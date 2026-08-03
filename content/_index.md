---
title: "Oliver Dixon"
date: 2026-08-03T12:43:12+01:00
draft: true
---

## Welcome

I'm Oliver Dixon, a software engineer in the final year of a degree in Computer Science at the University of York.
Thanks for visiting my website; I hope you find what you're looking for. If not, please email or call/text me using the
information in the below graphic, which has conveniently had the text converted to paths.

{{< figure src="/contactinfo.svg" width="50%" class="top-padding" >}}

This website is an amalgamation of personal and pseudo-professional content. If I have some opinion on a topic (which
I've been told is highly likely), it'll probably appear on here. The main categories, accessible on the top menu, are:

* [Projects](/projects): personal software-adjacent projects I'm currently working on, or have previously completed.
  Most of these were born out of personal interest, unrelated to the university curriculum, but some started as
  coursework and grew into something more. I open-source almost everything I do in my spare time, but project pages try
  to be more than an extended README.
* [Blog](/blog): short, more frequent articles covering topics that I find interesting.
* [CV](/cv): a Markdown rendering of my CV covering professional activities.
* [Non-Tech](/nontech): pages covering non-computing content.

## About me

I'm mainly interested in systems software: C++, formal methods, signal processing, WebAssembly/WebGPU, embedded systems,
and the unglamorous engineering work needed to make large-ish software projects stay understandable over time. Most of
what I build sits somewhere between software engineering, mathematics and hardware-adjacent computing.

My top three interests, in descending order, are:

1. Spending time with Maia (fiancée).
2. Deleting code.
3. Cycling, but not competitively.

Maybe those latter items should be broken down a little more. The first needs no explanation!

## About this site

I created this website to replace my existing university-hosted instance
([archived](https://web.archive.org/web/20260803122140/https://www-users.york.ac.uk/~od641/); [live](https://www-users.york.ac.uk/~od641/)).
The IT provision at York are best-in-class, especially for Linux tinkerers who like to break things, but I considered it
important to provision infrastructure capable of:

* Outlasting my time at the university.
* Hosting dynamic content if needed.
* Being used for "other stuff", which, while technically possible on York's infrastructure, might raise some eyebrows,
  such as running self-hosted GitHub Actions daemons.

The backend hardware is not exactly the [Viking supercomputer](https://vikingdocs.york.ac.uk/), but it has held up well
so far.

```
owd@semanticpad:~$ lscpu
Architecture:                x86_64
  CPU op-mode(s):            32-bit, 64-bit
  Address sizes:             39 bits physical, 48 bits virtual
  Byte Order:                Little Endian
CPU(s):                      4
  On-line CPU(s) list:       0-3
Vendor ID:                   GenuineIntel
  Model name:                Intel(R) Celeron(R) J4105 CPU @ 1.50GHz
    CPU family:              6
    Model:                   122
    Thread(s) per core:      1
    Core(s) per socket:      4
    Socket(s):               1
    Stepping:                1
    CPU(s) scaling MHz:      96%
    CPU max MHz:             2500.0000
    CPU min MHz:             800.0000
    BogoMIPS:                2995.20

owd@semanticpad:~$ lsmem
RANGE                                  SIZE  STATE REMOVABLE BLOCK
0x0000000000000000-0x000000006fffffff  1.8G online       yes  0-13
0x0000000100000000-0x000000027fffffff    6G online       yes 32-79

Memory block size:                128M
Total online memory:              7.8G
Total offline memory:               0B
```

The software stack isn't very exciting either: it's running Ubuntu Server 26.04 LTS. Most services are containerised
with Podman and orchestrated through systemd (with Quadlets) behind a Caddy reverse proxy.

Currently, we (as in myself, my father Mark, and my fiancée Maia), use the box for:

* This personal site, built with [Hugo](https://gohugo.io/).
* Mark's staging and development area for his WIP [SemanticPad web application](https://semanticpad.com) which uses
  Apache Tomcat for servlets and MariaDB for persistent relational storage.
* Maia's file storage and multimedia server.
* A constant reminder of paying the ISP £5/month for a static IP.

## Education

I'm due to graduate in the summer 2027, with a *Master of Engineering in Computer Systems (with a year in industry)*.
There are a few components to break down, especially if you're not familiar with the UK higher education system:

* "Master of Engineering": essentially a regular three-year Bachelor's degree, specialising in some form of engineering,
  immediately followed by a "top-up year" to reach (in theory) an equivalent level of education as a standalone Master's
  degree, sometimes called an MSc. Traditionally, the Master's level was intended to be more research-focused than the
  preceding years of study, with the goal of preparing students for doctoral level study, but this distinction has
  largely been lost.
* "Computer Systems": Computer Science. The name mismatch is due to a technicality involving the process of degree
  accreditation in the UK. At York, there are two courses identical in all but the name: *Computer Science*, and
  *Computer Systems*. The former is vetted for quality and rigour by
  the [British Computer Society](https://www.bcs.org/deliver-and-teach-qualifications/academic-accreditation/); the
  latter (despite having an identical curriculum) is not. By opting to take Computer Systems, students experience at
  least four things:

    1. Upon completion, a mildly less prestigious degree certificate for people "in the know".
    2. Freedom to enrol in modules that were not checked by the BCS.
    3. Requirement to take an additional exam to join the BCS as a member following graduation.
    4. A potential barrier to entering regulated industries post-graduation, though for computing this is far less
       poignant that, say, electronics or medicine.

  As I'm interested in mathematics, which isn't connected to the BCS, taking Computer Systems allows me to mix modules
  from the Department of Mathematics with the Department of Computer Science to have, in my opinion, a more well-rounded
  education. The CS course at York is quite traditional, insofar as training students in mathematical logic and
  fundamentals of CS, so the mathematics modules complement the core structure very well.

* "(with a year in industry)": the best part. Between the second and third years, students temporarily leave university
  to work in industry for around a year in a role relevant to their course. For most CS students, this involves working
  at a medium-to-large company in some SWE-adjacent position. I completed my placement
  at [Thales Underwater Systems in Stockport, Greater Manchester](https://en.wikipedia.org/wiki/Thales_Underwater_Systems#Cheadle_Heath).
