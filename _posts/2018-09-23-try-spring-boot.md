---
layout: post
title: Try Spring Boot without installing any software
tags: [spring, spring boot, heroku]
last_modified_at: 2018-09-28
---
Required preliminary steps:
- register [GitHub](https://github.com) account
- register [Heroku](https://heroku.com) account

Above steps require 3-4 minutes in total.
The easiest way to try Spring Boot is to fork (through GitHub site) [my repository](https://github.com/mshutov/demo-web-min) which consists of only 3 files:
- `pom.xml` - maven project file that contains dependencies and some project details
- `DemoApplication.java` - java class with single annotation `@SpringBootApplication` and `main` method
- `DemoController.java` - java class with several annotations that serves requests to `/`

After forking my repo you can go to [Heroku dashboard](https://dashboard.heroku.com) and create new app. 
Inside that app go to **Deploy** tab and in the **Deployment method** choose and connect to GitHub. After that choose your cloned repo.
After connecting to GitHub, select **Enable automatic deploys** in **Automatic deploys** section. And click **Deploy branch** in next section (we need to do this once).
After short time (after deploy finished) you will be able to see result - click **Open app** and you will see "Hello world"

Now let's go to repository and edit `DemoController.java` located in `src/main/java/com/example/demo`. 
Replace "Hello world" with your desired string and commit changes to `master`(not a good practice - usually it is better to work in branch and then create a pull request to `master`).
In Heroku dashboard you will notice that your commit triggered build (maybe you will have to wait 20-30 seconds). After all activity finished in **Activity** tab you can **Open app** again and see changed output.

As you can see we were able to create simple app working in a cloud that you can change and see results. And you can do that even without PC. 

In [next post]({% post_url 2018-09-27-add-services-to-store-translations-in-memory %}) we will add simple functionality to this app - ability to work as a dictionary.
