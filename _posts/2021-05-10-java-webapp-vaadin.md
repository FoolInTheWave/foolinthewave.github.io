---
layout: post
title: 'Writing a Web App in Java with Vaadin'
date: 2021-05-10 00:00:00
featured_image: '/assets/img/blog/2021-05-10/reindeer.png'
excerpt: There are many different ways to write a web application nowadays, but JavaScript frameworks like React and Vue reign supreme. Is it possible for a Java developer to write a web app with a limited knowledge of JavaScript? Yes, Vaadin makes it possible for Java developers to write rich web apps pretty much entirely in Java - let's review a simple example of such an app!
---

![](/assets/img/blog/2021-05-10/reindeer.png)

I'll be going over a simple app at a high level here: it's a tool that takes two JSON objects as input, and highlights differences between those objects as output. This is a single page web app that contains an input section with two text areas for the user to enter their JSON (one text area for each object) and a button that runs the object comparison on the back end, and an output section that displays the JSON objects in a table format, where rows in the table are highlighted if a common field between the objects contains different values. I'll refer to the app as `json-compare` from here on out.

Here is the input section:
![](/assets/img/blog/2021-05-10/input-section.png)

And the output section:
![](/assets/img/blog/2021-05-10/output-section.png)

`json-compare` is a [Spring Boot](https://spring.io/projects/spring-boot) app. Vaadin [integrates nicely with Spring Boot](https://vaadin.com/spring), and while `json-compare` doesn't use Spring's dependency injection mechanism it does make use of libraries that are included in the Spring Boot dependency set such as Jackson for JSON processing and JUnit for unit testing. Spring also includes embedded Apache Tomcat which allows one to run a web app as a regular ol' JAR! The main class of the application will look familiar to those who use Spring Boot in their projects:

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;

@SpringBootApplication
public class Application extends SpringBootServletInitializer {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

On the build tools front Vaadin supports [Maven](https://vaadin.com/docs/latest/guide/start/maven-archetype) and [Gradle](https://vaadin.com/docs/latest/guide/start/gradle) - having support for the two most popular Java build tools means that Vaadin based web apps should fit into most CI/CD setups. For example, we can use Gradle to run the initial build for `json-compare`. The following Gradle command will generate the HTML, CSS, and JavaScript from the Java code, then bundle those files with other assets via webpack to be served up when users access the website:

```shell
./gradlew vaadinBuildFrontend build
```

Vaadin provides facilities to turn a web app into a [PWA (progressive web app)](https://en.wikipedia.org/wiki/Progressive_web_application), just by specifying the `@PWA` annotation in the class that extends the Vaadin `AppShellConfigurator` class (the "main class" for the Vaadin portion of the app):

```java
import com.vaadin.flow.component.page.AppShellConfigurator;
import com.vaadin.flow.server.PWA;
import com.vaadin.flow.theme.Theme;

@Theme(value = "json-compare-java")
@PWA(name = "json-compare-java", shortName = "json-compare-java", offlineResources = {"images/logo.png"})
public class AppShell implements AppShellConfigurator {
}
``` 

Also notice the `@Theme` annotation in the `AppShell` class above. In Vaadin apps the theme contains the assets (CSS, fonts, images, etc) that are used to style the entire application. Vaadin provides two themes out of the box: [Lumo and Material](https://vaadin.com/docs/latest/ds/customization/using-themes), with Lumo being the default theme for Vaadin apps. `json-compare` uses the Lumo theme with a custom color palette. Custom theme CSS files will reside in the `frontend/{theme-name}` directory of the application (`frontend/json-compare-java` for `json-compare`). The `frontend/{theme-name}/views` directory contains the CSS files for the different web pages of the app. Notice that there is a single `json-compare-view.css` file in the `frontend/json-compare-java/views` - this is the CSS file that contains the styles needed by the `JsonCompareView` class, which implements the only web page for `json-compare` (we'll go over the `JsonCompareView` in more depth later on). Also notice the CSS files under the `frontend/json-compare-java/components` directory, these are CSS files that will be used to apply custom styling to the UI components that come with Vaadin, which is a nice segway into the next topic...

Let's now turn our attention to the building blocks of Vaadin apps: [components](https://vaadin.com/components). Components are where Vaadin really shines, as they provide rich functionality right out of the box.
