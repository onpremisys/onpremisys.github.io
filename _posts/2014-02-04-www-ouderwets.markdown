---
layout: post
title:  "www. ouderwets?"
date:   2014-02-04
author: Bas Roovers
---
Toen begin jaren 90 de eerste websites opgezet werden was het niet meer dan normaal dat je website geserveerd werd vanaf www.domeinnaam.com. Anno 2014 is de discussie nog steeds gaande tussen teams pro-www en anti-www wat het beste is.

Het onderdeel www heeft van oudsher een reden van bestaan. Dit gedeelte gaf namelijk aan dat je een webserver wilde bereiken en niet een mail of bestandserver. Toch staat het nergens in de documenten dat dit zo moet. Inmiddels is het voor velen ook duidelijk dat een website zonder www ook prima bereikbaar is. Heeft www nog wel nut dan?

World first web browser

Beiden kampen, pro en anti, hebben beiden hun eigen motivaties en zijn allen gegrond. De vraag is dus eigenlijk, wat is voor jouw situatie het beste?

**Anti-WWW**

* Het scheelt iedereen op de wereld tijd omdat je www. niet hoeft te typen.
* http:// geeft al aan dat het om een webserver gaat dus www is overbodig.

**Pro-WWW**

Een CNAME mag volgens [RFC1912 sectie 2.4](http://www.ietf.org/rfc/rfc1912.txt) niet geplaatst worden op de zone van het domein zelf en moet dus op een subdomein, zoals www. Deze technische afhankelijkheid zorgt ervoor dat redundantie op dns niveau enkel mogelijk in combinatie met het gebruik van een subdomein.
Zonder www worden cookies geserveerd op alle subdomeinen wat een mogelijk veiligheidsrisico met zich mee kan brengen.
We kunnen best stellen dat het gebruik van www te rechtvaardigen is wanneer je gebruik maakt van een omgeving met bijvoorbeeld meerdere webservers en daarmee redundantie wilt bieden. Ook het gebruik van een [CDN](https://web.archive.org/web/20140714080310/http://en.wikipedia.org/wiki/Content_delivery_network) vereist vaak dat je website via www bereikbaar is. Maar mocht dit allemaal niet van toepassing zijn, dan kun je met een gerust hart www achterwege laten.

Een ding is zeker in de wereld van "[Search Engine Optimization](http://nl.wikipedia.org/wiki/Zoekmachineoptimalisatie)", zorg dat je kiest voor 1 van beiden en blijf daarbij. Dus als je voor www kiest dan plaats je een [301 redirect](http://en.wikipedia.org/wiki/URL_redirection) op het kale domein. En vice versa bij een keuze voor het kale domein.