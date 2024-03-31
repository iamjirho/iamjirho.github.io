---
title: TryHackMe Sakura [CHALLENGE]
date: 2024-03-31 00:00:00 +0800
categories: [TryHackMe, Challenge]
tags: [walkthough,write-ups,osint]     # TAG names should always be lowercase
image:
    path: https://tryhackme-images.s3.amazonaws.com/room-icons/9a365314266266592078724ce18b248b.png
---

This room is designed to test a wide variety of different OSINT techniques. With a bit of research, most beginner OSINT practitioners should be able to complete these challenges. This room will take you through a sample OSINT investigation in which you will be asked to identify a number of identifiers and other pieces of information in order to help catch a cybercriminal.

## **TIP-OFF**
---
### **What username does the attacker go by?**

View the page source and found the highlighted text below which is the username.

![Desktop View](/assets/images/sakura/username.png){: width="972" height="589" }

## **RECONAISSANCE**
---
### **What is the full email address used by the attacker?**

If you have found her github page, you can look for PGP keys and extract the information using kleopatra. Install the application if it’s not built in to your machine.

```bash
sudo apt install kleopatra
```
Save the PGP keys to an extension .asc. Afterwards, import the key to Kleopatra

![Desktop View](/assets/images/sakura/email.png){: width="972" height="589" }

### **What is the attacker's full real name?**

Found her tweet from simple google search

![Desktop View](/assets/images/sakura/real-name.png){: width="972" height="589" }

[Twitter Profile](https://twitter.com/sakuraloveraiko?lang=en)

## **UNVEIL**
---
### **What cryptocurrency does the attacker own a cryptocurrency wallet for?**

On her GitHub repository, you can see an odd empty repo named **"ETH"**

![Desktop View](/assets/images/sakura/eth-wallet.png){: width="972" height="589" }

### **What is the attacker's cryptocurrency wallet address?**
Her ETH repository has been last updated on Jan 23, 2021, which means there are previous commits that happened on the repository before.

![Desktop View](/assets/images/sakura/eth-addy.png){: width="972" height="589" }

### **What mining pool did the attacker receive payments from on January 23, 2021 UTC?**
Knowing her ETH address, we can use an ETH blockchain explorer to hunt different transactions from that address. In my case, I use etherscan.io. There are many ETH blockchain explorer on the internet, just use whatever platforms you are comfortable using.

![Desktop View](/assets/images/sakura/eth-tx.png){: width="972" height="589" }

There are a total of 40 transaction for this specific address. In order to find the mining pool where Aiko receive payments from on January 23, 2021 UTC, click the Data Time (UTC) to give you a more proper format date rather than “x number ago”

### **What other cryptocurrency did the attacker exchange with using their cryptocurrency wallet?**

![Desktop View](/assets/images/sakura/eth-other.png){: width="972" height="589" }

Based on the blockchain explorer, the attacker withdraws USDT to the address `0xdAC17F958D2ee523a2206206994597C13D831ec7` from Bitfinex — another cryptocurrency exchange.

## **TAUNT**
---
### **What is the attacker's current Twitter handle?**
This one is pretty straightforward to find.

![Desktop View](/assets/images/sakura/twitter-handle.png){: width="972" height="589" }

### **What is the URL for the location where the attacker saved their WiFi  SSIDs and passwords?**
If you don’t feel like browsing through onion sites, you can just copy and paste the onion link shown in the image hint.

### **What is the BSSID for the attacker's Home WiFi?**
I use [wiggle](https://www.wigle.net/) to determine the BSSID which is basically the globally unique MAC address of the device. Based from the given hint, we can use `DK1F-G` to search for the attacker’s home wifi physical address.

![Desktop View](/assets/images/sakura/bssid.png){: width="972" height="589" }

## **HOMEBOUND**
---
### **What airport is closest to the location the attacker shared a photo from prior to getting on their flight?**

![Desktop View](/assets/images/sakura/bethesda.png){: width="972" height="589" }

Looking at this reposted tweet on 24th of January before Aiko travelled back home, you can see a word Bethesda which from a simple google search, Bethesda is found at the southern **Montgomery County, Maryland located just northwest of Washington, D.C**.

Again, from a simple web search, the nearest airport would be the  **Ronald Reagan Washington National Airport** with an approximate 45 minutes of travel time on a train

![Desktop View](/assets/images/sakura/near-airport.png){: width="972" height="589" }

### **What airport did the attacker have their last layover in?**

After conducting a reverse image search, it appears that JAL Sakura Lounges are found in several airports. However, one particular airport in Tokyo, Haneda, is similar to what the attacker shared on her Twitter account.

![Desktop View](/assets/images/sakura/haneda.png){: width="972" height="589" }

### **What lake can be seen in the map shared by the attacker as they were on their final flight home?**

Make use of google map search for the land comparison and zoom out.

![Desktop View](/assets/images/sakura/lake.png){: width="972" height="589" }

### **What city does the attacker likely consider "home"?**

Looking at the coordinates given by Wigle, we can determine which city in Japan she called as “Home”.

![Desktop View](/assets/images/sakura/coordinates.png){: width="972" height="589" }

![Desktop View](/assets/images/sakura/googlemaps.png){: width="972" height="589" }

## **Take Away Concepts**
- PGP keys can give you a wealth of information, such as usernames and emails.

- You can see the commit history in a GitHub repository, which might include sensitive information. Almost forgot this one.

- There are multiple lounges that seem to be similar in every airport.

- SVGs can contain useful information when being examined closely


