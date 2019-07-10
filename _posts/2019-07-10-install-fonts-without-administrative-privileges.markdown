---
layout: post
title:  "Install fonts without administrative privileges"
date:   2019-07-10
author: broovers
image: assets/images/heroes/hieroglyphs.jpg
---
A lot of companies have fonts that are used as part of corporate branding. These should therefore probably be installed on every computer. Deploying these has been part of an administrator’s job for a long time. There are several ways to do this, but they usually require administrative privileges.

Fonts are usually in the `C:\Windows\Fonts` directory. Which requires administrative privileges to alter. Luckily, as of Windows 10 version 1803 (released in April 2018), [non-admin font installation was added](https://blogs.windows.com/windowsexperience/2018/06/27/announcing-windows-10-insider-preview-build-17704/). Installing fonts without privileges now installs them in the local app data folder of the user. Doing this via a script automatically chooses the user folder if it didn’t have the necessary privileges. I’ve incorporated this behavior into a Powershell script which looks for fonts in the same directory as the script and installs them.

{% highlight powershell %}
$Destination = (New-Object -ComObject Shell.Application).Namespace(20)

$TempFolder = "$($env:windir)\Temp\Fonts\"

New-Item -Path $TempFolder -Type Directory -Force | Out-Null

Get-ChildItem -Path $PSScriptRoot\* -Include '*.ttf','*.ttc','*.otf' | ForEach {
    If (-not(Test-Path "$($env:LOCALAPPDATA)\Microsoft\Windows\Fonts\$($_.Name)")) {

        $Font = "$($env:windir)\Temp\Fonts\$($_.Name)"

        Copy-Item  $($_.FullName) -Destination $TempFolder
        
        $Destination.CopyHere($Font)

        Remove-Item $Font -Force

    }

}
{% endhighlight %}