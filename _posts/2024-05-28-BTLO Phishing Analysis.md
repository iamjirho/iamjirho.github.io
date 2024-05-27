---
title: BTLO Phishing Analysis
date: 2024-05-28 00:00:00 +0800
categories: [Blue Team Labs Online]
tags: [walkthough,write-ups,osint]     # TAG names should always be lowercase
image:
    #path: https://tryhackme-images.s3.amazonaws.com/room-icons/cbd741886fe1f0902d60525c81378eab.png
---

A user has received a phishing email and forwarded it to the SOC. Can you investigate the email and attachment to collect useful artifacts?

### **Who is the primary recipient of this email?**

You can use [Phishtool](https://www.phishtool.com/) to provide an automated analysis of the email header.
![Desktop View](/assets/images/phishing-analysis/recipient.png){: width="972" height="589" }

### **What is the subject of this email?**
![Desktop View](/assets/images/phishing-analysis/subject.png){: width="972" height="589" }

### **What is the date and time the email was sent?**
![Desktop View](/assets/images/phishing-analysis/dateandtime.png){: width="972" height="589" }

### **What is the Originating IP?**
Initially, I believed that the originating IP address could only be identified at the first hop from where the IP address originated. However, it appears that the originating IP address can also be determined from the X-headers.
![Desktop View](/assets/images/phishing-analysis/x-origin.png){: width="972" height="589" }


### **Perform reverse DNS on this IP address, what is the resolved host? (whois.domaintools.com)**
![Desktop View](/assets/images/phishing-analysis/resolved.png){: width="972" height="589" }
We can also use [dnschecker](https://dnschecker.org) to locate the name of host from which the IP is assigned to.

### **What is the name of the attached file?**
Website contact form submission.eml

### **What is the URL found inside the attachment?**
I think the lesson learned here is that the sender is trying to trick you into clicking the partial URL.
![Desktop View](/assets/images/phishing-analysis/url.png){: width="972" height="589" }

### **What service is this webpage hosted on?**
Using [urlscan](https://urlscan.io), we can determine that the link redirects us to main domain and seems like the service it is offering is a blogspot.
![Desktop View](/assets/images/phishing-analysis/service.png){: width="972" height="589" }

### **Using URL2PNG, what is the heading text on this page? (Doesn't matter if the page has been taken down!)**
Go to the [url2png](https://url2png.com/) and paste main domain you’ve found using the [urlscan](https://urlscan.io) and copy the heading text once the screenshot is shown.
![Desktop View](/assets/images/phishing-analysis/url2png.png){: width="972" height="589" }



## **Take Away Concepts**
- x-origin header can also give you an information about the source IP