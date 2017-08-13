---
layout: post
title: HTTPS Redirects and Let's Encrypt Renewals
published: true
excerpt: I have a site that requires SSL. The website in question needs to only use HTTPS for all requests because sensitive information is being transfered over the wire and because SSL is good!
---

I have a site that requires SSL. The website in question needs to only use HTTPS for all requests because sensitive information is being transfered over the wire and [because SSL is good!](https://www.troyhunt.com/dont-take-security-advice-from-seo-experts-or-psychics-neil-patel/). I decided the best route was to force SSL redirects in nginx. I don't want users of the site to pass any information over an insecure channel.

Recently, I accessed the website and only to get this message: `ERROR_SSL_CERTIFICATE_EXPIRED`. I was very sure that I had setup automagical renewal of certificates on my server. I had of course set this up. The cron job was already created using `crontab -e`.

```
30 2 * * 1 /usr/local/sbin/certbot renew --quiet
```

So why didn't `certbot` get the new certificates. Let's Encrypt's `certbot` will create a `.well-known` folder in the root of the web folder. When your website accepts HTTP requests, Let's Encrypt verifies the authenticity of the domain by reading the challenge file stored in that folder. Let's Encrypt will always and only connect to the site using HTTP (port 80). If the nginx configuration always redirects to HTTPS, the Let's Encrypt challenge verification will not pass (it wont use HTTPS). 

I've fixed this issue by updating the nginx configuration to redirect all traffic to HTTPS except the `.well-known` folder. I've tested the changes by manually running the renew command.

```bash
server {
        listen 80 default_server;

        server_name example.com www.example.com;
        #return 301 https://$server_name$request_uri;

        location / {
                # [...]
                return 301 https://$server_name$request_uri;
        }

        location /.well-known {
                alias /var/www/example.com/.well-known;
        }
}
```

By putting the `return 301 https://$server_name$request_uri;` inside the `location /` section rather than outside of the section, nginx will only redirect for request that match that path. If the request path is matched by `location /.well-known`, it will not force a redirect (because it is only scoped to `/`) and allow access.