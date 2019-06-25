---
layout: post
title:  "Fail-over SQL Server availability groups via Powershell"
date:   2019-06-25
author: broovers
image: assets/images/heroes/always-on.png
---
Using SQL Server's availability groups is a bliss for companies who want to provide redundancy. But things can get pretty tedious when you are not using an Enterprise license and have a lot of databases.

Let me explain. When not using an Enterprise license of SQL Server you need to create an availability group for every database. They are calling this feature "Basic Availability Group". Every availability group needs a listener on it's own ip address which means firewall rules also need to be added in order for clients to access this database over the listener. Although these steps are only needed in the setup phase, there is more to come.

When you need to fail-over in case of maintenance, you need to set all availability groups in synchronous commit. This makes sure no data is lost during the fail-over. When this is done you can safely fail-over each and every one availability group. These steps will take you a lot of time and there is a high chance you forgotting something, believe me I've been there.

I've written a Powershell script for this exact scenario. It will probably save you a lot of time, even if you only have 3 availability groups.

Here is the complete script:

{% highlight powershell %}
#Requires -RunAsAdministrator

Param
(
    [Parameter(Mandatory=$true)][string]$primarySqlInstance,
    [Parameter(Mandatory=$true)][string]$secondarySqlInstance
)

$availabilityGroups = Invoke-Command -ComputerName $primarySqlInstance {
    [System.Reflection.Assembly]::LoadWithPartialName("Microsoft.SqlServer.SMO") | Out-Null
    $SqlServer = New-Object Microsoft.SqlServer.Management.Smo.Server
    return $SqlServer.AvailabilityGroups | Select-Object Name, PrimaryReplicaServerName
}  -ErrorAction Stop

$Title = "SQL Fail-Over Tool"
$Info += $availabilityGroups | Select-Object Name,PrimaryReplicaServerName | Out-String

$options = [System.Management.Automation.Host.ChoiceDescription[]] @("Fail-&over to $secondarySqlInstance", "Fail-&back to $primarySqlInstance", "&Take blue pill")
[int]$defaultchoice = 2
$opt = $host.UI.PromptForChoice($Title , $Info , $Options,$defaultchoice)
switch($opt)
{

0 { 
    "You choose to fail over to $secondarySqlInstance"
    # Establish high availability
    Invoke-Command -ComputerName $primarySqlInstance -ArgumentList $primarySqlInstance,$secondarySqlInstance,$availabilityGroups.Name -ScriptBlock {
        $primarySqlInstance = $args[0] -replace "\..*"
        $secondarySqlInstance = $args[1] -replace "\..*"
        $agGroups = $args[2]

        foreach ($agName in $agGroups){
            "Setting availability group $agName on replica $primarySqlInstance to synchronous commit"
            Set-SqlAvailabilityReplica -AvailabilityMode "SynchronousCommit" -FailoverMode "Manual" -Path SQLSERVER:\Sql\$primarySqlInstance\DEFAULT\AvailabilityGroups\$agName\AvailabilityReplicas\$primarySqlInstance | Out-Null
            "Setting availability group $agName on replica $secondarySqlInstance to synchronous commit"
            Set-SqlAvailabilityReplica -AvailabilityMode "SynchronousCommit" -FailoverMode "Manual" -Path SQLSERVER:\Sql\$primarySqlInstance\DEFAULT\AvailabilityGroups\$agName\AvailabilityReplicas\$secondarySqlInstance | Out-Null
        }
    } -ErrorAction Stop

    # Fail over availability group without data loss
    Invoke-Command -ComputerName $secondarySqlInstance -ArgumentList $secondarySqlInstance,$availabilityGroups.Name -ScriptBlock {
        $secondarySqlInstance = $args[0] -replace "\..*"
        $agGroups = $args[1]

        foreach ($agName in $agGroups){
            "Switch availability group $agName primary replica to $secondarySqlInstance"
            Switch-SqlAvailabilityGroup -Path "SQLSERVER:\Sql\$secondarySqlInstance\DEFAULT\AvailabilityGroups\$agName"
        }
    } -ErrorAction Stop

}

1 {
    "You choose to fail back to $primarySqlInstance"
    # Fail back availability group without data loss
    Invoke-Command -ComputerName $primarySqlInstance -ArgumentList $primarySqlInstance,$availabilityGroups.Name -ScriptBlock {
        $primarySqlInstance = $args[0] -replace "\..*"
        $agGroups = $args[1]

        foreach ($agName in $agGroups){
            "Switch availability group $agName primary replica to $primarySqlInstance"
            Switch-SqlAvailabilityGroup -Path "SQLSERVER:\Sql\$primarySqlInstance\DEFAULT\AvailabilityGroups\$agName"
        }
    } -ErrorAction Stop

    # Remove latency induced outage
    Invoke-Command -ComputerName $primarySqlInstance -ArgumentList $primarySqlInstance,$secondarySqlInstance,$availabilityGroups.Name -ScriptBlock {
        $primarySqlInstance = $args[0] -replace "\..*"
        $secondarySqlInstance = $args[1] -replace "\..*"
        $agGroups = $args[2]

        foreach ($agName in $agGroups){
            "Setting availability group $agName on replica $primarySqlInstance to asynchronous commit"
            Set-SqlAvailabilityReplica -AvailabilityMode "AsynchronousCommit" -FailoverMode "Manual" -Path SQLSERVER:\Sql\$primarySqlInstance\DEFAULT\AvailabilityGroups\$agName\AvailabilityReplicas\$primarySqlInstance | Out-Null
            "Setting availability group $agName on replica $secondarySqlInstance to asynchronous commit"
            Set-SqlAvailabilityReplica -AvailabilityMode "AsynchronousCommit" -FailoverMode "Manual" -Path SQLSERVER:\Sql\$primarySqlInstance\DEFAULT\AvailabilityGroups\$agName\AvailabilityReplicas\$secondarySqlInstance | Out-Null
        }
    } -ErrorAction Stop

}
2 {Write-Host "You took the blue pill, the story ends. You wake up in your bed and believe whatever you want to believe." -ForegroundColor Green}
}
{% endhighlight %}
