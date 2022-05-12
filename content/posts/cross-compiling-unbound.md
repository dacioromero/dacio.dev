---
title: How I Updated, Built, and Deployed to Launchpad an Ubuntu/Debian Package
date: "2022-05-11T18:12:23-07:00"
tags:
- ubuntu
- debian
- ppa
- sbuild
- unbound
- arm
- pihole
---

I wanted to build [Unbound](https://www.nlnetlabs.nl/projects/unbound/about/) from source for my [Pi-hole](https://pi-hole.net/) running  [Armbian](https://www.armbian.com/bananapi-m2-zero/) (a debian fork for ARM SBC) for my [Banana Pi M2 Zero](https://wiki.banana-pi.org/Banana_Pi_BPI-M2_ZERO) (a clone of the [Raspberry Pi Zero](https://www.raspberrypi.com/products/raspberry-pi-zero/)) so that I could enable its cachedb module following [this thread](https://discourse.pi-hole.net/t/pihole-ftl-unbound-with-cache-db-module-options-redis/24676) for potentially speeding up DNS queries on my local network.

This lead me down the rabbit hole of building C-based Debian packages from source.

## Setup

`sudo apt-get install -y sbuild debhelper ubuntu-dev-tools git-buildpackage`

[sbuild](https://manpages.ubuntu.com/manpages/focal/en/man1/sbuild.1.html) is the tool we'll to compile,
[debhelper](https://manpages.ubuntu.com/manpages/focal/en/man7/debhelper.7.html) contains many necessary commands for buiding debian packages,
[ubuntu-dev-tools](https://packaging.ubuntu.com/html/ubuntu-dev-tools.html) some useful commands (`pull-lp-source` and `mk-sbuild`), and
[git-buildpackage](https://manpages.ubuntu.com/manpages/focal/en/man1/gbp-buildpackage.1.html) is an incredibly useful tool for Debian development

I recommend having the minimum of the following in your `~/.gbp.conf`

```toml
[DEFAULT]
pristine-tar = True
```

This tries to ensure that `git-buildpackage` generates the same tar files byte-for-byte so that Launchpad doesn't reject them. See [DebianPackaging--pristine-tar-option-explained](https://wiki.debian.org/DebianPackaging--pristine-tar-option-explained).

## Pulling the source
### From original package repository
`pull-lp-source --download-only unbound`
or
`apt-get source --download-only unbound`

Both of these commands have the same output, but `apt-get source` requires `deb-src` entries in your `/etc/apt/sources.list`.

`gbp import-dsc ./unbound_1.13.1-1ubuntu5.dsc`

`cd unbound`

You can omit `--download-only` from `pull-lp-source` or `apt-get source` to unpack the `.dsc`, but then you'll be unable to use any of the `gbp` commands which are a major help in managing Debian packages.

### From Git

`git clone https://salsa.debian.org/dns-team/unbound.git`

`cd unbound`

`git branch upstream origin/upstream`

`git branch pristine-tar origin/pristine-tar`

I initially went with this method, but I couldn't find a Git source for Ubuntu's version of Unbound, and I suspect that the Git repositories are often not published.

## Updating upstream
To update our upstream package to the latest version we're going to use [gbp import-orig](https://manpages.ubuntu.com/manpages/focal/en/man1/gbp-import-orig.1.html). If your primary branch isn't master, which would only happen when pulling the source [from git](#from-git),  you can should use `--debian-branch` option to specify when using `gbp` commands in the future.

### uscan
`--uscan` automatically gets the latest upstream and signature as defined in [debian/watch](https://wiki.debian.org/debian/watch) in an incredibly developer-friendly way; if you plan on maintaining your PPA I'd recommend looking into adding one of these.

`gbp import-orig --uscan`

### file/url

If uscan didn't work or isn't defined, you can also import compressed tarballs from file or URL. In Unbound's case it was https://www.nlnetlabs.nl/downloads/unbound/unbound-1.15.0.tar.gz.

`gbp import-orig https://www.nlnetlabs.nl/downloads/unbound/unbound-1.15.0.tar.gz`

---

`wget https://www.nlnetlabs.nl/downloads/unbound/unbound-1.15.0.tar.gz -P ..`

`gbp import-orig ../unbound-1.15.0.tar.gz`

## Making changes
You can now make changes that you'd like, I'd recommending commiting smaller and frequently as normal for Git so that `gbp dch` can generate useful changelogs.

If you haven't already you should set your name and email in Git.

`git config --global user.name "Dacio Romero"`

`git config --global user.email "dacioromero@gmail.com"`

Update [debian/changelog](https://www.debian.org/doc/manuals/maint-guide/dreq.en.html#changelog) with:

`DEBEMAIL="Dacio Romero <dacioromero@gmail.com>" gbp dch --release --commit`

When publishing a PPA it's important to allow for official versions to take precedence. This can become a bit complicated, so let's take a look at one of the versions of my packages:

`1.15.0-0ubuntu0ppa1~focal1`

`1.15.0` is the version of the upstream package, in the case the actual version of unbound. `-0` is the Debian revision `ubuntu0` is the Ubuntu revision. `ppa1` is our PPAs revison. `~focal1` is additional versioning I used to create builds for every active Ubuntu release.

With the Debian and Ubuntu revisions I set both to `0` since there were no official versions for Ubuntu 1.15.0 in either repository.

See [Building a source package](https://help.launchpad.net/Packaging/PPA/BuildingASourcePackage) and [the version section of the Debian Policy Manual](https://www.debian.org/doc/debian-policy/ch-controlfields.html#version) for more information on versioning.

`--newversion`, placed after  is the best argument I've found for `gbp dch` for manually specifying a version.

`DEBEMAIL` is passed onto [debchange](https://manpages.ubuntu.com/manpages/focal/man1/debchange.1.html) to fill the new entry with your info in the changelog, but if your `NAME` and `EMAIL` environment variables happen to already be set you don't need this.

## Creating the build environment
https://wiki.ubuntu.com/SimpleSbuild
https://wiki.debian.org/PackagingWithGit
https://wiki.debian.org/CrossCompiling
http://hypernephelist.com/2015/03/09/cross-compilation-made-easy-on-ubuntu-with-sbuild.html

sbuild requires something called a [schroot](https://wiki.debian.org/Schroot), I honestly don't know how to make one manually, but luckily ubuntu-dev-tools includes a command called [mk-sbuild](https://manpages.ubuntu.com/manpages/focal/man1/mk-sbuild.1.html) to automate this.

`mk-sbuild focal`

You can specify a different host architecture, the architecture that your package wil run on, with `--target` which defaults to the build architecture. The build architecture from can be chosen with `--arch`.  If the specified arch differs from your machine it will use [qemu](https://www.qemu.org/) through [qemu-user-static](https://manpages.ubuntu.com/manpages/focal/man1/qemu-user-static.1.html),  .

I'd recommend cross-compiling through the `--target` option as it's often faster and you don't have to worry about qemu's emulation; personally I've run the following error with Ubuntu 20.04 Focal while building with qemu:

`semop(1): encountered an error: Function not implemented`

As far as I can tell this is a known error ([bug report](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=965109)) that's fixed in newer versions of Ubuntu. It worked for me in Ubuntu 22.04 Jammy, but cross-compiling for armhf from amd64 would error as Jammy doesn't have `pkg-config-arm-linux-gnueabihf` at the time of writing.

Upon running `mk-sbuild` for the first time it'll prompt you to edit `~/.sbuildrc` which the same as [sbuild.conf](https://manpages.ubuntu.com/manpages/focal/en/man5/sbuild.conf.5.html). I'd reccommend commenting out `$mailto` and setting `$maintainer_name` as a minimum. Afterwards you can run the same command as before.

## Building
Finally we can build the package

`gbp buildpackage --git-builder="sbuild --dist=focal --jobs=32 --source-only-changes"`

This is a lot so let's break it down. [gbp buildpackage](https://manpages.ubuntu.com/manpages/focal/en/man1/gbp-buildpackage.1.html) does a number of checks, but most importantly to us is automatically recreating `unbound_1.13.1.orig.tar.gz` in the same directory that our package's directory is in which is required for sbuild to start.

You can alternatively run `git export-orig` to generate that file and then the command in `--git-builder`.

git-buildpackage by default will use [debuild](https://manpages.ubuntu.com/manpages/focal/en/man1/debuild.1.html) and cowbuilder/pbuilder with the option `--git-pbuilder`.  debuild, cowbuilder/pbuilder, and sbuild are all similar but each have their pros and cons (see [Builder choices](#Alternate-builder-choices)).

sbuild `--arch`  is the same as mk-sbuild and sbuild `--host` is the same as mk-sbuild --`target`, so you should match these if you specified them. `--dist` is the same as the final argument in mk-sbuild. `--jobs` enables multiprocessing, you can set this to `--jobs=$(nproc)` to use all of your threads automatically.

### Builder choices
### Debuild
Debuild is a very simple way of building comparitively, but its major drawback is working within the the same environment as your normal one requiring you to dirty your environment with build dependencies (which you have to satisfy yourself).

`debuild`

### cowbuilder/pbuilder
`sudo apt-get install -y pbuilder`

`cowbuilder` is a wrapper around `pbuilder` and use chroot's like sbuild does, but isn't as straightforward as sbuild to set up. It does, however, have official integration with `git-buildpackage` and built-in apt caching.

You should specify  `PBUILDERSATISFYDEPENDSCMD="/usr/lib/pbuilder/pbuilder-satisfydepends-apt"` in your `~/.pbuilderrc` and `/root/.pbuilderrc` (can be omitted if you add `--configfile ~/.pbuilderrc`) for packages get satisfied correctly. Set `--mirror` when creating a chroot for another distribution than you current and specify `--basepath`  because it defaults to `/var/cache/pbuilder/base.cow` limiting you to one chroot.

Many options are equivalent between pbuilder and sbuild:

| sbuild | cowbuilder |
| --- | --- |
| --dist=\[distribution\] | --distribution \[distribution\] |
| --arch=\[architecture\] | --architecture \[architecture\] |
| --target \[architecture\] | --host-arch \[architecture\] |
| --jobs=\[n\] | --debbuildopts "--jobs=\[n\]" |
| --debbuildopts="\[options\]" | --debbuildopts "\[options\]" |

#### Create the schroot
`sudo cowbuilder create --distribution focal --basepath /var/cache/pbuilder/focal-amd64-base.cow`

If you're cross-compiling (`--host-arch` is set) you might need to edit its `sources.list` as cowbuilder doesn't automatically add the necessary sources for non amd64 or i386 architectures.

For me I had to change:

```
deb http://archive.ubuntu.com/ubuntu/ focal main universe
#deb-src http://ports.ubuntu.com/ubuntu-ports/ focal main universe
```

to:

```
deb [arch=amd64] http://archive.ubuntu.com/ubuntu/ focal main universe
#deb-src http://archive.ubuntu.com/ubuntu/ focal main universe
deb [arch=armhf] http://ports.ubuntu.com/ubuntu-ports/ focal main universe
#deb-src http://ports.ubuntu.com/ubuntu-ports/ focal main universe
```

#### Build the package
`pdebuild --pbuilder cowbuilder -- --basepath /var/cache/pbuilder/focal-amd64-base.cow --distribution focal --debbuildopts '--jobs=32'`
or
`gbp buildpackage --git-pbuilder --git-no-pbuilder-auto-conf --git-pbuilder-options="--basepath /var/cache/pbuilder/focal-amd64-base.cow --distribution focal --debbuildopts '--jobs=32'"`

### Why did we choose sbuild
`mk-sbuild` is sbuild's saving grace as sbuild's [sbuild-createchroot](https://manpages.ubuntu.com/manpages/focal/en/man8/sbuild-createchroot.8.html) doesn't nearly do enough compared to cowbuilder's built in `create` command. It does a lot of the configuration for you compared to `cowbuilder create` including not worrying about `PBUILDERSATISFYDEPENDSCMD`, defining mirrors (including arch's), and specifying schroot locations.

## Signing and uploading to Launchpad
Before uploading to Launchpad you have to:

1. [Create a key](https://keyring.debian.org/creating-key.html)
2. [Upload the key to Ubuntu's keyserver](https://help.ubuntu.com/community/GnuPrivacyGuardHowto#Uploading_the_key_to_Ubuntu_keyserver)
3. [Import the key into Launchpad](https://help.launchpad.net/YourAccount/ImportingYourPGPKey)
4. [Sign the Ubuntu Code of Conduct](https://help.launchpad.net/Signing%20the%20Ubuntu%20Code%20of%20Conduct)

It may be necessary to edit the `source.changes` file before uploading. There seem to be [an issue](https://bugs.launchpad.net/launchpad/+bug/1699763) with Launchpad preventing some buiildinfo files from being accepted. Adding `--buildinfo-option=-O` to `--debbuildopts` (output buildinfo to stdout) in sbuild or cowbuilder seem to help, but the easiest solution I've found is deleting every line referencing the file in source.changes.

You may get an error email back from Launchpad with `.orig.tar.gz` files if it already exists in on their fileservers, but your version differs. `pristine-tar` is supposed to help with this, but it's possible that somehow another version ended up on Launchpad.

There are a few resolutions for this problem:

- Download the file in Launchpad and rebuild with it
- Add `sd` to wherever options are passed to [dpkg-buildpackage](https://manpages.ubuntu.com/manpages/focal/en/man5/gbp.conf.5.html) and subsequently [dpkg-genchanges](https://manpages.ubuntu.com/manpages/focal/en/man1/dpkg-genchanges.1.html) (`--debbuildopts` for sbuild and cowbuilder)
- Remove the `.orig.tar.gz` referenced lines from the source.changes file like with .buildinfo

Next is signing the `source.changes` file.

`debsign unbound_1.15.0-1_source.changes`

Upload the package (See [Launchpad: Uploading a package to a PPA](https://help.launchpad.net/Packaging/PPA/Uploading))

`dput ppa:dacio/unbound unbound_1.15.0-1_source.changes`

## If all has gone well your package should build and be used once completed!
