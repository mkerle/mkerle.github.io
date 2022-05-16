---
layout: single
title:  "Windows Event Log using Powershell"
date:   2022-05-16 18:30:00 +1000
toc: true
categories: windows event powershell security
---

Powershell can be used to query event logs.

Can check highlevel information about the Windows event logs using the below command.  Note to view the security logs you must have administrator rights.

{% highlight powershell %}
> Get-WinEvent -ListLog Application,Security,Setup,System
{% endhighlight %}

## Querying the Event Logs

To query the event logs we specify the `LogName` (security, application etc.).  With no filter this command will output all events and there could be many.
To limit the amount returned we can use `Select-Object`.

Query to get the first 10 log entries from Security.
{% highlight powershell %}
> Get-WinEvent -LogName Security | Select-Object -First 10
{% endhighlight %}

## Filtering the Event Logs 
A simple way to filter event logs is by using `Where-Object` cmdlet.  This cmdlet makes use of the special pipeline variable `$_` which is a reference to the output from the previous command in the pipeline.  To query for logon events we can check for Id = 4624.  We can combine this with `Select-Object` to only display the columns we are interested in.

{% highlight powershell %}
> Get-WinEvent -LogName Security | Where-Object { $_.Id -eq "4624" } | Select-Object -Property TimeCreated,Message -First 10
{% endhighlight %}

## Filtering with FilterHashTable

`Get-WinEvent` includes a property called `FilterHashTable` where we can specify a hashtable of filters as an alternative to `Where-Object`.


{% highlight powershell %}
> Get-WinEvent -FilterHashTable @{LogName='Security';StartTime="16/05/2022 19:00:00"; EndTime="16/05/2022 19:10:00"; ID=4624} | Select-Object -Property TimeCreated,Message
{% endhighlight %}

## Microsoft Documentation for Event Data

Microsoft documents the XML format of the Event Data so that we can extend our queries using these additional fields.  While not mentioned
each attribute can be indexed by integer in the order that it appears (0 is the first index).  For example, [event 4624][ms-event-4624] LogonType is index 8.

We can use this knowledge to check for `LogonType = 10` (remote interactive logon).  `Format-List` is used to give a more detailed output of any matching events.

{% highlight powershell %}
> Get-WinEvent -FilterHashTable @{LogName='Security';StartTime="16/05/2022 19:00:00"; EndTime="16/05/2022 19:10:00"; ID=4624} | Where-Object { $_.properties[8] -eq 10 } | Format-List
{% endhighlight %}



[ms-event-4624]: https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4624

