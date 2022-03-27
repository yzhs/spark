# Spark

Spark is an [Ansible][1] playbook meant to provision a personal machine running
[Arch Linux][2]. It is intended to run locally on a fresh Arch install (ie,
taking the place of any [post-installation][3]), but due to Ansible's
idempotent nature it may also be run on top of an already configured machine.

Spark assumes it will be run on a laptop and performs some configuration based
on this assumption. This behaviour may be changed by removing the `laptop` role
from the playbook or by skipping the `laptop` tag.

If Spark is run on either a ThinkPad or a MacBook, it will detect this and
execute platform-specific tasks.

**Note:** If you would like to try recreating all the tasks that are currently 
included in the ansible playbook, through a VM, you would need a disk of at least 
**16GB** in size.

## Running

First, sync mirrors and install Ansible:

    $ pacman -Syy python-passlib ansible

Second, install and update the submodules:

    $ git submodule init && git submodule update
    
Run the playbook as root.

    # ansible-playbook -i localhost playbook.yml

When run, Ansible will prompt for the user password. This only needs to be
provided on the first run when the user is being created. On later runs,
providing any password -- whether the current user password or a new one --
will have no effect.

## SSH

By default, Ansible will attempt to install the private SSH key for the user. The
key should be available at the path specified in the `ssh.user_key` variable.
Removing this variable will cause the key installation task to be skipped.

### SSHD

If `ssh.enable_sshd` is set to `True` the [systemd socket service][4] will be
enabled. By default, sshd is configured but not enabled.

## Dotfiles

Ansible expects that the user wishes to clone dotfiles via the git repository
specified via the `dotfiles.url` variable and install them with [rcm][5]. The
destination to clone the repository to is defined by the `dotfiles.destination`
variable. This is relative the user's home directory.

These tasks will be skipped if the `dotfiles` variable is not defined.

## Tagging

All tasks are tagged with their role, allowing them to be skipped by tag in
addition to modifying `playbook.yml`.

## AUR

All tasks involving the [AUR][6] are tagged `aur`. To provision an AUR-free
system, pass this tag to ansible's `--skip-tag`.

AUR packages are installed via the [ansible-aur][7] module. Note that while
[yay][8], an [AUR helper][9], is installed by default, it will *not* be used
during any of the provisioning.

## Mail

### Receiving Mail

Receiving mail is supported by syncing from IMAP servers via both [isync][13]
and [OfflineIMAP][14]. By default isync is enabled, but this can be changed to
OfflineIMAP by setting the value of the `mail.sync_tool` variable to
`offlineimap`.

### Sending Mail

[msmtp][15] is used to send mail. Included as part of msmtp's documentation are
a set of [msmtpq scripts][16] for queuing mail. These scripts are copied to the
user's path for use. When calling `msmtpq` instead of `msmtp`, mail is sent
normally if internet connectivity is available. If the user is offline, the
mail is saved in a queue, to be sent out when internet connectivity is again
available. This helps support a seamless workflow, both offline and online.

### System Mail

If the `email.user` variable is defined, the system will be configured to
forward mail for the user and root to this address. Removing this variable will
cause no mail aliases to be put in place.

The cron implementation is configured to send mail using `msmtp`.

### Syncing and Scheduling Mail

A shell script called `mailsync` is included to sync mail, by first sending any
mail in the msmtp queue and then syncing with the chosen IMAP servers via
either isync or OfflineIMAP. The script will also attempt to sync contacts and
calendars via [vdirsyncer][17]. To disable this behavior, set the
`mail.sync_pim` variable to `False`.

Before syncing, the `mailsync` script checks for internet connectivity using
NetworkMananger. `mailsync` may be called directly by the user, ie by
configuring a hotkey in Mutt.

A [systemd timer][18] is also included to periodically call `mailsync`. The
timer is set to sync every 5 minutes (configurable through the `mail.sync_time`
variable).

The timer is not started or enabled by default. Instead, the timer is added to
`/etc/nmtrust/trusted_units`, causing the NetworkManager trusted unit
dispatcher to activate the timer whenever a connection is established to a
trusted network. The timer is stopped whenever the network goes down or a
connection is established to an untrusted network.

To have the timer activated at boot, change the `mail.sync_on` variable from
`trusted` to `all`.

If the `mail.sync_on` variable is set to anything other than `trusted` or
`all`, the timer will never be activated.


## Tor

[Tor][23] is installed by default. A systemd service unit for Tor is installed,
but not enabled or started. instead, the service is added to
`/etc/nmtrust/trusted_units`, causing the NetworkManager trusted unit
dispatcher to activate the service whenever a connection is established to a
trusted network. The service is stopped whenever the network goes down or a
connection is established to an untrusted network.

To have the service activated at boot, change the `tor.run_on` variable
from `trusted` to `all`.

If you do not wish to use Tor, simply remove the `tor` variable from the
configuration.

[1]: http://www.ansible.com
[2]: https://www.archlinux.org
[3]: https://wiki.archlinux.org/index.php/Installation_guide#Post-installation
[4]: https://wiki.archlinux.org/index.php/Secure_Shell#Managing_the_sshd_daemon
[5]: https://thoughtbot.github.io/rcm/
[6]: https://aur.archlinux.org
[7]: https://github.com/pigmonkey/ansible-aur
[8]: https://github.com/Jguer/yay
[9]: https://wiki.archlinux.org/index.php/AUR_helpers
[12]: https://github.com/pigmonkey/nmtrust
[13]: http://isync.sourceforge.net/
[14]: http://offlineimap.org/
[15]: http://msmtp.sourceforge.net/
[16]: http://sourceforge.net/p/msmtp/code/ci/master/tree/scripts/msmtpq/README.msmtpq
[17]: https://github.com/pimutils/vdirsyncer
[18]: https://wiki.archlinux.org/index.php/Systemd/Timers
[23]: https://www.torproject.org/
