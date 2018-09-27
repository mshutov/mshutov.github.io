---
layout: post
title: Add service to store translations in memory
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
<script src="https://gist.github.com/mshutov/d46f6e6d8e98d598edc73b1eb9c5ad4f.js"></script>
