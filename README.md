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
* Does it boot - No!

## Environment

Host is a desktop with Intel I7-4770K, 32GB RAM and SSD storage running Debian Bullseye. I also have a laptop with I7-8550U CPU and 16GB RAM running Bookworm. Either of these can be used for hosting the build.

## Questions

* Where should I be getting the desired kernel source?
* How should I be managing configuration? (e.g. `.config`)

## Procedure

* Install toolchain
  * apt install libssl-dev libelf-dev zlib1g-dev lz4 pahole gcc-10-aarch64-linux-gnu gcc-arm-linux-gnueabihf ccache kernel-wedge
* Download the tarball from https://www.kernel.org/ for 5.19.8 and unpack.
* Copy the config from a Pi running 5.19.0 on Bookworm (Actually from the SSD mounted on the desktop)

```text
hbarta@olive:~/Downloads/linux-5.19.8$ cp /mnt/sdg2/boot/config-5.19.0-1-arm64 .config
```

* Add to PATH `PATH=/usr/lib/ccache:$PATH`
* Execute build command `make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- bindeb-pkg -j8`

## Result 

The build asked a few more questions to which I generally answered 'n' or accepted the default selection. Signing modules concerned me a bit. The config has been added to this repo.

Broke on `No rule to make target 'n', needed by 'certs/x509_certificate_list'` set to 'n' (should be ''.)

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
make[4]: *** No rule to make target 'n', needed by 'certs/x509_certificate_list'.  Stop.
make[4]: *** Waiting for unfinished jobs....
make[3]: *** [Makefile:1847: certs] Error 2
make[3]: *** Waiting for unfinished jobs....
  CHK     include/generated/compile.h
make[2]: *** [debian/rules:7: build-arch] Error 2
dpkg-buildpackage: error: debian/rules binary subprocess returned exit status 2
make[1]: *** [scripts/Makefile.package:83: bindeb-pkg] Error 2
make: *** [Makefile:1553: bindeb-pkg] Error 2
hbarta@olive:~/Downloads/linux-5.19.8$ 
```

Solution: `make menuconfig` and navigate
* `Cryptographic API`
  * `Certificates for signature checking` (hint: last option)
    * `Additional X.509 keys for default system keyring` And set to '', save and exit.

This appears to have restarted config with an empty slate. :-/

```text
  SYNC    include/config/auto.conf.cmd
*
* Restart config...
*
*
* Platform selection
*
Actions Semi Platforms (ARCH_ACTIONS) [N/y/?] (NEW) 
.
.
.
```

Copied working config as above and restarted make. When it camt to

```text
    Automatically sign all modules (MODULE_SIG_ALL) [Y/n/?] (NEW) n
```

Answer 'n' to sidestep the certificate issue. There were a few other questions that I left default.

## Contributing

Please do! Feel free to submit PRs or create issues to convey improvmeents in this process. Any additional information is very much appreciated. And please point out typos, awkward phrasing, anything not abundantly clear and so on.