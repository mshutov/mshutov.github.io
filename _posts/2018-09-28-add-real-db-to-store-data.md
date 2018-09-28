---
layout: post
title: Add real DB to store translations
excerpt_separator: <!--more-->
last_modified_at: 2018-09-28
---
We have a problem with our app inplementation that uses `ConcurrentHashMap` - it looses all data on app reload.
To solve this problem we need to add real DB. To avoid installation of any local DB let's continue with approach used in [initial post]({% post_url 2018-09-23-try-spring-boot %}) - let's add a DB to our app in Heroku.
<!--more-->
You need to open app dashboard and go to `Resources` tab: 
![Resources tab](/assets/img/1-Resources-1.png)

In search field (that you see on previous screenshot) type `postgres` and choose proposed variant. You will see popup:
![Resource provisioning popup](/assets/img/2-Resources-provision.png)

Confirm and you're done:
![Postgres added to resources](/assets/img/3-Resources-provision-done.png)

Heroku sets all variables that Spring 'searches' so you don't need to set host/port and credentials to use this DB in your app.

Ok, so far so good. But how we will add tables to DB? Of course we can use SQL console or UI app, but there is a better approach that is not hard to implement - DB migration libs. I know two popular - Liquibase and Flyway. For our app let's use former. Spring boot already have all required dependencies in dependency management section of pom, so we only need to know group and artifact. Let's also use dependencies for Postgres:
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

Let's create a `changelog` folder under `db` and add a single file `001-initial.xml`:
```xml
<databaseChangeLog
        xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd">
    <changeSet author="initial.setup" id="201809281000">
        <createTable tableName="translations" remarks="A table to contain all the translations">
            <column name="id" type="int" autoIncrement="true">
                <constraints nullable="false" primaryKey="true"/>
            </column>
            <column name="word" type="varchar(100)">
                <constraints nullable="false"/>
            </column>
            <column name="meaning" type="varchar(1024)">
                <constraints nullable="false"/>
            </column>
        </createTable>
    </changeSet>
</databaseChangeLog>
```

As you can see, now we added `changeset` tag with actual required actions. Here we `createTable` with `tableName`="translations". Table have 3 columns: `id` with auto increment that is used as primary key, `word` and `meaning`.

If you build you app now table will be created. Some details you can find in [this article](https://medium.com/@harittweets/evolving-your-database-using-spring-boot-and-liquibase-844fcd7931da) or in [liquibase documentation](http://www.liquibase.org/documentation/index.html)

Now we only need to add a `TranslationRepository` that will be used to talk with DB and hide details about that:
```java
package com.example.demo;

import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Optional;

@Repository
public class TranslationRepository {
    private final JdbcTemplate jdbcTemplate;

    public TranslationRepository(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public void addTranslation(String word, String meaning) {
        jdbcTemplate.update("insert into translations (word, meaning) values (?,?)", word, meaning);
    }

    public Optional<String> findByWord(String word) {
        return jdbcTemplate.queryForList("select meaning from translations where word=?", String.class, word)
                .stream().findFirst();
    }

    public List<TranslationPair> findAll() {
        return jdbcTemplate.query("select word, meaning from translations",
                (rs, rowNum) -> new TranslationPair(rs.getString(1), rs.getString(2)));
    }
}
```

`JdbcTemplate` has a lot of different methods that we can use to perform any DB query. `@Repository` - is stereotype annotation annotated with `@Component` so Spring can detect such classes during component scanning. `TranslationPair` is simple `data` class:
```java
package com.example.demo;

public class TranslationPair {
    private final String word;
    private final String meaning;

    public TranslationPair(String word, String meaning) {
        this.word = word;
        this.meaning = meaning;
    }

    public String getWord() {
        return word;
    }

    public String getMeaning() {
        return meaning;
    }
}
```

Now we only need to use `TranslationRepository` in `TranslationService`. Here what our service will look like after changes:
```java
package com.example.demo;

import org.springframework.stereotype.Service;

import java.util.HashMap;
import java.util.Map;
import java.util.Optional;

@Service
public class TranslationService {
    private final TranslationRepository repository;

    public TranslationService(TranslationRepository repository) {
        this.repository = repository;
    }

    public void addTranslation(String word, String meaning) {
        repository.addTranslation(word, meaning);
    }

    public Optional<String> findByWord(String word) {
        return repository.findByWord(word);
    }

    public Map<String, String> findAll() {
        Map<String, String> result = new HashMap<>();
        repository.findAll().forEach(pair -> result.put(pair.getWord(), pair.getMeaning()));
        return result;
    }
}
```

Becuase signature of methods remains unchanged we don't need to do any modifications inside `TranslationController`.

Commmit and push your changes now and see how app is working. Now you can restart (deploy) your app and translations don't disappear anymore.
