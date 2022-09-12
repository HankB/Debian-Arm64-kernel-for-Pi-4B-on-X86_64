# Debian Arm64 kernel for Pi 4B on X86_64

Instructions to build a Debian kernel package for Raspberry Pi 4B hosted on an X86_64 Debian host.

## Motivation

I'd like to contribute to the Debian on Raspberry Pi effort and one way to do that is to test the upcoming release. At present I have Bookworm running on a Pi 4B and with the 5.19 kernel in the repo, it has some significant problems. It has been suggested that I try the latest updates to 5.19 kernel to see if the issues have been resolved.

I have built a Debian kernel in the past on a Pi 4B (running from an SSD) and it took about 24 hours. I'm striving to do that on my desktop PC (equipped with I7-4770K, 32GB RAM and SSD storage.) My first attempt that ran to completion took about 1 hour. Unfortunately it does not complete boot. It also does not respond to a USB keyboard. I suspect that USB support is missing. This is most likely a result of stuff missing from the configuration.

The kind and patient folk at the `#debian-raspberrypi` IRC have been helping me with this but I'm afraid I'm a bit too lost on this to be able to proceed with the kind of interaction that IRC is suited for. Therefore I'm putting this up on Github to describe what I've tried, the results I achieved and what do do differently to achieve success.

In exchange for the help I get, I will commit to maintaining this description so that it remains current and can become a helpful source of information for anyone who endeavors to do the same.

And lastly, if there is already a ready source of this information packaged in a way to make it accessible to someone who is not familiar with building kernels, please point me to it! I've perused [chapter 4 of the kernel handbook](https://kernel-team.pages.debian.net/kernel-handbook/ch-common-tasks.html) and was not able to follow it to accomplish the task at hand.

## Status

* Built a kernel - Yay!
* Does it boot - No!
