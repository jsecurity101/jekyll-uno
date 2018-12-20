---
title:  "Kerberos Enumeration Detection"
date:   2018-12-20 
categories: Detection
tags: 
---
Background:
---
Before we begin I want to explain what attack was done and how I performed it. Kerbos Enumeration in simple terms is a way to enumerate or collect user's on a certain domain or workstation. I used Kali Linux and Metasploit to perform this attack. I will not be showing how to perform this attack, as I don't think that is ethical, but I will be showing how I Detected this attack while I perfromed it.

Lab:
--
I have built a couple of personal labs on my own, but in this example (and many in the future) I will be using Chris Long's Detection Lab. Now I won't go into detail what this lab has inside but, if you haven't already built his lab and enjoy detection, then I would. He is constantly making great updates and adding great tools. One of those tools is the Threat Hunting App built inside of Splunk by Olaf Hartong. This is an amazing app, with many great features. If either you read this, thank you both so much for your guys hard work, and making these tools available. 

On to the fun stuff:
--
I will be showing results from Microsoft Advanced Threat Analytics, Splunk (Threat Hunting App), Suricata, and Wireshark. Inside my AD/DC I created a variety of users with a variety of privileges. I did this so I could understand the alerting/detection systems a better and how to personalize these alerts to better fit my enviroment. 

This first image is from my Kali Linux box after I performed the kerberos enum attack:

![Kerberos-Enumeration](/images/kerb-enum.jpg)

This shows the result of the actual attack and what users were actually on the DC. (You might enjoy the user theme I have going..) **Note** you will see there is the password for one user in here, this is because I put the password in the wordlist to see if it would return anything for results. 

This next image shows what Microsoft Advance Threat Analytics came up with. As you will see with the first image, it simply gives you a brief summary of the detection. This was only found as a "Medium" threat. 

![Mic-Anal](/images/anal1.png)

This next image shows the result's when you open the alert. One really neat thing I would like to point out is, it shows the existing and non-existing accounts inside of the wordlist. In other words, which accounts the attacker successfully acheived on retrieving. 

![Mic-Anal2](/images/anal2.png)

I have been dealing with Splunk quite a bit lately, specifically with Olaf Hartong's Threat Hunting App. This next image shows the results that Splunk returned after the attck. What I enjoy about this alert is the amnount of information that comes with it. 

![Splunk](/images/splunk.png)

This next image is from suricata. This was easy to capture, all you have to do is "tail -f /var/log/suricata/fast.log" during the attack. This alert is saying that someone/something has retrieved accounts and the "administrator privilege gain" is saying that one of those is an administrator. 

![Suricata](/images/suricata.png)

Last but not least! Wireshark. I have been doing alot of Traffic Analysis recently so I thought it would be cool to show this as well. I ran this on the Kali box while I was doing the attack. If you don't have CNameString as a column, I highly suggest. It comes in use when doing analysis. 
In this image you will see the accounts that were retrieved:

![Wireshark](/images/ws.png)

(I didn't screenshot the data of each one, didn't want to over do the images). After this you can then correlate that list with the list of accounts you have and investigate further.  

Summary
--
This was to show a basic detection for a basic attack. I thought it would be cool to show the alerts of different applications and how they all work very well together. Feel free to contact me with any questions.
I hope you all have enjoyed. 

