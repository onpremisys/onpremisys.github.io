---
layout: post
title:  "Local admin permissions? Doing it the right way!"
date:   2019-06-27
author: mvosslamber
image: assets/images/heroes/useless-gate.jpg
---
There's always a case that some users, or even a complete department need local admin permissions. But how can you control this, and more important, how to do this the right way?

If you've created a domain group and added that group to all the local Administrators group on all machines, then you are doing it wrong! Why? Because you've made everyone local administrator on all of these machines. Handy right? Sure, but they're also able to access all machines' hidden shares. So user A is able to access all drives of user B's laptop. What's the right way I hear you ask? You should add the user of the machine to the local Administratos group of that machine, that's it, simple right? But what if you have hundreds or even thousands of machines? Then you have got something to do coming weekend! Just kidding, here is an easier way.

First we create a GPO that adds a domain group to the local Administratos group with the computername in it. You do this as follows:

1. Navigate to: Computer Configuration\Preferences\Control Panel Settings\Local Users and Groups\
2. Create a new Local Group
3. Choose as action for Update
4. In Group name drop down choose: Administrators (build-in)
5. Click on Add... and fill this in by name: ```%DomainName%\Local Admin %ComputerName%```
6. Save the GPO and scope it to the group of machines you want

Now every machine will have a domain group (with the machine name in it) as member of the local Administratos group. But the groups arent in the Active Directory right? Well not yet, with a little bit of PowerShell we create this in a minute, here is the script:

{% highlight powershell %}
$searchFilter = "*"
$computerNames = Get-ADComputer -Filter {Name -like $searchFilter} -SearchBase "OU=Laptops, OU=Computers, DC=CompanyDomain, DC=local"
foreach ($computerName in $computerNames) {
    New-ADGroup -Name "Local Admin $($computerName.Name)" -GroupCategory Security -GroupScope Global -Path "OU=Local Admin, OU=Groups, DC=CompanyDomain, DC=local"
    }
{% endhighlight %}

What it does, it scans an OU for computer names and it creates a Domain Group in an different OU. The group names contains the machinename and it has the same name as the one you added to the local machines. All you have to do is add the correct users, and you're done. That saved a lot of time right?