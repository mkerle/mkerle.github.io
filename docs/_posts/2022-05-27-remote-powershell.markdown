---
layout: single
title:  "Windows Remote Powershell - WinRM"
date:   2022-05-27 18:00:00 +1000
toc: true
categories: windows remote powershell security winrm administration
---

Powershell can be used to remotely login to a windows machine to obtain a remote Powershell session also known as WinRM (Windows Remote Management).

## Remote Login

To remotely login to a windows machine use the `Enter-PSSession` cmdlet.  Depending on the security settings of the remote machine you may need to ensure that you are connected to the same domain and the user has enough privileges to remotely connect to the target machine (more info below).

Note that using the domain name of the remote machine can help avoid some kerberos issues depending on where\what your souce machine is.

{% highlight powershell %}
> Enter-PSSession win10box.some.domain -Credential Administrator -Authentication Negotiate
{% endhighlight %}

## Allowing WinRM on a Domain Joined Windows Box

If the target machine is joined to a domain, you will need to ensure that the WinRM service is enabled, and firewall will allow external connection to the WinRM port (TCP/5985 or TCP/5986).

The [instructions here][enable-winrm-gpo] explain how to setup a GPO for a domain joined machine to allow WinRM.

In some cases it may also be required to whitelist an IP or host to allow WinRM by updating the trusted hosts.

See the current list of trusted hosts:

{% highlight powershell %}
> Get-Item WSMAN:\Localhost\Client\TrustedHosts
{% endhighlight %}

[Set trusted hosts][set-trusted-hosts]  with a specific IP:
{% highlight powershell %}
> Set-Item WSMan:\localhost\Client\TrustedHosts -Value 192.168.1.10
{% endhighlight %}


[enable-winrm-gpo]: https://support.auvik.com/hc/en-us/articles/204424994-How-to-enable-WinRM-with-domain-controller-Group-Policy-for-WMI-monitoring
[set-trusted-hosts]: https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_remote_troubleshooting?view=powershell-7.2