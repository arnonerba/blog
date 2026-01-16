---
title: "Finding the Serial Number of a Windows PC With PowerShell"
---
It's now possible to locate your PC's serial number in the Settings app, but nothing beats a simple command-line query.

## PowerShell Method (Modern)

Since [the WMIC utility is now deprecated](https://techcommunity.microsoft.com/blog/windows-itpro-blog/wmi-command-line-wmic-utility-deprecation-next-steps/4039242), the modern way to perform this query is with [the `Get-CimInstance` cmdlet](https://learn.microsoft.com/en-us/powershell/module/cimcmdlets/get-ciminstance) in PowerShell:
```powershell
Get-CimInstance -ClassName Win32_BIOS | Select-Object SerialNumber
```

That's the right way to do it if you're running this query in a script. However, if you're doing this by hand, that's a lot to remember and type, so I'd recommend using an abbreviated version instead:
```powershell
(gcim Win32_BIOS).SerialNumber
```

You can also use the following version, which returns more than just the serial number but is even easier to remember:
```powershell
gcim win32_bios
```

**Note:** Some sources suggest using the `Get-WmiObject` cmdlet, but [that's deprecated too](https://learn.microsoft.com/en-us/powershell/scripting/learn/ps101/07-working-with-wmi).

## WMIC Method (Legacy)

This is hardcoded into my brain from years of manual Windows deployments, so even though it no longer works on recent Windows installations, I'll include it here for posterity:
```
wmic bios get serialnumber
```

**Remember:** [wmic.exe is deprecated and will eventually stop working](https://stackoverflow.com/q/57121875).
