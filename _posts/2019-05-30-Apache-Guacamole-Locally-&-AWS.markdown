---
title:  "Apache Guacamole Local and/or AWS Install"
date:   2019-04-14 
categories: Detection
---

Introduction:
---
Apache Guacamole was built on top of Chris Long's <a href="https://github.com/clong/DetectionLab/tree/master/Terraform"><strong>Detection Lab</strong></a>. This guide will work locally and on AWS as well if you have your own lab, depending on how you have it set up. I will discuss the direct differences between the Detection Lab, home lab, and AWS when it comes to the configurations. However this was built and centered around Detection Lab and AWS. 

Being new to AWS, I wanted to really customize my experience, as well as make my experience as pain free as possible. . To do so, I had to answer a couple of questions:

1. How can I get access to my lab, without having to look up my public ip and edit my security groups everytime moved between devices? This was done by setting up a VPN, Algo/Wireguard specficially. There are some really great guides on this, but Chris Long has one here - https://clo.ng/blog/algo_vpn/

2. Inside of AWS, everytime you Stop then Start an instance the Public IP will change. Yes you can fix this by setting an Elastic IP, but I didn't want to spend the money. A free DNS would work amazing for this, which a friend of mine Ben (twitter: @UsernameIsBen) brought to my attention that DuckDNS is perfect for this. This allowed me to not have to look at the Public IP everytime I wanted to get to Guacamole, Splunk, etc. Just a nice easy use the duckdns you create, ex: logger-duckdns.org. 

 3. Lastly, I wanted to be able to acces my lab enviroment without 1- Transferring my logger's private key, or creating a new one for every device. 2. Downloading Remote Desktop or SSH clients on each device I want to access my lab from. This is where Apache Guacamole came in perfectly. 

<strong>Note</strong> Detection Lab AWS does require that you login with SSH key's vs. username/password. I will show the difference in the configuration in Guacamole below if you have your lab set up which ever way. 

Guide:
---
<strong>Note</strong> This guide is for Ubuntu 16.04, if you have 18.04 I have a small write-up for that as well, just contact me. There are also other guides out there was well. 

 First thing you will want to do is add yor Public IP to the logger's security group for port 8080 (if you want HTTP), port 8443 (if you want HTTPS), port 8080 & 8443 (HTTP & HTTPS). At the bottom of this guide in the "Securing" Section I do forward 8080 to 8443, so I suggest either just putting 8443, or 8080 & 8443 if you want to see the redirect process. 

![security-groups](/images/AWS/securitygroups.PNG)

  ```sudo apt-get install libcairo2-dev libjpeg62-dev libpng12-dev libossp-uuid-dev libfreerdp-dev libpango1.0-dev libssh2-1-dev libssh-dev tomcat7 tomcat7-admin tomcat7-user```


  ```wget http://sourceforge.net/projects/guacamole/files/current/source/guacamole-server-0.9.9.tar.gz``` 

```tar zxf guacamole-server-0.9.9.tar.gz```

  ```cd guacamole-server-0.9.9``` 

  ```sudo ./configure```

  ```sudo make```

  ```sudo make install```

  ```sudo ldconfig```

  ```cd /var/lib/tomcat7```

  ```wget http://sourceforge.net/projects/guacamole/files/current/binary/guacamole-0.9.9.war```

  ```mv guacamole-0.9.9.war /var/lib/tomcat7/webapps/guacamole.war```

  ```mkdir /etc/guacamole```

  ```mkdir /usr/share/tomcat7/.guacamole```

 Create guacamole.properties in /etc/guacamole

```nano /etc/guacamole/guacamole.properties``` 



        guacd-hostname: localhost
        
        gucad-port: 4822
        
        user-mapping: /etc/guacamole/user-mapping.xml
        
        auth-provider: net.sourceforge.guacamole.net.basic.BasicFileAuthentiionProvider
        
        basic-user-mapping: /etc/guacamole/user-mapping.xml



  ```ln -s /etc/guacamole/guacamole.properties /usr/share/tomcat7/.guacamole/```

 Create user-mapping.xml in /etc/guacamole

