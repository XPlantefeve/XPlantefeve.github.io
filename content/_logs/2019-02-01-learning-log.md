---
layout: post
title:  "01 Feb. 2019"
date:   2019-02-01
excerpt: "More WMI and .NET."
log: true
tag:
- PowerShell
- WMI
- dotnet
comments: true
---

# 2019-02-01

- Powershell.org forums [don't help me more](https://powershell.org/forums/topic/silly-question-about-inputs-outputs-comments-based-help-format/) than Reddit.
- Several ways of distinguishing Desktops and Laptops:
  - [Win32_SystemEnclosure.ChassisTypes](https://blogs.technet.microsoft.com/heyscriptingguy/2004/09/21/how-can-i-determine-if-a-computer-is-a-laptop-or-a-desktop-machine/)
  - [Win32_SystemComputer.PCSystemType](http://stonywall.com/2017/05/01/wmi-filtering-based-on-computer-type-desktop-laptop/)
  - [Win32_Battery](http://woshub.com/sccm-and-wmi-query-to-find-all-laptops-and-desktops/)
- [PKGDat/PKGX files](https://social.technet.microsoft.com/wiki/contents/articles/17235.description-of-ue-v-files-stored-in-the-settings-storage-path.aspx)
- System.Security.Principal.WindowsIdentity
  - ::GetCurrent() : info about current user
  - ::New($username) : info about another user
- [Win32_UserAccount](https://docs.microsoft.com/en-gb/windows/desktop/CIMWin32Prov/win32-useraccount) reads every user in the domain: awfully slow
  - You can Trick the machine with New-CIMInstance -Clientonly -Key
- Win32_SystemUsers (still slow)
- No CIM cmdlet equivalent of Get-WMIObject (shame)
-There are WMI classes that cannot be enumerated: Win32_SID
- SID/Username translation:
  - `$NTA =New-Object Security.Principal.NTAccount -ArgumentList 'DOMAIN\User'`
  - `$NTA.Translate([Security.Principal.SecurityIdentifier])`
  - $SecID = New-Object Security.Principal.SecurityIdentifier -Arg '<SID>'
  - $SecID.Translate([Security.Principal.NTAccount])
- Win32_UserProfile : profiles saved locally, with SID
- `[DirectoryServices.AccountManagement.Principal]::FindByIdentity()`

There are several types in AccountManagement (user, computer, group, etc.), but *Principal* has a FindByIdentity that works for everything. It even works for local accounts,  without having to switch from *Domain* to *Machine* context.

```powershell
Add-Type -AssemblyName System.DirectoryServices.AccountManagement
$context = [DirectoryServices.AccountManagement.PrincipalContext]::new('Domain')
ciminstance Win32_UserProfile | % {
    [DirectoryServices.AccountManagement.Principal]::FindByIdentity($context,$_.SID)
}
```

Dug through Chocolatey doc and code to find how to install to non-default paths:
`choco install ruby --params "/InstallDir:c:\stuff\ruby"`

Installed Ruby, discovered Gem, thanks to searching how to create a github.io page with Jekyll. Got me a blank page at first, that's what you get for trying to use a Gem not on github, apparently. Found my way through Jekyll thanks to [the beautiful Moon theme](https://taylantatli.github.io/Moon/), by [Taylan Tatlı](https://taylantatli.com/)