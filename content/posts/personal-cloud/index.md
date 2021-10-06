---
title: "OYD Chapter 1: Personal Cloud"
date: 2021-10-05T23:14:34-04:00
draft: true
---
Clouds can be light and fluffy. They can also be... not.

![A Picture of a Hurricane, taken by NASA](hurricane.webp)

One massive convience (and source of lock in) offered by major tech players is Cloud Storage. Its a big deal because its
(relatively) easy to set up, has predictable costs, and produces lots of *recurring* revenue.

In this first dive into personal data ownership, I'm going to walk through setting up and using an open-source
alternative to offerings like DropBox and iCloud: [SyncThing](https://syncthing.net/).

## Moving stuff around

In the beginning was the punchcard. This primitve source of "non-volatile" storage (just don't light it on fire)
actually predates the popular concept of a computer by over 100 years: in 1801, the textile indusry began using punched
"number cards" to automate complex weaving patterns in looms.

The punchcard was pretty much the go-to for transportable digital storage (minus specialty, bespoke solutions like [Magnetic-core
memory](https://en.wikipedia.org/wiki/Magnetic-core_memory)) until magnetic storage in the form of tape and floppy
drives finally superceded it.

From there, we moved briefly to optical storage (CDs, DVDs), but that fell out of favor as solid-state storage
(essentially, specialized portable computers), offering much smaller size, resilliance, and performance took over with
the help of Moore's law.

And now, in 2021, even flash drives and SD cards are beginning to feel... quaint. Plugging in a thumb drive, manually
copying folders, physically travelling or sending the drive to the other device, manually pulling it from the drive; it
just cannot compete with the magical concept of installing a program, dropping the file into a special folder on your
computer and **bam**, it is everywhere you have an internet connection -- and if SpaceX has anything to say about it,
that means pretty much the entire planet.

But all this magic takes a lot of behind-the-scenes work. Its easy to forget that, under all the abstraction, ultimately
matter or energy is getting moved all the way from point A to point B, in a path that you could trace if you were
inclined to.

Along the way, that data often sits on someone else's server. And increasingly, modern culture is discovering all that
implies, both good and bad. As it turns out, a lot of big cloud providers, while saying niceties like "encryption at
rest", "industry-leading security" and more, reading the privacy policies reveals all the major free services are not
only able to see your data, they are methodically combing through it at the behest of advertisers, governments, and in
some really bad cases, curious employees.

There are a lot of really great tools to limit this. I would comfortably say that any major provider is *probably* not
snooping (beyond some [creative ways](https://www.apple.com/child-safety/pdf/CSAM_Detection_Technical_Summary.pdf)ᵖᵈᶠ 
of "checking without looking" for stuff like illegal material). But do we really have to just stop at mights and maybes? 
What if we could short-circuit this whole thing and make the magical transfers happen without the ugly in-between
process?

## My first try: Nextcloud

If you really want the whole 9 yards of collabrative document mangement, editing, sharing, etc., that a fully-featured
online productivity suite can offer (think Google Drive), the main name in the do-it-yourself space has got to be
[Nextcloud](https://nextcloud.com/).

I ran my own Nextcloud server on a home computer, and then a purpose-built NAS, for a few years. But I need to worry
about sharing files only once in a blue moon, and I was using a lot of resources to run this thing. It is also brittle;
you cannot set-and-forget this sort of thing. 

I could manage it with only an hour here or there of work, but getting computers to do stuff is my day job. I sat down
and reconsidered what I really *needed*, and got this:

1. I need to be able to easily move an arbitrary file or set of files from computer A to computer B
1. I need to be able to occasionally share moderately large files with friends and family; not *huge*, but enough that
   an email attachment is pushing it.
1. I want my data to persist in a way that even if all my computers simultaneously became unusable or unaccessable, I
   could recover my files
1. I want all this to happen without requiring upkeep or getting in my way

Nextcloud is pretty good at the first and second items; it is pretty darn close in feature parity and usage to Google
Drive. The third item is a problem, since it is running on a computer in my house, that is a single point of failure.
And the forth item? Forget it! I have had to learn way too much about nginx configuration, php-fpm sockets, in-memory
caching, databases... This is something that is really only worth doing if you have an organization that wants its own
private cloud.

Nextcloud does point you toward [pre-existing free providers](https://nextcloud.com/signup/), and I guess I'm a bit more
comfortable knowing that if I really, *really* wanted to I could look at their source code, but this seems like going
back to Google Drive with a nice coat of paint.

## The Current Setup

Luckily, I found [SyncThing](https://syncthing.net). On the surface, this tool offers a much smaller set of features.
However, when I considered how much I used all the other bells and whistles outside of file sync on Nextcloud, I expect
I won't mind. What I stand to gain is much better.

SyncThing seeks to replicate the original promise of a tool like DropBox: you have a folder or folders that you want to
exist on more than one device. When you modify, add, or remove a file in one place, the other places update to reflect
that. There are some more tweaks you can do if you want to get in the weeds, but this is the out of the box experience.

![The SyncThing Logo](syncthing-logo.svg)

SyncThing by itself is just a light program running in the background, configurable by going to a special "localhost"
address in your browser (essentially, you are going to a website that you are running inside your own computer).
SyncThing's official documentation reccomends instead choosing one of the community tools that offers native desktop
support:

* For Windows, there is [SyncTrayzor](https://github.com/canton7/SyncTrayzor/releases/latest)
* For macOS; [syncthing-macos](https://github.com/syncthing/syncthing-macos/releases/latest)
* And Linux as always has a plethora of options. The most popular seems to be the cross-platform
  [syncthingtray](https://github.com/Martchus/syncthingtray)

There are others as well, including for iPhone and Android, listed in the SyncThing documentation: https://docs.syncthing.net/users/contrib.html#contributions

Using SyncThing is disarmingly straightforward. I was sure I missed a step setting it up, but nope! I just install a
copy of SyncThing on the computers I want to distrubte files amonst. Then, to get the party started you exhange each
device's ID to each other device you want. It seems like a pain at first, but once you realize this is legitimately a
one-time thing it isn't much different than logging in manually to DropBox. Plus, once you've trusted one machine you
can mark it as a trusted "introducer", and it can *automagically*, well,
"[introduce](https://docs.syncthing.net/users/introducer.html)" your machine to the other machines
it knows, allowing zero-configuration propagation of a file through a connected mesh of machines.

You can then go to any of the connected devices and select a folder, then share it with a few clicks. It will now
continuously ensure anything you change, anywhere, is made the same *everywhere*.

There are plenty more awesome tweaks you can do from here. If, like me, you are a [massive geek and have a dedicated
computer](reddit.com/r/homelab) for storage and other miscellany, you can tell that machine to only recieve changes from
other devices and *not* to broadcast its changes otherwise. As noted in the docs, "This mode is useful for replication
mirrors, backup destinations, or other cases where no local modifications are expected or allowed", which is exactly
what I've done with my own setup.

On the other hand, if you have a primary computer that you do most of your work on, and you just want to be able to
reference but not nessessary change the files on other devices, you can choose to mark *that* computer to only send
changes, ignoring changes on other devices.

Each device can choose to keep a certain number of local versions of files, too: if you accidentaly delete or modify a
file and want to undo it, this setting will let you restore an older version.

This is all extra, though. The default "send & recieve" setup is going to be just fine for 95% of people.

<br>

The best way, ultimately, to find out if you're comfortable making this sort ot change is to try it in a safe and
limited manner. You can install it on two devices, and make a new folder somewhere (really, it can be _anywhere_) called
"synced files", then share it. Just try dropping something in the folder and looking on the other device.

Keep that going for a while (maybe copy everything from your DropBox or iCloud folder into it), then adjust and add as
you feel comfortable.

---

If you go back to my short list of needed things, I feel I can comfortably cross off item 1 with SyncThing. Item 2 is
possible as well, but if I don't want to deal with setting up a folder for a single transfer, my favorite option is
[Bitwarden Send](https://bitwarden.com/products/send/). I get the full features for free by running my own instance, but
this is legitimately one tool I'm more okay using the hosted version of: by its nature, these files are meant to be
emphemeral and auto-delete after a preset number of days of your choosing. That is more comfortable to me than letting
years of random stuff (including tax returns at one point, yikes!) just accumulate in someone else's computer, which is
what the cloud ultimatly is.

The 3rd point has a few potential solutions, from the super-manual (every month, copy the files to a USB SSD, stick that
SSD in a safe deposit box) to the completely hands-off (pay for an encrypted backup service).

This has a few potential solutions, and I will cover those in the next post, along with my personal solution.

Until then, be well, be safe, and be wise with your data.

<br>
All the best,
Quentin
