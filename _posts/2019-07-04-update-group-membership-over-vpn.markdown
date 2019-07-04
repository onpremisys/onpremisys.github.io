---
layout: post
title:  "Updating user group membership over VPN"
date:   2019-07-04
author: mvosslamber
image: assets/images/heroes/world-map.jpg
---
You probably already know that group membership is being updated at system logon, but you need to be able to connect with your domain controller. Unless you're using DirectAccess or Always on VPN with device tunneling, you're not able to contact your domain controller at the system logon. There are several posts on the internet about ```klist purge```. I tried that but it didn't work for me. I found an easier solution that actually works.

With this small script you will be able to update the group membership. It is important that you are connected with the VPN and that all programmes are closed.

{% highlight powershell %}
#Requires -RunAsAdministrator
taskkill /F /IM explorer.exe
$username = "$env:userdomain\$env:username"
$credential = Get-Credential -UserName $username -Message 'Enter password'
Start-Process explorer.exe -Credential $credential
{% endhighlight %}

You can check it by running the following command: ``` whoami /groups ```. The output shows your users group memberships.
