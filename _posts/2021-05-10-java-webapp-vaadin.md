---
layout: post
title: 'Writing a Web App in Java with Vaadin'
date: 2021-05-10 00:00:00
featured_image: '/assets/img/blog/2021-05-10/reindeer.png'
excerpt: There are many different ways to write a web application nowadays, but JavaScript frameworks like React and Vue reign supreme. Is it possible for a Java developer to write a web app with a limited knowledge of JavaScript? Yes, Vaadin makes it possible for Java developers to write rich web apps pretty much entirely in Java - let's review a simple example of such an app!
---

![](/assets/img/blog/2021-05-10/reindeer.png)

I'll be going over a super simple app here: it's a tool that takes two JSON objects as input, and highlights differences between those objects as output. This is a single page web app that contains an input section with two text areas for the user to enter their JSON (one text area for each object) and a button that runs the object comparison on the back end, and an output section that displays the JSON objects in a table format, where rows in the table are highlighted if a common field between the objects contains different values. I'll refer to the app as `json-compare` from here on out.

Here is the input section:
![](/assets/img/blog/2021-05-10/input-section.png)

And the output section:
![](/assets/img/blog/2021-05-10/output-section.png)

`json-compare` is a [Spring Boot](https://spring.io/projects/spring-boot) app. Vaadin [integrates nicely with Spring Boot](https://vaadin.com/spring), and while `json-compare` doesn't use Spring's dependency injection mechanism it does make use of libraries that are included in the Spring Boot dependency set such as Jackson for JSON processing and JUnit for unit testing. Spring also includes embedded Apache Tomcat which allows one to run a web app as a regular ol' JAR! On the build tools front Vaadin supports [Maven](https://vaadin.com/docs/latest/guide/start/maven-archetype) and [Gradle](https://vaadin.com/docs/latest/guide/start/gradle) - having support for the two most popular Java build tools means that Vaadin based web apps should fit into most CI/CD setups. For example, we can use Gradle to run the initial build for `json-compare` using the following command:
```shell
./gradlew vaadinBuildFrontend build
```

This will generate the HTML, CSS, and JavaScript from the Java code, then bundle those files with other assets via webpack to be served up when users access the website.

