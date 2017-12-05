---
layout: post
title:  "How to run a virus scanner on Azure"
date:   2018-03-01 13:45:00 +0100
categories: c-sharp clamav virus-scanner azure webjob
disqus: true
---

In this post we'll discuss how to install and run clamav on azure. By the end of the post you would have seen how to:

1. run clamav via in a console app
2. deploy the console app as a web job
3. trigger the virus scanner to run on a blob creation trigger

If you are like me and have googled your way around the web only to find how to’s, tutorials and stackoverflow answer from 2012 that dont actually tell you mich. you’ll probably come to the conclusion that ising a 3rd party service might be easier. Although it would be easier, I was told i could not transfer confidenial data outside an application or network you control, regardless of their service contracts.

so lets get into it. Step 1, lets get clamav running.

(run ClamAv code)

this console app is a very simple example of how to exectute clamav. there is a library by .... that can do the same. it has limited finctionality but for some of you it maybe a better solution.

the app first boots up ClamAv as a seperate service. we then update the virus database. once the service is running we can call it directly passing in the location of a file.

I have hosted this example on github. Feel free to download it and