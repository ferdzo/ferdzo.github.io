---
layout: post
title: Using Nginx as subdomain proxy for Weechat relay  
--- 

Weechat is an IRC client with relay options that enables us to use external applications as it's frontend.
In example, Glowingbear or Weechat for Android are such applications. The relay can be directly
exposed to the internet, and you can also use self-signed SSL certificate, but I don't really like the idea 
of doing that and exposing a possibly vulnerable app to the internet.


So I stumbled upon the idea of using Nginx to proxy the internal port to a custom subdomain and open
it up to the internet. Using Nginx:

1. It's much more secure to open up ports and expose application such as Nginx, which is designed to
do exactly that.

2. Easier to use SSL certificates. It's much more easier to import and manage SSL certificates for Nginx, than
exporting them and copying to the Weechat folder and importing them and so on and so on.

3.  Most of the time I have Nginx already on my server for various jobs, such as proxying other apps, or hosting something else. So it is easy to add just one more config file and make it work perfectly and without any hassle.

We are doing this assuming the internal communication on the server is secure. So we can use SSL just on the Nginx side, because if use SSL on both sides it get's much more complicated and it's not worth the inconvenience. Anyway if the internal communication on our server is not secure there's no point of doing any of this.

```
    printf("test");
```
The post is not finished...