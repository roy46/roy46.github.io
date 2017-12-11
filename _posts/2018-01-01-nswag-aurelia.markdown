---
layout: post
title:  "Swagger definition to generate front end clients via Nswag"
date:   2018-03-01 13:45:00 +0100
categories: c-sharp sql server integration testing
disqus: true
live: false
---

In this post we'll discuss how to automatically generate client side services that can call your api using a swagger definition and nwag.

For thise that haven’t heard of nswag it is a .................(find decription)

On the past i have used other methods to generate client side repositories or services that can call the backend api. mainly the use of T4 templates and refelection. However these were always limitibg as you need the dll that houses the controllers to generate anything.

The use of swagger in the company I currently work for has been a breath of fresh air. Easy to use, standardised ways for sharing api definitions. its as if we hav WCF back.

I will assume you have a json swagger definition. so the first step is installing nwsag into your application via npm (or yarn)

`npm install nswag —save-dev`

Now we need a .nswag defibition file for pur settings:


.......sample file

the documenetation for each if these settings can be found here: [nseag settings]()

Once we have all the settibgs we want we can run:

`nswag run`

This will find any .nswag config we have and generate then services we have asked for. Note that you can have as many .nswag configs as you like. meaning you you can generate service for muliple swagger defibitions.

In my next post i’ll show you how to extend the return models so you can add client side finction and propteries to the generated models.
