---
layout: post
title:  "Visual Studio kan niet bij de IIS metabase"
date:   2015-07-27
author: Bas Roovers
---
Het komt regelmatig voor dat nieuwe installaties van Visual Studio met foutmeldingen komen als:

> The Web Application Project Frontend is configured to use IIS. Unable to access the IIS metabase. You do not have sufficient privilege to access IIS web sites on your machine.

Ongeacht of je voldoende rechten op je machine hebt of zelfs aangemeld bent als Administrator blijf je deze melding krijgen. Het probleem zit hem in de standaard vergrendelde mappen van IIS, waar Visual Studio niet bij kan.

Je kunt dit op 2 manieren oplossen, echter is oplossing 2 de mooiste.

**Oplossing 1**

Je gaat op zoek naar de executable van Visual Studio en zorgt ervoor dat deze standaard met elevated rights geopend wordt. Iedere keer dat je dan Visual Studio opent zal UAC om bevestiging vragen dat deze met elevated rights wordt geopend.

**Oplossing 2**

Open de volgende mappen in verkenner en bevestig dat Windows de betreffende map permanent open zet qua permissies.

C:\Windows\System32\inetsrv
C:\Windows\System32\inetsrv\config\export