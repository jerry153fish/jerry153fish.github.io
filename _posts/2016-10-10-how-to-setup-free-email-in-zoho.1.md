---
layout: post
title:  How To Setup Free Email Host In Zoho
key:   20161010
categories: website
tags: domain email free zoho
comment: true
---

This article introduce how to set up a free business email in zoho.

### Preparation

- Must have a domain name first
- a phone number never apply zoho email before

### Zoho features

- 10 free accounts per domain ( possible get more )

- 5G for each accounts

- CRM, Project, Docs and other -- Free version of Google business


### Step one - register

Go to <a href="https://www.zoho.com/mail/" target="_blank">Zoho mail</a>

![zoho register](/assets/img/website/zoho-1.png)

Choose business email and select the free one

![zoho register 2](/assets/img/website/zoho-2.png)

Fill the whole form and click sign up then you should see this  

![register successfully](/assets/img/website/zoho-3.png)


### Step two - setup

Then you come to the setup steps

![zoho setup process](/assets/img/website/zoho-4-1.png)

- Verify domain -- modify DNS record to prove you have the ownship of certain domain

Open a new browser tab to your Domain DNS manage page of the registrar (eg godaddy),

![DNS manage](/assets/img/website/godaddy-manage-two.png)

You can choose CNAME or TXT method

Copy the record

![DNS manage](/assets/img/website/zoho-4.png)

Add new CNAME record in the DNS zone

![DNS manage](/assets/img/website/zoho-5.png)

*Wait for sometime ( an hour ) to click CNAME lookup button on center buttom*

![DNS manage](/assets/img/website/zoho-6-domain-verify.png)

After the domain is verfied. You can choose add users or groups according to youself, or you can add them later.

- add or edit mx record in DNS Zone

Now you need to tell you domain that zoho is hosting your email services.

Exactly same step as add CNAME record in the DNS zone. *If there were already mx records in the DNS, just edit them*.

Copy record

![DNS manage](/assets/img/website/zoho-7-mail-mx.png)

Add or edit

![DNS manage](/assets/img/website/godaddy-manage-mx.png)

Again, you need to wait for a while unitll the changes take effect.

![DNS manage](/assets/img/website/zoho-7-mail-mx-check.png)

You can set or skip the following steps in terms of you needs.


### Finally

Done! You can can enter your business email now. You can test it by sending mails to your other accounts.