```nano /etc/guacamole/user-mapping.xml```


		<user-mapping>


			<authorize
				username="guacadmin" <!--Login Username-->
				password="5cbd438413e8e3ca0e14e200fde621a9" <!--Login password, if you want to change this, command is below-->
				encoding="md5">


				<connection name="logger">
					<protocol>ssh</protocol>
					<param name="hostname">192.168.38.105</param>
					<param name="port">22</param>
					<param name="username">vagrant</param>
							  <param name="private-key">privatekey</param>
				</connection>

				<connection name="wef">
					<protocol>rdp</protocol>
					<param name="hostname">192.168.38.103</param>
					<param name="port">3389</param>
					<param name="username">vagrant</param>
					<param name="password">vagrant</param>
					<param name="security">nla</param>
					<param name="ignore-cert">true</param>
				</connection>


				 <connection name="win10">
					<protocol>rdp</protocol>
					<param name="hostname">192.168.38.104</param>
					<param name="port">3389</param>
					<param name="username">vagrant</param>
					<param name="password">vagrant</param>
					<param name="security">nla</param>
					<param name="ignore-cert">true</param>
				</connection>



				<connection name="dc">
					<protocol>rdp</protocol>
					<param name="hostname">192.168.38.102</param>
					<param name="port">3389</param>
					<param name="username">vagrant</param>
					<param name="password">vagrant</param>
					<param name="security">nla</param>
					<param name="ignore-cert">true</param>
				</connection>

			</authorize>

		</user-mapping>




To change the password and print it in md5, command is: ``` printf '%s' "password" | md5sum```

<strong>Note:</strong> If your lab doesn't require ssh keys to log in, the logger's connection can look like below:



                <connection name="logger">
                        <protocol>ssh</protocol>
                        <param name="hostname">192.168.38.105</param>
                        <param name="port">22</param>
                        <param name="username">vagrant</param>
                </connection>




  ```service tomcat7 start```

  ```/usr/local/sbin/guacd &```

This starts the guacamole process, if you want this to start on boot (suggested so  you don't have to manually start everytime machine boots) do the following:

  ```sudo mkdir /etc/guacamole/guacd/```
  
 
  ```sudo nano /etc/guacamole/guacd/guac-start.sh```
  
  ```crontab -e ```

Add this to crontab -e 

   ```@reboot /usr/local/sbin/guacd &```

Test by going to: http://ip-address:8080/guacamole

![security-groups](/images/AWS/guac.PNG)


<strong>Login guacadmin:guacadmin</strong> 



Securing:
---
These are some basic things you can do to lock down the Tomcat7 server. There are ALOT more things to do, these are some basic things I wanted to implement. 

 ```cd /var/lib/tomcat7/webapps```

  ```rm -r Root/```

  ```cd /etc/tomcat7```

Create keygen cert: 
 ```sudo keytool -genkey -alias tomcat -keyalg RSA -keystore /etc/tomcat7/.keystore```

- Put guacadmin for all passwords

  ```nano server.xml```

 Change 'Connector port=8443' to:
	

		  <Connector SSLEnabled="true" acceptCount="100" clientAuth="false"
		  disableUploadTimeout="true" enableLookups="false" maxThreads="25"
		  port="8443" keystoreFile="/etc/tomcat7/.keystore" keystorePass="guacadmin"
		  protocol="org.apache.coyote.http11.Http11NioProtocol" scheme="https"
		  secure="true" sslProtocol="TLS" />

	

 Change 'Server port' to:

	

		<Server port="8789" shutdown="THISPASSWORDISTOOLONGFORYOUTOTRYTOGUESS1234567890">

	
Save and exit

  ```nano web.xml```

Add following between 'web-app' & '/web-app' tags:

	

		<security-constraint>
		<web-resource-collection>
		<web-resource-name>Protected Context</web-resource-name>
		<url-pattern>/*</url-pattern>
		</web-resource-collection>
		<user-data-constraint>
		<transport-guarantee>CONFIDENTIAL</transport-guarantee>
		</user-data-constraint>
		</security-constraint>

	

  Between <strong>session-config</strong> change to look like this:
	

		   <session-timeout>30</session-timeout>
			  <cookie-config>
			  <http-only>true</http-only>
			  <secure>true</secure>
			  </cookie-config>
		      </session-config>


 Save and Close

  ```chmod 600 /etc/guacamole/user-mapping.xml```

Conclusion:
---
I wanted to give the community a simple guide to follow when it comes to Apache Guacamole, especially when it comes to AWS. I hope you enjoy! If you have any questions or corrections never hesitate to contact me! :) 

References:
---

https://www.tecmint.com/guacamole-access-remote-linux-windows-machines-via-web-browser/

https://guacamole.apache.org/

https://github.com/clong/DetectionLab/tree/master/Terraform

https://www.duckdns.org/

https://clo.ng/blog/algo_vpn/




