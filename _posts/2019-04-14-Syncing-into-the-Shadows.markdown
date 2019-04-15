---
title:  "Syncing Into the Shadows"
date:   2019-04-14 
categories: Detection
---

Introduction:
---
<p>As an adversary, one of the goals is to capture Domain Admin (DA) credentials, change/modify objects inside of Active Directory, and to be able to evade any detection systems that an environment may have in place.</p>
<p>One way you can capture DA credentials is through an attack technique called “DCSync”. DCSync is an attack technique that many security professionals, like <a href="https://adsecurity.org/?p=1729"><strong>Sean Metcalf</strong></a> and <a href="http://www.harmj0y.net/blog/redteaming/mimikatz-and-dcsync-and-extrasids-oh-my/"><strong>Will Schroeder</strong></a> have talked about. Once an adversary has DA privileges, they can then perform a defensive evasion technique attack, by injecting objects into the Active Directory Infrastructure. This attack technique is called “DCShadow”. There is a great presentation on DCShadow that was done by <a href="https://www.dcshadow.com/"><strong>Benjamin Delpy and Vincent Letoux</strong></a> which I highly suggest going to, to read and watch.</p>
<p>DCSync and DCShadow sound very similar and could be confusing to understand the differences if not explained. I am going to talk about the differences in DCSync and DCShadow when it comes to their functionality as an attack technique, along with differences when it comes to Indicators of Compromise (IOC) and hunting/detecting these two techniques.</p>
<p>When running these two attacks I wanted to have some fun with it, as I am a big Marvel fan, let me know if you catch any of the references and WHY some of the users were used. I will explain at the end ☺</p>

Background:
---
<p>To start taking a look at these two attacks in a Tactics, Technique, and Procedure (TTP) standpoint, can help give a baseline at the differences in these two attack and what they are used for. This is from the <a href="https://attack.mitre.org/"><strong>Mitre Att&amp;ck Framework</strong></a>:</p>
<table>
<thead>
<tr class="header">
<th><strong>Tatics</strong></th>
<th><strong>Techniques</strong></th>
<th><strong>Procedure</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Credential Access</td>
<td>DCSync</td>
<td>Mimikatz</td>
</tr>
<tr class="even">
<td>Defense Evasion</td>
<td>DCShadow</td>
<td>Mimikatz</td>
</tr>
</tbody>
</table>
<p><strong>DCSync:</strong></p>
<p>DCSync has been around for a while and is used quite often. This technique is used to retrieve and dump credentials of a specified account. In order to do this, the user you are using to ‘sync’ to the Domain Controller (DC), must be a part of a high privileged group: Administrators, Domain Admins, Enterprise Admins, or a Domain Controller computer accounts. This is done by impersonating the Domain Controller, while doing so it requests user account credentials from the targeted DC. This example is demonstrated in the “On to the Attack” section.</p>

<p><strong>DCShadow:</strong></p>
<p>DCShadow is used for Defense Evasion by modifying/pushing object changes inside of the Active Directory (AD) Infrastructure. For example, say you have Domain Admin (DA) credentials and want to avoid being caught in an environment, you can push a user into the Domain Administrators group, then use that user account to move around in the environment/modify other objects and attributes in the AD infrastructure. How is this done?</p>

<p><strong><i>Non-Technical Overview:</i></strong> This is done by registering the host machine you are on as a (rogue) Domain Controller, creating/modifying objects then pushing them out to the legitimate Domain Controller in the environment.</p>

<p><strong><i>Technical Overview:</i></strong> Inside of each Domain Controller, there is a built-in process called the knowledge consistency checker <a href="https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/replication/active-directory-replication-concepts"><strong>(KCC)</strong>.</a> This handles the replication topology for the Active Directory forest. Also, inside of the Domain Controller is a Directory System Agent (DSA) called Ntdsa.dll, that runs. This is to provide access to the directory database inside of Active Directory (AD). Within the DSA is a forest-wide object <a href="https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-srpl/4c62c74a-b55c-47d1-b575-33395a727d97"><strong>(nTDSDSA)</strong></a> that represents the DSA on the Domain Controller. DCShadow allows the adversary to create a new nTDSDSA object in the rogue Domain Controller and replicate that change to the legitimate Domain Controller because of the KCC.</p>
<strong>Note</strong> In order to successfully complete this attack you MUST already have Domain Admin or Enterprise Admin privileges.

