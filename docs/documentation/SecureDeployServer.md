# Deploy NPL Web Server With SSL (https)
This post shows `How To Configure Nginx with SSL as a Reverse Proxy for NPL Web Server`.

### Why NPL Does Not Support SSL Natively?
NPL protocol is TCP based, which can run HTTP and NPL's TCP protocol on the `same` port. 
If the TCP connection is encoded with SSL, it will break NPL's internal TCP protocol. 
This is why NPL does not support SSL natively. However, NPL server can fetch data via `https`, 
so it is perfectly fine to call SSL protected rest API within the NPL server.

### Introduction
By default, NPL comes with its own built in web server, which listens on port 8099. 
This is convenient if you run a private NPL instance, or if you just need to get something up quickly 
and don't care about security. Once you have real production data going to your host, though, 
it's a good idea to use a more secure web server proxy in front like [Nginx](https://nginx.org/).

### Prerequisites

This post will detail how to wrap your site with SSL using the Nginx web server as a reverse proxy for your NPL instance. This tutorial assumes some familiarity with Linux commands, a working NPL Runtime installation, and a Ubuntu 14.04 installation.

You can install NPL runtime later in this tutorial, if you don't have it installed yet.

### Step One — Configure Nginx
Nginx has become a favored web server for its speed and flexibility in recents years, so that is the web server we will be using.

#### Install Nginx
Update your package lists and install Nginx:
```
sudo apt-get update
sudo apt-get install nginx
```

#### Get a Certificate
Next, you will need to purchase or create an SSL certificate. These commands are for a self-signed certificate, but you should get an officially signed certificate if you want to avoid browser warnings.

Move into the proper directory and generate a certificate:
```
cd /etc/nginx
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/cert.key -out /etc/nginx/cert.crt
```
You will be prompted to enter some information about the certificate. You can fill this out however you'd like; just be aware the information will be visible in the certificate properties. We've set the number of bits to 2048 since that's the minimum needed to get it signed by a CA. If you want to get the certificate signed, you will need to create a CSR.

#### Edit the Configuration
Next you will need to edit the default Nginx configuration file.
```
sudo nano /etc/nginx/sites-enabled/default
```
Here is what the final config might look like; the sections are broken down and briefly explained below. You can update or replace the existing config file, although you may want to make a quick copy first.
```
server {
    listen 80;
    return 301 https://$host$request_uri;
}

server {

    listen 443;
    server_name npl.domain.com;

    ssl_certificate           /etc/nginx/cert.crt;
    ssl_certificate_key       /etc/nginx/cert.key;

    ssl on;
    ssl_session_cache  builtin:1000  shared:SSL:10m;
    ssl_protocols  TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers HIGH:!aNULL:!eNULL:!EXPORT:!CAMELLIA:!DES:!MD5:!PSK:!RC4;
    ssl_prefer_server_ciphers on;

    access_log            /var/log/nginx/npl.access.log;

    location / {

      proxy_set_header        Host $host;
      proxy_set_header        X-Real-IP $remote_addr;
      proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header        X-Forwarded-Proto $scheme;

      # Fix the “It appears that your reverse proxy set up is broken" error.
      proxy_pass          http://localhost:8099;
      proxy_read_timeout  90;

      proxy_redirect      http://localhost:8099 https://npl.domain.com;
    }
  }
```
In our configuration, the cert.crt and cert.key settings reflect the location where we created our SSL certificate. You will need to update the servername and `proxyredirect` lines with your own domain name. There is some additional Nginx magic going on as well that tells requests to be read by Nginx and rewritten on the response side to ensure the reverse proxy is working.

The first section tells the Nginx server to listen to any requests that come in on port 80 (default HTTP) and redirect them to HTTPS.
```
...
server {
   listen 80;
   return 301 https://$host$request_uri;
}
...
```
Next we have the SSL settings. This is a good set of defaults but can definitely be expanded on. For more explanation, please read this tutorial.
```
...
  listen 443;
  server_name npl.domain.com;

  ssl_certificate           /etc/nginx/cert.crt;
  ssl_certificate_key       /etc/nginx/cert.key;

  ssl on;
  ssl_session_cache  builtin:1000  shared:SSL:10m;
  ssl_protocols  TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers HIGH:!aNULL:!eNULL:!EXPORT:!CAMELLIA:!DES:!MD5:!PSK:!RC4;
  ssl_prefer_server_ciphers on;
  ...
```
The final section is where the proxying happens. It basically takes any incoming requests and proxies them to the NPL instance that is bound/listening to port 8099 on the local network interface. This is a slightly different situation, but this tutorial has some good information about the Nginx proxy settings.
```
...
location / {

    proxy_set_header        Host $host;
    proxy_set_header        X-Real-IP $remote_addr;
    proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header        X-Forwarded-Proto $scheme;

    # Fix the “It appears that your reverse proxy set up is broken" error.
    proxy_pass          http://localhost:8099;
    proxy_read_timeout  90;

    proxy_redirect      http://localhost:8099 https://npl.domain.com;
}
...
```
A few quick things to point out here. If you don't have a domain name that resolves to your NPL server, then the proxy_redirect statement above won't function correctly without modification, so keep that in mind. 

### Step Two — Configure NPL Runtime
As stated previously, this tutorial assumes that NPL is already installed. [This tutorial](InstallGuide) will show you how to install NPL if necessary. You will probably need to switch to the root user for that article.

For NPL to work with Nginx, we need to update the NPL config to listen only on the localhost interface instead of all (0.0.0.0), to ensure traffic gets handled properly. This is an important step because if NPL is still listening on all interfaces, then it will still potentially be accessible via its original port (8099). 

For example, one may start npl webserver like [this](https://github.com/NPLPackages/WebServerExample/blob/master/start.sh)
```
pkill -9 npl
npl -d root="WebServerExample/" ip="127.0.0.1" port="8099" bootstrapper="script/apps/WebServer/WebServer.lua" 
```
Notice that the `ip="127.0.0.1"` setting needs to be either added or modified.

Then go ahead and restart NPL and Nginx.

```
sudo service nginx restart
```
You should now be able to visit your domain using either HTTP or HTTPS, and the NPL web site will be served securely. You will see a certificate warning if you used a self-signed certificate.

### Conclusion

The only thing left to do is verify that everything worked correctly. As mentioned above, you should now be able to browse to your newly configured URL - npl.domain.com - over either HTTP or HTTPS. You should be redirected to the secure site, and should see some site information, including your newly updated SSL settings. As noted previously, if you are not using hostnames via DNS, then your redirection may not work as desired. In that case, you will need to modify the proxy_pass section in the Nginx config file.

You may also want to use your browser to examine your certificate. You should be able to click the lock to look at the certificate properties from within your browser.

### References
> This is a rewrite of this [post](https://www.digitalocean.com/community/tutorials/how-to-configure-nginx-with-ssl-as-a-reverse-proxy-for-jenkins) in terms of NPL. 