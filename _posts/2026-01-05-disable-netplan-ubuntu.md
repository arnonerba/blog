---
title: "Non-Destructively Disabling Netplan on Ubuntu"
---
Let's talk about disabling [Netplan](https://netplan.io/) on Ubuntu.

First: Why not just leave it alone? Netplan has been part of Ubuntu [for almost a decade now](https://ubuntu.com/blog/a-declarative-approach-to-linux-networking-with-netplan), and it's actually pretty good at its job, which is to provide a unified, human-readable interface for configuring either systemd-networkd (on Ubuntu Server) or NetworkManager (on Ubuntu Desktop). Let's be honest, manually configuring systemd-networkd isn't for the faint of heart, especially for more complex scenarios like bonding.

Here's my problem: I don't need Netplan, because I can configure systemd-networkd directly [with Puppet](https://forge.puppet.com/modules/puppet/systemd) (or possibly [OpenVox](https://voxpupuli.org/openvox/) in the future). Netplan can play nice in this scenario, since it writes to `/run/systemd/network/` instead of `/etc/systemd/network/`, but I don't want it fighting with my configuration management tools over something as critical as network configuration.

So: Why not just purge Netplan with `apt purge netplan.io && apt autoremove`? Well, because doing so breaks the `ubuntu-minimal` metapackage, which, in my opinion, is a great way to shoot yourself in the foot:

> This package [ubuntu-minimal] depends on all of the packages in the Ubuntu minimal system ... It is also used to help ensure proper upgrades, so it is recommended that it not be removed.

That leaves us with a third option: Leave Netplan installed, but purge its config files, and then reapply its now-empty configuration. Fortunately, this is pretty simple. [According to the FAQ](https://netplan.io/faq), Netplan config files can exist in three different directories, in order of precedence:
1. `/run/netplan/`
2. `/etc/netplan/`
3. `/lib/netplan/`

Once you've removed the relevant files from those directories, regenerate Netplan's configuration with `netplan apply`. **Note:** This is a disruptive action, so make sure you have an alternate network configuration ready to take over after Netplan lets go of the reins.

All together, the whole process might look something like this:
```console
$ sudo rm -f /run/netplan/*.yaml /etc/netplan/*.yaml /lib/netplan/*.yaml && sudo netplan apply
```

Or, if you're a Puppet aficionado:
```puppet
file { '/etc/netplan/':
  ensure  => directory,
  owner   => 'root',
  group   => 'root',
  mode    => '0755',
  purge   => true,
  recurse => true,
  force   => true,
} ~>
exec { 'regenerate_netplan_config':
  command     => 'netplan apply',
  path        => [ '/sbin', '/usr/sbin', '/bin', '/usr/bin' ],
  refreshonly => true,
}
```
