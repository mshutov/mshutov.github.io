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

Ok, so far so good. But how we will add tables to DB? Of course we can use SQL console or UI app, but there is a better approach that is not hard to implement - DB migration libs. I know two popular - Liquibase and Flyway. Wo our app let's use former. Spring boot already have all required dependencies in dependency management section of pom, so we only need to know group and artifact. Let's also use dependencies for Postgres.
