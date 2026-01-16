---
title: "Finding the Serial Number of a Windows PC from the Command Line"
---
On Windows 11, it's usually possible to locate your PC's serial number in the Settings app, but nothing beats a simple command-line query.

## PowerShell Method (Modern)

Since [the WMIC utility is officially deprecated](https://techcommunity.microsoft.com/blog/windows-itpro-blog/wmi-command-line-wmic-utility-deprecation-next-steps/4039242), the modern way to perform this query is with [the `Get-CimInstance` cmdlet](https://learn.microsoft.com/en-us/powershell/module/cimcmdlets/get-ciminstance) in PowerShell:
```powershell
Get-CimInstance -ClassName Win32_BIOS | Select-Object SerialNumber
```

That's great if you're performing this query from a script. However, if you're doing this by hand, that's a lot to remember and type, so I'd recommend using the abbreviated version instead:
```powershell
(gcim win32_bios).serialnumber
```

You can also use `gcim win32_bios`, which returns a few extra BIOS properties besides the serial number, but is even easier to remember.

**Note:** Several sources online recommend using the `Get-WmiObject` cmdlet, but [that's deprecated too](https://learn.microsoft.com/en-us/powershell/scripting/learn/ps101/07-working-with-wmi).

## WMIC Method (Legacy)

This is hardcoded into my brain from years of manual Windows deployments, so I'll include it here for posterity:
```
wmic bios get serialnumber
```

**Remember:** [wmic.exe is deprecated, and will eventually stop working](https://stackoverflow.com/q/57121875).
