# Security Incident Report

## Section 1. Identify the network protocol involved in the incident  

The incident affected the Hypertext Transfer Protocol (HTTP). To detect the issue, we ran tcpdump and accessed the yummyrecipesforme.com website. This helped us capture protocol and traffic data in a DNS & HTTP traffic log file. Based on this evidence, we concluded that the malicious file was delivered to users' computers through the HTTP protocol at the application layer.

## Section 2. Document the incident  

The security analyst team was notified by our manager through the website hosting provider for [yummyrecipesforme.com](https://yummyrecipesforme.com) that the customers who are trying to access the website are prompted to download a file to update their browsers.
After doing so, the customers claimed that the address for the website changed and their PCs began running more slowely. In response to this, the owner tries to log in to the admin panel but is unable to, so they reach out to the website hosting provider.

To address the incident, we create a sandbox enviromnent to recreate this suspicious website behavior. we run the network protocol analyzer tcpdump, then type in the URL for the website.
As soon as the website loads, we are prompted to download an executable file to update my browser. we accept the download and allow the file to run. Immediately we noticed that the browser redirects us to a different URL, [greatrecipesforme.com](https://greatrecipesforme.com), which is desigend to look like the original site.

In the DNS & HTTPS traffic log, a few lines is worth addressing:  

```
14:18:36.786589 IP your.machine.36086 > yummyrecipesforme.com.http:
Flags [P.], seq 1:74, ack 1, win 512, options [nop,nop,TS val
3302576859 ecr 3302576859], length 73: HTTP: GET / HTTP/1.1
```
> 1. The log entry with the code HTTP:GET / HTTP/1.1 shows the browser is requesting data from [yummyrecipesforme.com](https://yummyrecipesforme.com) with the HTTPS:GET method using HTTP protocol version 1.1.
> This could be the download request for the malicious file.

```
14:20:32.192571 IP your.machine.52444 > dns.google.domain: 21899+ A?
greatrecipesforme.com. (24)
14:20:32.204388 IP dns.google.domain > your.machine.52444: 21899
1/0/0 A 192.0.2.172 (40)
14:25:29.576493 IP your.machine.56378 > greatrecipesforme.com.http:
Flags [S], seq 1020702883, win 65495, options [mss 65495,sackOK,TS
val 3302989649 ecr 0,nop,wscale 7], length 0
14:25:29.576510 IP greatrecipesforme.com.http > your.machine.56378:
Flags [S.], seq 1993648018, ack 1020702884, win 65483, options [mss
65495,sackOK,TS val 3302989649 ecr 3302989649,nop,wscale 7], length
0
```

> 2. It appears that there's a sudden change in the logs. The traffic is routed from the source computer to the DNS server again using port **.52444**.
> This time, the DNS server routes the traffic to a new IP address **(192.0.2.172)** and its associated URL [greatrecipesforme.com](https://greatrecipesforme.com).
> This indicates that the traffic changes to a route between the source computer and the spoofed website. 
>  
>    Outgoing traffic: **IP your.machine.56378 > greatrecipesforme.com.http**
> 
>    Incomiing traffic: **greatrecipesforme.com.http > IP your.machine.56378**


## Section 3. Recommend one remediation for brute force attacks  

Since we noticed that the website owner for [yummyrecipesforme.com](https://yummyrecipesforme.com) used a default password for their admin account, we strongly recommend that they change it to a more complex passwords.
Perhaps something with 1 uppercase letter, 1 symbol, at least 1 number and more than 8 letters long.  

Next, we recommend the website to enforce a MFA/2FA, install a software to monitor login attempts and limiting the number of login attempts to prevent malicious threat actors to have the ability to try out all combinations of passwords.

