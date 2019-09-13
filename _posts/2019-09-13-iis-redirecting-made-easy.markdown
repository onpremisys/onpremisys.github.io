---
layout: post
title:  "IIS redirecting made easy"
date:   2019-09-13
author: broovers
image: assets/images/heroes/crowd-redirect.jpg
---

Redirecting web traffic in IIS has been done for ages. Most people use the [URL Rewrite plugin](https://www.iis.net/downloads/microsoft/url-rewrite) to achieve this. It provides a relatively easy way to create redirect rules.


Most rules I see are very specific with the exact domain name in the rule and manually created over and over again for each project. Fortunately there are rules crafted in such a way that they can be applied for every website. This saves me a lot of time and hopefully now yours.

Let me share the 2 most commonly used ones:

## Prepending www to 2nd level domain names
Credits to [Scott Forsyth](https://weblogs.asp.net/owscott/prepending-www-to-2nd-level-domain-names)
{% highlight xml %}
<rule name="Prepend www to 2nd level domain names" enabled="true" stopProcessing="true">
    <match url=".*" />
    <conditions trackAllCaptures="true">
        <add input="{HTTP_HOST}" pattern="^([^.]+\.[^.]+)$" />
        <add input="{CACHE_URL}" pattern="^(.+)://" />
    </conditions>
    <action type="Redirect" url="{C:2}://www.{C:1}/{R:0}" />
</rule>
{% endhighlight %}

## HTTP to HTTPS
{% highlight xml %}
        <rule name="Redirect to https" stopProcessing="true">
          <match url="(.*)"/>
          <conditions>
            <add input="{HTTPS}" pattern="off" ignoreCase="true"/>
          </conditions>
          <action type="Redirect" url="https://{HTTP_HOST}{REQUEST_URI}" />
        </rule>
{% endhighlight %}

