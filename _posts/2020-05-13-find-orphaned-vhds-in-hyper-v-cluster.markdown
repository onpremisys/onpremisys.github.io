---
layout: post
title:  "Find orphaned VHDs in Hyper-V cluster"
date:   2020-05-13
author: broovers
image: assets/images/heroes/magnifying-glass-laptop.jpg
---

If you're running a Hyper-V cluster for a while, you probably ended up with a couple of virtual hard disks for which the actual VM is already gone. These orphaned VHDs are occupying valuable space and are kinda difficult to trace. I decided to dive into this and hoped that someone has already solved this puzzle.

I found the following scripts which advertise to do the job:
* [Free script find orphaned Hyper-V VM files](https://www.altaro.com/hyper-v/free-script-find-orphaned-hyper-v-vm-files)  
This one is 346 lines of code and always ends up with an error for me.
* [Get-OrphanedVHDs.ps1 from alexinslc](https://github.com/alexinslc/powershell/blob/master/Get-OrphanedVHDs.ps1)  
Simple set-up but uses csv files and looks to be made for local storage Hyper-V nodes.

That didn't really help so I decided to be bold and write my own version. Mine is meant to be run on clusters where all of your Hyper-V nodes use the same storage like with a SAN-based setup. It's 36 lines and gets the job done within a minute for over 200 VMs.

You can run this Powershell script against any one of your nodes directly or by running it through Invoke-Command.

{% highlight powershell %}
$clusterNodes = Get-ClusterNode | Select-Object Name -ExpandProperty Name

if(!$clusterNodes){Throw "No nodes found, please run this script on one of the nodes."}

Write-Host "The following nodes are part of the cluster: "
$clusterNodes | ForEach-Object { Write-Host "- $($_.Name)" }
Write-Host ""

Write-Host "Fetch VHDs from nodes: " -NoNewline
$ActiveVHDs = (Get-VM -ComputerName $clusterNodes | Get-VMHardDiskDrive | Select-Object -Property Path).Path
Write-Host "V" -ForegroundColor Green
Write-Host ""


Write-Host "Fetch VHDs from storage: " -NoNewline
$HyperVFileLocation = ("C:\ClusterStorage")
$Dir = Get-ChildItem $HyperVFileLocation -Recurse -ErrorAction Ignore | Where-Object {$_.FullName -notmatch "\\Replica\\?" } 
$AllVHDs = ($Dir | Where-Object { $_.Extension -eq ".vhd" }).FullName + ($Dir | Where-Object { $_.Extension -eq ".vhdx" }).FullName
Write-Host "V" -ForegroundColor Green
Write-Host ""

Write-Host "Compare VHDs of storage with nodes: " -NoNewline
$orphanedVHDs = @()
foreach($vhd in $AllVHDs){
    if($vhd -notin $ActiveVHDs){
        $orphanedVHDs += $vhd
    }
}
Write-Host "V" -ForegroundColor Green
Write-Host ""
if($orphanedVHDs){
    Write-Host "List of orphaned VHDs:"
    $orphanedVHDs
} else {
    Write-Host "Nice work, no orphaned VHDs found!" -ForegroundColor Green
}
{% endhighlight %}