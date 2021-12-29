---
layout: post
title:  "Rejoin quarantined fail-over cluster nodes"
date:   2021-12-29
author: broovers
image: assets/images/heroes/barb-wire.jpg
---

If you ever encounter a flapping fail-over cluster you might end up with one or more quarantined cluster nodes.

Quarantined nodes are drained from their roles and are not allowed to rejoin the cluster for a set amount.
You can find out the duration and threshold using this command:
{% highlight powershell %}
Get-Cluster | Format-List -Property Quarantine*
{% endhighlight %}

In practise though, you might find your fail-over cluster with quarantined nodes outside of these boundaries.

![Fail-over cluster green quarantined node](\assets\images\fail-over-cluster-green-quarantined-node.png)

You can use the following Powershell script to find quarantined nodes and rejoin them.

{% highlight powershell %}
Write-Host "Find quarantined nodes: " -NoNewline
$QuarantinedClusterNodes = Get-ClusterNode | Where-Object StatusInformation -eq Quarantined

if($QuarantinedClusterNodes){
    Write-Host $QuarantinedClusterNodes.count
    foreach($QuarantinedClusterNode in $QuarantinedClusterNodes){
        Write-Host "@($QuarantinedClusterNode.Name)"
        Write-Host "Stopping cluster node: " -NoNewline
        $QuarantinedClusterNode | Stop-ClusterNode
        Write-Host "V"
        Write-Host "Clear quarantine and start cluster node" -NoNewline
        $QuarantinedClusterNode | Start-ClusterNode -ClearQuarantine
        Write-Host "V"
    }
} else {
    Write-Host "No quarantined nodes found"
}

{% endhighlight %}