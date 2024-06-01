---
layout: post
title: "Moving to FreeBSD"
categories: code
---

So I've finally moved my personal computer off to FreeBSD, and away from any Linux-based system. I have my reasons too -

- I want a simpler system to tinker with. At some level, I want a system that forces me to delve beneath the surface. 
- I also want to delve into jails and maybe some virtualisation with bhyve.
- The documentation seems (at least upto now) much clearer and shorter. It invites me to read it more than the standard Linux manpage, that much is certain.

## Simplicity, doas and the Cathedral

I think the overarching theme is simplicity. For example, `doas` has a much simpler configuration than `sudo` (and I know that `doas` can be installed on Linux). The `/etc/doas.conf` to give all members of the familiar `wheel` group super-powers is one line. It might not be quite as safe on production machines to add `nopass`, but for my personal machine, it should be alright. The only caveat is that `keepenv` has to be specified as well, otherwise the current environment will be reset prior to execution.

```bash
# /usr/local/etc/doas.conf
permit nopass keepenv :wheel
```

Naming the package manager `pkg` is comically good - if I want to install a package, I think "package install _thing_", and I type `pkg install thing`. It's great. It makes me think of the Cathedral and the Bazaar - a great little read on design philosophies and open source. The BSDs seem to be firmly in the Cathedral business i.e. everything is well thought out and is organised into a coherent whole.


## Drawbacks

Of course, working on essentially a niche operating system definitely has drawbacks. For instance, hardware support, especially for Wi-Fi is bad. I tried a TP-Link TL-WN881ND, which had spotty connection at best. Switching to a card with an Intel chipset was almost plug and play. Following the [FreeBSD handbook](https://docs.freebsd.org/en/books/handbook/network/) to configure my interface and hten connect was seamless. Lesson learned - when it works, it _really_ works. When it doesn't, Prometheus.

There's also a problem I have with Python packages - not all of them provide wheels for BSDs (I'm not counting macOS for now). For example, a library that I use, [Ray](https://github.com/ray-project/ray/tree/master) (or more specifically, RLlib), can't be installed with a `pip install` and a smile. This gives me a few options, which I will need to thoroughly explore.

Lastly, there's configuration hell. I've fallen prey to the Ricing fever (again), and am now slowly but surely inching towards my perfect, keyboard-centric workflow on this machine. To be honest, configuring the audio (the tools are great, but I didn't know them yet) took me a while since I couldn't find the right sources. I still need to fix a few shortcuts on my keymaps, but otherwise, everything's golden.

## Linux, Python and Compatibility

So, to run my Python programs with all the packages I want, there's a few options I can look at.

- Run everything in the [Linuxulator](https://wiki.freebsd.org/Linuxulator#Linuxulator_.28Linux_Emulation.29). This is essentially `chroot`-ing into what looks like a Debian/Ubuntu userspace. I still need to understand how this works, but I was able to set it up painlessly.
- Run everything in a virtual machine - enter `bhyve`.
- See if some packages can be manually installed, or verify if there's even a way to do that.

In either case, I'll need to learn a few things, and that's the good part :)
