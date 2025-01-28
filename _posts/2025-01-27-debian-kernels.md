---
layout: post
title: "Compiling Custom Kernels for Debian"
categories: code
---

Happy New Year to everyone! I've just upgraded my rusty, trusty old Xeon for a 14-core behemoth (still a Xeon). Considering that I'm a Linux nut, there's only one good way to put all those cores to good use - compile my own kernel. I was using Debian 12 (bookworm) with the `6.1.28` kernel by default, and am now using the latest (at the time of writing, we've moved to the `6.12.11` stable version) stable kernel, `6.12.10`. Are there any big changes ? No. Was it a good learning experience ? Yes.

# Getting the Source and Configuration

Getting the kernel source is the easy part : simply hit the big yellow button on [kernel.org](https://kernel.org/). Now, extract the tarball and `cd` into it.

```bash
(Downloads) $ tar xvf linux-6.12.10.tar.gz
(Downloads) $ cd linux-6.12.10
```

Of course, change the `6.12.10` with the version you have. 

# Basic Steps

The path to running a self-compiled kernel is quite simple:

1. Get the kernel sources.
2. Create a `.config` file using the kernel Makefile. There's a number of helpful Makefile targets to do this.
3. Once generated, this `.config` file will allow us to `make` (or compile and link) the kernel. This also includes any optional modules that the config file specifies.
4. A good cup of coffee later, the kernel should be built. Install this in the right location (`/boot`) + optional modules, generate the `initramfs`, update the GRUB (or other bootloaders) entries to recogize this new kernel.

After (4), you should be good to go. The entire process is something like:

```bash
(linux-6.12.10) $ make menuconfig   # creates the .config file
(linux-6.12.10) $ make              # compiles the kernel
(linux-6.12.10) $ sudo make modules_install  # installs compiled modules
(linux-6.12.10) $ sudo make install  # installs compiled kernel
```

Note that the last two need superuser rights, since we will be writing to `/usr/lib/modules` and `/boot`, both well outside the home directory. Now, the first line uses the `menuconfig` target, which is not the only one (and not the one I used anyway). Now, there's different Makefile targets used to compile the kernel, `menuconfig` being the most commonly used. To list them all, use the very helpful

```bash
(linux-6.12.10) $ make help   # list all Makefile targets
```


## .config

The `.config` file is the configuration file used to build the kernel and associcated modules. It is automatically generated when running `make menuconfig` (or when using similar Makefile targets). An alternative is to re-use the distribution kernel configuration, stored in `/boot` to give us a hot-start.

```bash
(linux-6.12.10) $ cp /boot/config-6.1.0-28-amd64 ./.config
```

The classical way (as far as I've made out) is to use the `menuconfig` target. This will present you with an `ncurses`-style menu with an ocean of options to choose from. This gives you full configurability (and responsibility) on what to include in the kernel. Most options can also be added as a kernel module i.e. they're only added when necessary, such as when the relevant device is plugged in, for example. Now, there's a lot of modules, and a lot of reading to find out if you can skip something or not. I've gone down this route with mixed results, but the main risk is that you'll be compiling a lot of unnecessary modules which will explode your compile times, 14 cores or otherwise.

In order to get a lean, mean kernel with nothing more than needed, I used the `localmodconfig` target. This will try to build only the modules that are currently loaded into your computer - definitely an improvement over manually searching for each installed module. It will prompt you for several optional modules just to be sure, but sticking to the defaults seemed to work just fine for me. 

<div class="remark" text="Debian Specifics">
Since we're on Debian, the OS expects that all modules are signed, or that module signing is explicitly disabled. This is a great security feature, but when compiling my kernel, I did not be sign them. Instead, I disabled module signing using the following command : (after getting a configuration file and before starting the compile process)

```bash
(linux-6.12.10) $ ./scripts/config --file .config --disable MODULE_SIG
```

Refer the [Debian Kernel Team](https://kernel-team.pages.debian.net/kernel-handbook/ch-common-tasks.html#s-common-building) for more information.
</div>


## Compiling and Installing a Kernel

This is the simplest part of the whole process. Running steps (3) and (4) should be just fine. To speed up the compilation by using all available cores on your machine, use the `-j` option as: 

```bash
(linux-6.12.10) $ make -j28
```

This instruction will try to parallelise the compilation using 28 different jobs - I supplied 1 for each thread I have, hence $14 \times 2 = 28$. Do the same for `modules_install` as well. The `install` target will handle putting the kernel in the right place, as well as updating GRUB, so that should be the end of that. If all goes well, you can reboot and use your freshly baked kernel :)


# Post-Op: Firmware Upgrades

Upgrading the kernel meant that I also had to upgrade the firmware of my Wi-Fi driver. When booting up the new kid for the first time, I didn't have any Wi-Fi or Bluetooth etc. Reading through `dmesg` quicklkly showed me the problem : the new kernel expected new firmware to go with it. So, peering through `dmesg`, I saw the very telling 

```bash
(linux-6.12.10) $ dmesg | grep -i Wi-Fi
...
[    9.954457] iwlwifi 0000:0b:00.0: Detected Intel(R) Wi-Fi 6 AX200 160MHz
[    9.954804] iwlwifi 0000:0b:00.0: Direct firmware load for iwlwifi-cc-a0-77.ucode failed with error -2
[    9.954812] iwlwifi 0000:0b:00.0: no suitable firmware found!
[    9.954840] iwlwifi 0000:0b:00.0: iwlwifi-cc-a0-77 is required
[    9.954859] iwlwifi 0000:0b:00.0: check git://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git
...
```

Once I knew which file to go for, I basically had to download the firmware (preferably from [Debian sources](https://packages.debian.org/trixie/firmware-iwlwifi)), and copy/move the `iwlwifi-cc-a0-88.ucode` file to `/usr/lib/firmware`. This fixed my problem, and hopefully, you shouldn't have something more complicated than this to do.


# Conclusions

So, the results of this experiment:

1. A brand-new, shiny _stable_ kernel on a Debian system.
2. `dmesg` is a friend.
3. Knowing where the system firmware is stored ([_insert evil smile_](https://tenor.com/view/evil-smile-gif-9846496955827135924)).

A nicer, Debian specific guide was listed [here](https://debian-handbook.info/browse/stable/sect.kernel-compilation.html). I'll look at it later.
