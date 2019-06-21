---
layout: post
title:  "Hyper-V Linux Integration Service for Ubuntu 18.04"
date:   2019-06-21 09:17:21 +0200
categories: jekyll update
author: Bas Roovers
---
Linux Integration Services. Linux Integration Services (LIS) is a package of drivers and services that enhance the performance of Linux-based virtual machines on Hyper-V. It enables features like:
* Graceful Shutdown
* Heartbeat to Hyper-V Manager
* Firewall Management IP Address Visibility
* Hot-add memory

According to the [Microsoft documentation][microsoft-docs] on Ubuntu virtual machines, they state that Linux Integration Service is built into the OS. However this is not the case. So in order to make use of the benefits of LIS, you need to perform some manual steps.

{% highlight bash %}
# Add hv_modules to /etc/initramfs-tools/modules
echo 'hv_vmbus' >> /etc/initramfs-tools/modules
echo 'hv_storvsc' >> /etc/initramfs-tools/modules
echo 'hv_blkvsc' >> /etc/initramfs-tools/modules
echo 'hv_netvsc' >> /etc/initramfs-tools/modules

# Replace Out of Box Kernal with linux-virtual
apt -y install linux-virtual linux-cloud-tools-virtual linux-tools-virtual

# Update Initramfs
update-initramfs -u

# Reboot Server
reboot
{% endhighlight %}

Sources: [Hyper-V Lab](https://hypervlab.co.uk/2019/04/configure-linux-integration-services-on-ubuntu-server-18-04-02-lts/)

[microsoft-docs]: https://docs.microsoft.com/en-us/windows-server/virtualization/hyper-v/supported-ubuntu-virtual-machines-on-hyper-v