Onto the Attack:
---
<p><strong>DCSync:</strong></p>

<a href="https://jsecurity101.tinytake.com/sf/MzQ1NjQzNV8xMDM1NTA1Mg">![dcsync](/images/DCSync-vs-DCShadow/DCSync.PNG)</a>

<p>Let’s take a closer look and talk about what is going on during the DCSync attack and go over why I demonstrated the DCSync first along with the privileges used to perform this attack:</p>
<p>An adversary has enumerated the user: ironman@windomain.loca, which is in the Administrators Group, what does this mean? This user has full control over the Domain Controller(s) in this domain. BUT we want Domain Admin (DA) so the adversary can have control over the whole domain.</p>
<p>A next step might be to check to see who is in the DA group, so we can target said user. To check this, the command ran was:</p>
<ul>
<li><p><strong>domain groups “domain admins” /domain</strong></p></li>
</ul>
<p>See that “vision” is a user in the DA group. This could be a good target. To grap the user’s ntlm hash, this command can be ran inside of <a href="https://github.com/gentilkiwi/mimikatz"><strong>Mimikatz</strong></a>:</p>
<ul>
<li><p><strong>lsadump::dcsync /domain:windomain.local /user:vision</strong></p></li>
<li><p><strong>lsadump::dcsync /push</strong></p></li>
</ul>
<p>Users ntlm hash:</p>
<ul>
<li><p><strong>ac8e786b4305cf56937c8a08b175ed6b</strong></p></li>
</ul>
<p>After cracking this, Vision’s password is <strong>LastStone1!</strong> (hint hint)</p>

![vision](/images/DCSync-vs-DCShadow/vision.webp)

<p>Now that DA credentials have been captured and since the adversary knows that DCSync can be detected, why not evade detection and inject another user into the DA Group?</p>

<p><strong>DCShadow:</strong></p>

<a href="https://jsecurity101.tinytake.com/sf/MzQ1NjczOF8xMDM1NTk4NQ">![dcsync](/images/DCSync-vs-DCShadow/DCShadow.PNG)</a>

