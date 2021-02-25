---
title: 'Implementing a Simple Cache in Java'
date: 2021-02-25 00:00:00
featured_image: '/images/blog/2021-02-25/floppy.jpg'
excerpt: Every programmer keeps a set of reusable utility programs, patterns, and algorithms that they have accumulated over the years. One program in my tool belt that I've made use of multiple times in the past is a simple in-memory cache - let's see how it works!
---

![](/images/blog/2021-02-25/floppy.jpg)

## The need for fast data access

We all know about databases, they're a great solution for storing large amounts of data with facilities for managing data access from a multitude of consuming systems. However, there is a cost that comes with the security and efficiency that a database offers: processing overhead and added system complexity are a part of that cost. Sometimes we need to store a small amount of data that we need quick access to, especially when it comes to real-time applications such as system locking or session management, an in-memory "database" would do the trick here, otherwise known as a cache.

![](/images/blog/2021-02-25/cash-money.jpg)

Cash, cache... clever yeah? Bad jokes with old memes aside, let's see what the implementation of a simple cache in the Java programming language can look like!

