---
layout: post
title:  "Groeplidmaatschap bijwerken zonder herstart"
date:   2016-10-29
author: Bas Roovers
---
Ik vraag me wel eens af waarom het anno 2015 nog nodig is om zo vaak opnieuw te starten voor bijna iedere wijziging die we uitvoeren. Gelukkig kennen enkele wijzigingen een oplossing of omweg, waaronder een groepslidmaatschap voor computers.

Het onderstaande commando zal ervoor zorgen dat de Kerberos tickets als ongeldig worden bestempeld en nieuwe worden aangemaakt. Hierdoor worden ook alle lidmaatschappen van de computer opgehaald en hoef je Windows dus niet opnieuw te starten.

{% highlight cmd %}
klist -lh 0 -li 0x3e7 purge
{% endhighlight %}