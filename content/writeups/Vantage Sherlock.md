---
title: Vantage Sherlock
date: 2026-04-20
tags:
  - ctf
  - dfir
description: HTB's sherlock Vantage
showToc: true
---
## Overview

#### Sherlock Scenario
A small company moved some of its resources to a private cloud installation. The developers left the redirect to the dashboard on their web server. The security team got an email from the alleged attacker stating that the user data was leaked. It is up to you to investigate the situation.

```Files
controller.2025-07-01.pcap  
web-server.2025-07-01.pcap
```


***
## Solution
#### What tool did the attacker use to fuzz the web server ? (Format- include version e.g, [nmap@7.80](mailto:nmap@7.80))

Query that i used
```Queries
http.request.method== "GET"
```

and user-agent of brute-force packets
```user-agent
User-Agent: Fuzz Faster U Fool v2.1.0-dev\r\n
```

GOOGLE SAYS: [ffuf](https://www.google.com/search?client=firefox-b-e&channel=entpr&q=ffuf&mstk=AUtExfBIiC0HNxDu-NEq8EKQU6efUy35xFOKCnS3zLCOAY6VbqCLmMkCLCXR1btrz96dl6wbn265D41L55ZGyQ-Pa5uBIgy_k9ls9xAEA-zn-y-o2hd3abIsIVb9e-hx4ykLfZwa73wXo_z-p614iaiP16EEtetvUHkQqhv4W6LWp8PS-Gk&csui=3&ved=2ahUKEwjwsoDnjsOTAxULgP0HHeJKLDkQgK4QegQIARAD) (Fuzz Faster U Fool) is an open-source, high-performance web fuzzer written in Go, designed for rapid web application security testing and enumeration.

```Answer
ffuf@2.1.0
```

#### Which subdomain did the attacker discover?

Queries 
```Queries
(http.request.method== "GET") && (http.user_agent == "Fuzz Faster U Fool v2.1.0-dev")

(frame.time >= "2025-07-01T14:38:58.121751000+0500") &&(http.request.method=="GET")
```

```Notes
Arrival Time: Jul  1, 2025 14:38:58.121751000 +05
UTC Arrival Time: Jul  1, 2025 09:38:58.121751000 UTC
Epoch Arrival Time: 1751362738.121751000

```

![](/blog/images/Pasted-image-20260420204700.png)

Answer
```Answer
cloud
```


#### How many login attempts did the attacker make before successfully logging in to the dashboard?

Queries 
```queries
(((frame contains "login") && (ip.src == 117.200.21.26) ) ) && (http.request.method == "POST")
```

![](/blog/images/Pasted-image-20260420204907.png)

```Answer
3
```

#### When did the attacker download the OpenStack API remote access config file? (UTC)

![](/blog/images/Pasted-image-20260420204930.png)

![](/blog/images/Pasted-image-20260420204938.png)

```Answer
2025-07-01 09:40:29
```

#### When did the attacker first interact with the API on controller node? (UTC)
We are continuing the investigation on second pcap file. Beause it's controller traffic and we need one of it's nodes.

Before that let's look at the API config file that attacker downloaded.  We'll see the controlled node's IP address. 

![](/blog/images/Pasted-image-20260420205014.png)

```Queries
ip.src==117.200.21.26 && ip.dst==134.209.71.220
```

![](/blog/images/Pasted-image-20260420205027.png)

```Answer
2025-07-01 09:41:44
```

#### What is the project id of the default project accessed by the attacker?
```Queries
ip.src==117.200.21.26 && ip.dst==134.209.71.220
```

![](/blog/images/Pasted-image-20260420205039.png)

```Answer
9fb84977ff7c4a0baf0d5dbb57e235c7
```

#### Which OpenStack service provides authentication and authorization for the OpenStack API?

![](/blog/images/Pasted-image-20260420205048.png)

```Answer
keystone
```
#### What is the endpoint URL of the swift service?
* **Swift is basically OpenStack's version of Google Drive** — it stores files/objects in the cloud.

```
(frame.time_utc >= "2025-07-01T09:43:27.279520000Z") && (ip.src == 117.200.21.26)
```

![](/blog/images/Pasted-image-20260420205059.png)

```Answer
http://134.209.71.220:8080/v1/AUTH_9fb84977ff7c4a0baf0d5dbb57e235c7
```

#### How many containers were discovered by the attacker?
Containers in Swift = Folders
"After the attacker logged into Swift, how many **folders** did they find/list?"

![](/blog/images/Pasted-image-20260420205114.png)

```Answer
3
```

#### When did the attacker download the sensitive user data file? (UTC)

```Queries
 (frame.time_utc >= "2025-07-01T09:43:27.279520000Z") && (ip.src == 117.200.21.26)
```

![](/blog/images/Pasted-image-20260420205131.png)

![](/blog/images/Pasted-image-20260420205142.png)


```Answer 
2025-07-01 09:45:23
```

#### How many user records are in the sensitive user data file?

![](/blog/images/Pasted-image-20260420205152.png)

```Answer
28
```

#### For persistence, the attacker created a new user with admin privileges. What is the username of the new user?

```Queries
 (frame.time_utc >= "2025-07-01T09:43:27.279520000Z") && (ip.src == 117.200.21.26)
```

We just need to scroll.

![](/blog/images/Pasted-image-20260420205204.png)

```Answer
jellibean
```

#### What is the password of the new user?

```Queries
((frame.time_utc >= "2025-07-01T09:43:27.279520000Z") && (ip.addr == 117.200.21.26)) && (frame contains "jellibean")
```

![](/blog/images/Pasted-image-20260420205214.png)

```Answer
P@$$word
```

#### What is MITRE tactic id of the technique in task 12?
just googled

```Answer
T1136.003
```

We nailed it, man
***
#### IOC
Attacker IP: 117.200.21.26
Web Server's IP: 157.230.81.229
API Controlled node's IP: 134.209.71.220
***
### Learned Lessons
* **OpenRC** is a dependency-based [init system](https://en.wikipedia.org/wiki/Init "wikipedia:Init") for Unix-like systems that maintains compatibility with the system-provided [init system](https://wiki.gentoo.org/wiki/Init_system "Init system"), normally located in /sbin/init.
* OpenRC will start necessary system services in the correct order at boot, manage them while the system is in use, and stop them at shutdown