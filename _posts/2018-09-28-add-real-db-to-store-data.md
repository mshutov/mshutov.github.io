---
layout: post
---
We have a problem with our app inplementation that uses `ConcurrentHashMap` - it looses all data on app reload.
To solve this problem we need to add real DB. To avoid installation of any local DB let's continue with approach used in [initial post]({% post_url 2018-09-23-try-spring-boot %}) - let's add a DB to our app in Heroku. You need to open app dashboard and go to `Resources` tab: 
![Resources tab](/assets/img/1-Resources-1.png)

In search field (that you see on previous screenshot) type `postgres` and choose proposed variant. You will see popup:
![Resource provisioning popup](/assets/img/2-Resources-provision.png)

Confirm and you're done:
![Postgres added to resources](/assets/img/3-Resources-provision-done.png)

Heroku sets all variables that Spring 'searches' so you don't need to set host/port and credentials to use this DB in your app.

Ok, so far so good. But how we will add tables to DB? Of course we can use SQL console or UI app, but there is a better approach that is not hard to implement - DB migration libs. I know two popular - Liquibase and Flyway. Wo our app let's use former. Spring boot already have all required dependencies in dependency management section of pom, so we only need to know group and artifact. Let's also use dependencies for Postgres:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
</dependency>
<dependency>
    <groupId>org.liquibase</groupId>
    <artifactId>liquibase-core</artifactId>
</dependency>
```

We've added 3 dependencies - spring-boot-starter-jdbc (we now mainly insterested in it's feature to add ability to use `JdbcTemplate`. Other 2 dependencies I already explained.

Now, we need to add actual migration scripts. We have two options here - use Spring's default place for liquibase file or to specify it explicitly. Let's stick with latter variant - add `resources` folder under `src/main`. Create file named `application.properties` with single string:
```
spring.liquibase.change-log=classpath:db/liquibase-changelog.xml
```
This string tells Spring where to look for liquibase' changelog. Now create a folder `db` under `resorces` and create a file named `liquibase-changelog.xml` with content:
```xml
<databaseChangeLog
        xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd">

    <include file="changelog/001-initial.xml" relativeToChangelogFile="true"/>
</databaseChangeLog>
```

We really need to care about line `<include file="changelog/001-initial.xml" relativeToChangelogFile="true"/>` - this tag allows us to include other changelogs - this is a common approach to use `liquibase-changelog.xml` as file that includes other changelogs but have no actual migration in itself.

Let's create a `changelog` folder under 
