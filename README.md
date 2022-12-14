# Debian Arm64 kernel for Pi 4B on X86_64

Instructions to build a Debian kernel package for Raspberry Pi 4B hosted on an X86_64 Debian host.

## Motivation

I'd like to contribute to the Debian on Raspberry Pi effort and one way to do that is to test the upcoming release. At present I have Bookworm running on a Pi 4B and with the 5.19 kernel in the repo, it has some significant problems. It has been suggested that I try the latest updates to 5.19 kernel to see if the issues have been resolved.

I have built a Debian kernel in the past on a Pi 4B (running from an SSD) and it took about 24 hours. I'm striving to do that on my X86_64 desktop PC My first attempt that ran to completion took about 1 hour. Unfortunately it does not complete boot. It also does not respond to a USB keyboard. I suspect that USB support is missing. This is most likely a result of stuff missing from the configuration.

The kind and patient folk at the `#debian-raspberrypi` IRC have been helping me with this but I'm afraid I'm a bit too lost on this to be able to proceed with the kind of interaction that IRC is suited for. Therefore I'm putting this up on Github to describe what I've tried, the results I achieved and what do do differently to achieve success.

In exchange for the help I get, I will commit to maintaining this description so that it remains current and can become a helpful source of information for anyone who endeavors to do the same.

And lastly, if there is already a ready source of this information packaged in a way to make it accessible to someone who is not familiar with building kernels, please point me to it! I've perused [chapter 4 of the kernel handbook](https://kernel-team.pages.debian.net/kernel-handbook/ch-common-tasks.html) and was not able to follow it to accomplish the task at hand.

## Status

* Built a kernel - Yay!
* Does it boot - Yes!

## Environment

Host is a desktop with Intel I7-4770K, 32GB RAM and SSD storage running Debian Bullseye. I also have a laptop with I7-8550U CPU and 16GB RAM running Bookworm. Either of these can be used for hosting the build.

## Questions

* From `kibi` on IRC
> I recommend this: git clone https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git (which by default will be known as 'origin') then git remote add stable https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git && git fetch stable

```text
git clone https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
cd linux
git remote add stable https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git
git fetch stable
```

* How should I be managing configuration? (e.g. `.config`) - Copy config from a working system (in `/boot` for Debian/Pi) and

```text
make ARCH=arm64 defconfig
```

## Procedure

### Install toolchain

```text
apt install libssl-dev libelf-dev zlib1g-dev lz4 gcc-10-aarch64-linux-gnu gcc-arm-linux-gnueabihf ccache kernel-wedge dwarves wget crossbuild-essential-arm64 flex bc
```

### fetch kernel source

This can come from two sources (at least.) A tarball for a particular release can be downloaded from kernel.org. For example

```text
wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.19.8.tar.xz
tar xf linux-5.19.8.tar.xz
cd linux-5.19.8
```

A local copy can be cloned from a remote repo when trying to narrow down a commit that introduced a bug. (e.g. when using `git bisect ...`)

```text
git clone https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
cd linux
git remote add stable https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git
git fetch stable
```

Copy the config from a Pi running 5.19.0 on Bookworm. This will be in /boot on the running Pi. Save the file in the directory created when unpacking the linux source (e.g. `.../linux-5.19.8`) and make a copy (or rename) to be used to guide the build.

```text
cp config-5.19.8 .config
```

Adjust path and kick off the build

```text
PATH=/usr/lib/ccache:$PATH
time -p make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- bindeb-pkg -j$(nproc)
```
[First try notes](try-1.md)

### Try 2

```text
hbarta@olive:~/Downloads/linux-5.19.8$ make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- bindeb-pkg -j8
sh ./scripts/package/mkdebian
dpkg-buildpackage -r"fakeroot -u" -a$(cat debian/arch)  -b -nc -uc
dpkg-buildpackage: info: source package linux-upstream
dpkg-buildpackage: info: source version 5.19.8-1
dpkg-buildpackage: info: source distribution bullseye
dpkg-buildpackage: info: source changed by hbarta <hbarta@olive>
dpkg-buildpackage: info: host architecture arm64
 dpkg-source --before-build .
 debian/rules binary
make KERNELRELEASE=5.19.8 ARCH=arm64    KBUILD_BUILD_VERSION=1 -f ./Makefile
  DESCEND bpf/resolve_btfids
  CALL    scripts/atomic/check-atomics.sh
  CALL    scripts/checksyscalls.sh
  CHK     include/generated/compile.h
  GEN     .version
  CHK     include/generated/compile.h
make[4]: Nothing to be done for '__modpost'.
  MODINFO modules.builtin.modinfo
  GEN     modules.builtin
BTF: .tmp_vmlinux.btf: pahole (pahole) is not available
Failed to generate BTF for vmlinux
Try to disable CONFIG_DEBUG_INFO_BTF
make[3]: *** [Makefile:1168: vmlinux] Error 1
make[2]: *** [debian/rules:7: build-arch] Error 2
dpkg-buildpackage: error: debian/rules binary subprocess returned exit status 2
make[1]: *** [scripts/Makefile.package:83: bindeb-pkg] Error 2
make: *** [Makefile:1553: bindeb-pkg] Error 2
hbarta@olive:~/Downloads/linux-5.19.8$ sudo apt install -t stable pahole
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Some packages could not be installed. This may mean that you have
requested an impossible situation or if you are using the unstable
distribution that some required packages have not yet been created
or been moved out of Incoming.
The following information may help to resolve the situation:

The following packages have unmet dependencies:
 pahole : Depends: libbpf0 (>= 1:0.4.0) but 1:0.3-2 is to be installed
E: Unable to correct problems, you have held broken packages.
hbarta@olive:~/Downloads/linux-5.19.8$ 
```

For Try #3 will try to disable `pahole`. No option. ENV VAR is a value. 0 in .config and 123 in the source config.

Stuck for now.

Per IRC user kibi, `pahole` is also in the `dwarves` package for Bullseye. I installed that and the build completed. It booted and came up and the bugs I experienced previously seemed to be banished. However the SD card timeouts have returned. Bug filed at <https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=1019700>

## Testing

Working this out is fraught with opportunities for errors. Iterative configuration and installation steps can pile up to produce a system that is not easily reproduced. For this reason, the procedure will be tested on another host that is more or less a vanilla install so that the steps and commands can be verified to work as expected and desired.

## Contributing

Please do! Feel free to submit PRs or create issues to convey improvmeents in this process. Any additional information is very much appreciated. And please point out typos, awkward phrasing, anything not abundantly clear and so on.