---
layout: single
title:  "Linux auditctl Logging"
date:   2022-09-06 18:00:00 +1000
toc: true
categories: linux logging auditctl alerting aureport
---

We can configure the linux audit daemon to trigger on certain events.

## File Modifications Monitoring

`auditctl` can be used to trigger log entries if files have been modified.  An example command to achieve this is shown below. The `-w` switch is to watch the file, `-p` is the change operation with `wa` looking for write and attribute changes.  The `-k` switch is to specify a keyword to be used in log entries.

{% highlight bash %}
sudo auditctl -w /home/user/.bashrc -p wa -k privesc
{% endhighlight %}

To apply the changes and validate run:

{% highlight bash %}
sudo auditctl -l
{% endhighlight %}

Specifying rules in this way are not persistant across reboots.  To make permenant changes the rules need to be added to `/etc/audit/rules.d/audit.rules`.

To view log entries that have been trigged by the conditions use:

{% highlight bash %}
sudo auditctl -k
{% endhighlight %}

It should be noted that the id for users are "auid" and may not directly map to the "uid" on the system.  The `audit.log` can be used check which uid was assigned to a pariticular "auid".

## Logging Set-UID (SUID) Processes

The SUID bit can be abused to gain root privileges on a system.  To monitor instances of SUID processes we can use auditctl as below.

{% highlight bash %}
sudo auditctl -a exit,always -F arch=b64 -F euid=0 -S execve -k root_cmds
sudo auditctl -a exit,always -F arch=b32 -F euid=0 -S execve -k root_cmds
{% endhighlight %}

The `-a` flag indicates "[list,action]" or "[action,list"] (can be either order).  The `-F` flag is used to indicate a field, operation, value rule.  The `-S` specifies a syscall name or number (in this case the [execute program][execve-man-page] syscall).  Refer to the [auditctl][auditctl-man-page] for the possible values.

## Using ausearch

There is a tool called [ausearch][ausearch-man-page] that allows searching of the audit logs.

An example of this command is below:

{% highlight bash %}
sudo ausearch -k root_cmds -i -x bash
{% endhighlight %}



[auditctl-man-page]: https://man7.org/linux/man-pages/man8/auditctl.8.html
[ausearch-man-page]: https://man7.org/linux/man-pages/man8/ausearch.8.html
[execve-man-page]: https://man7.org/linux/man-pages/man2/execve.2.html