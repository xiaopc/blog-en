---
title: Speaking of httpoxy
id: 20
categories:
  - backend
date: 2020-05-27 16:04:37
---

*original post date: July 20, 2016*

1.  Service who get HTTP_PROXY from environment `PATH` could be infected (wget/curl will be fine).
2.  Service data could be stolen while connecting outbound destination by attackers' given `HTTP_PROXY`.
3.  Under CGI/FPM mode.
4.  Fix for nginx: add these to `fastcgi.conf`:
{% codeblock lang:nginx %}
fastcgi_param HTTP_PROXY "";
{% endcodeblock %}
5.  Apache has official update.

<!--more-->

* * *

References:

[1] https://httpoxy.org/

[2] http://www.laruence.com/2016/07/19/3101.html
