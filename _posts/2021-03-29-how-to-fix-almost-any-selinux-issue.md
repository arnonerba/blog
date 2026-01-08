---
title: "Troubleshooting SELinux Issues"
modified_date: "2026-01-07"
last_modified_at: "2026-01-07"
---
SELinux has a well-earned reputation for being hard to use. It's infamous for causing strange, illogical faults that can't be fixed via normal troubleshooting routines, and, as a consequence, many guides and blog posts recommend disabling it outright. However, SELinux is a great way to secure and harden Linux systems, and with a few simple steps it's possible to fix most common problems you might encounter while using it.

## Examples of Common Issues

Let's start by looking at a few issues I've had in the past that turned out to be caused by SELinux:
1. A user could no longer log in with an SSH key after their home directory was restored from a backup. Their `authorized_keys` file was configured correctly but was being ignored by SSH.
2. A service wouldn't start after replacing its config file with a modified version that had been uploaded via SFTP. The service complained about the config file being inaccessible even though its permissions were set correctly.
3. Postfix couldn't communicate with OpenDKIM when the latter was set to use a UNIX socket instead of a TCP/IP socket. The Postfix user was in the correct security group and the socket was configured correctly.

Without a general understanding of how SELinux works, you might guess that the issues above were caused by bad file permissions. That's why it's important to understand SELinux and to identify it as a possible culprit early in the troubleshooting process.

## What is SELinux, Exactly?

At its core, SELinux is a set of rules that tell applications what they can and can't do. SELinux is separate from the regular Linux file permissions model and is therefore able to protect against issues like misconfigured permissions or privilege escalation exploits. In order for an operation to succeed on an SELinux-enabled system, it must be permitted by file permissions as well as by the active SELinux policy.

Regular file permissions are a form of discretionary access control, or DAC. On the other hand, SELinux is a form of mandatory access control, or MAC. With DAC, a user or service can do anything they have permission to do, even if it's something undesirable or dangerous. With MAC, malicious or dangerous actions can be stopped, even if a DAC policy would otherwise permit them to happen.

Here's an example of why you'd want to keep SELinux enabled. Normally, Apache shouldn't be able to read `/etc/shadow`, and the default file permissions prevent that from happening. However, if those permissions were misconfigured and Apache was configured to serve files from `/etc`, it would be possible for anyone with a web browser to download `/etc/shadow`. A properly configured SELinux policy would override both misconfigurations and prevent Apache from serving sensitive system files from `/etc`.

## Putting Things in Context

Extra protection is great, but what happens when SELinux interferes when it shouldn't? If SELinux is interfering with something "normal" that should otherwise work, chances are you have one simple problem: incorrect file security contexts. Security contexts are how SELinux categorizes files and decides which applications can access them. By default, security contexts are applied to files based on their location. For example, files in home directories get different security contexts from files in `/etc` or `/tmp`.

You can inspect a file's security context with `ls -Z`, but you're probably better off using [restorecon](https://linux.die.net/man/8/restorecon) to reset contexts to their default values if you suspect a problem. To save time, you can run `restorecon -rv /path/to/directory` to recursively reset the security contexts for an entire directory. If things are bad enough, you can relabel your entire filesystem by running `touch /.autorelabel` and then rebooting.

The `restorecon` tool was the solution to problems #1 and #2 from the list at the beginning of this post. Incorrect security contexts can be applied when files are restored from a backup or copied from a nonstandard location.

## Adjusting the Policy

In most mainstream Linux distributions, the default SELinux policy is carefully crafted by a group of upstream maintainers. Creating a perfect one-size-fits-all policy is impossible, so the maintainers provide built-in policy exceptions in the form of SELinux booleans. SELinux booleans can be easily enabled or disabled to cover common use cases where the default SELinux policy falls short. If you have an SELinux problem that can't be fixed by restoring default file security contexts, you should check to see if an available SELinux boolean covers your use case.

You can use [getsebool](https://linux.die.net/man/8/getsebool) to retrieve a list of available booleans on your system and then use [setsebool](https://linux.die.net/man/8/setsebool) to enable or disable them. Alternatively, you can use [semanage](https://linux.die.net/man/8/semanage) to see more detailed information about available booleans. Examples of SELinux booleans include:
- `use_nfs_home_dirs`: Support NFS home directories.
- `httpd_can_network_connect`: Allow HTTPD scripts and modules to connect to the network.
- `ftpd_full_access`: Allow full filesystem access over FTP.

## Rewriting the Policy

If fixing security contexts and enabling booleans hasn't worked, ask yourself if you're doing something abnormal. "Abnormal" in this context might include running a service on a nonstandard port, serving web files from an unconventional location, or moving config files out of their default directory. If you are, there's a good chance your system's default SELinux policy won't cover your use case.

Before you proceed, you should think hard about what benefit you're getting from running a nonstandard configuration. Standards exist for good reasons: troubleshooting is easier, malicious activity is simpler to detect, and applications can be configured to behave more predictably. With that said, there's plenty of vendor software out there that relies on an "abnormal" configuration to work properly.

If you've evaluated your configuration and decided to proceed, you have two options. First, you may have discovered a bug in your platform's SELinux policy, which means you should submit a bug report so that the policy can be fixed upstream. This is the course I ended up pursuing for the [OpenDKIM issue](https://bugzilla.redhat.com/show_bug.cgi?id=1563423) mentioned above, and the Red Hat maintainers updated the upstream policy after a few months.

Alternatively, you can write and compile a custom SELinux policy module. This is not as difficult as it sounds, as [audit2allow](https://linux.die.net/man/1/audit2allow) can generate SELinux modules directly from audit log entries. A brief description of how to make use of the audit log is below, but a full explanation is beyond the scope of this post.

## The Audit Log

By default, SELinux violations are logged to the audit log, which is generally located at `/var/log/audit/audit.log`. The best way to troubleshoot potential SELinux issues is to consult the audit log, but the default log format is not particularly user-friendly and raw entries are not always easy to understand. Instead of reading the audit log file directly, you can search the log with [ausearch](https://linux.die.net/man/8/ausearch) or generate comprehensive, human-readable reports from it with [sealert](https://linux.die.net/man/8/sealert). Additional resources for how to use those tools are provided at the bottom of this post.

## Wrapping Up

SELinux has been around for a long time, and many mainstream Linux distributions now ship with robust SELinux policies that cover a range of use cases. Additionally, configuration management tools like Puppet can automatically set SELinux contexts for you and help you avoid inadvertently mislabeling files.

That said, the default SELinux policy can't possibly cover all possible use cases, so you may still need to enable SELinux booleans or compile custom policy modules to make SELinux work for you. In any case, you should avoid disabling it outright, especially if you're running a derivative of Fedora such as RHEL or CentOS where SELinux is intended to be the primary form of mandatory access control.

---

**References:**
- [What is SELinux? - Red Hat](https://www.redhat.com/en/topics/linux/what-is-selinux)
- [HowTos/SELinux - CentOS Wiki](https://wiki.centos.org/HowTos/SELinux)
- [Getting started with SELinux :: Fedora Docs](https://docs.fedoraproject.org/en-US/quick-docs/selinux-getting-started/)
- [Troubleshooting SELinux :: Fedora Docs](https://docs.fedoraproject.org/en-US/quick-docs/selinux-troubleshooting/)
- [Basic SELinux Troubleshooting in CLI - Red Hat Customer Portal](https://access.redhat.com/articles/2191331)
- [Using SELinux - Red Hat Enterprise Linux 8 - Red Hat Customer Portal](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html/using_selinux/index)
