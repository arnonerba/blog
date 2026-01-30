---
title: "Enabling Duo Unix on Ubuntu With \"pam-auth-update\""
---
Unless you're using the legacy `login_duo` approach, enabling [Duo Unix](https://duo.com/docs/duounix) on an Ubuntu system involves integrating the `pam_duo` module into your system's [Pluggable Authentication Modules (PAM)](https://en.wikipedia.org/wiki/Pluggable_Authentication_Module) stack. This can be done by manually editing files in `/etc/pam.d/`, but Ubuntu provides a safer way to do this in the form of `pam-auth-update`.

## Ubuntu's PAM Config Framework

The [pam-auth-update](https://manpages.ubuntu.com/manpages/noble/en/man8/pam-auth-update.8.html) script and its associated [PAMConfigFrameworkSpec](https://wiki.ubuntu.com/PAMConfigFrameworkSpec) were [originally written by Steve Langasek](https://raphaelhertzog.com/2011/05/06/people-behind-debian-steve-langasek-release-wizard/)[^1] and were added to Debian and its derivatives in 2008[^2].

The spec page linked above is the best resource for understanding this framework that I am aware of, but I'll provide a brief summary here:
1. Packages -- or enterprising administrators -- can install custom PAM profiles into `/usr/share/pam-configs/`. This allows packages to specify how their PAM modules should be integrated into the system PAM stack.
2. Once installed, these custom PAM profiles can be enabled or disabled with the `pam-auth-update` script.
3. When PAM profiles are enabled or disabled, `pam-auth-update` modifies the system-wide PAM configuration files at `/etc/pam.d/common-*` accordingly.
4. The script can be run interactively by an administrator or non-interactively by configuration management tools or package maintainer scripts.

## Understanding Ubuntu's System-Wide PAM Configuration

First, let's review some PAM basics. As described in the [man page for pam.d](https://manpages.ubuntu.com/manpages/noble/en/man5/pam.d.5.html), there are four types of PAM modules:
1. **account**

    > this module type performs non-authentication based account management. It is typically used to restrict/permit access to a service based on the time of day, currently available system resources (maximum number of users) or perhaps the location of the applicant user -- 'root' login only on the console.

2. **auth**

    > this module type provides two aspects of authenticating the user. Firstly, it establishes that the user is who they claim to be, by instructing the application to prompt the user for a password or other means of identification. Secondly, the module can grant group membership or other privileges through its credential granting properties.

3. **password**

    > this module type is required for updating the authentication token associated with the user. Typically, there is one module for each 'challenge/response' based authentication (auth) type.

4. **session**

    > this module type is associated with doing things that need to be done for the user before/after they can be given service. Such things include the logging of information concerning the opening/closing of some data exchange with a user, mounting directories, etc.

The PAM module for Duo Unix falls solely into the **auth** management group, so, for the purposes of this integration, we only need to focus on that module type.

Second, Ubuntu provides five system-wide PAM configuration files in `/etc/pam.d` that are designed to be included by services that rely on PAM, such as `sshd` and `sudo`. These are the files that `pam-auth-update` is able to modify:
1. `/etc/pam.d/common-account`: Authorization settings common to all services.
2. `/etc/pam.d/common-auth`: Authentication settings common to all services.
3. `/etc/pam.d/common-password`: Password-related modules common to all services.
4. `/etc/pam.d/common-session`: Session-related modules common to all services.
5. `/etc/pam.d/common-session-noninteractive`: Session-related modules common to all non-interactive services.

For context, this means our PAM profile for Duo Unix will only need to modify `/etc/pam.d/common-auth`. With all this in mind, we can move on to writing the profile.

## Writing the PAM Profile

Here is the PAM profile I've come up with:
```
Name: Duo two-factor authentication
Default: no
Priority: 128
Auth-Type: Additional
Auth:
	required			/usr/lib64/security/pam_duo.so
```

The table below describes what each individual field does and explains why I chose the options I did:

| Field | Description | Justification |
| ----- | ----------- | ------------- |
| `Name` | A human-readable string that's displayed when `pam-auth-update` is run in interactive mode. | This can be anything that makes sense to you. |
| `Default` | Whether the profile should be enabled by default. | My subjective opinion is that Duo Unix should not be enabled by default because it requires additional configuration to work properly. In any case, this is irrelevant for the purposes of this post, since we're not providing this profile as part of a package. |
| `Priority` | Where to insert this module in the relevant system-wide PAM configuration file (`/etc/pam.d/common-auth`). Higher-priority modules will be listed first. | I've chosen `128` based on the framework spec page and on the choices made by other packages. **Note:** This value depends on the value of `Auth-Type` below, since priorities are evaluated **separately** for the `Primary` and `Additional` sections. |
| `Auth-Type` | This field supports two options, `Primary` and `Additional`, that correspond to separate sections in the system-wide PAM configuration files listed above. The `Primary` section is intended for modules where the success of any one module indicates an overall success, such as when `pam_unix.so` and `pam_sss.so` are enabled simultaneously[^3]. The `Additional` section is intended for modules that should run regardless of the success of other modules. | Based on that guidance, I've chosen to add Duo Unix as an "Additional" module, since it should only run after the primary authentication flow has completed. |
| `Auth` | The desired PAM-API control, the name of the PAM module on disk, and any optional module arguments. | I'm using `required` here so that the PAM-API will deny access if Duo authentication fails but will still process the rest of the PAM stack first. (Other valid control values can be found in the [man page for pam.d](https://manpages.ubuntu.com/manpages/noble/en/man5/pam.d.5.html).) I've also included the canonical path to the module since it resides outside of PAM's normal search path. |

Finally, the profile must be installed at `/usr/share/pam-configs/duo-unix` in order for it to be readable by `pam-auth-update`. I've chosen to use `duo-unix` as the filename for the profile since convention dictates it should match the related package or module name.

## Enabling the PAM Profile

Once the profile is installed, it can be activated with `pam-auth-update`. There are three ways to do this:
1. Interactively, as an administrator: Run `pam-auth-update` as root, check the box next to "Duo two-factor authentication", and select "OK".
2. Non-interactively, as a package maintainer script: Run `pam-auth-update --enable duo-unix --package`.
3. Non-interactively, as an administrator or as a configuration management tool: Run `pam-auth-update --enable duo-unix --force`.

> [!NOTE]
> The third option will overwrite your current system-wide PAM configuration without prompting, which should be safe unless you have manually edited the aforementioned system-wide PAM configuration files in `/etc/pam.d`.

## Example Puppet Code

Here's a minimal working example for how to do all of this in a Puppet module:
```puppet
file { '/usr/share/pam-configs/duo-unix':
  ensure  => file,
  owner   => 'root',
  group   => 'root',
  mode    => '0644',
  content => file("${module_name}/pam-config-duo-unix"),
} ~>
exec { "${module_name}_enable_pam_module":
  command => 'pam-auth-update --enable duo-unix --force',
  unless  => 'grep -q pam_duo\.so /etc/pam.d/common-auth',
  path    => [ '/sbin', '/usr/sbin', '/bin', '/usr/bin' ],
}
```

## Closing Thoughts

In doing things this way, I'm breaking with Duo's official recommendation for Ubuntu systems, which is to modify the default call to `pam_unix.so` in `/etc/pam.d/common-auth` and insert a call to `pam_duo.so` directly afterwards. However, what I propose here should be both functionally identical and safer. Also, if you're using `pam_sss.so` in conjunction with (or instead of) `pam_unix.so` to enable authentication via [SSSD](https://sssd.io/), Duo's recommendation won't work anyway.

It's also worth mentioning that enabling Duo Unix in the manner described above should enable Duo for any authentication flow that relies on PAM's `common-auth` stack, including `sudo`. One notable exception to this rule is key-based authentication in SSH, which evidently [bypasses the PAM auth stack](https://serverfault.com/a/592591) entirely. Enabling Duo for SSH key authentication is outside the scope of this post[^4].

[^1]: Tragically, [Steve passed away early last year](https://discourse.ubuntu.com/t/remembering-and-thanking-steve-langasek/52665) after decades of prolific and lasting contributions to Ubuntu and open-source as a whole.
[^2]: Those of you who are familiar with other distros may have noted that Fedora and its derivatives use [authselect](https://github.com/authselect/authselect) (which replaces the older `authconfig` utility) instead.
[^3]: For example, this might be done to allow both local and domain accounts to log into a system.
[^4]: Hint: This can be accomplished by editing the package-provided `sshd` PAM stack at `/etc/pam.d/sshd`, but this carries its own risks. Caveat emptor.
