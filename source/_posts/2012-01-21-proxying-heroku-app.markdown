---
layout: post
title: "Proxying Heroku App"
date: 2012-01-21 14:21
comments: true
categories: [howto]
tags: [heroku, apache, proxy]
---
A few weeks ago I've challenged to make some routing with Apache's `mod_rewrite`.
The task was to show `http://someapp.heroku.com` when user visit `http://site.com/heroku`.
Ok, it seems to be not a problem to extend `.htaccess` file one `RewriteRule` string:
{% codeblock lang:apache %}
RewriteRule ^heroku/?(.*) http://someapp.heroku.com/$1 [P,L]
{% endcodeblock %}
But our `http://site.com` running Wordpress, so the new rule must be separated with conditions:
{% codeblock lang:apache %}
RewriteCond %{REQUEST_URI} ^\/heroku
RewriteRule ^heroku/?(.*) http://someapp.heroku.com/$1 [P,L]
{% endcodeblock %}
Looks nice, and it works, but Heroku returns `Bad Request` error when I've tried to visit "rewrited" page. After some derping over internet, I found, that Heroku needs
request header `host` to be set with an app domain, `someapp.heroku.com` in my case. But looking forward, I've noted that
Wordpress using request header `host` for it's own purposes (e.g. link generation in admin), so I need to make another
condition for request header. And here it comes - the final result:
{% codeblock lang:apache %}
SetEnvIf Request_URI ^\/heroku heroku
RequestHeader set host "someapp.heroku.com" env=heroku
RewriteCond %{REQUEST_URI} ^\/heroku
RewriteRule ^heroku/?(.*) http://someapp.heroku.com/$1 [P,L]
{% endcodeblock %}
And after that manipulations - it works like charm now!