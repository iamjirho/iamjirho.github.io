---
title: BTLO Phishing Analysis 2
date: 2024-05-28 00:00:00 +0800
categories: [Blue Team Labs Online]
tags: [walkthough,write-ups,osint]     # TAG names should always be lowercase
image:
    #path: https://imgur.com/zWkIpZg
---

Put your phishing analysis skils to the test by triaging and collecting information about a recent phishing campaign.

### **What is the sending email address?**
We can easily see the sender email address using phishtool if you prefer the automated method.
![Desktop View](/assets/images/phishing-analysis2/sending.png){: width="972" height="589" }

You can also manually search the eml file.
![Desktop View](/assets/images/phishing-analysis2/manual-sending.png){: width="972" height="589" }

### **What is the recipient email address?**
![Desktop View](/assets/images/phishing-analysis2/recipient.png){: width="972" height="589" }

### **What is the subject line of the email?**
![Desktop View](/assets/images/phishing-analysis2/subject.png){: width="972" height="589" }

### **What company is the attacker trying to imitate?**
![Desktop View](/assets/images/phishing-analysis2/amazon.png){: width="972" height="589" }
Aside from the sender email address, you can also see the footer of the email imitating amazon

### **What is the date and time the email was sent? (As copied from a text editor)**
![Desktop View](/assets/images/phishing-analysis2/date.png){: width="972" height="589" }

### **What is the URL of the main call-to-action button?**
Since I don't have Thunderbird or any email client installed on my virtual machine, I used an online tool to upload the EML file to view the content and see the email's main call-to-action button. I hovered over the button and copied the link by right-clicking on it.
![Desktop View](/assets/images/phishing-analysis2/url.png){: width="972" height="589" }


### **Look at the URL using URL2PNG. What is the first sentence (heading) displayed on this site? (regardless of whether you think the site is malicious or not)**
I don’t use the ur2png this time. I am using a virtual machine in opening this malicious email.
![Desktop View](/assets/images/phishing-analysis2/content.png){: width="972" height="589" }

### **When looking at the main body content in a text editor, what encoding scheme is being used?**
We can start looking for this information at the eml file itself.
![Desktop View](/assets/images/phishing-analysis2/encoding.png){: width="972" height="589" }

We can verify this using base64 decoder. Looks like this is the rendered mail body
![Desktop View](/assets/images/phishing-analysis2/rendered.png){: width="972" height="589" }

While this one is the html format of the body.
![Desktop View](/assets/images/phishing-analysis2/html.png){: width="972" height="589" }

### **What is the URL used to retrieve the company's logo in the email?**
The first question that comes to mind is, "Where can we find the URL?" We can find the URL from the decoded big chunk of base64 earlier since that is the HTML format of the body.

Since we know that the URL is encoded, we can just use [urldecoder.org](https://urldecoder.org/) to decode that for us.
![Desktop View](/assets/images/phishing-analysis2/retrieve.png){: width="972" height="589" }

### **For some unknown reason one of the URLs contains a Facebook profile URL. What is the username (not necessarily the display name) of this account, based on the URL?**

There is a link embedded i in the “Amazon Support Team” and upon hovering, we can see the username of the Facebook redirect. This is pretty easy so you can just find it by yourself.