---
title: How to reverse proxy Google Analytics and reCaptcha via Nginx
id: 13
categories:
  - backend
date: 2020-05-26 09:32:22
---

*original post date: July 18, 2016*

> 2018/3/11 update
> 
> There are a lot of problems with this approach (lack of access to real IP, etc.) , and given the improved availability of Google Analytics services in China, it is no longer recommended. If reliability is sought, it is recommended to use domestic services or self-built statistical services.

You can find nginx.conf for reversing proxy Google Analytics on Google, but most of them are for the old version. 
And so do reCaptcha's. After solving some issues, here's my code.

<!--more-->

Attention:
1. `ngx_http_substitutions_filter_module module` needed。
2. DO NOT ADD `X-Frame-Options DENY` WHILE CONFIGURING SSL (`SOMEORIGIN` would be fine for self using, not tested)
3. After reverse proxy reCaptcha, challenge will be difficult. (Using proxies will reduce credibility, and some code uses complex obfuscations that result in client-side identification code can't be modified)

{% codeblock lang:nginx %}
# reCaptcha reverse proxy
 location /recaptcha {
     default_type text/html;
     subs_filter_types text/css text/xml text/javascript;
     subs_filter 'www.gstatic.com' 'lab.xpc.im/wwwgstatic' g;
     proxy_set_header X-real-ip $remote_addr;
     proxy_pass https://www.google.com/recaptcha;
     proxy_set_header Accept-Encoding ""; #see [3]
     proxy_set_header User-Agent $http_user_agent;
     break;
 }
 location /wwwgstatic {
     subs_filter_types text/css text/xml text/javascript;
     subs_filter 'www.gstatic.com' 'lab.xpc.im/wwwgstatic' g;
     subs_filter 'www.google.com/recaptcha' 'lab.xpc.im/recaptcha' g;
     subs_filter 'ssl.gstatic.com' 'lab.xpc.im/sslgstatic' g;
     proxy_set_header X-real-ip $remote_addr;
     proxy_pass https://www.gstatic.com/;
     proxy_set_header Accept-Encoding "";
     proxy_set_header User-Agent $http_user_agent;
     break;
 }
 location /sslgstatic {
     subs_filter_types text/css text/xml text/javascript;
     subs_filter 'www.gstatic.com' 'lab.xpc.im/wwwgstatic' g;
     subs_filter 'www.google.com/recaptcha' 'lab.xpc.im/recaptcha' g;
     proxy_set_header X-real-ip $remote_addr;
     proxy_pass https://ssl.gstatic.com/;
     proxy_set_header Accept-Encoding "";
     proxy_set_header User-Agent $http_user_agent;
     break;
 }
{% endcodeblock %}

{% codeblock lang:nginx %}
# Google Analytics reverse proxy
 location /ga {
     proxy_set_header X-real-ip $remote_addr;
     rewrite ^/ga/(.*)$ /$1?$args&amp;uip=$remote_addr;
     proxy_pass https://www.google-analytics.com;
     proxy_set_header User-Agent $http_user_agent;
     break;
 }
 location /analytics {
     proxy_set_header X-real-ip $remote_addr;
     rewrite ^/ga/(.*)$ /$1?$args&amp;uip=$remote_addr;
     proxy_pass https://www.google.com;
     proxy_set_header User-Agent $http_user_agent;
     break;
 }
 location /analytics.js {
     default_type text/html;
     subs_filter_types text/css text/xml text/javascript;
     subs_filter 'www.google-analytics.com' 'lab.xpc.im/ga' g;
     subs_filter 'www.google.com/analytics' 'lab.xpc.im/analytics' g;
     proxy_set_header X-real-ip $remote_addr;
     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
     proxy_set_header Referer https://www.google-analytics.com;
     proxy_set_header Host www.google-analytics.com;
     proxy_pass https://www.google-analytics.com;
     proxy_set_header Accept-Encoding "";
 }
{% endcodeblock %}

* * *

References:

[1] https://ruby-china.org/topics/27400

[2] https://rocfang.gitbooks.io/dev-notes/content/guan_yu_proxy_pass_de_can_shu_lu_jing_wen_ti.html

[3] https://www.v2ex.com/t/234923
