---
title:  "Kerberos Enumeration Detection"
date:   2018-12-20 
categories: Detection
tags: 
---
Background:
--
Before we begin I want to explain what attack was done and how I performed it. Kerbos Enumeration in simple terms is a way to enumerate or collect user's on a certain domain or workstation. I used Kali Linux and Metasploit to perform this attack. I will not be showing how to perform this attack, as I don't think that is ethical, but I will be showing how I Detected this attack while I perfromed it.

Lab:
--
I have built a couple of personal labs on my own, but in this example (and many in the future) I will be using Chris Long's Detection Lab. Now I won't go into detail what this lab has inside but, if you haven't already built his lab and enjoy detection, then I would. He is constantly making great updates and adding great tools. One of those tools is the Threat Hunting App built inside of Splunk by Olaf Hartong. This is an amazing app, with many great features. If either you read this, thank you both so much for your guys hard work, and making these tools available. 

On to the fun stuff:
--
Inside my AD/DC I created a variety of users with a variety of privileges. I did this so I could understand the alerting/detection systems a better and how to personalize these alerts to better fit my enviroment. 

This first image is from my Kali Linux box after I performed the kerberos enum attack:
![Kerb-Enum:](kerb-enum.png)

