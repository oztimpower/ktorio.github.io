---
title: Fat JAR
caption: Fat JAR
category: servers
permalink: /servers/deploy/packing/fatjar.html
ktor_version_review: 1.0.0
---

A *fat-jar* (or *uber-jar*) archive is a normal jar file single archive packing all the dependencies together
so it can be run a standalone application directly using Java:

`java -jar yourapplication.jar`

This is the preferred way for running it in a container like [docker](/servers/deploy/containers.html#docker), when deploying to [heroku](/servers/deploy/hosting/heroku.html)
or when being reverse-proxied with [nginx](/servers/deploy/containers.html#nginx). 

## Gradle
{: #fat-jar-gradle}

When using Gradle, you can use the [`shadow`](https://imperceptiblethoughts.com/shadow/) gradle plugin to generate it. For example,
to generate a fat JAR using netty as an engine:

{% capture build-gradle %}
```groovy
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.github.jengelman.gradle.plugins:shadow:5.1.0' // For Gradle versions 5.x
    }
}

apply plugin: 'com.github.johnrengelman.shadow'
apply plugin: 'kotlin'
apply plugin: 'application'

//mainClassName = 'io.ktor.server.netty.DevelopmentEngine' // For versions < 1.0.0-beta-3
mainClassName = 'io.ktor.server.netty.EngineMain' // Starting with 1.0.0-beta-3

// This task will generate your fat JAR and put it in the ./build/libs/ directory
shadowJar {
    manifest {
        attributes 'Main-Class': mainClassName
    }
}
```
{% endcapture %}

{% capture build-gradle-kts %}
```kotlin
plugins {
    application
    kotlin("jvm") version "1.3.21"
    // Shadow 5.0.0 requires Gradle 5+. Check the shadow plugin manual if you're using an older version of Gradle.
    id("com.github.johnrengelman.shadow") version "5.0.0"
}

application {
    mainClassName = "io.ktor.server.netty.EngineMain"
}

tasks.withType<Jar> {
    manifest {
        attributes(
            mapOf(
                "Main-Class" to application.mainClassName
            )
        )
    }
}
```
{% endcapture %}

{% include tabbed-code.html
    tab1-title="build.gradle" tab1-content=build-gradle
    tab2-title="build.gradle.kts" tab2-content=build-gradle-kts
    no-height="true"
%}

## Maven
{: #fat-jar-maven}

When using Maven, you can generate a fat JAR archive with the `maven-assembly-plugin`. For example, to generate
a fat JAR using netty as an engine:

{% capture pom-xml %}
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>3.1.0</version>
    <configuration>
        <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
        <archive>
            <manifest>
                <addClasspath>true</addClasspath>
                <mainClass>io.ktor.server.netty.EngineMain</mainClass>
            </manifest>
        </archive>
    </configuration>
    <executions>
        <execution>
            <id>assemble-all</id>
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```
{% endcapture %}

{% include tabbed-code.html
    tab1-title="pom.xml" tab1-content=pom-xml
    no-height="true"
%}
