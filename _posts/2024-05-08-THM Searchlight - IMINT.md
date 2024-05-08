---
title: TryHackMe Searchlight - IMINT
date: 2024-05-08 00:00:00 +0800
categories: [TryHackMe, Challenge]
tags: [walkthough,write-ups,osint]     # TAG names should always be lowercase
image:
    path: https://tryhackme-images.s3.amazonaws.com/room-icons/cbd741886fe1f0902d60525c81378eab.png
---

OSINT challenges in the imagery intelligence category

## **Your first challenge!**
---
### **What is the name of the street where this image was taken?**

Simply examine the image

![Desktop View](/assets/images/searchlight/streetname.png){: width="972" height="589" }

## **Just Google it!**
---
### **Which city is the tube station located in?**

Searching for “Public Subway Undergound”

![Desktop View](/assets/images/searchlight/subway.png){: width="972" height="589" }

### **Which tube station do these stairs lead to?**

You can easily search for “public subway underground london circus”

![Desktop View](/assets/images/searchlight/stairs.png){: width="972" height="589" }

### **Which year did this station open?**

![Desktop View](/assets/images/searchlight/year.png){: width="972" height="589" }

### **How many platforms are there in this station?**

![Desktop View](/assets/images/searchlight/platforms.png){: width="972" height="589" }

Piccadilly Circus station in London serves the Piccadilly and Bakerloo lines of the London Underground. The station has two platforms for each of these lines, totaling four platforms

## **Keep at it!**
---
### **Which building is this photo taken in?**

Search for “yvr connects”

![Desktop View](/assets/images/searchlight/building.png){: width="972" height="589" }

### **Which country is this building located in?**

![Desktop View](/assets/images/searchlight/country.png){: width="972" height="589" }

### **Which city is this building located in?**

![Desktop View](/assets/images/searchlight/city1.png){: width="972" height="589" }
![Desktop View](/assets/images/searchlight/city2.png){: width="972" height="589" }

## **Coffee and a light lunch**
---
### **Which city is this coffee shop located in?**

Using web browser, I tried searching for “edinburgh woollen mill scotland” and it gave me few articles about it. It turns out that this is clothing store with branches across multiple locations. While exploring, I found this news articles “***Edinburgh Woollen Mill Group rescued but uncertainty remains for region's stores*”** that has the same physical building infrastracture of the given picture.

![Desktop View](/assets/images/searchlight/coffeecity.png){: width="972" height="589" }

### **Which street is this coffee shop located in?**

After finding the location of the clothing store which is taken from inside the coffee shop, it would be easy for us to determine the exact location since the coffee is just in front of the clothing store and by using google map we can just pin point exactly which street is the coffee shop located.

![Desktop View](/assets/images/searchlight/pinpoint.png){: width="972" height="589" }

### **What is their phone number?**

You can easily find it along the basic information that the google map gave you

![Desktop View](/assets/images/searchlight/phone.png){: width="972" height="589" }

### **What is their email address?**

They have their facebook page!

![Desktop View](/assets/images/searchlight/email.png){: width="972" height="589" }

### **What is the surname of the owners?**

I just saw this little unclickable link which seems to be their website. Found this from the results given from google maps

![Desktop View](/assets/images/searchlight/owner.png){: width="972" height="589" }

## **Reverse your thinking**
---
### **Which restaurant was this picture taken at?**

You can just use Yandex image search here.

![Desktop View](/assets/images/searchlight/resto.png){: width="972" height="589" }


### **What is the name of the Bon Appétit editor that worked 24 hours at this restaurant?**

Simple search “katz's deli editor”. Turns out, Bon Appétit is a youtube channel where the editor worked for 24 hrs at Katz’s Deli.

![Desktop View](/assets/images/searchlight/youtube.png){: width="972" height="589" }

## **Locate this sculpture**
---
### **What is the name of this statue?**

Tried reversing the image first using Yandex which gave me quite accurate results and eventually landed on the “[www.visitoslo.com/](http://www.visitoslo.com/)” page. Luckily, they gave a map full of statues, enough to give us the name of the creepy looking creature.

![Desktop View](/assets/images/searchlight/visitoslo.png){: width="972" height="589" }

![Desktop View](/assets/images/searchlight/rudolph.png){: width="972" height="589" }

### **Who took this image?**

![Desktop View](/assets/images/searchlight/photographer.png){: width="972" height="589" }

## **...and justice for all**
---
### **What is the name of the character that the statue depicts?**

![Desktop View](/assets/images/searchlight/statue1.png){: width="972" height="589" }

I usually do a reverse image search and translate the results

![Desktop View](/assets/images/searchlight/translate.png){: width="972" height="589" }

Upon translating, looks like we have our clue “the goddess of justice”

![Desktop View](/assets/images/searchlight/ladyjustice.png){: width="972" height="589" }

Falling down the rabbit hole, we can determine that the the goddess of justice is called Themis also known as “Lady Justice”

### **where is this statue located?**

From a reverse image search, I was able to find an article that uses the same image of statue crediting the location where it was taken

![Desktop View](/assets/images/searchlight/credit.png){: width="972" height="589" }

### **What is the name of the building opposite from this statue?**

When we have the location, we can easily use google maps for it to know which establishments or infrastructure surrounds the point of interest.

![Desktop View](/assets/images/searchlight/statue_building.png){: width="972" height="589" }


## **The view from my hotel room**
---
### **What is the name of the hotel that my friend stayed in a few years ago?**

Looks like the building was destroyed some time after the video footage was created.

![Desktop View](/assets/images/searchlight/hotel1.png){: width="972" height="589" }

![Desktop View](/assets/images/searchlight/hotel2.png){: width="972" height="589" }


## **Take Away Concepts**
- PGP Keys can give you a wealth of information such as username and emails.
- You can see the commit history in github repository which might include sensitive information.
- There are multiple lounge that seems to be similar in every airport.
- SVG can contain useful information when is being examined closely.