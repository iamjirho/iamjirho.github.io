---
title: TryHackMe OhSINT [CHALLENGE]
date: 2024-03-31 00:00:00 +0800
categories: [TryHackMe, Challenge]
tags: [walkthough,write-ups,osint]     # TAG names should always be lowercase
image:
    path: https://tryhackme-images.s3.amazonaws.com/room-icons/9c6bc7e6db746ea68ecaa99e328923f1.png
---

What information can you possible get with just one photo?

### **What is this user's avatar of?**

We need to use [exiftool](https://exiftool.org/) to extract any information of the given picture.

Use any preferred operating system and follow the installation process on their documentation page.

```bash
exiftool WindowsXP.jpg
```

![Desktop View](/assets/images/ohsint/name.png){: width="972" height="589" }

Now that We have an author name `OWoodflint`. A basic google search is our first go to method to find any related information.

![Desktop View](/assets/images/ohsint/google.png){: width="972" height="589" }

### **What city is this person in?**

In one of the Google Search results, you can find the link pointing to github. See [here](https://github.com/OWoodfl1nt/people_finder)

![Desktop View](/assets/images/ohsint/peoplefinder.png){: width="972" height="589" }

### **What is the SSID of the WAP he connected to?**
Create an account for [wigle.net](http://wigle.net/) to access the Advanced Search.

Paste the BSSID you got from his [Twitter Post](https://twitter.com/OWoodflint/status/1102220421091463168)

![Desktop View](/assets/images/ohsint/ssid.png){: width="972" height="589" }

![Desktop View](/assets/images/ohsint/ssid2.png){: width="972" height="589" }

### **What is his personal email address?**
Visit his [github accont](https://github.com/OWoodfl1nt/people_finder)

![Desktop View](/assets/images/ohsint/email.png){: width="972" height="589" }

### **What site did you find his email address on?**
[Github accont](https://github.com/OWoodfl1nt/people_finder)

### **Where has he gone on holiday?**

It looks like his wordpress has been removed. Use wayback machine. Type `https://oliverwoodflint.wordpress.com/`

![Desktop View](/assets/images/ohsint/holiday.png){: width="972" height="589" }

### **What is the person's password?**

We can use inspect element to see what is inside.

![Desktop View](/assets/images/ohsint/hidden.png){: width="972" height="589" }

There is an interesting text inside the <p> tag and has class of “has-text-color” which could be defined to be hidden in the CSS which makes it to blend in the front-end.