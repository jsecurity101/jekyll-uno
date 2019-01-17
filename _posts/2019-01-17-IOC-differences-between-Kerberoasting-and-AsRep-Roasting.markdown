---
title:  "IOC differences between Kerberoasting and As-Rep Roasting"
date:   2019-01-17
categories: Detection
tags: 
---
Background:
---
Hello everyone! Thank you for tuning in. I was running a Kerberoast and As-Rep Roasting attack's on my Detection Lab, and noticed some really cool IOC (Indicator of Compromise) differences between the two.
Before we get started though I want to explain these two attacks. Alot of people categories these attacks as the same, but they are two pretty different attacks. So lets break it down.

**Kerberoasting:**
You have an account, and with that account you have a SPN (Service Principal Name). *This doesn't come automatically when you create the user, you HAVE to go and create the SPN.* This allows a client to request a service authentication without having the actual account name through Kerberos authentication. 
The SPN is going to be a unique identifier of that account. Through kerberos, when you request a ticket for that specific spn it will send back an ecrypted ticket. So what the attacker will do is request a service ticket (TGS) specifically in RC4 format by default. That encryption with the NTLM hash of that account.
The attacker can then crack that hash with hashcat 13100 and a wordlist to find the password for that/those accounts. 

**Note** I will not show how I did this attack, but I did this attack with Kali Linux/Impacket. 

**As-Rep Roasting:**
As-Rep Roasting has the same IDEA of kerberoasting but is different in the fact that an account needs "Do not require Kerberos preauthentication". For Kerberos v5 you have to manually go in and disable Kerberos pre-auth. The only reason I can think of someone to actually want to do this is for backwards compantibility with Kerberos v4 libraries, which by default a password was not required for authentication. Another difference between the two, is As-Rep requests a Kerberos Authentication Ticket (TGT) not a service authenitcation ticket (TGS).
The hashes you get between As-Rep and Kerberoasting are different. To crack the hash (if using hashcat you will need to change from 13100 to 18200 this is because Kerberoast requests TGS and As-Rep request TGT)

*If you want to disable Kerberos Pre-Auth this is where you want to go*
![Pre-Auth](/images/pre-auth-disabled.png)

---
Detection:
---
