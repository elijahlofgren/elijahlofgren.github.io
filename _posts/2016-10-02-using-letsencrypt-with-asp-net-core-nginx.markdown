---
published: true
title: Using LetsEncrypt to provide https support to ASP.NET Core app hosted with nginx
layout: post
permalink: 2016-10-02-using-letsencrypt-with-asp-net-core-nginx
---
Let's say your hosting on Ubuntu after following this guide: [Publishing an ASP.NET Core website to a cheap Linux VM host](http://www.hanselman.com/blog/PublishingAnASPNETCoreWebsiteToACheapLinuxVMHost.aspx)

Add a controller to your app that allows for serving files like /.well-known/acme-challenge/abc. An example one is available at the bottom of [http://www.westerndevs.com/Tools/Lets-Encrypt-ASPNET-Core/](http://www.westerndevs.com/Tools/Lets-Encrypt-ASPNET-Core/)

Let's say you have your app running from the following directory:  		
	~/webapps/myapp/bin/Debug/netcoreapp1.0/publish/

Generate SSL cert by installing Let's Encrypt and then running:
	./certbot-auto certonly --webroot -w ~/webapps/myapp/bin/Debug/netcoreapp1.0/publish/wwwroot -d example.org -d www.example.org

Now edit your /etc/nginx/sites-available/default file and change it from something like:


	server {
		listen 80;
		location / {
			proxy_pass http://localhost:5123;
			proxy_http_version 1.1;
			proxy_set_header Upgrade $http_upgrade;
			proxy_set_header Connection keep-alive;
			proxy_set_header Host $host;
			proxy_cache_bypass $http_upgrade;
		}
	}
 
to:


	server {
		listen 80;
		listen 443 ssl;
		server_name example.org www.example.org;
		ssl_certificate /etc/letsencrypt/live/example.org/fullchain.pem;
		ssl_certificate_key /etc/letsencrypt/live/example.org/privkey.pem;
		location / {
			proxy_pass http://localhost:5123;
			proxy_http_version 1.1;
			proxy_set_header Upgrade $http_upgrade;
			proxy_set_header Connection keep-alive;
			proxy_set_header Host $host;
			proxy_cache_bypass $http_upgrade;
		}
	}

For additional info on forcing HTTPS and using a stronger cipher see [How To Secure Nginx with Let's Encrypt on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-14-04)