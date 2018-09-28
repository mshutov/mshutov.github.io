---
layout: post
title: Add service to store translations in memory
last_modified_at: 2018-09-28
---
To dive into Spring Boot and related libraries let's create simple app that could be used as vocabulary.
Let's add new endpoints for our needs:
- `GET /translations` - to return all saved translations.
- `GET /translations/{word}` - to return translation for given word or `[No translation found]`
- `POST /translations` - to add new translation

To implement proposed functionality we need to create a service that will provide this features. 
Let's name it `TranslationService`. We can use `HashMap` or `ConcurrentHashMap` as a storage. 
To simplify things we can use `Map<String, String>` as return type for all translations.
I will use such methods in this Service:
- `void addTranslation(String, String)`
- `Map<String, String> getTranslations()`
- `Optional<String> getTranslation(String)`

One thing that we need to do is annotate our created service with `@Service` annotation so it will be discoverable when Spring prepares context.

We can add new methods to our existing controller, but it is better to create new one called `TranslationController`. Let's annonate it with one more annotation: `@RequestMapping(path = "/translations")`. 
By adding it we specify that this controller should handle all requests that start with `/translations/*`.
Now add 3 methods for each of our endpoints with specified method and path:
- `@GetMapping Map<String, String> getTranslations()`
- `@GetMapping(path = "/{word}") String getTranslation(@PathVariable String word)`
- `@PostMapping void addTranslation(@RequestParam String word, @RequestParam String meaning)`

Let's discuss these annotations. `*Mapping` marks method as handler for requests with corresponding `HTTP` method. Also it allows specify path - we already specified it for our class, so full path for getTranslation methods is `/translations/{words}`. `{word}` in path allows us to have a method parameter with same name, bu we need to annotate it with `@PathVariable` (we also can change name of variable we capture by adding name='smth' in annotation, e.g. we could use `@GetMapping(path = "/{word}") String getTranslation(@PathVariable(name="word") String term)` and this will work fine).

`@RequestParam` also interesting because allows us to get this params from different places - forms, query params. We also can specify name of param if we want to name our variables using other names).

Now you can commit this changes to branch you are working with or to `master` directrly (`master` is also just a branch but it is usually have a special role)

Here is my code for this controller:
```java
package com.example.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.util.Map;

@RestController
@RequestMapping(path = "/translations")
public class TranslationController {
    private final TranslationService translationService;

    public TranslationController(TranslationService translationService) {
        this.translationService = translationService;
    }

    @PostMapping
    public void createTranslation(@RequestParam String word, @RequestParam String meaning) {
        translationService.addTranslation(word, meaning);
    }

    @GetMapping
    public Map<String, String> getTranslations() {
        return translationService.findAll();
    }

    @GetMapping(path = "/{word}")
    public String getTranslations(@PathVariable String word) {
        return translationService.findByWord(word).orElse("[No translation]");
    }
}
```

And for service:
```java
package com.example.demo;

import org.springframework.stereotype.Service;

import java.util.Collections;
import java.util.Map;
import java.util.Optional;
import java.util.concurrent.ConcurrentHashMap;

@Service
public class TranslationService {
    private final ConcurrentHashMap<String, String> translations = new ConcurrentHashMap<>();

    public void addTranslation(String word, String meaning) {
        translations.put(word, meaning);
    }

    public Optional<String> findByWord(String word) {
        return Optional.ofNullable(translations.get(word));
    }

    public Map<String, String> findAll() {
        return Collections.unmodifiableMap(translations);
    }
}
```

In [next post]({% post_url 2018-09-28-add-real-db-to-store-data %}) we will use add real db (without installing any software as we do in initial post).
