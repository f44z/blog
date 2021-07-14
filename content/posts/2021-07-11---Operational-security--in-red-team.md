---
title: Operational Security in Red Team
date: "2021-07-11T08:40:32.169Z"
template: "post"
draft: false
slug: "opsec-red-team"
category: "RedTeam"
tags:
  - "RedTeam"
description: "@TODO."
socialImage: "/media/oswe-badge.png"
---
(some words about what is operational securty and why it's important) - and that I will focus on AWS

## EC2 and Security Groups
keep with least privilage - only open necessary ports
tell about IaC 
show diagram how infrastructure should look inside AWS

## Redirectors 
HTTPS and DNS redirectors - why they are important
tell about automating this stuff - using C2Concealer and mod2rewrite automation - always change default ones be creative
in next article couple of words about lambda redirector in AWS

good idea to separate redirectors and team server 

## Infrastructure monitoring - Red ELK or custom solution
maybe incorporate AWS for logging 
implementing red elk not only for security but also notifications - in.ex. send e-mail when you get new beacon (will do article how to do this in AWS without ELK)
- monitoring using cloud watch for apache2 - https://aws.amazon.com/blogs/mt/simplifying-apache-server-logs-with-amazon-cloudwatch-logs-insights/
- maybe it's possible to monitor CobaltStrike using cloud watch?

## Other stuff
hidden WHOIS in domain 

interesting presentation about OPSEC

redirecting DNS:
https://medium.com/rvrsh3ll/redirecting-cobalt-strike-dns-beacons-e3dcdb5a8b9b
opsec:
https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Cobalt%20Strike%20-%20Cheatsheet.md




