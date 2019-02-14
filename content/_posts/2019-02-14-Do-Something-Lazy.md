---
layout: post
title: Do-Something, the very lazy way
date:   2019-02-14
excerpt: GUI is lazy, typing is lazier, typing less is the laziest
log: true
tag:
    - Powershell
    - Alias
comments: true
---

This one is more of an exercise, as in "can it be done?". A few days ago, I was testing
a service. I had to stop it, start it, stop it, start it, and so on. Automation is all
about cutting down repetitive tasks. And it appeared to me that typing the following
was very repetitive:

```powershell
Get-Service SvcName | Stop-Service
Get-Service SvcName | Start-Service
```

(For some reason, that's close to what I had to type, rather that `Stop-Service SvcName` and
`Start-Service SvcName`)

I wondered: can I cut down the typing? After all, as once said Saint Jeffrey Snover:

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">The secret is that the best engineers are lazy.  They love automation &amp; will spend days to automate something. <a href="https://t.co/X1Ab8pNURk">https://t.co/X1Ab8pNURk</a></p>&mdash; Jeffrey Snover (@jsnover) <a href="https://twitter.com/jsnover/status/622025108400398336?ref_src=twsrc%5Etfw">July 17, 2015</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Ok, spending days on that would have been a bit much, but it stayed somewhere on my mind.
I had an idea about how to do it, but didn't see how to use the pipeline for a cmdlet you
had the name of in a string. Then while falling asleep yesterday: it dawned to me: *alias*...

Suppose you have, say, a service. It has a type:

```powershell
PS> $svc = Get-Service SvcName
PS> $svc.GetType()
IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     False    ServiceController                        System.ComponentModel.Component
```

You can get commands by parameter type:
```powershell
PS> Get-Command -ParameterType $svc.GetType()

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Cmdlet          Get-Service                                        3.1.0.0    Microsoft.PowerShell.Management
Cmdlet          Restart-Service                                    3.1.0.0    Microsoft.PowerShell.Management
Cmdlet          Resume-Service                                     3.1.0.0    Microsoft.PowerShell.Management
Cmdlet          Set-Service                                        3.1.0.0    Microsoft.PowerShell.Management
Cmdlet          Start-Service                                      3.1.0.0    Microsoft.PowerShell.Management
Cmdlet          Stop-Service                                       3.1.0.0    Microsoft.PowerShell.Management
Cmdlet          Suspend-Service                                    3.1.0.0    Microsoft.PowerShell.Management
```
You can get commands by verb, too:

```powershell
PS> Get-Command -Verb stop

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Function        Stop-BgpPeer                                       3.0.0.0    RemoteAccess
Function        Stop-DscConfiguration                              1.1        PSDesiredStateConfiguration
Function        Stop-Dtc                                           1.0.0.0    MsDtc
Function        Stop-DtcTransactionsTraceSession                   1.0.0.0    MsDtc
...
```

Now suppose you write a function, called Stop-Something. In it, you get the type of objects sent
in the pipeline, find the command whose verb is 'Stop' that accepts that particular type. Stop-Something is
no shorter to type than Stop-Service, so you just create a *Stop* alias:

```powershell
Get-Service SvcName | Stop
Get-Process Notepad | Stop
```

Nice. All that's left to do is to create a function for *Stop*, one for *Restart*, etc. Wait, they would
all be the same, but for the verb used. And another thing you can do, is to get the verb of your
current function:

```powershell
PS> Function Get-Lazy {
>>   $MyInvocation.MyCommand.Verb
>> }
PS> Get-Lazy
Get
```

Now you can have one single scriptblock with the function definition, and create those on the fly.
I had to take into account the fact that some verbs are well-known aliases or reserved words, and because I
just wrote it very quickly, I don't really handle the case where there would be several commands that fit, but here is
a result (Oh, BTW, did you know that *Something* is always an alias for *Get-Something*?):

```powershell
PS> service BITS | restart
PS> module MyTools | remove
PS> job 5 | stop
PS> ps | ? responding -EQ $false | stop # it handles the pipeline, too.
```

I put the whole thing in a gist:

<script src="https://gist.github.com/XPlantefeve/9f1090df3b4446e2f22215eb2730a837.js"></script>
