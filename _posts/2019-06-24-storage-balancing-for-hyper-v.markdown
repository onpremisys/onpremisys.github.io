---
layout: post
title:  "Storage balancing for Hyper-V"
date:   2019-06-24
author: broovers
image: assets/images/heroes/balance.jpg
---
Although Hyper-V works flawlessly for our needs. We are still missing out on some features which VMWare has had since 2011. One of which is [Storage DRS](https://kb.vmware.com/s/article/2149938). It's a feature that allows you to balance the storage usage over several datastores.

With Storage DRS missing we need to manually move VM storage to lesser occupied cluster shared volumes. That's something I would rather automate. Luckily [Yusuf Ozturk](http://www.yusufozturk.info/windows-server/hyper-v-storage-drs-start-vmstorageoptimization.html) made a very nice Powershell script to do exactly that. The script was somewhat outdated, but still elegant. With some minor editing on my side I got it to run on our Windows Server 2016 Hyper-V cluster and had immediate success.

![Hyper-V storage balancing result graph](/assets/images/hyper-v-storage-balancing-result-graph.png)

As you can see in the graph the lines are slowly coming to the same level once the script nears it's completion. I would love for Microsoft to add this natively to fail-over cluster manager. But for now this script will suffice.

Here is the complete script:

{% highlight powershell %}
function Start-VMStorageOptimization {
 
<#
    .SYNOPSIS
 
        Function to optimize storage usage on CSV volumes by using Hyper-V Live Storage Migration.
 
    .DESCRIPTION
 
        If you use dynamic virtual disks with VMs, you may need to monitor free space on your CSV volumes.
	This script moves virtual machines into different CSV volumes by using Hyper-V Live Storage Migration.
	Note: run this as Administrator
 
    .PARAMETER  WhatIf
 
        Display what would happen if you would run the function with given parameters.
 
    .PARAMETER  Confirm
 
        Prompts for confirmation for each operation. Allow user to specify Yes/No to all option to stop prompting.
 
    .EXAMPLE
 
        Start-VMStorageOptimization
 
    .EXAMPLE
 
        Start-VMStorageOptimization -AllowPT
 
    .INPUTS
 
        None
 
    .OUTPUTS
 
        None
 
    .NOTES
 
        Author: Yusuf Ozturk (Small updates by Bas Roovers)
        Website: http://www.yusufozturk.info
        Email: ysfozy@gmail.com
        Date created: 08-June-2014
        Last modified: 06-June-2019
        Version: 1.5
 
    .LINK
 
        http://www.yusufozturk.info
        http://twitter.com/yusufozturk
 
#>
 
[CmdletBinding(SupportsShouldProcess = $true)]
param (
 
	# Allow Passthrough Disks
	[Parameter(
		Mandatory = $false,
		HelpMessage = 'Allow Passthrough Disks')]
	[switch]$AllowPT = $false,
 
	# Debug Mode
	[Parameter(
		Mandatory = $false,
		HelpMessage = 'Debug Mode')]
	[switch]$DebugMode = $false
)
	# Enable Debug Mode
	if ($DebugMode)
	{
		$DebugPreference = "Continue"
	}
	else
	{
		$ErrorActionPreference = "silentlycontinue"
	}
 
	# Informational Output
	Write-Host " "
	Write-Host "------------------------------------------" -ForegroundColor Green
	Write-Host " Hyper-V Storage Optimization Script v1.4 " -ForegroundColor Green
	Write-Host "------------------------------------------" -ForegroundColor Green
	Write-Host " "
 
	# Set Storage Status
	$StorageStatus = $True
 
	while ($StorageStatus -eq $True)
	{ 
		# Get Worst CSV Volume
		$WorstVolume = ((Get-ClusterSharedVolume | Select -ExpandProperty SharedVolumeInfo | Select @{label="Name";expression={(($_.FriendlyVolumeName).Split("\"))[-1]}},@{label="FreeSpace";expression={($_ | Select -Expand Partition).FreeSpace}} | Sort FreeSpace -Descending)[-1])
 
		# Worst CSV Volume Information
		$WorstVolumeName = $WorstVolume.Name
		$WorstVolumeFreeSpace = $WorstVolume.FreeSpace
 
		# Get Best CSV Volume
		$BestVolume = ((Get-ClusterSharedVolume | Select -ExpandProperty SharedVolumeInfo | Select @{label="Name";expression={(($_.FriendlyVolumeName).Split("\"))[-1]}},@{label="FreeSpace";expression={($_ | Select -Expand Partition).FreeSpace}} | Sort FreeSpace -Descending)[0])
 
		# Best CSV Volume Information
		$BestVolumeName = $BestVolume.Name
		$BestVolumeFreeSpace = $BestVolume.FreeSpace
 
		# Calculate Space Gap
		[int64]$SpaceGap = [math]::round((([int64]$BestVolumeFreeSpace - [int64]$WorstVolumeFreeSpace) / 1GB), 0)
 
		if ($SpaceGap -lt "100")
		{
			# Informational Output
			Write-Host "Storage is already optimized." -ForegroundColor Green
			Write-Host " "
 
			# Set Storage Status
			$StorageStatus = $False
		}
		else
		{
			# VM Reports
			$VMReports = $Null
 
			# Get Cluster Nodes
			$ClusterNodes = Get-Cluster | Get-ClusterNode
 
			# Informational Output
			Write-Host "Measuring VMs to find the most suitable virtual machine for the migration.. Please wait.." -ForegroundColor Cyan
			Write-Host " "

			if ($AllowPT)
			{
				# Enable Resource Metering
				$EnableMetering = Get-VM -ComputerName $ClusterNodes | Where ConfigurationLocation -like "C:\ClusterStorage\$WorstVolumeName\*" | Enable-VMResourceMetering
 
				# Get Resource Usage
                $VMReports = Get-VM -ComputerName $ClusterNodes | Where ConfigurationLocation -like "C:\ClusterStorage\$WorstVolumeName\*" | Measure-VM
			}
			else
			{
				# Enable Resource Metering
				$EnableMetering = Get-VM -ComputerName $ClusterNodes | Where ConfigurationLocation -like "C:\ClusterStorage\$WorstVolumeName\*" | Where {(($_ | Select -Expand HardDrives).Path -notlike "Disk*") -and (($_ | Select -expand FibreChannelHostBusAdapters) -eq $Null)} | Enable-VMResourceMetering
 
				# Get Resource Usage
				$VMReports = Get-VM -ComputerName $ClusterNodes | Where ConfigurationLocation -like "C:\ClusterStorage\$WorstVolumeName\*" | Where {(($_ | Select -Expand HardDrives).Path -notlike "Disk*") -and (($_ | Select -expand FibreChannelHostBusAdapters) -eq $Null)} | Measure-VM

			}

 
			# Calculate Space Gap Average
			[int64]$SpaceGapAverage = [math]::round(([int64]$SpaceGap / 2), 0)
			[int64]$SpaceGapAverageMb = [int64]$SpaceGapAverage * 1024
 
			# Find Most Suitable VM
			$BestVM = ($VMReports | Where TotalDisk -lt $SpaceGapAverageMb | Sort TotalDisk -Descending)[0]
 
			if ($BestVM)
			{		
				# Best VM Information
				$BestVMName = $BestVM.VMName
				$BestVMHost = $BestVM.ComputerName
 
				# Informational Output
				Write-Host "Moving $BestVMName from $WorstVolumeName to $BestVolumeName by using Hyper-V Storage Live Migration.." -ForegroundColor Yellow
 
				# Move Virtual Machine
				Move-VMStorage -ComputerName "$BestVMHost" -VMName "$BestVMName" -DestinationStoragePath "C:\ClusterStorage\$BestVolumeName\$BestVMName"
 
				# Informational Output
				Write-Host "Done." -ForegroundColor Green
				Write-Host " "
			}
			else
			{
				# Informational Output
				Write-Host "No suitable VM is found.. $WorstVolumeName contains VMs with large spaces." -ForegroundColor Red
				Write-Host " "
 
				# Set Storage Status
				$StorageStatus = $False				
			}
		}
	}
}
{% endhighlight %}
