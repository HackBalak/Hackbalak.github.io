---
title: Setting up an Nginx Redirector for covenant C2
author: HackBalak
date: 2021-09-08 14:10:00 +0100
categories: [Tutorial, Red Teaming]
tags: [red teaming, reverse proxy, covenant C2]
toc: false
image:
  src: ./Aseets/set-up-nginx-c2-redirector/C2-redirector_principe.png
  width: 500   # in pixels
  height: 400   # in pixels
  alt: C2 redirector overview

---

## What is a C2 redirector ?

Redirectors are an essential component for advanced red teaming. Redirectors allow malicious traffic to come and go as it pleases, but remain hidden from detection.

The objective of a redirector is to mask the core C2 infrastructure from prying blue team eyes, and allow the red team operator hidden communication with a compromised machine.

Redirectors seek to mask and protect their backend server, the main orchestration server for all C2.

The picture below explains the principe idea of C2 redirector :

<img src="https://github.com/HackBalak/Hackbalak.github.io/blob/main/_posts/Aseets/set-up-nginx-c2-redirector/C2-redirector.png?raw=true" width="600" height="400">

Simplified, but you get the idea...

Redirectors offer many advantages around obfuscation but they also offer a resilience and persistence advantage. If the blue team are able to successfully identify and block an IP address associated with the C2 infrastructure, the red team operator can quickly spin up a redirector and continue to keep the core backend server IP address hidden. 
There is many solutions to do so, but we choose to use an Nginx web server and make it behave like a **reverse proxy** to redirect the traffic communication between the covenant c2 server and the victim machine.

Now and after getting an Idea about what is a C2 redirector, let's move to the Implementation :) .
## Implementation


