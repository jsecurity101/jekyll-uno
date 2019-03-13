---
title:  "Injecting Into The Hunt"
date:   2019-03-12
categories: Detection
tags: 
---
Background:
---
Process Injection is a very common Defense Evasion/Privilege Escalation technique. Typically this will include injecting custom code into another processes address space.
There are many different routes one can go when it comes to the actual procedure of doing this attack. Really good article explaining the various process injection types: *[Endgame 10 Process Injection Techniques](https://www.endgame.com/blog/technical-blog/ten-process-injection-techniques-technical-survey-common-and-trending-process)*.

For this post, I performed a Reflective DLL Injection using the *[psinject](https://github.com/EmpireProject/Empire/blob/dev/lib/modules/powershell/management/psinject.py)* module from *[Empire](https://github.com/EmpireProject/Empire)*. Psinject is based off of Stephen Fewer's *[Reflective DLL Injection Technique](https://github.com/stephenfewer/ReflectiveDLLInjection)*, which will execute a Powershell Script from memory into a remote process. 
Reflective DLL injection will work by creating a DLL that maps itself into memory when executed, instead of relying on the Window’s loader. Which makes the injection process the same as a shellcode injection, just the shellcode is replaced with a self-mapping DLL. 
This Script will do the following to inject itself into a Remote Process:

1. Targets a process for injection
2. Calls VirtualAllocEx - to have a space to write its path to the DLL.
3. Calls WriteProcessMemory - to write the path in allocated memory.
4. Writes CreateRemoteThread - in order for the code to be executed.
5. Writes OpenThreadToken - to access the thread.

Here is a screenshot of what it will look like after injecting into a process:

![injection](/images/Injection.png)

You can see that the original agent was Powershell with a PID of 5636, then injected into the process win32calc with a PID of 5576.


Onto the Hunt:
---
In order to hunt, we need to create a hypothesis, this will help prevent analysis paralysis - Over analyzing an abundance of logs. To put this into perspective, say we are a Detection/Hunt Team and we got this alert from Microsoft ATA:

![Microsoft-ATA](/images/ATA.png)

Then upon further analysis we come across this:

![ATA](/images/ATA2.png)

This is a red flag, this shows someone has accessed our enviroment from an unkown source and could be moving around, collecting data, whatever they might be doing. So the Hunt begins to find this adversary and find the Technique they used to infiltrate our enviroment. 
As you can see from the above image of the injection, the original powershell PID is 5636. Inside of Sysmon Event ID, we see ID 1 - Create Process: 

![SYSMON](/images/cmd.png)

Does anything stick out? The parent process was Command Prompt, but moved into Powershell, how do we know this? Look at the command line argument.  You can see a payload (not showing full payload) was executed, lets investigate further. 

If we move into Olaf Hartong's *[Threat Hunting App](https://github.com/olafhartong/ThreatHunting)*, we can see these logs as well. The first is going to be a raw sysmon log. What I want to point out is the "Destination Host" has the IP address of the box I used to perform this attack. This isn't in our Network - big red flag: 

![ThreatRAW](/images/Injection-Raw-Sysmon.png)

The second log you can see a Powershell Execution with the Parent Process of cmd.exe, then if you look to the right you can see that initial payload we saw before. Right after, you can see 'RuntimeBroker.exe - Embedding'. 
Why is this significant? RuntimeBroker is used to help assist with permissions for applications and also acts as a middle man between Universal Windows Plateforms (UWP) and Full Windows API's. RuntimeBroker will also provide the required level of access for the UWP - which win32calc is a UWP. **SPOILER**
Which could be why we see RuntimeBroker -Embedding (because we embedded inside of a UWP).

![Threat-Hunting-Runtime](/images/Runtime.png)

The next step in our hunting process, after seeing the Powershell Execution (with Command Prompt being the Parent Process) and also seeing RuntimeBroker -Embedding, would be to go to see the Powershell Events. See what the script block logs are collecting. When doing so we come across Event ID 4104. We see here "Invoke-PSInject" was used along with the payload:

![Powershell-Invoke](/images/Invoke.png)

This can then lead our Hunt to finding where the adversary might be. Then used Jared Atkinson's *[Get-InjectedThread.ps1](https://gist.github.com/jaredcatkinson/23905d34537ce4b5b1818c3e6405c1d2)*.

![Get-InjectedThread](/images/Get-InjectedThread.png)

As you can see, there was a process injected into win32calc.exe. We have found the adversary! 

Conclusion:
---
This Hunt/Detection was meant to give an overview or a start, on how to possibly hunt for Process Injection Attack Techniques. I have always been facinated with Process Injection and Memory and thought this would be a cool blog to do. I plan on doing another blog on Process Injection/Memory in the future, but wanted to throw this blog out first. 
I want to say thank you to Olaf Hartong, Chris Long, Jared Atkinson for their work on the tools that I used in my blog. They have all been linked. I hope you have enjoyed this blog and found it informational! 

Resources:
---
***[Get-InjectedThread.ps1](https://gist.github.com/jaredcatkinson/23905d34537ce4b5b1818c3e6405c1d2)*** - **Jared Atkinson**

***[DetectionLab](https://github.com/clong/DetectionLab)*** - **Chris Long**

***[Threat Hunting App](https://github.com/olafhartong/ThreatHunting)*** - **Olaf Hartong**

***[Endgame 10 Process Injection Techniques](https://www.endgame.com/blog/technical-blog/ten-process-injection-techniques-technical-survey-common-and-trending-process)***

***[Mitre Attack- Process Injection](https://attack.mitre.org/techniques/T1055/)***
