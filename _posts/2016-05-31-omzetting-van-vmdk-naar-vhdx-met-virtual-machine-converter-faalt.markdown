---
layout: post
title:  "Omzetting van VMDK naar VHDX met Virtual Machine Converter faalt"
date:   2016-05-31
author: Bas Roovers
---
Probeer je een VMware VMDK schijf om te zetten naar Hyper-V via de Virtual Machine Converter van Microsoft? Dan kan het wel eens voor komen dat je een foutmelding krijgt die hier op lijkt en dat de omzetting niet kan worden gestart.

The entry 046a40e4 is not a supported disk database entry for the descriptor.

Dit houdt feitelijk in dat er iets in het descriptor bestand staat waar de converter niet mee overweg kan. Je kunt dit probleem oplossen door het descriptor bestand te openen in een tekstverwerker en de volgende regel te verwijderen. Het descriptor bestand is het kleine bestand van ongeveer 1 KB welke naast het -flat bestand staat.

{% highlight cmd %}
ddb.alternateParentCID = "046a40e4"
{% endhighlight %}

De code 046a40e4 zal waarschijnlijk anders zijn dus kijk voor jouw code in de foutmelding die je kreeg.