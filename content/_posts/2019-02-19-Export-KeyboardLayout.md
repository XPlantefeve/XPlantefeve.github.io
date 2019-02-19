---
layout: post
title: Exporting keyboard layouts
date:   2019-02-14
excerpt: Where to find the information when you're asked to "simply" export keyboard layouts.
post: true
tag:
    - Windows 10
    - Keyboard Layout
    - Registry
comments: true
---

Sometimes, the boss enters the room to ask for a simple thing: "If ever you've
got a couple of minutes..." And sometimes, that very simple thing is not as
simple as expected.

It happened to me recently, as I was asked to export a list of the keyboard
layouts: "We're going to migrate computers, we need that list to reinstall
them." OK, it's quite easy. I mean: it's just this, right?

![Screenshot of the keyboard layouts menu](/assets/img/post/KeyboardLayouts.png)

It appears that gathering that info is a bit convoluted.

It starts with the Keyboard Layout\Preload key in the user's registry:

```powershell
PS> Get-ItemProperty 'HKCU:\Keyboard Layout\Preload'


2            : 00000407
3            : d0010407
1            : 0000040c
4            : 00000809
```

Those DWORDs are hexadecimal encoded keyboard layout IDs, the lower word being
the language LCID (Language Code IDentifier). We can translate those to decimal
and get information about the language thanks to .NET:

```powershell
PS> [int]('0x0407')
1031

PS> [Globalization.CultureInfo]::GetCultureInfo(1031) |
        select LCID,Name,NativeName,EnglishName

LCID Name  NativeName            EnglishName
---- ----  ----------            -----------
1031 de-DE Deutsch (Deutschland) German (Germany)
```

From there, either the keyboard layout is the default one for the language, or not. If
it is not, there will be info in the substitute key:

```powershell
PS> Get-ItemProperty 'HKCU:\Keyboard Layout\Substitutes\'


d0010407     : 00090407
0000040c     : 0007080c
00000809     : 00000452
```

If there a substitute, the keyboard layout ID is that substitute. If there's
none, we keep the DWORD from the first step (in our example, all but one German
keyboard have substitutes).

Each keyboard layout has its own key under *Control\Keyboard Layouts* in the
ControlSet. Each key lists the file containing the layout, and may
contain a text description, a display name, or info about where to find it,
and sometimes an ID.

```powershell
gci 'HKLM:\SYSTEM\ControlSet001\Control\Keyboard Layouts\'


    Hive: HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Control\Keyboard Layouts


Name                           Property
----                           --------
00000401                       Layout Display Name : @C:\WINDOWS\system32\input.dll,-5084
                               Layout File         : KBDA1.DLL
                               Layout Text         : Arabic (101)
00000402                       Layout Display Name : @C:\WINDOWS\system32\input.dll,-5053
                               Layout File         : KBDBU.DLL
                               Layout Text         : Bulgarian (Typewriter)
00000404                       Layout Display Name : @C:\WINDOWS\system32\input.dll,-5065
                               Layout File         : KBDUS.DLL
                               Layout Text         : Chinese (Traditional) - US Keyboard
00000405                       Layout Display Name : @C:\WINDOWS\system32\input.dll,-5031
                               Layout File         : KBDCZ.DLL
                               Layout Text         : Czech
00000406                       Layout Display Name : @C:\WINDOWS\system32\input.dll,-5007
                               Layout File         : KBDDA.DLL
                               Layout Text         : Danish
...


PS> Get-ItemProperty 'HKLM:\SYSTEM\ControlSet001\Control\Keyboard Layouts\00090407'


layout text  : Company Qwertz
Layout id    : 737
layout file  : kbdgr.dll

PS> Get-ItemProperty 'HKLM:\SYSTEM\ControlSet001\Control\Keyboard Layouts\00000452'


Layout Display Name : @C:\WINDOWS\system32\input.dll,-5145
Layout File         : KBDUKX.DLL
Layout Text         : United Kingdom Extended
```

For standard keyboard layout, the display name is translated in the current
version language, so the multilingual description is read from a DLL. In
PowerShell, it can be done by p/invoking .Net functions, as explained in
[this StackOverflow answer](https://stackoverflow.com/questions/45953778/how-to-use-powershell-to-extract-data-from-dll-or-exe-files/)

As shown with the *00090407* layout in my last example, some ad-hoc layouts
do not contain that kind of nicety.

That's about it. Now, you just have to get that info for the default user
(that's the one used on the logon screen), and load every user registry hive
to gather everybody's settings. Oh, and choose what to display, but everything
you need should be at your disposal now.

I've had to chase after every little piece of this jigsaw puzzle. Take it brother,
may it serve you well.