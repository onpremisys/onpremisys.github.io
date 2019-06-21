---
layout: post
title:  "Prepend www using URL Rewrite in IIS"
date:   2015-05-20
author: Bas Roovers
---
Ervoor zorgen dat alle url’s een www. voor de domeinnaam krijgen lijkt heel simpel maar om een universele redirect te krijgen zijn er nog wat truukjes nodig.
Voeg de volgende code toe aan je web.config in het system.webserver blok.

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