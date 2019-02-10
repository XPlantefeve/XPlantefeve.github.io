---
layout: post
title: Create Scheduled Tasks on an event with PowerShell
date:   2019-02-03
excerpt: A look at how to create a scheduled task that triggers on an event, the PowerShell way.
log: true
tag:
    - CIM
    - Scheduled Task
    - Event
comments: true
---

I wrote a small script that I needed to run every time my computer was
connecting to a network. A quick search told me that each connection triggers
an event of ID 10000 in the operational event log for NetworkProfile.

I've done that before: you launch the event log viewer, find the event,
right-click, and choose "Attach task". But because I've done it before,
I now wanted to do it on the command line. A quick `Get-Command *sched*` tells
me that there is a whole module dedicated to scheduled tasks. Of interest to me
right now are the `Get-ScheduledTask` cmdlet, that will allow me to analyse
the existing tasks, and `New-ScheduledTaskTrigger`, as it's the one I'll
have to understand.

I launch `Get-ScheduledTask`, see that it gives me what I want, isolate
an "on event" task, and see that it has a *triggers* property, a collection.
Let's have a look at the first one:

```powershell
PS C:\> $task = Get-ScheduledTask | ? taskpath -match 'event' | select -first 1
PS C:\> $trigger = $task.Triggers | select -first 1
PS C:\> $trigger
```

I recognize a task I created recently that triggers on a DHCP event:

```powershell
Enabled            : True
EndBoundary        :
ExecutionTimeLimit :
Id                 :
Repetition         : MSFT_TaskRepetitionPattern
StartBoundary      :
Delay              :
Subscription       : <QueryList><Query Id="0" Path="Microsoft-Windows-Dhcp-Client/Admin"><Select
                     Path="Microsoft-Windows-Dhcp-Client/Admin">*[System[Provider[@Name='Microsoft-Windows-Dhcp-Client'] and
                     EventID=1001]]</Select></Query></QueryList>
ValueQueries       :
PSComputerName     :
```

Nice. But when I look at the help for New-ScheduledTaskTrigger, I can see
switches like -Daily, -AtStartup, -AtLogon, etc. Nothing for events. A quick
search shows me that I'm not the first one hitting that wall, and the solutions
range from doing it in the GUI to using *schtasks.exe*, or getting objects out
of an existing task. Really, there's no cleaner solution? What's a trigger,
anyway?

```powershell
PS C:\> $trigger.GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     CimInstance                              System.Object
```

Oh, that's a CIM object. Interesting, let's have a deeper look at the type:

```powershell
PS C:\> $trigger.psobject.TypeNames

Microsoft.Management.Infrastructure.CimInstance#Root/Microsoft/Windows/TaskScheduler/MSFT_TaskEventTrigger
Microsoft.Management.Infrastructure.CimInstance#ROOT/Microsoft/Windows/TaskScheduler/MSFT_TaskTrigger
Microsoft.Management.Infrastructure.CimInstance#MSFT_TaskEventTrigger
Microsoft.Management.Infrastructure.CimInstance#MSFT_TaskTrigger
Microsoft.Management.Infrastructure.CimInstance
System.Object
```

Now that is great, we have a complete CIM path! Let's try to create an instance:

```powershell
PS C:\> $class = cimclass MSFT_TaskEventTrigger root/Microsoft/Windows/TaskScheduler
PS C:\> $trigger = $class | New-CimInstance -ClientOnly
PS C:\> $trigger

Enabled            :
EndBoundary        :
ExecutionTimeLimit :
Id                 :
Repetition         :
StartBoundary      :
Delay              :
Subscription       :
ValueQueries       :
PSComputerName     :
```

It looks exactly like what we need. I won't detail it, but I had a look at
the *MSFT_TaskRepetitionPattern* in the example task, and it apparently was
nothing but the default values, so I'll forget about it. What I need to do
is to enable the trigger, and give it a definition.

Once again, I turn to the example, and it doesn't look extremely
complicated: the path is written a couple of times, then we have the
provider and the event ID. I'll just replace those with the values I need:

```powershell
PS C:\> $trigger.Enabled = $true
PS C:\> $trigger.Subscription = '<QueryList><Query Id="0" `
Path="Microsoft-Windows-NetworkProfile/Operational"><Select `
Path="Microsoft-Windows-NetworkProfile/Operational">`
*[System[Provider[@Name=''Microsoft-Windows-NetworkProfile''] `
and EventID=10000]]</Select></Query></QueryList>'
```

And voilà, if I have another look at my trigger, it looks exactly as I wanted
it to be.

(Note: I broke it down on several lines for this post, but did it
in a single one in real life. I even tried to format the XML and it
works OK, but then the GUI sees it as a "custom trigger" and it cannot be
easily edited if someone else doesn't want to do it our way.)

Let's finish the job, the rest is pretty straightforward:

```powershell
$ActionParameters = @{
    Execute  = 'C:\Windows\system32\WindowsPowerShell\v1.0\powershell.exe'
    Argument = '-NoProfile -File C:\scripts\NetworkConnectionCheck.ps1'
}

$Action = New-ScheduledTaskAction @ActionParameters
$Principal = New-ScheduledTaskPrincipal -UserId 'NT AUTHORITY\SYSTEM' -LogonType ServiceAccount
$Settings = New-ScheduledTaskSettingsSet

$RegSchTaskParameters = @{
    TaskName    = 'Network checks'
    Description = 'runs at network connection'
    TaskPath    = '\Event Viewer Tasks\'
    Action      = $Action
    Principal   = $Principal
    Settings    = $Settings
    Trigger     = $Trigger
}

Register-ScheduledTask @RegSchTaskParameters
```

Here we are, a 100% PowerShell solution.