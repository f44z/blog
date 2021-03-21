---
title: Web application white box pentesting methodology
date: "2021-03-21T08:40:32.169Z"
template: "post"
draft: false
slug: "white-box-methodology"
category: "Pentesting"
tags:
  - "Pentesting"
description: "Recently, I finished the Offensive Security WEB-300 course and successfully passed OSWE certification. The goal of this article is to share the methodology I'm using in white box engagements and which helped me in 48h long OSWE exam."
socialImage: "/media/oswe-badge.png"
---
Recently, I finished the Offensive Security WEB-300 course and successfully passed OSWE certification. The goal of this article is to share the methodology I'm using in white box engagements and which helped me in 48h long OSWE exam.

Before diving into the methodology, it's worth spearing some words about the course and exam itself. The Offensive Security WEB-300 course is all about white box web testing, so application source code is available. It means exposure to a lot of code in five different languages: Java, JavaScript, C#, Python, PHP. The full course syllabus can be found [here](https://www.offensive-security.com/documentation/awae-syllabus.pdf).

In my opinion, you have to obtain four types of skills to become OSWE:
1. Understanding the syntax of many languages and their web frameworks to efficiently analyze big codebases.
2. Knowledge of web vulnerabilities - how to discover and exploit them.
3. Scripting skills - know how to pack a kill chain of vulnerabilities in one script in the language of your choice (WEB-300 course uses python).
4. Having a solid white box testing methodology.

Resources that I used for learning about 1 and 2 are linked at the [end of this article](#resources). 

## White box methodology

- [Reconnasaince](#reconnasaince)
- [Threat modeling](#threat-modeling)
- [Tests preperation](#tests-preperation)
- [Understand application](#understand-application)
- [Automatic scans](#automatic-scans)
- [Manual code review](#manual-code-review)

## Reconnaissance
This step is all about a clicking-through application from a user perspective with Burp proxy (or similar tool) turned on. Note all of the application functionalities, technology, how it handles authentication, authorization, is there anything odd that triggers your pentesting instinct? Note the structure of requests, cookies, local storage, etc. Write your observation down and save the Burp session. 

## Threat modeling
It's time to make use of your notes from the reconnaissance phase. List all of the application functionalities and map this to known vulnerabilities. What can go wrong if the attacker misuses the presented functionality? There are threat modeling lessons that I find very useful, it's from Jakub Kaluzny from SecuRing - you can watch it [here](https://www.youtube.com/watch?v=e_haUCYppYs&list=PL-lO2xrptAtav4SZgCdDkVxChWhVU3kmP).

## Tests preparation
In this step, you should prepare a code base for tests. This depends on the type of application that you are auditing, but in general:
1. Turn on the application, database, and system logging policy to the most detailed (like debug) and find a place where logs are stored.
2. Set up the environment for application debugging. It's very handy to work with a debugger instead of a console.logging/printing messages.
3. Sometimes it's also useful to observe interactions between application and system in. ex. [windows sysinternals](https://docs.microsoft.com/en-us/sysinternals/) Procmon and ProcessHacker can become handy.

## Understand the application
Before starting to look for vulnerabilities, make sure you know how the application is structured and get yourself familiar with coding patterns. It can become easier when the application is based on some framework or uses well-known architecture like MVP. However, make sure to note all exceptions from standards, for example, look for places where database query is done without ORM or parts of the application that use custom code instead of framework capabilities. A debugger can be helpful in this step.

In this phase I'm trying to answer the following questions:
- What framework application uses?
- How routing works?
- How requests with user input are handled?
- Where is the application configuration located?
- How session is handled?
- How it interacts with a database?
- How it handles authorization?
- How it handles authentication?

## Automatic scans
**Warning. You can't use automatic scanners in OSWE exam** but it becomes really handy in real-life engagements, to localize low-hanging fruits. I have a local lab with three tools:
- [SonarQube Community](https://www.sonarqube.org/downloads/) for static code analysis
- [OWASP Dependency check](https://owasp.org/www-project-dependency-check/) for finding vulnerabilities in application dependencies
- [Burp Professional](https://portswigger.net/burp/pro) for application dynamic analysis utilizing scanner module

## Manual code review
Here real fun starts. Now you have:
basic knowledge of the application and its functionalities, 
threat model, 
insight into how the application is structured and how it works,
low-hanging fruits from the automatic analysis. 
Of course, the OffSec exam and CTF's are different from the real-life scenario. Usually, in real engagement, you should stick to penetration testing methodologies like [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/assets/archive/OWASP_Testing_Guide_v4.pdf) to make sure your tests for all well-known vulnerabilities and deliver a comprehensive report.

In CTF the goal is usually to get full control over the application and underlying system. Firstly, look for some kind of authentication bypass as in general it's easier to find more serious vulnerabilities like RCE as admin user than as non-logged in. When looking for authentication bypass, focus on all application functionalities related to login and registering user, such as:
- simple logic flaws related to login in. ex. possibility to register as a user with the highest privileges
- password reset
- try bruteforcing login panel - pay attention to account blocking mechanisms 
- weak crypto related to session handling
- SQL injection vulnerabilities, to extract password hashes or plaintext passwords from DB
- how an application is protected against XSS, maybe it's possible to hijack admin sessions using XSS-based vulnerability?

Having access to the application as a privileged user, you can utilize all of its functionalities to get RCE or find flags. Things like insecure deserialization flaws, unrestricted file upload vulnerabilities, or SQL injections could be an important ingredient in your RCE kill chain recipe.

## Resources
- web application vulnerability refresher - [PortSwigger Academy](https://portswigger.net/web-security)
- javascript for pentesters in [Pentester Academy](https://www.pentesteracademy.com/course?id=11)
- burp refresher from [Bugcrowd University](https://www.youtube.com/watch?v=h2duGBZLEek)
- resource mentioned in [wetw0rk github repo](https://github.com/wetw0rk/AWAE-PREP)
- security code review series from [Paul Ionescu](https://medium.com/@paul_io/security-code-review-101-a3c593dc6854)
- [Nathan Rague OSWE Review and Exam Preperation Guide](https://hub.schellman.com/blog/oswe-review-and-exam-preparation-guide)