<p>As shown in the video, the user logged in is: vision@windomain.local (member of the Domain Administrator’s Group). The adversary wants to evade any detection or hunting that is currently being done due to logs that have propagated (which I show below). Great way to do so is to inject a user into the DA Group, then use that user. Say there was a user in the domain named: thanos@windomain.local. Sounds like a perfect fit, since historically Vision practically gave Thanos the last infinity stone, making him all powerful.</p>
<p>I want to point out, that in this attack we are modifying two things. Firstly, the computers (win10.windomain.local) attribute to classify as a Global Catalog (GC- a role given to one or more Domain Controllers in the environment so that it can store data about every object in the forest). This allows the computer to be a rogue Domain Controller and push out modifications to objects to the legitimate Domain Controller. Secondly, we are modifying Thanos’ privileges to give him DA rights.</p>
<p>How do can this be done? By running these commands inside of <a href="https://github.com/gentilkiwi/mimikatz">Mimikatz</a>:</p>
<ul>
<li><p><strong>privilege::debug</strong> (Allows user to debug a process they wouldn’t otherwise have access to)</p></li>
<li><p><strong>!+</strong> (Registers and starts a service with system level privileges)</p></li>
<li><p><strong>!processtoken</strong> (Gives the System Token to Mimikatz so it has the appropriate privileges to run the following commands)</p></li>
<li><p><strong>lsadump::dcshadow /object:thanos /attribute:primaryGroupID /value:512</strong> (This will will use the Security Identifier (512) of the DA Group to inject thanos into the DA Group.</p></li>
<li><p><strong>lsadump::dcshadow /push</strong> (Pushes the changes we made with the rogue Domain Controller (us) to the actual Domain Controller).</p></li>
</ul>
<p>Thanos has now been injected into the DA Group, this can be verified by:</p>
<ul>
<li><p><strong>domain group “domain admins” /domain</strong></p></li>
</ul>

![thanos](/images/DCSync-vs-DCShadow/thanos-gif.gif)

Onto the Hunt (the best part):
---
<p><strong><i>DCSync:</i></strong></p>
<p>In order to hunt, creating a hypothesis is key. This will help prevent analysis paralysis - Over analyzing an abundance of logs.</p>
<p>During this hunt, I will show how the indicator of compromises (IOC’s) for DCSync, will help us discover and hunt the defense evasion of DCShadow. Keep in mind the IOC’s for DCShadow will stay the same regardless if the adversary did attempt a DCSync beforehand or if they used a different type of technique to collect a DA’s credentials.</p>
<p>To put this into perspective, say we are a Detection/Hunt Team and we got this alert from Microsoft ATA:</p>

![DCSync-DCShadow](/images/DCSync-vs-DCShadow/ATA.png)

<p>Then upon further analysis we come across this:</p>

![DCSync-DCShadow](/images/DCSync-vs-DCShadow/ATA2.png)

<p>Here we see a Directory Service Replication attempted and succeeded with a user in our environment (Tony Stark: ironman@windomain.local). Let’s discuss WHY this is a red flag. Replication should only be done between registered Domain Controllers through the KCC. After understanding this there shouldn’t be a reason an account (with Administrator privileges or any account) should be doing a replication on directory services from win10.windomain.local, which is not a registered Domain Controller in our forest. Keep in mind this is before we implemented win10 as a rogue Domain Controller. This screams “Credentials Stolen” to me</p>
<p>After seeing that the replication happened to dc.windomain.local, we go to investigate logs. What are we looking for? There are going to be a lot of logs that will be propagated so we want a hunt hypothesis or a game plan before investigating.</p>
<p>After googling the alert from Microsoft ATA, we get this – “In this detection, an alert is triggered when a replication request is initiated from a computer that is not a Domain Controller.” We know that the replication happened from the win10.windomain.local (not a Domain Controller) and with user account ironman@windomain.local (Administrator privileges).</p>
<p>Inside of dc@windomain.local we see this log:</p>

<p><strong>Event 4662:</strong> An object was performed as an object.</p>

![DCSync-DCShadow](/images/DCSync-vs-DCShadow/Event4662.png)

<p>This looks suspicious, after further investigation we see these properties mean:</p>
<p><strong>Properties:</strong> Control Access:</p>
<p>{19195a5b-6da0-11d0-afd3-00c04fd930c9} <strong>- Domain-DNS Class</strong></p>
<p>{1131f6ad-9c07-11d1-f79f-00c04fc2dcd2}- <strong>DS-Replication-Get-Changes-All</strong></p>
<p>We see the replication happening here, the DS-Replication-Get-Changes-All is giving rights for replication of secret domain data. What is ironman replicating or what is he trying to accomplish. Let’s check the network logs.</p>

<p><strong><i>Wireshark:</i></strong></p>
<p>One thing to note, we are getting different directory service requests from an IP address that is NOT a Domain Controller in our environment. This shows an adversary is replicating data from their IP from our Domain Controller’s IP address.</p>

![DCSync-DCShadow](/images/DCSync-vs-DCShadow/dsuapi.png)

<p>DCSync is easier to detect once we look at the network. You see these different directory service operations: DsGetDomainControllerInfo, DsCrackNames, DsGetNChanges. As Sean Metcalf explains in his <a href="https://adsecurity.org/?p=1729">post</a>, a way to detect bad activity is to configure the IDS to trigger when you DsGetNChanges request originates from a non-Domain Controllers IP address.</p>
<p>Here is another an example of what dce_rpc.log in BRO will look like as well:</p>

![DCSync-DCShadow](/images/DCSync-vs-DCShadow/bro-dcsync.png)

<p>You can see the directory service operations being done over the network here as well.</p>

<p><strong><i>DCShadow:</i></strong></p>

<p>So far, we have successfully confirmed that there was a DCSync technique handled above. What did the adversary do with the information it requested from the Domain Controller? How can we successfully hunt the adversary now? We follow the same process. So far we have found tracks from the adversary, not the actual adversary yet.</p>
<p>For DCShadow we are going to work backwards, move from the network logs- Wireshark/Bro, then move into the Window Event ID’s.</p>
<p>Imagine you are still in the network and you see this from Bro:</p>

![DCSync-DCShadow](/images/DCSync-vs-DCShadow/bro-dcshadow.png)

<p>Why are we seeing DRSUpdateRefs, DRSReplicaAdd? This isn’t involved with the DCSync. This can be proven this by looking inside of Wireshark:</p>

![DCSync-DCShadow](/images/DCSync-vs-DCShadow/dsuapi-shadow.png)

<p>What is different then the DCSync network log? DRS_REPLICA_ADD ,DRS_REPLICA_DEL, and DsReplicaUpdateRefs. Take a closer look as to what is going on in this capture:</p>

![DCSync-DCShadow](/images/DCSync-vs-DCShadow/replica-shadow.png)

<p>This is modifying the nTDSDA object. You can correlate these logs with a Windows Event 4662 that populates inside of the legitimate Domain Controller:</p>

![DCSync-DCShadow](/images/DCSync-vs-DCShadow/nTDSDSA.png)

<p>This is activating the replication process:</p>

![DCSync-DCShadow](/images/DCSync-vs-DCShadow/replication.png)

<p>We see this replication is happening over the network. How can we tell that this is a DCShadow attack? This can be seen by filtering out LDAP in the packet capture:</p>

![DCSync-DCShadow](/images/DCSync-vs-DCShadow/ldap.png)

<p>Inside of the AddRequest you see that Win10 is added into the Servers CN.</p>

![DCSync-DCShadow](/images/DCSync-vs-DCShadow/addrequest.png)

<p>Inside of the ModifyRequest we see that Win10 is modified to being a Domain Controller</p>

![DCSync-DCShadow](/images/DCSync-vs-DCShadow/modifyrequest.png)

<p>Notice the GC part- this proves win10 was registered as a DC. GC stands for global catalog and is a role given to one or more Domain Controllers in the environment. DC's store data about objects in its own domain where GC stores data about every object in in the forest.</p>
<p>Inside of the Windows Event Security logs you will find another Event ID 4662, showing this change. You can tell by the Object Type, along with the Operation Type/Accesses:</p>

![DCSync-DCShadow](/images/DCSync-vs-DCShadow/modify4662.png)

<p>You will also see a Event ID of 4742, which populates when you make a change to a computers attribute.</p>

![DCSync-DCShadow](/images/DCSync-vs-DCShadow/4742.png)

Notice while looking into the details of the 4742 log - underneath ‘Service Principle Names’, anything sticking out? This shows the change done on the win10.windomain.local was changed to classify as a GC.

![DCSync-DCShadow](/images/DCSync-vs-DCShadow/SPN.png)

<p>Back to the Packet Capture, you will see the request to remove Win10 as a Domain Controller:</p>

![DCSync-DCShadow](/images/DCSync-vs-DCShadow/removing.png)

<p>The delReqeust is requesting to remove Win10 from Servers CN:</p>

![DCSync-DCShadow](/images/DCSync-vs-DCShadow/delRequest.png)

<p>Modify Request is removing WIN10 From GC. Officially removing Win10 as Domain Controller:</p>

![DCSync-DCShadow](/images/DCSync-vs-DCShadow/RemoveModify.png)

<p>In between the Addition and removal of win10.windomain.local into the Servers CN, you see that there is a user being searched and modified.</p>

![DCSync-DCShadow](/images/DCSync-vs-DCShadow/thanos.png)

<p>When we look into the packet we see this:</p>

![DCSync-DCShadow](/images/DCSync-vs-DCShadow/ThanosWireshark.png)

<p>Focus on the attribute “Thanos” with the user account control 512? Does 512 sound familiar? That is the Group ID of Domain Admins Group. We set 512 as the primary group ID of Thanos, injecting him into the Domain Admins Group.</p>
<p>Now that there was a user injected in the Domain Admins group and a computer win10.windomain.local was registered as a Domain Controller, we are missing one piece. Who injected this user? Why is this important? We want to know where the adversary has been, who they have compromised, and where they are. We know where they have been, where they are (or could be), but we are missing who all they have compromised. We know they have compromised: ironman@windomain.local and thanos@windomain.local, but who was compromised between those two? How do we know there was a user used between the two? Remember the privileges it takes to implement these two attacks. For DCSync: Administrators, Domain Admins, Enterprise Admins, or a Domain Controller computer accounts. DCShadow: Domain Admin. ironman@windomain.local is in the Administrator Group and thanos@windomain.local was injected into the Domain Administrator Group, but WHO did the injection?</p>
<p>This is actually pretty simple to find, especially over the network. Remember when we had to give system level privileges to Mimikatz in order to run the DCShadow attack? Then we used another Mimikatz window to make the push?</p>

![DCSync-DCShadow](/images/DCSync-vs-DCShadow/TGS.png)

<p>You notice this before any of the DRS or LDAP logs inside of the packet capture.</p>

Final Thoughts:
---
<p>DCSync and DCShadow at first to me sounded very similar. I wanted to give others the clear difference in the two attacks. DCSync is used to capture credentials, where DCShadow is used for Defense Evasion by having the ability to inject objects into the Active Directory Infrastructure. I hope you found this informational and also enjoyed the Marvel references. ☺</p>

Marvel Explanation:
---
If you follow Marvel, you know in the movie "Avengers: Age of Ultron", Tony Stark had a AI (which he created) named "JARVIS". JARVIS later turned into Vision, who had the Mind Stone attached to his head. In "Avengers: Endgame", Thanos removed the stone from Vison (killing him), giving Thanos the last infinity stone. Which made him all powerful, which led to him wiping out half of the world. 

Resources:
---
<p>If any of the following read this blog, I would like to say thank your work, along with your write ups. They are a huge help.</p>
<p><strong>DCSync:</strong></p>
<ul>
<li><p><a href="https://adsecurity.org/?p=1729"><strong>Mimikatz DCSync Usage, Exploitation, and Detection</strong></a> by Sean Metcalf</p></li>
<li><p><a href="http://www.harmj0y.net/blog/redteaming/mimikatz-and-dcsync-and-extrasids-oh-my/"><strong>Mimikatz and DCSync and ExtraSids, Oh My</strong></a> by Will Schroeder</p></li>
<li><p><a href="https://github.com/Cyb3rWard0g/mordor"><strong>Mordor Gates</strong></a> by Roberto Rodriguez</p></li>
</ul>
<p><strong>DCShadow:</strong></p>
<ul>
<li><p><a href="https://blog.alsid.eu/dcshadow-explained-4510f52fc19d"><strong>DCShadow explained</strong></a> by Luc Delsalle</p></li>
<li><p><a href="https://www.dcshadow.com/"><strong>DCShadow</strong></a> by Benjamin Delpy and Vincent Letoux</p></li>
<li><p><a href="https://pentestlab.blog/2018/04/16/dcshadow/"><strong>DCShadow</strong></a> by netboisx</p></li>
</ul>
<p><strong>Other:</strong></p>
<ul>
<li><p><a href="https://github.com/clong/DetectionLab"><strong>Detection Lab</strong></a> by Chris Long</p></li>
<li><p><a href="https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/replication/active-directory-replication-concepts"><strong>KCC</strong></a> by Microsoft</p></li>
</ul>
