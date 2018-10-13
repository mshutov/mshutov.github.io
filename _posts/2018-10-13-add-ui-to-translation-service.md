---
layout: post
title: Adding UI to our service
tags: [spring, spring boot, heroku]
last_modified_at: 2018-10-13
---
In this post we are going to achieve 2 goals:
- add UI (using mustache)
- add method to choose random translation

For UI let's use mustache - it is easy to use template engine. To add it to our project we need to add one dependency:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mustache</artifactId>
</dependency>
```
As everything in spring this starter has nice defaults:
```
spring.mustache.prefix=classpath:/templates/
spring.mustache.suffix=.mustache
```

So let's create folder `templates` under `resources`. Three things that we need to know about mustache syntax for now:
- `{{ var }}` is to replace this with value of `var`
- `{{> header }}` is to replace this with content of file header.mustache. We will use it to keep actual pages simpler
- `{{# var }} smth {{/var}}` - will output smth only if var is true (or not null)
Full documentation on syntax you can find at http://mustache.github.io/mustache.5.html

We previously created RestController, but now let's create controller for UI - `TranslationWebController` and add dependency to `TranslationService` through costructor.
One of the ways to pass variables to templates is to accept Spring's `Model` as method params, for example:
```java
@GetMapping
public String home(Model model) {
    model.addAttribute("title", "Main page");
    return "index";
}
```
what also interesting here is that we return `"index"` - this is a name of view (or template) that should be used. Because we added dependency on mustache starter Spring Boot knows that it should pass this name to mustache.
According to defaults there should be a file name `index.mustache` under `templates`.
Here it is:
```
{{> header }}
<div>
<h1>Demo</h1>
<div>Hello World</div>
<div><a href="/random">random</a></div>
</div>
{{> footer }}
```
And here content of `header.mustache`:
```
<!doctype html>
<html lang="en">
<head>
    <!-- Required meta tags -->
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

    <!-- Bootstrap CSS -->
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css"
          integrity="sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO" crossorigin="anonymous">
    {{#title}}
        <title>{{title}}</title>
    {{/title}}
</head>
<body>
<div class="container mt-2">
    <div class="row justify-content-md-center">
```
and `footer.mustache`:
```
</div>
</div>
</body>
</html>
```

Using such includes we can keep actual files with content pretty small and simple.
Also let's add two more templates
- `form.mustache` (will be used to create new translation):
```
{{> header }}
<div>
<h1>Create new translation</h1>
<form action="new" method="POST">
    <div class="form-group">
        <label for="word">Word:</label>
        <input type="text" id="word" name="word"/>
    </div>
    <div class="form-group">
        <label for="meaning">meaning:</label>
        <input type="text" id="meaning" name="meaning"/>
    </div>
    <input type="submit" value="Create"/>
</form>
{{#word}}
    <div><a href="/word/{{word}}">{{word}}</a></div>
{{/word}}
<div><a href="/">Home</a></div>
</div>
{{> footer }}
```
- `card.mustache` (to display translation pair):
```
{{> header }}
<div class="card">
    <div class="card-body">
        <h5 class="card-title">{{word}}</h5>
        <p class="card-text">{{meaning}}</p>
        <a href="/random" class="card-link">Open another</a>
        <a href="/" class="card-link">Home</a>
        <a href="/new" class="card-link">Add new</a>
    </div>
</div>
{{> footer }}
```

As you can see in `card.mustache` I have links to `/new` and `/random`. First one will open a form. Here is a method in our new controller:
```java
@RequestMapping(path = "new", method = {RequestMethod.GET, RequestMethod.POST})
public String createForm(@RequestParam(required = false) String word,
                         @RequestParam(required = false) String meaning,
                         Model model) {
    if (word != null && meaning != null) {
        translationService.addTranslation(word, meaning);
        model.addAttribute("word", word);
    }
    return "form";
}
```
What interesting here that this method is used for both `GET` and `POST`. 
If word and meaning are specified, then we save them and add `word` and `meaning` to `model`. 
In `form.mustache` this is used to display a link to just added word.

Here is method that used to display translation:
```java
@GetMapping(path = "word/{word}")
public String meaning(@PathVariable String word, Model model) {
    translationService.findByWord(word).ifPresent(tp -> {
        model.addAttribute("word", tp.getWord());
        model.addAttribute("meaning", tp.getMeaning());
    });
    return "card";
}
```
Nothing to explain here after previous explanations and posts.
