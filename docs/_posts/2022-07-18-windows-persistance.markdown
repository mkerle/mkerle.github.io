---
layout: single
title:  "Windows Persistance"
date:   2022-07-18 18:30:00 +1000
toc: true
categories: windows event sysmon security wmi powershell persistance
---

## Types of Persistance

## Windows Services
-- Look for changes in HKLM\System\CurrentControlSet\Services\, have startup type of 2 (auto) - provides persistance across reboots.

## Scheduled Tasks

{% highlight powershell %}
schtasks /query /tn MicrosoftEdgeUpdateTaskMachineCore
{% endhighlight %}

Scheduled tasks are located in `C:\Windows\System32\Tasks` with a XML file per scheduled tasks containing the task configuration.

Look in Security Evetns for ID = 4698 (A scheduled task was created ...).  These events have the exact XML as used in the `C:\Windows\System32\Tasks` directory.

Look for SYSTEM user creating scheduled tasks as this is not common.

## DLL Sideloading and Hijacking

The process Windows takes to load a DLL is:
1. Check if the DLL has been loaded into memory
2. Check HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\KnownDLLs -> This stores the names and locations of common DLLs
3. If HKLM\System\CurrentControlSet\Control\Session Manager\SafeDllSearchMode is enabled (default):
    1. It will search the current directory where the exectuable was started
    2. Next it will search the system and windows directories
    3. Current directory
    4. Directories in PATH
4. If HKLM\System\CurrentControlSet\Control\Session Manager\SafeDllSearchMode is disabled:
    1. It will search the current directory where the exectuable was started
    2. Search the current directory of the user
    3. Search the system and windows directories
    4. Directories in PATH

The detection of these events can involve the creation of DLL on the disk as well as activity from rundll32.exe.  Sysmon is capable of logging DLL load events (Event ID 7) but due to the large amount of DLL imports that occur on a system this can generate too much noise in logs.

### Sideloading

For DLL sideloading a malicious DLL is placed in the same directory as the application.  [Mitre ATT&CK - DLL Side-Loading][mitre-dll-side-loading]

### Hijacking

Involves placing the DLL in a directory that will be searched before the legitimate DLL.  [Mitre ATT&CK - DLL Hijacking][mitre-dll-hijacking]

## Registry Persistance

A common method of persistance is the use of `Run`, `RunOnceEx` and `RunOnce` registry keys.  These keys are created under current user and local machine hives with the below paths:
- HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
- HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run
- HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunOnce
- HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnce
- HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunOnceEx
- HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnceEx

Applications listed in these keys are exectured when a user logs in.  `RunOnce` is removed after exection so not useful for ongoing persistance.  `RunOnceEx` is similar to `RunOnce` but is not removed until the application has finished execution.

To check what applications are configured to run under these keys we can query the registry with:

{% highlight powershell %}
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run\
{% endhighlight %}

<pre>
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
    SecurityHealth    REG_EXPAND_SZ    %windir%\system32\SecurityHealthSystray.exe
</pre>

The output displays a name, data type and the path.
- REG_EXPAND_SZ -> This enables environment variables to be interpreted.
- REG_SZ -> Human readable string value.

Adding new application to the `Run` registry key can be done by:

{% highlight powershell %}
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" /v RunCmd /t REG_SZ /d "C:\Windows\System32\cmd.exe" /f
{% endhighlight %}

An alternative way to create registry keys is using the `New-ItemProperty` cmdlet in powershell.

We can use sysmon logs to find registry key updates with event ID 13.  The `image` field of these events will indicate the program that was used to create the reg key i.e. reg.exe or powershell.exe.

[Mitre ATT&CK Registry Run Keys][mitre-registry-run]

## Windows Logon

When a user logs on to windows the `WinLogon` process is invoked to handle the login process.  This process will load the user profile, unlock the workstation etc.  The configuration of `WinLogon` is contained in the registry at:

`HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon`

| SubKey    |   Description |
|-----------|---------------|
| Shell      |   This is the executable to use as the default shell for users.  A malicious executable can be loaded along side the default shell. |
| UserInit   |   This executable is run on every login and by default is `userinit.exe`.  This loads fonts, colours, wallpaper etc. |
| Notify    |   Found in Windows 7 and earlier.  This value could be changed to a malicious DLL. |

As an example, to load an additional malicous executable alongside the default shell (explore.exe) we could use the below command to update the registry:

{% highlight powershell %}
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Winlogon" /v Shell /t REG_SZ /d "explorer.exe, C:\somepath\dodgieExe.exe" /f
{% endhighlight %}

On a logon event to the workstation the above command would be executed.  This could be used as a persistance technique.  Searching Sysmon events for process create events (event ID 1), we would find that our `dodgieExe.exe` executed by the `ParentImage` of `userinit.exe`.

[Mitre ATT&CK Logon Autostart Execution][mitre-logon-autostart]

[mitre-dll-side-loading]: https://attack.mitre.org/techniques/T1574/002/
[mitre-dll-hijacking]: https://attack.mitre.org/techniques/T1574/001/
[mitre-registry-run]: https://attack.mitre.org/techniques/T1547/001/
[mitre-logon-autostart]: https://attack.mitre.org/techniques/T1547/004/