---
title: '[Updated] Getting FreeBSD + xfce4 up and running on Hyper-V'
date: 2021-11-12T20:22:14-04:00
tags: [tutorial, BSD, Unix, OS, VM, Hyper-V, xfce]
summary: 'A tutorial that walks through installing and setting up the xfce desktop environment on FreeBSD, running as a VM in Hyper-V.'
draft: false 
---

## (April 2023) Update!

When I first wrote this tutorial at the end of 2021, Hyper-V generation **2** VMs did not correctly
capture mouse and keyboard input from BSD guest VMs. I just got notice that this bug was
[closed](https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=210175)!

I haven't been using Hyper-V as VM manager recently, so I haven't confirmed, but if you want to
give it a shot, you can try these instructions with a Gen 2 VM and see how it goes. Feel free to
email me (my first name at this website domain) if you get it working and I'll post your
experience.

Original content continues below...

![A FreeBSD Banner](freebsd_3.gif)
![Windows Hyper-V](hyperv.png)

This document began as a checklist for myself, as I tried and failed repeatedly to get a desktop environment working in FreeBSD inside Hyper-V on Windows. I used to use VMWare Player for hosting BSD, but it is locked down and an extra program to run. A Microsoft employee and personal friend pointed me to Hyper-V  Manager, a tool built-in to Windows 10+11 Pro/Enterprise that allows for the easy creation and management of VMs. Did I mention its built-in?

FreeBSD has had Hyper-V virtualization support for some time, and is mostly out-of-the-box... if you are sticking to a virtual terminal. Command line is fine for most users running BSD on Hyper-V, I'm sure, but I need a desktop environment to maintain a few BSD desktop packages.

After several days of scrounging around, I had managed to cobble together the process. There are a couple "gotchas" that aren't well-known, so for posterity I have written this guide in the hope that it might save others down the road the frustrations I went through.

Indeed, this is going on my blog so I myself can reference it in the future!

