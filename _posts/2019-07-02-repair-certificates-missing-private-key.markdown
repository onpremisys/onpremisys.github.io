---
layout: post
title:  "Repair certificates missing private key"
date:   2019-07-02
author: broovers
image: assets/images/heroes/locksmith.jpg
---
Every now and then I receive a renewed certificate without ever creating a certificate signing request for it. In this case the previous certificate signing request was used. This means the renewed certificate belongs to the private key of the earlier request. When you try to complete the certificate and make a key pair you're most likely missing the private key on your computer. By the time you're aware of this you import the older key pair to make sure you have the private key for the new certificate. Unfortunately Windows doesn't bind the private key automatically to the already existing certificate.

In order to fix this you need to "repair" the certificate by finding the private key and binding it to the certificate. This process involves looking up the thumbprint and running the certutil command with this thumbprint. I created a powershell script to make this a little bit easier for myself.

{% highlight powershell %}
#Requires -RunAsAdministrator

Write-Host "List certificates without private key: " -NoNewline

$certsWithoutKey = Get-ChildItem Cert:\LocalMachine\My | Where-Object {$_.HasPrivateKey -eq $false}


if($certsWithoutKey) {

    Write-Host "V" -ForegroundColor Green

    $Choice = $certsWithoutKey | Select-Object Subject, Issuer, NotAfter, ThumbPrint | Out-Gridview -Passthru

    if($Choice){

        Write-Host "Search private key for $($Choice.Thumbprint): " -NoNewline
        $Output = certutil -repairstore my "$($Choice.Thumbprint)"

        $Result = [regex]::match($output, "CertUtil: (.*)").Groups[1].Value
        if($Result -eq '-repairstore command completed successfully.') {

            Write-Host "V" -ForegroundColor Green

        } else {

            Write-Host $Result -ForegroundColor Red

        }

    } else {

        Write-Host "No choice was made." -ForegroundColor DarkYellow

    }

} else {

    Write-Host "There were no certificates found without private key." -ForegroundColor DarkYellow

}
{% endhighlight %}
