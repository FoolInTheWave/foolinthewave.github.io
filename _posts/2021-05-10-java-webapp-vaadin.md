---
layout: post
title: 'Writing a Web App in Java with Vaadin'
date: 2021-05-10 00:00:00
featured_image: '/images/blog/2021-05-10/reindeer.png'
excerpt: There are many different ways to write a web application nowadays, but JavaScript frameworks like React and Vue reign supreme. Is it possible for a Java developer to write a web app with a limited knowledge of JavaScript? Yes, Vaadin makes it possible for Java developers to write rich web apps pretty much entirely in Java - let's review a simple example of such an app!
---

![](/assets/img/blog/2021-05-10/reindeer.png)

I'll be going over a super simple app here: it's a tool that takes two JSON objects as input, and highlights differences between those objects as output. This is a single page web app that contains an input section with two text areas for the user to enter their JSON (one text area for each object) and a button that runs the object comparison on the back end, and an output section that displays the JSON objects in a table format, where rows in the table are highlighted if a common field between the objects contains different values.

Here is the input section:
![](/assets/img/blog/2021-05-10/input-section.png)

And the output section:
![](/assets/img/blog/2021-05-10/output-section.png)