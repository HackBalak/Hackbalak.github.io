---
title: Setting up an Nginx Redirector for covenant C2
author: HackBalak
date: 2021-09-08 14:10:00 +0100
categories: [Tutorial, Red Teaming]
tags: [red teaming, reverse proxy, covenant C2]
image:
  src: https://github.com/HackBalak/Hackbalak.github.io/blob/main/_posts/Aseets/set-up-nginx-c2-redirector/C2-redirector_principe.png?raw=true
  width: 600   # in pixels
  height: 400   # in pixels  
---

## What is covenanat C2 ?

<img src="https://raw.githubusercontent.com/wiki/cobbr/Covenant/covenant.png?raw=true" width="1200" height="300">

[Covenant](https://github.com/cobbr/Covenant) is a .NET command and control framework and web application that aims to highlight the attack surface of .NET, make the use of offensive .NET tradecraft easier, and serve as a collaborative **command and control (C2)** platform for red teamers.
Covenant is an ASP.NET Core, cross-platform application that includes a web-based interface that allows for multi-user collaboration.

### Features

Covenant has several key features that make it useful and differentiate it from other command and control frameworks:

- **Intuitive Interface** - Covenant provides an intuitive web application to easily run a collaborative red team operation.
- **Multi-Platform** - Covenant targets .NET Core, which is  multi-platform. This allows Covenant to run natively on Linux, MacOS,  and Windows platforms. Additionally, Covenant has docker support,  allowing it to run within a container on any system that has docker  installed.
- **Multi-User** - Covenant supports multi-user  collaboration. The ability to collaborate has become crucial for  effective red team operations. Many users can interact with the same  Covenant server and operate independently or collaboratively.
- **API Driven** - Covenant is driven by an API that enables  multi-user collaboration and is easily extendible. Additionally,  Covenant includes a Swagger UI that makes development and debugging  easier and more convenient.
- **Listener Profiles** - Covenant supports listener  “profiles” that control how the network communication between Grunt  implants and Covenant listeners look on the wire.
- **Encrypted Key Exchange** - Covenant implements an  encrypted key exchange between Grunt implants and Covenant listeners  that is largely based on a similar exchange in the Empire project, in  addition to optional SSL encryption. This achieves the cryptographic  property of forward secrecy between Grunt implants.
- **Dynamic Compilation** - Covenant uses the Roslyn API for  dynamic C# compilation. Every time a new Grunt is generated or a new  task is assigned, the relevant code is recompiled and obfuscated with  ConfuserEx, avoiding totally static payloads. Covenant reuses much of  the compilation code from the SharpGen project, which I described in  much more detail in a previous post.
- **Inline C# Execution** - Covenant borrows code and ideas  from both the SharpGen and SharpShell projects to allow operators to  execute C# one-liners on Grunt implants. This allows for similar  functionality to that described in the SharpShell post, but allows the  one-liners to be executed on remote implants.
- **Tracking Indicators** - Covenant tracks “indicators”  throughout an operation, and summarizes them in the Indicators menu.  This allows an operator to conduct actions that are tracked throughout  an operation and easily summarize those actions to the blue team during  or at the end of an assessment for deconfliction and educational  purposes. This feature is still in it’s infancy and still has room for  improvement.
- **Developed in C#** - Personally, I enjoy developing in C#, which may not be a surprise for anyone that has read my latest blogs or tools. Not everyone might agree that development in C# is ideal, but  hopefully everyone agrees that it is nice to have all components of the  framework written in the same language. I’ve found it very convenient to write the server, client, and implant all in the same language. This  may not be a true “feature”, but hopefully it allows others to  contribute to the project fairly easily.


The [Wiki](https://github.com/cobbr/Covenant/wiki) documents most of Covenant's core features and how to use them.

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

After Installing Covenant C2 tools on your machine and making sure that is work correctly, you will just need to keep in mind its **@ip address**, because we will need it later for the nginx setup configuration file. 

Now prepare another Linux machine -in my case I am using ubuntu- and install Nginx server. 

The next step is nothing more than editing the _"/etc/nginx/sites-enabled/default"_ file, delete all the previous content , and add the following configuration :
```bash

server {
        listen 80 default_server;
        listen [::]:80  default_server;

        root /var/www/html;

        index index.html index.htm index.nginx-debian.html;

        server_name *.example.org;

        location / {
                try_files $uri $uri/ @c2;
        }

        location @c2 {
                proxy_pass http://@covenant-ip-address;
                proxy_redirect off;
                proxy_ssl_verify off;
                proxy_set_header Host $host;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
}

```

then Restart the Nginx service.
```bash
service nginx restart
```

Now your Nginx redirector is ready, on your futur attack make sure to put the **@ip adsress** of the machine where nginx is installed and configured on the covenant Listener Tab.

The video below is the PoC of this setup ;) 
Enjoy, and don't forget to leave a beautifull comment below ...
![]({https://github.com/HackBalak/Hackbalak.github.io/blob/main/_posts/Aseets/set-up-nginx-c2-redirector/attack_PoC.mkv})
<iframe width="420" height="315" src="https://github.com/HackBalak/Hackbalak.github.io/blob/main/_posts/Aseets/set-up-nginx-c2-redirector/attack_PoC.mkv" frameborder="0"></iframe>

