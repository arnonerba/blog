---
title: "Automatically Updating Stata on Linux with Cron"
modified_date: "2026-01-08"
last_modified_at: "2026-01-08"
---
On Windows and macOS, Stata can be configured to check for updates automatically with the `set update_query` command. However, this feature isn't present on the Linux version of Stata. Also, it doesn't actually update Stata -- it just enables update notifications. Stata will still need to be manually updated by someone with the permission to do so.

If you're running Stata on a standalone Linux server or an HPC cluster, you may be interested in having Stata update itself without any user interaction. This is especially useful if Stata users do not have permission to update the software themselves, as is often the case on shared Linux systems.

We can enable true automatic updates with a cron job and a Stata batch mode hack:
```
0 0 * * 0 echo 'update all' | /usr/local/stata19/stata > /dev/null
```

Adding this line to root's crontab will cause the `update all` command to be run every Sunday at 12am. Standard output is piped to `/dev/null` to prevent cron from sending unnecessary emails.

As always, think carefully before enabling automatic updates for mission-critical pieces of software. However, StataCorp's quality control has historically been pretty good.
