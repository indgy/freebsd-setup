# FreeBSD Setup 

A simple provisioning script to setup a freshly installed FreeBSD server, ideal for quick setup of 
VPS from Digital Ocean, Vultr, AWS etc.

This is a fairly un-opinionated setup, it will install only sane, lightweight and performant 
additional utilities where absolutely necessary. I prefer lightweight and simle solutions over 
the complex and involved, this philosophy is reflected in this setup.


* Installs `doas` instead of `sudo`
  Doas ~100Kb, Sudo ~7,000kb
* Installs `BastilleBSD` rather than using the built in jails for convenience. BastilleBSD has
  no dependencies other than `/bin/sh` and is very convenient for it's lightweight size ~195Kb
* No Sendmail
  The built in DMA mailer is used for sending out all system messages
* UTF-8 charset
  Defaults to UTF-8 only
* No bash
  Bash is great on MacOS (now uses Zsh) or Linux, but the built in `tcsh` is perfectly good for 
  interactive use. (If you really need bash just `pkg install bash` and `chsh -s /usr/local/bin/bash`)


#### Shameless plug
Please use one of the following links to sign up to a VPS provider and we will both get some free 
credit at no extra cost to you.
[Vultr](https://www.vultr.com/?ref=8852149) - [Get $100 free!](https://www.vultr.com/?ref=8852150-6G) 
[Digital Ocean](https://m.do.co/c/3d666f171318) Get $100 free!


## Requirements

A *freshly installed* FreeBSD server running 11.4-RELEASE or 12.2-RELEASE or higher.

**Note** This script will replace your existing config files!

## What it installs

The following additional packages will be installed, requiring a tiny (~0.25Mb) additional download

* [Doas](https://www.freebsd.org/cgi/man.cgi?query=doas) (45Kb) - Simple sudo alternative from OpenBSD to run commands as another user
* [BastilleBSD](https://bastillebsd.org) (195Kb) - The container (jail) automation framework
* CA Root NSS - Root certificate bundle from the Mozilla Project, required for encrypted communications

## What it does

* Enables AESNI hardware encryption.
* Configures PKG (the package management tool) to use the Latest packages.
* Sets up the timezone, defaults to UTC, override with `--timezone=GB`.
* Sets the charset to UTF-8 and locale to C.UTF-8 override with `--locale=en_GB`.
* Reduces boot delay to 4 seconds.
* Removes unnecessary ttys.
* Disables Sendmail and removes cruft from /etc/mail.
* Configures the PF firewall to support BastilleBSD jails, Blacklistd and the SSH server.
* Configures the SSH server to accept keys only, deny root logins and deny ssh forwarding.
* Configures the Blacklist daemon to protect SSH.
* Configures the Network Time daemon.
* Configures the [DragonFly Mail Agent](https://www.freebsd.org/cgi/man.cgi?query=dma) to send outgoing mail.
* Installs the Root security certificates from Mozilla to enable encrypted communications.
* Installs 'doas' the lightweight and secure 'sudo' alternative.
* Installs 'BastilleBSD' to manage jails (containers).
* Adds daily OS update checks.
* Adds an admin user `--ssh-user` who can SSH in and elevate privileges to root via 'doas'.
* Forwards all mail to root to the specified email address `--mail-to`.
* Adds some configuration to the default shell `tcsh` to make it easier to live with.

## Usage

As root on a freshly installed FreeBSD system type the following to fetch the setup script and make it executable:

```sh
fetch https://raw.githubusercontent.com/indgy/freebsd-setup/main/freebsd-setup \
&& chmod 0700 freebsd-setup
```

To see all options:

```sh
./freebsd-setup -h
```

Then provide your config as arguments.

```sh
./freebsd-setup \
  --ssh-user="admin" \              # the name of the user who will ssh in
  --ssh-user-key="ssh-rsa ..." \    # provide a string, a local file or the url of a remote file
  --mail-to="admin@yourdomain.com"  # all mail will be forwarded to this address
```

Or generate an editable config file 

```sh
./freebsd-setup -g setup.conf
edit setup.conf
```

Reference the config file using the `-c` or `--config` arguments

```sh
 # A local file
./freebsd-setup -c=/path/to/setup.conf
 # A remote file via http or ftp
./freebsd-setup -c=https://yourdomain.com/setup/setup.conf
```

The external IP address and default NIC are determined from the first DHCP enabled NIC in /etc/rc.conf.
If the defaults are not correct you may override with the `--ext-if` and `--ext-ip` arguments

## What next

Once you have setup the script and the server has rebooted you have a great setup to build on.

Depending on your requirements you may want to:

* Add a monitoring solution
* Install apps in jails, see the [Bastille Templates](https://gitlab.com/bastillebsd-templates) repository for pre built apps

If you're new(ish) to FreeBSD checkout these resources:

* Learn more about FreeBSD at [www.freebsd.org](https://www.freebsd.org)
* Refer to the documentation at [docs.freebsd.org](https://docs.freebsd.org/) especially the [Handbook](https://docs.freebsd.org/en_US.ISO8859-1/books/handbook/)
* Refresh your memory with the manual pages at [man.freebsd.org](https://man.freebsd.org) 

<!--
A quick guide to -CURRENT, -STABLE, -RELEASE, the FreeBSD development branches

Though not quite accurate, FreeBSD branches can be thought of like so:

* -CURRENT The bleeding edge, aimed at developers and tinkerers.
* -STABLE Aimed at testers and early adopters, working updates from CURRENT appear here.
* -RELEASE Aimed at everyone else, this is the most thoroughly tested branch.

While -STABLE is generally reliable it is best to use -RELEASE, especially on public facing servers.
Think of -RELEASE as the 'shrink-wrapped' version ;-)
-->

## Security 
* Further configuration of the firewall is recommended
* Additionally some thought needs to be given to the arrangement and management of the jails
* If you are concerned about the physical security of your virtual machine you may want to set the
  console to insecure in `/etc/ttys`
