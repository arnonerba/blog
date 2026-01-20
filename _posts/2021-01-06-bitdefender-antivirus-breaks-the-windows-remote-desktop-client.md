---
title: "Bitdefender Antivirus Breaks RDP (Remote Desktop) on Windows"
modified_date: "2026-01-07"
last_modified_at: "2026-01-07"
---
**Update:** This may have been fixed by now, but back in 2021, [the free edition of Bitdefender Antivirus](https://www.bitdefender.com/solutions/free.html) was interfering with Remote Desktop Protocol (RDP) connections on Windows. The remainder of this post has been preserved for posterity.

---

Affected users on Windows endpoints with Bitdefender Antivirus installed may receive the following error when they try to log on to a remote PC or server with Network Level Authentication (NLA) enabled:
```
An authentication error has occurred.
The Local Security Authority cannot be contacted.
This could be due to an expired password.
```

While an expired password or a server-side misconfiguration can cause this error, it may also indicate a client-side issue. In this case, the error appears to be caused by Bitdefender Antivirus replacing the remote computer's certificate in order to inspect encrypted RDP traffic. This process breaks Network Level Authentication and causes the connection to fail.

## Solution: Add File-Level Exclusions for `mstsc`

One workaround is to add file-level exclusions in Bitdefender for both the 64-bit and 32-bit versions of the Windows RDP client:
- `C:\Windows\system32\mstsc.exe`
- `C:\Windows\syswow64\mstsc.exe`

This is not an ideal solution, but the free version of Bitdefender Antivirus has a limited control panel and does not provide alternative workarounds.

---

**References:**
- [Bitdefender Antivirus Free Edition breaking RDP : BitDefender](https://www.reddit.com/r/BitDefender/comments/hqmcqu/bitdefender_antivirus_free_edition_breaking_rdp/)
- [Remote Desktop Connection Issue - Microsoft Community](https://answers.microsoft.com/en-us/windows/forum/windows_10-networking/remote-desktop-connection-issue-solved/8e382bf1-b26a-4730-b9d0-c8a166fa33a3)
- [Bitdefender Antivirus Free - remote Desktop block - The Bitdefender Expert Community](https://community.bitdefender.com/en/discussion/82563/bitdefender-antivirus-free-remote-desktop-block)
- [AWS ec2 windows login error - Stack Overflow](https://stackoverflow.com/questions/62531042/aws-ec2-windows-login-error-saying-an-authentication-error-has-occured-the-loca)

