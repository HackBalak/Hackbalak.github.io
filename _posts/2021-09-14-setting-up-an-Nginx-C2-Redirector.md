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
  
driveId: 1wrYcCAjLUpVK_4k0k3HafbB7FKs61r3n/preview

---

## What is covenanat C2 ?

<img src="https://raw.githubusercontent.com/wiki/cobbr/Covenant/covenant.png?raw=true" width="1200" height="300">

[Covenant](https://github.com/cobbr/Covenant) is a .NET command and control framework and web application that aims to highlight the attack surface of .NET, make the use of offensive .NET tradecraft easier, and serve as a collaborative **command and control (C2)** platform for red teamers.
Covenant is an ASP.NET Core, cross-platform application that includes a web-based interface that allows for multi-user collaboration.

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

Now prepare another Linux machine -- in my case I am using ubuntu -- and install Nginx server on it. 

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

Now your Nginx redirector is ready, on your futur attack make sure to put the **@ip address** of the machine where nginx is installed and configured on the covenant Listener Tab.

The video below is the PoC of this setup ;) 

Enjoy, and don't forget to leave a beautifull comment below ...

{% include googleDrivePlayer.html id=page.driveId %}

