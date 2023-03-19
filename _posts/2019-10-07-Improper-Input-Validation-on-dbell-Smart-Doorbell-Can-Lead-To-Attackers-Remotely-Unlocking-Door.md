---
layout: post
title: "Improper Input Validation on dbell Smart Doorbell Can Lead To Attackers Remotely Unlocking Door"
excerpt: "CVE-2019-13336"
tags: [post, CVE]
comments: true
header-img: "https://drive.google.com/uc?export=view&id=1_CySfS1Gx7RM6__cCYRLC3q2gU6eiSgP"
---

 - CVE Number: CVE-2019-13336

<center><h3>Foreword</h3></center>

Before I get into this writeup, I'd like to thank Tamir Israel from the CIPPIC for helping me with legal issues regarding this disclosure and the EFF for referring me to him.
&nbsp;

This vulnerability allows any user to launch commands with no authentication verification through the doorbell’s web server. More specifically, if there is a lock connected to the relay switch on the doorbell, you can unlock the door locally on the network or remotely if it is exposed to the internet. Multiple email exchanges took place between me and dbell. **This vulnerability remains unpatched.**
&nbsp;

<center><h3>Details</h3></center>

After connecting the doorbell to my network, I started with a simple nmap port scan.

<img src="/assets/images/nmap.png" alt="nmap scan">

After going to port 81 in my browser, it led me to a GoAhead web server login request.

<img src="/assets/images/rsz_login.png" alt="GoAhead login">

I tried the default password which is listed on the back of the doorbell (admin:blank) and was given this interface.

<img src="/assets/images/rsz_page.png" alt="Page">

There were many cases of hardcoded credentials throughout the source code on the doorbell's webserver.
&nbsp;

<img src="/assets/images/rsz_creds1.png" alt="hardcoded creds 1">
<img src="/assets/images/rsz_creds2.png" alt="hardcoded creds 1">

But I was mostly interested in this commented out URL..
&nbsp;

<img src="/assets/images/rsz_url.png" alt="commented out URL">

&nbsp;
This looked to me as a gateway to reverse engineer the source code and call any of the functions.

Looking through the functions I discovered one that unlocks the door. For some background info, there is a small relay switch on the back of the doorbell. This switch can plug into a door lock, which can allow you unlock the door for a guest remotely.
&nbsp;

Upon entering this URL with your doorbell’s IP address into your browser, the doorbell lets out a “door is unlocked” voice message and will unlock the door for you.

> - **hxxp://xxx.xxx.xxx.xxx:81/openlock.cgi?loginuse=admin&loginpass=888888**

To make matters even worse, I accidentally discovered that those credentials don’t even matter. You can put absolutely whatever you want as the username and password values and it will execute.

> - **hxxp://xxx.xxx.xxx.xxx:81/openlock.cgi?loginuse=?????&loginpass=?????**

It is not limited to opening the lock either, any “.cgi” function that is on the webserver can be executed without it properly validating the input.
&nbsp;

Below is a POC video of the doorbell performing the unlocking function.
&nbsp;

[![Doorbell Unlocking POC](https://img.youtube.com/vi/SkTKt1nV57I/0.jpg
)](https://www.youtube.com/watch?v=SkTKt1nV57I)


<center><h3>Timeline</h3></center>

 - **July 4th, 2019** – Privately disclosed vulnerability to dbell.
 - **July 4th, 2019** – Reply from dbell thanking me and explaining its been discontinued for 3 years.
 - **July 4th, 2019** – I requested if the vulnerability could be published earlier since the product has been discontinued.
 - **July 5th, 2019** – dbell replies asking me not to disclose anything or else they’d take legal action and that I have malicious intent.
 - **July 9th, 2019** – I replied explaining that I am being responsible and will be giving them 90 days.
 - **July 9th, 2019** – dbell replied saying that there aren’t any apps on the app store to control this device, they care about their customers security and privacy…
 - **July 16th, 2019** – I explained there are still apps on the app store that can be used to control this device, which helped in finding this vulnerability.
 - **July 18th, 2019** – dbell replied with a very hostile email explaining again that I have malicious intent. They said that the email chain has been forwarded to their legal team and if I don’t stop in 10 days, they will file a case for extortion with the local police before taking legal action.
 - **August 27th, 2019** – Tamir from CIPPIC sent a letter through email explaining that their accusations of malicious intent and extortion have no legal basis.
 - **September 26th, 2019** – Tamir sent a 2 weeks notice to public disclosure in an email to dbell .
 - **October 7th, 2019** – Vulnerability is publicly disclosed.