Before we get into things, I would like to thank the authors of two incredible resources. The first is the many contributors that have built the [FreeBSD Handbook](https://docs.freebsd.org/en/books/handbook). I reference it later on, but this is to BSD what [ArchWiki](https://wiki.archlinux.org/) is to Linux.

Second is the author of [unixsheikh.com](https://unixsheikh.com/index.html). This site first [introduced me to BSD](https://unixsheikh.com/articles/freebsd-is-an-amazing-operating-system.html), [persuaded me to try it](https://unixsheikh.com/articles/why-you-should-migrate-everything-from-linux-to-bsd.html), and walked me through getting it set up. Many parts of this tutorial are based on a similar tutorial series from here (linked to in the Epilogue). I still use Linux quite a lot, but it isn't the sole OS in my stack, thanks to this site.

Anyway, to the good stuff!

---

## Getting Started

[Download](https://www.freebsd.org/where/) the install **ISO**. Yes, there is a pre-made virtual machine image, but beware its temptation! The .vhd file is setup with a hilariously small size, and resizing partitions in FreeBSD on Hyper-V is a bit of a pain.[^0]

[^0]: Okay its not too terrible; the official Hyper-V docs explain how to do it in [the notes](https://docs.microsoft.com/en-us/windows-server/virtualization/hyper-v/supported-freebsd-virtual-machines-on-hyper-v#BKMK_notes). They _say_ its fixed in FreeBSD 11.0+ but I was having issues unless I did a `gpart recover` in FreeBSD 13.0, so ehhh... <br>Anyways, its not really any faster to use the pre-made image when you factor in expanding the `.vhd` in Hyper-V, then booting into single-user, resizing the partition, and getting the FS inflated. Additionally, the pre-made image uses UFS instead of the newer and generally better ZFS.

### Hyper-V Setup

Create  a new **Generation 1** VM. ~~[At the time of this writing, FreeBSD does not have a mouse
driver for Gen 2 VMs](https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=210175).~~ (04-2023 Note:
This bug has been marked as fixed, but I haven't tested it.)

Once you've set up your VM and pointed it to your ISO, go ahead and follow the standard install prompts. If you want a well-written walkthrough of this process, [the FreeBSD Handbook has you covered](https://docs.freebsd.org/en/books/handbook/bsdinstall/#bsdinstall-start). In fact, I **highly recommend** keeping the handbook open in a browser whenever you're trying something new-ish. I like the [version with the whole text on one page](https://docs.freebsd.org/en/books/handbook/book/): It makes it easy to `ctrl`+`F` for a term I need clarification on. Fair warning though, it takes a second to load.

Anyway, once set up, log in as root (or as your user and then use `su`). Going forward, I will denote commands executed as root with a hash symbol (#) and user commands with a dollar sign ($).

## Building the Base

Once logged in, setup `pkg` by running the command `pkg` and following the first-time setup prompts.

If you intend to do ports development (like I am) you should switch from the default (quarterly) branch to the "latest" branch for packages.

  1. Create a configuration file:

``` text
# mkdir -p /usr/local/etc/pkg/repos
# vi /usr/local/etc/pkg/repos/FreeBSD.conf
```

  2. Insert the following:

```text
FreeBSD: {
 url: "pkg+https://pkg.FreeBSD.org/${ABI}/latest"
}
```

  3. Update the package database:

```text
# pkg update
```

  4. Verify `pkg` is now on the "latest" repository:

```text
# pkg -vv
...
Repositories:
  FreeBSD: {
 url             : "pkg+https://pkg.FreeBSD.org/FreeBSD:13:amd64/latest",
 enabled         : yes,
 priority        : 0,
 mirror_type     : "SRV",
 signature_type  : "FINGERPRINTS",
 fingerprints    : "/usr/share/keys/pkg"
  }
```

The important part here is the `url` field. If you aren't on a 64-bit PC chip, the `amd64` may be different (e.g. x86, aarch64, etc.)

If `vt` is not the default virtual console set it as such by adding this line to `/boot/loader.conf`:

  ```shell
kern.vty="vt"
  ```

If you aren't comfortable using the pre-installed `vi`, go ahead and install your editor of choice with `pkg` at this point, i.e. `pkg install nano`.

* (Optional) speed up boot process by lowering the pause at the boot screen by adding this below the line we just added in `loader.conf`:

  ```shell
  autoboot_delay="2"
  ```

## Some other useful tools

I do most of my text editing in neovim. I still like to use the `vim` command though, and I don't like setting up aliases in every single shell/user combo.
So instead I install neovim, and symlink the `nvim` binary file to `vim`:

```text
# pkg install neovim
# ln -s /usr/local/bin/nvim /usr/local/bin/vim
```

I also install a couple other utilities at this time:

```text
# pkg install git curl doas
```

## Setting up `doas`

`doas` is basically an alternative tool to `sudo` from Linux land. There is also a port of `sudo` to the BSDs, but `doas` is [preferred for several reasons](https://www.reddit.com/r/freebsd/comments/klqt7k/why_are_people_sold_on_doas/ghgk1rn/), chief among them safety and ease of configuration[^1].

[^1]: <https://flak.tedunangst.com/post/doas> is a great writeup on the creation of this tool by its author.

I configure `doas` to allow `pkg` and shutdown/reboot commands without a password, and other commands with verification. You do this by adding the following to `/usr/local/etc/doas.conf`:[^2]

```shell
permit :wheel
permit nopass :wheel cmd shutdown
permit nopass :wheel cmd pkg
```

`:wheel` in this case means "any user in the group `wheel`". If you wanted to only permit a specific user (i.e. user 'jane') you would so so like this:

```shell
permit jane
permit nopass jane cmd shutdown
permit nopass jane cmd pkg
```

(Note that jane has no ":" before it.)

[^2]: **Note**: `doas` works by following the preferences of the *last matching line*. That means if you put line 1 (`permit :wheel`) at the end, then everything will require a password. Start with the most general, and progressively get more specific.

Some versions of BSD (and Linux ports of `doas`) also support a `persist` keyword, which allows subsequent uses of `doas` to not require another password auth for a time, similar to `sudo`. FreeBSD does not support this; adding this keyword will do nothing. If you need to do a bunch of commands as root in a row, it is [recommended](https://github.com/slicer69/doas/issues/37) to use `doas -s`, run the commands, then `exit`.

## Getting (or `Git`ing) ports

FreeBSD *used* to be distributed on a code management system called subversion, or `svn`. There is still some legacy support for this, but all the main code repositories have switched over to using `git`. Because of this, I recommend `git clone`-ing the Ports tree instead of the old `portsnap` method:

```text
# git clone https://git.FreeBSD.org/ports.git /usr/ports
```

Note that this downloads the HEAD of the tree, which will be the bleeding edge latest packages. If for some reason, you want to switch to a different quarterly branch (i.e. for stability) you can simply checkout the relevant branch:

```text
# git -C /usr/ports switch 2020Q4
```

Where `2020Q4` can be the year and quarter of your preference.

This may take a minute. As of writing the Ports tree is just under 1GB in size. This isn't downloading all the code for every package, mind you: the Ports tree is just a bunch of directories that _point_ to where the code exists on the internet for any given software. It will download and compile the software as you `make` a given folder. Still, there are enough packages to make hefty even in this minimal state.

At this point, feel free to switch off from root to your user account. We can do everything else with `doas` if needed.

## Shell setup

I like zshell (zsh) with ohmyzsh and the powerlevel10k theme. To add zsh, install the packages:

```zsh
pkg install zsh zsh-completions ohmyzsh
```

Go ahead and switch the root session to zsh by typing `zsh`. Then change your normal user's shell to use it by default:

```zsh
chpass -s /usr/local/bin/zsh <user>
```

Create a new zsh config file for ohmyzsh:

```zsh
cp /usr/local/share/ohmyzsh/templates/zshrc.zsh-template /home/<user>/.zshrc
chown <user>:<user> /home/<user>/.zshrc
```

Now you can clone your theme of choice (mine is p10k):

```zsh
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

And finally, set the theme in your user's `.zshrc`:

```zsh
ZSH_THEME="powerlevel10k/powerlevel10k"
```

The next time you log in as your non-root user, p10k will drop you into a configuration wizard.

## Pretty Chrome - Installing the DE

I like xfce. You can use whatever you prefer, of course, but you'll have to look elsewhere at this point, because this guide is going to go forward with the xfce4 desktop environment. I am also still using `xorg` as my display manager, wayland isn't *quite* ready for my workflow yet.

Start by installing the `xfce` package (and optional goodies), plug the X-org server:

```zsh
doas pkg install xfce xfce4-goodies xorg
```

At this point, things may be wildly different depending on the hardware you have. Refer to the [FreeBSD Handbook](https://docs.freebsd.org/en/books/handbook/book/#x-config-video-cards) for information best suited to your video hardware.

I have an nVidia card (I know, boo), so I install the nvidia driver from Ports[^3]:

```zsh
cd /usr/ports/x11/nvidia-driver
doas make -DBATCH install clean
```

[^3]: The `-DBATCH` flag tells the package and any dependencies to compile with defaults without asking. Normally its recommended to do a `make config-recursive` and ensuring everything is as you like it. In this case, since this is a development VM that I don't have to worry about uptime/stability/data loss, I don't mind skipping this.

Its also recommended to enable `dbus` at this point, by adding the following lines to `/etc/rc.conf`:

```zsh
dbus_enable="YES"
```

Add the user(s) that you expect to be using a GUI to the group `video`

```zsh
doas pw groupmod video -M <user>
```

## Getting the Keyboard to work {#keyboard}

*(2023 Note: This is for Gen 1 VMs. If you try a Gen 2 VM, this may not be needed).

Hyper-V is an interesting creature - for Gen 1 VMs, the mouse and keyboard are emulated not as USB devices, but as the older PS/2-style devices -- even if your actual device is USB, FreeBSD will see a PS/2 device because that is what Hyper-V presents it.

Since PS/2 is a pretty old technology and mostly deprecated, recent versions of FreeBSD no longer check for it by default. To re-enable this, add the following line to `/etc/sysctl.conf`[^4]:

```zsh
sysctl kern.evdev.rcpt_mask=6
```

[^4]: Full disclosure: I know that this is modifying a scan setting for device discovery, but I have _no idea_ why this makes the keyboard work. Its been recommended in a couple places (bugs.freebsd.org, reddit, FreeBSD forums) and works, but it is never explained **why**. If anyone has an answer or can point me to the documentation, I'll definitely write a post as a follow-up.

Go ahead and do a reboot.

## Getting to the Desktop

Log in as your user, and create a file in your home directory called `.xinitrc`. This file tells the Xorg server what to try and launch when you call `startx` (or use a Desktop Manager to log in).

For xfce, we add the following line:

```zsh
exec startxfce4
```

Now for the fun part. Type `startx`, sit back, and wait for things to load. If everything has gone right, you will now be in a default XFCE desktop. Click on the screen area to capture the mouse (`ctrl`+`alt`+`left arrow` releases the mouse again). Try typing in a terminal.

At this point, you will be at a basic starting point to experiment with BSD, do some dev work, etc.

## Conclusion

### Extra Credit

If you want to see some more tips and tricks, and try really getting a nice desktop setup, I cannot recommend the [unixsheikh.com how to setup FreeBSD with a riced desktop](https://unixsheikh.com/tutorials/how-to-setup-freebsd-with-a-riced-desktop-part-1-basic-setup.html) series[^5]. If you notice it looks similar to this guide, that isn't a coincidence. I used this guide myself to install FreeBSD and eventually port my first desktop program from Linux.

[^5]: Parts 1 and 2 will cover getting set up in general, installing xfce, and doing some quality of life tweaks. Part 3 is for those who prefer a [tiling window manager](https://en.wikipedia.org/wiki/Tiling_window_manager) and is not required if you don't want to install one of those.

### Why BSD? Reason 0: The Ports Collection

There are a ton of cool features of all the BSDs, each worth considering and exploring. One that we very briefly touched on is the Ports collection. If you followed these steps, you will have already downloaded the Ports tree. The step where I installed the nvidia drivers is a high-level walkthrough of installing a port in itself, but if you want the brief steps:

1. Find the name of your desired app and its category. [There are multiple ways to find a package](https://docs.freebsd.org/en/books/handbook/book/#ports-finding-applications), but I like to use the `pkg search -o` command:

   ```zsh
   $ pkg search -o xfce4-docklike
   ...
   x11/xfce4-docklike-plugin      Modern, minimalist taskbar for Xfce
   ```

2. Go to the path listed in the output

3. Type `make config`

4. Type `make install clean`

That's pretty much it. This is a super powerful tool that allows extreme customizability with ease, because you are with these three simple steps actually downloading and compiling from source!

The BSD Ports system has been replicated in a couple other places, most notably in Gentoo Linux.

---

At the end of the day, use the tools that best suit you. BSD is worth a look, though!
<br>

-Quentin \
\
\
<br>
