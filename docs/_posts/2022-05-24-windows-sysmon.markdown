---
layout: single
title:  "Windows Sysmon Introduction"
date:   2022-05-24 17:30:00 +1000
toc: true
categories: windows event sysmon security
---

Sysmon is part of the [sysinternals][sysinternals] package provided by Microsoft.  Sysmon can be deployed to a windows enpoint to provide additional logging as a provider under "Applications and Services Logs".

## Events

Sysmon generates events similar to standard windows events but sysmon uses its own event IDs.  The [sysmon][sysmon] page gives a quick overview of the event classification.

Sysmon reports events based on a configuration file that specifies the "configuration entries" and "event filtering entries".  Configuration entries specify the behaviour of sysmon while the event filtering entries specifies the rules to report or filter events.
This configuration file is in XML format.  Sysmon can also include metadata with each event such as IMPHASH that can create a hash based on the imports made by a PE.

## Community Configurations

A community project is available on GitHub known as [sysmon-config][sysmon-config].  This project focuses on detection coverage.

## Event Viewer

Sysmon events can be viewed using the Event Viewer by navigating through `Applications and Services -> Microsoft -> Windows -> Sysmon -> Operational`.

## Viewing Sysmon Events with Powershell

Sysmon events can be viewed by specifying the `-LogName` parameter to `Get-WinEvent`.

{% highlight powershell %}
> Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational"
{% endhighlight %}

To save specifying the long LogName parameter each time a powershell module can be created with a function that acts as a shortcut for the previous command.  To make this a module it must be saved as `psm1` rather than `ps1` used for normal powershell scripts.

{% highlight powershell %}
function Get-SysmonEvent{
    param (
        $eventid,
        $start,
        $end
    )
    $filters = @{LogName = "Microsoft-Windows-Sysmon/Operational"}
    
    if ($eventid -ne $null) {
        $filters.ID = $eventid
    }
    
    if ($start -ne $null) {
        $filters.StartTime = $start
    }

    if ($end -ne $null) {
        $filters.EndTime = $end
    }
    Get-WinEvent -FilterHashtable $filters
}
{% endhighlight %}

To use the module use the `Import-Module` command and specifying the path to the module.

{% highlight powershell %}
> Import-Module C:\PowershellModules\Get-Sysmon.psm1
{% endhighlight %}

Note that updating the module may require the user of `Remove-Module` and then re-importing.

To query all events between a start and end time we can call the function as below:
{% highlight powershell %}
> Get-SysmonEvent $null "24/05/2022 19:50:00" "24/05/2022 20:00:00"
{% endhighlight %}

Another example if we want to get all the DNS query events until a certain end time:

{% highlight powershell %}
> Get-SysmonEvent 22 $null "24/05/2022 20:00:00"
{% endhighlight %}

The same concept described above can be applied so that we get a shortcut to view the "Security" logs using:

{% highlight powershell %}
function Get-SecurityEvent{
    param (
        $eventid,
        $start,
        $end
    )
    $filters = @{LogName = "Security"}
    
    if ($eventid -ne $null) {
        $filters.ID = $eventid
    }
    
    if ($start -ne $null) {
        $filters.StartTime = $start
    }

    if ($end -ne $null) {
        $filters.EndTime = $end
    }
    Get-WinEvent -FilterHashtable $filters
}
{% endhighlight %}

[sysinternals]: https://docs.microsoft.com/en-us/sysinternals
[sysmon]: https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon
[sysmon-config]: https://github.com/Neo23x0/sysmon-config