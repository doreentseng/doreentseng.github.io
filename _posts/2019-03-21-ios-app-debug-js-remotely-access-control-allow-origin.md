---
layout: post
title: iOS App Debug JS Remotely with "No 'Access-Control-Allow-Origin'"
date: 2019-03-21
excerpt: "Black screen after debugging JS remotely."
category: iOS
tags: [iOS, app, ReactNative]
comments: true
---

## Description
I run my iOS app on a real device, everything works fine. Once I shake my device and click `Debug JS Remotely`, 
black screen shows up like this:  
<figure class="third">
	<img src="../assets/img/post/2019-03-21-1.png">
	<figcaption></figcaption>
</figure>

The following message shows on the console of Chrome: 
<figure>
	<img src="../assets/img/post/2019-03-21-2.png">
	<figcaption></figcaption>
</figure>
Only when I stop debugging JS remotely do the app work fine again.


## What I've Tried

Install ["Allow-Control-Allow-Origin" plugin](<https://chrome.google.com/webstore/detail/allow-cors-access-control/lhobafahddgcelffkeicbaginigeejlf> "Click") on Chrome and reload the app.
It works for some people except me, I've got another error...surprise!

## Solution

Make sure that your PC and device are connected to the same Wi-Fi network.
In the browser address bar, change url from 

**localhost:8081/debugger-ui**

to 

**http://\<your-pc-ip\>:8081/debugger-ui**

e.g. mine is http://192.168.1.209:8081/debugger-ui