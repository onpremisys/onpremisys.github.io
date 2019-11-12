---
layout: post
title:  "Domain join through read-only domain controller"
date:   2019-11-12
author: broovers
image: assets/images/heroes/businessman-stop-dominoes.jpg
---

It is known that you should not have a domain controller in your DMZ. But this leaves the Windows servers in a workgroup. Which is more difficult to manage because you lack group policies. Fortunately there is a way by running a read-only domain controller.

Running an RODC is quite simple. You install the server like any other domain controller except you mark it as read-only during the installation.

The difficult part is joining Windows servers to the domain. You basically have 2 options:
1. Join the Windows server while it is outside your DMZ so it has full access to a writeable domain controller.
2. Prepare a computer object on a writeable domain controller and join the domain through an RODC.

The second option is the one we are discussing here. Mainly because you might not always have the option to start of your Windows server in any other network than DMZ.

## Step 1 - Prepare computer object
{% highlight powershell %}
#requires -RunAsAdministrator

Param
(
    [Parameter(Mandatory=$true)][string]$serverName,
    [Parameter(Mandatory=$true)][string]$ouPath,
    [Parameter(Mandatory=$true)][string]$fqdnRODC,
    [Parameter(Mandatory=$true)][string]$fqdnRWDC,
    [Parameter(Mandatory=$true)][Security.SecureString]$computerPassword,
    [Parameter(Mandatory=$true)][string]$rodcWhitelist
)

If ($null -ne (Get-WmiObject -class win32_optionalfeature | Where-Object { $_.Name -eq 'RemoteServerAdministrationTools'})) {
    Throw "RSAT not installed."
}

## Preparing variables
$cred = Get-Credential
$distinguishedServerName = "CN=" + $serverName + "," +$ouPath
$SecurePassword=ConvertTo-SecureString $computerPassword -asplaintext -force

## Create new computer object
New-ADComputer -Name $ServerName -SamAccountName $ServerName -Path $ouPath -Credential $cred -Server $fqdnRWDC

## Set temporary password on new computer object
Set-ADAccountPassword -Identity $distinguishedServerName -Credential $cred -NewPassword $SecurePassword -Server $fqdnRWDC

## Add new computer object to AD group which is allowed to replicate passwords to read-only domain controller
Add-ADGroupMember -Identity $rodcWhitelist -members $distinguishedServerName -Credential $cred -Server $fqdnRWDC

## Sync new computer object to read-only domain controller
Sync-ADObject -Object $distinguishedServerName -Source $fqdnRWDC -Destination $fqdnRODC -PasswordOnly -PassThru
{% endhighlight %}

## Step 2 - Join domain through RODC
Credits to Jorge from [Jorge's Quest For Knowledge!](https://jorgequestforknowledge.wordpress.com/2017/03/16/domain-join-through-an-rodc-instead-of-an-rwdc-update-2/)
{% highlight powershell %}
#requires -RunAsAdministrator

Param
(
    [Parameter(Mandatory=$true)][string]$serverName,
    [Parameter(Mandatory=$true)][string]$ouPath,
    [Parameter(Mandatory=$true)][string]$fqdnRODC,
    [Parameter(Mandatory=$true)][Security.SecureString]$computerAccountPassword,
    [Parameter(Mandatory=$true)][string]$domainName,
    [Parameter(Mandatory=$true)][string]$dynamicSiteName
)

## Set the DNS suffix search list
Set-DnsClientGlobalSetting -SuffixSearchList @("$domainName")

# Set DynamicSiteName
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\Netlogon\Parameters" -Name "DynamicSiteName" -Value $dynamicSiteName -PropertyType String -Force

## Domain join with RODC
Write-Host "Joining domain" -BackgroundColor Black -ForegroundColor Green
Set-Variable JOIN_DOMAIN -option Constant -value 1                  # Joins a computer to a domain. If this value is not specified, the join is a computer to a workgroup
Set-Variable MACHINE_PASSWORD_PASSED -option Constant -value 128    # The machine, not the user, password passed. This option is only valid for unsecure joins
Set-Variable NETSETUP_JOIN_READONLY -option Constant -value 2048    # Use an RODC to perform the domain join against

$readOnlyDomainJoinOption = $JOIN_DOMAIN + $MACHINE_PASSWORD_PASSED + $NETSETUP_JOIN_READONLY
$localComputerSystem = Get-WMIObject Win32_ComputerSystem
$localComputerSystem.Rename($serverName)
$returnErrorCode = $localComputerSystem.JoinDomainOrWorkGroup($fqdnADdomain+"\"+$fqdnRODC,$computerAccountPassword,$null,$null,$readOnlyDomainJoinOption)

$returnErrorDescription = switch ($($returnErrorCode.ReturnValue)) {
	0 {"SUCCESS: The Operation Completed Successfully."} 
	5 {"FAILURE: Access Is Denied."} 
	53 {"FAILURE: The Network Path Was Not Found."}
	64 {"FAILURE: The Specified Network Name Is No Longer Available."}
	87 {"FAILURE: The Parameter Is Incorrect."} 
	1219 {"FAILURE: Logon Failure: Multiple Credentials In Use For Target Server."}
	1326 {"FAILURE: Logon Failure: Unknown Username Or Bad Password."} 
	1355 {"FAILURE: The Specified Domain Either Does Not Exist Or Could Not Be Contacted."} 
	2691 {"FAILURE: The Machine Is Already Joined To The Domain."} 
	default {"FAILURE: Unknown Error!"}
}
	
If ($($returnErrorCode.ReturnValue) -eq "0") {
	Write-Host "Domain Join Result Code...: $($returnErrorCode.ReturnValue)" "SUCCESS"
	Write-Host "Domain Join Result Text...: $returnErrorDescription" "SUCCESS"
} Else {
	Write-Host "Domain Join Result Code...: $($returnErrorCode.ReturnValue)" "ERROR"
	Write-Host "Domain Join Result Text...: $returnErrorDescription" "ERROR"
}
	
If ($($returnErrorCode.ReturnValue) -eq "0") {
	Write-Host "All went well, please reboot."
}
{% endhighlight %}

