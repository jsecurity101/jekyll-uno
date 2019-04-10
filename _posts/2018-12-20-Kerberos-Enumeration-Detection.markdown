---
title:  "Kerberos Enumeration Detection"
date:   2018-12-20 
categories: Detection
tags: 
---
Background:
---
Before we begin I want to explain what attack was done and how I performed it. Kerberos Enumeration in simple terms is a way to enumerate or collect user's on a certain domain or workstation. I used Kali Linux and Metasploit to perform this attack. This attack is different then kerberoasting, this attack is simply using a wordlist to see if those accounts can authenticate on the system. I will not be showing how to perform this attack, as I don't think that is ethical, but I will be showing how I Detected this attack while I perfromed it.

Lab:
--
I have built a couple of personal labs on my own, but in this example (and many in the future) I will be using Chris Long's Detection Lab. Now I won't go into detail what this lab has inside but, if you haven't already built his lab and enjoy detection, then I would. He is constantly making great updates and adding great tools. One of those tools is the Threat Hunting App built inside of Splunk by Olaf Hartong. This is an amazing app, with many great features. If either you read this, thank you both so much for your guys hard work, and making these tools available. The link to their github's are at the bottom of the blog.

On to the fun stuff:
--
I will be showing results from Microsoft Advanced Threat Analytics, Splunk (Threat Hunting App), Suricata, and Wireshark. Inside my AD/DC I created a variety of users with a variety of privileges. I did this so I could understand the alerting/detection systems a better and how to personalize these alerts to better fit my enviroment. 

This first image is from my Kali Linux box after I performed the kerberos enum attack:

![Kerberos-Enumeration](/images/Kerberos-Enumeration-Detection/kerb-enum.jpg)

This shows the result of the actual attack and what users were actually on the DC. (You might enjoy the user theme I have going..) **Note** you will see there is the password for one user in here, this is because I put the password in the wordlist to see if it would return anything for results. 

This next image shows what Microsoft Advance Threat Analytics came up with. As you will see with the first image, it simply gives you a brief summary of the detection. This was only found as a "Medium" threat. 

![Mic-Anal](/images/Kerberos-Enumeration-Detection/anal1.png)

This next image shows the result's when you open the alert. One really neat thing I would like to point out is, it shows the existing and non-existing accounts inside of the wordlist. In other words, which accounts the attacker successfully acheived on retrieving. 

![Mic-Anal2](/images/Kerberos-Enumeration-Detection/anal2.png)

I have been dealing with Splunk quite a bit lately, specifically with Olaf Hartong's Threat Hunting App. This next image shows the results that Splunk returned after the attack. What I enjoy about this alert is the amount of information that comes with it. 

![Splunk](/images/Kerberos-Enumeration-Detection/splunk.png)

This next image is from suricata. This was easy to capture, all you have to do is "tail -f /var/log/suricata/fast.log" during the attack. This alert is saying that someone/something has retrieved accounts and the "administrator privilege gain" is saying that one of those is an administrator. 

![Suricata](/images/Kerberos-Enumeration-Detection/suricata.png)

Last but not least! Wireshark. I have been doing alot of Traffic Analysis recently so I thought it would be cool to show this as well. I ran this on the Kali box while I was doing the attack. I use the tag "Kerberos.CNameString" as a column in wireshark to help find credentials transferred in traffic. While using wireshark I have noticed that the Kerberos.CNameString tag comes in handy when an account was used to try and authenticate, very useful if someone get access to a workstation you can see what account was compromised. If you don't have Kerberos.CNameString as a column, I highly suggest. It comes in use when doing analysis. 
In this image you will see the accounts that were retrieved:

![Wireshark](/images/Kerberos-Enumeration-Detection/ws.png)

(I didn't screenshot the data of each one, didn't want to over do the images). After this you can then correlate that list with the list of accounts you have and investigate further.  

Summary
--
This was to show a basic detection for a basic attack. I thought it would be cool to show the alerts of different applications and how they all work very well together. Feel free to contact me with any questions.
I hope you all have enjoyed. 

Chris Long's Detection Lab:
https://github.com/clong/DetectionLab
Olaf Hartong's Threat Hunting App:
https://github.com/olafhartong/ThreatHunting
