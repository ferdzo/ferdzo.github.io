---
layout: post
title: Using Nginx as subdomain proxy for Weechat relay  
--- 

**Weechat** is an IRC client with relay options that enables us to use external applications as the frontend for Weechat.
For example, Glowingbear or Weechat for Android is such application. The relay can be directly
exposed to the internet, and you can also use a self-signed SSL certificate, but I don't really like the idea 
of doing that and exposing a possibly vulnerable app to the internet.


So I stumbled upon the idea of using Nginx to proxy the internal port to a custom subdomain and open
it up to the internet. Using Nginx:

1. It's much more secure to open up ports and expose application such as Nginx, which is designed to
do exactly that.

2. Easier to use SSL certificates. It's much easier to import and manage SSL certificates for Nginx, than
exporting them and copying to the Weechat folder and importing them and so on and so on.

3.  Most of the time I have Nginx already on my server for various jobs, such as proxying other apps or hosting something else. So it is easy to add just one more config file and make it work perfectly and without any hassle.

We are doing this assuming the internal communication on the server is secure. So we can use SSL just on the Nginx side because if use SSL on both sides it gets much more complicated and it's not worth the inconvenience. Anyway, if the internal communication on our server is not secure there's no point in doing any of this.

{% if post.excerpt != post.content %}
    <a href="{{ site.baseurl }}{{ post.url }}">Read more</a>
{% endif %}

## Nginx configuration

First, we need to make a config file with the subdomain we are using in /etc/nginx/sites-available. In example, /etc/nginx/sites-available/wchat.example.xyz. After this step, we need to create a symbolic link for the config file to the /etc/nginx/sites-enabled/ folder with the command
```
ln -s /etc/nginx/sites-available/wchat.example.xyz /etc/nginx/sites-enabled/wchat.example.xyz
```
In the next step, we go back to the Nginx config file and set up the proxy. Our subdomain should point to the same IP address as our server. The proxy forwards all the connections to port 9001 on our local machine. 

```
map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}
server {
    listen      443 ssl http2;
    server_name wchat.example.xyz;

    location / {
        proxy_pass http://127.0.0.1:9001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_read_timeout 4h;
    }

}
```
In server_name we write our domain and at proxy_pass the local IP address and the port of the Weechat relay. The other parameters can be copied and are not important.
### SSL certificate using certbot
In this step, we need to make a new SSL certificate, in this case, we are using certbot to make a self-signed SSL certificate using Let's Encrypt. To create a certificate we use the following command:
```
sudo certbot --nginx -d wchat.example.xyz 
```
Using the --nginx parameter, certbot will automatically configure Nginx with the SSL certificate.

Now, we test our Nginx configuration
```
nginx -t
```
If the test passes we can continue and restart our Nginx service
```
systemctl restart nginx.service
```

## Configuring Weechat
After we configured our Nginx proxy, we are now going the configure the Weechat relay. We enter the following commands in Weechat
```
/set relay.network.password thepasswordiwant
/set relay.network.bind_address "127.0.0.1"
/relay add ipv4.weechat 9001
```
This will set up a Weechat password and set the relay listening port to 9001.

## Connecting to the relay
And in the last step, we should try and connect to our relay. The port should be 443 and not 9001 because our proxy forwards the traffic from 443 to 9001. And port 9001 is not open to external connections.


Thanks for reading my first post on the blog and stay tuned for more new posts!