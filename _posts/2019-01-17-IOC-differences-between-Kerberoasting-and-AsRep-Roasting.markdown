---
title:  "IOC differences between Kerberoasting and AS-REP Roasting"
date:   2019-01-17
categories: Detection 
---
**Background**:
---
Hello everyone! Thank you for tuning in. I was running some Kerberoast and AS-REP Roasting attack techniques on my Detection Lab, and noticed some really cool IOC (Indicator of Compromise) differences between the two.
Before we get started though I want to explain these two attacks. Although you could categorize these two attack as the same, they are two pretty different attacks. So lets break it down.

**Kerberoasting:**
To understand Kerberoasting, there is an item we need to define that plays a huge part during this attack technique.
Service Principal Names (SPN) is used to uniquely identify a Windows Service. Kerberos authentication requires that with each service logon account there must be a SPN associated. This allows a client to request a service authentication without having the actual account name through Kerberos authentication. You can read more about this at [MITRE ATT&CK](https://attack.mitre.org/techniques/T1208/). 

*The SPN is not automatically created when you create the user in Active Dirtectory, you HAVE to go and create the SPN. You can see below how to do this:* 

![set-spn](/images/set-spn.png)
![set-spn2](/images/set-spn2.png)

When the Kerberoasting attack technique is executed, an adversary can use Domain credentials captured on any user to request Kerberos TGS tickets for accounts that are associated with the SPN records in Active Directory (AD). The TGS tickets are signed with the targeted user or services NTLM hash. This can then be cracked offline to retrieve the clear text password. By default, the tools to automate this process will retrieve the TGS ticket in the encrypted RC4 algorithm. This is where we can start to build our baseline in detecting this attack. 
The adversary can then crack that hash with hashcat 13100 and a wordlist to find the password for that/those accounts. 

**AS-REP Roasting:**
AS-REP Roasting has the same IDEA of Kerberoasting but is different in the fact that an account needs "Do not require Kerberos preauthentication". For Kerberos v5 you have to manually go in and disable Kerberos pre-auth. The only reason I can think of someone to actually want to do this is for backwards compantibility with Kerberos v4 libraries, which by default a password was not required for authentication. Another difference between the two, is AS-REP requests a Kerberos Authentication Ticket (TGT) not a service authenitcation ticket (TGS).
The hashes you get between AS-REP and Kerberoasting are different. To crack the hash (if using hashcat you will need to change from 13100 to 18200 this is because Kerberoast requests TGS and AS-REP request TGT)

![Pre-Auth](/images/pre-auth-disabled.png)

*If you want to disable Kerberos Pre-Auth this is where you want to go*

**Detection (Same Logs):**
---
When doing this attack, I did it with the intent of collecting logs and IOC's. While doing this, I was honestly surprised at what tools turned out to use the same logs to detect these two attacks, but there were 2 tools in particular that detected these attacks differently and they gave some pretty cool logs. Before I show those tools I want to show the tools that gave the same logs. *Because they are the same I will only show one log, not both*

**Suricata:**

![suricata](/images/suricata-golden.png)


As you can see with this log you see a *Lateral movement* alert and *Exploit possible GoldenPac Priv Esc.* I was pretty impressed with this log. I wouldn't have thought suricata would have caught anything like this, but they did. So that was interesting.

**Splunk - Threat Hunting App:**

***Credential_Access:***

![Splunk-Golden](/images/splunk-golden.png)

***Raw Logs:***

![raw](/images/rawlogs.png)

*Image is very hard to see, I apologize, main thing I want to point out in this image is lsass.exe. I explain below.*

I really enjoy this Threat Hunting App, by Olaf Hartong. This tool has really helped me understand attacks and have given really good logs. With this app, I got the same logs with both attacks. As you can see there is alot of useful information within these logs, even though Im not going to explain these particular logs I do suggest you look into Splunk and this particular app and get familiar with logs like these. 

I will say though the 2 things that caught my eye was the 

1. *Credential_Access and Credenital Dumping* which means someone used their credentials to gain access and dumping credentials. Which is obviously a red flag.  

2. Raw Logs: *lsass.exe* which is Local Security Authority Subsystem Service. It verifies users logging in on Windows enviroments. These credentials are stored in protected memory and anyone with Domain Access can actually dump those credentials. This file is often faked by malware or malicous attacks that are being ran against your system. 



**Detection (Difference in Logs):**
---
I am going to show these logs, give a brief explanantion then do a *Difference* section to show and explain the differences in the logs and how you can detect one attack from the other. 

Kerberoast:
--
Native Windows Event Logging can be used to detect and alert the execution of the Kerberoast attack technique. For the robustness of this Detection to succeed, the Domain Controllers’ advanced security auditing policy, will need to be configured and enabled to log the Kerberos Authentication Service and Service Ticket Operations. This will allow the Domain Controller to log Kerberos Service Ticket requests. 

***Windows Event ID 4769:*** *Kerberos service ticket was requested* 

![4769](/images/windows-4769.png)


***Windows Event ID 4768:*** *Kerberos authentication ticket was requested* 

![4768](/images/windows-4768-kerberoast.png)

As you can see with this log, a kerberos service ticket was request. What I want to point out is a service *and* a authentication ticket was request when this attack was performed. Keep this in mind as we move forward to AS-REP. 


***Wireshark:*** *TGS-REQ/TGS-REP*

![tgs-req1](/images/kerberoast-wireshark.png)

With this Kerberoast attack you will see both AS-REQ/AS-REP before TGS-REQ/TGS-REP, because that is going to be the inital user authentication then you will see the TGS-REQ/TGS-REP because it authorizing the session ticket. That is because TGS stands for Ticket Granting Service and AS stands for Authentication Service. With version 5, Kerberos has 2 components for authorization: Ticket Granting Service (as you see here) and Authentication Service (as you will see here and in AS-REP). So it is authorizing the user through AS-REQ/AS-REP, then sending service ticket that was requested TGS-REQ/TGS-REP. 

***Wireshark:*** *TGS-REQ*

![tgs-req2](/images/kerberoast-request.png)

***Wireshark:*** *TGS-REP*

![tgs-req3](/images/kerberoast-response.png)

Through these two packets you can see what user was used to authenticate. Now I would like to point out, that I may have only showed one user, you will see more then one user TGS-REQ/TGS-REP under the sname tab. This is gathering other service namees being requested, this is now giving the attacker the tickets for other users, which the attacker can crack the hash for their passwords as well.  


AS-REP Roasting:
--

***Windows Event ID 4769:*** *Kerberos service ticket was requested* 

![as-4768](/images/windows-4768.png)


***Windows Event ID 4625:*** *An account failed to log on* 

![4625](/images/windows-4625.png)

I would like to point out, as you saw before with the Kerberoasting Windows Event's you saw both 4768 AND 4769. With AS-REP you see 4768 and 4625. This is because, the user does not need to be pre authenticated, so you don't need to know the password. If you have a password to a user, there is no need to do an AS-REP, but just do a Kerberoast. Then you will see the logs like the ones above. 

***Wireshark:*** *Invalid creds* ***Red Flag***

![invalid](/images/wireshark-invalid.png)

This stuck out to me immediately. You see a *bindRequest* with NTLMSSP-AUTH then then the user's id. Then you see *Invalid credentials.* Sure you get user's who forget their passwords and this might come up, but followed by As-REQ/AS-REP? I think not. That is way too coincidental for me. 

***Wireshark:*** *AS-REQ/AS-REP*

![asrep-asreq](/images/wireshark-asrep-asrec.png)


***Wireshark:*** *AS-REP*

![asrec](/images/asrec.png)


***Wireshark:*** *AS-REQ*

![asrep](/images/asrep.png)

In AS-REP I showed cname in the packet, where in Kerberoast I showed only sname. The reason for that is this, in Kerberoast you will see both and get a value for both because it is authenitcating through one user:cname and grabbing other service names: sname. I did this to show the difference in the attacks: AS-REP you will only get a value for cname. No sname. Why? It is not requesting a service names and not requesting service authentication. AS-REP is requesting Kerberos Authentication and because the user has "Pre-Authentication" disabled it doesn't need the right password to recieve the ticket. 

Difference:
-
1. In Windows Security Logs, Kerberoast will contain Event ID's 4768 and 4769, where in AS-REP contains Event ID's 4768 and 4625. The biggest indicator to me that one was AS-REP vs Kerberoast was the Failed login attempt along with there was no service ticket requested. 


2. I have pointed out a couple of differences of the attacks and their IOC's, but this is to summarize it. Kerberoast has AS-REQ/AS-REP AND  TGS-REQ/TGS-REP. AS-REP Roasting ONLY has AS-REQ/AS-REP. That is because Kerberoast is requesting a Service Account Authorization Ticket, where AS-REP is only requesting a Kerberos Authentication Ticket. 

Summary:
--
In this article I have shown the main IOC differences between two different attacks. Kerberoasting and AS-REP Roasting. The Detection for Kerberoasting I would admit would be hard, simply because requesting for service tickets does happen alot, **BUT** if you look for service request formated in RC4, then I am sure you will get better luck. 

I hope you enjoyed this article, feel free to email me with any questions or concerns. Add me on my Twitter, Linkedin, Github as well! My links are on the main screen. Thank you for tuning in. 

Sources:
--
Detection Lab - Chris Long - https://github.com/clong/DetectionLab

Impacket - Secure Auth - https://www.secureauth.com/labs/open-source-tools/impacket

Kerberoast - Tim Medin - https://www.sans.org/cyber-security-summit/archives/file/summit-archive-1493862736.pdf

AS-REP - harmj0y - https://www.harmj0y.net/blog/activedirectory/roasting-as-reps/

Kerberos - https://adsecurity.org
