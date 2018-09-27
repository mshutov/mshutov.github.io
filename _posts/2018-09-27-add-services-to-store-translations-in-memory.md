---
layout: post
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

