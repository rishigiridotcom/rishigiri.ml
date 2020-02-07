---
layout: post
title: "Kworker and High CPU Usage"
comments: true
keywords: "linux, kernel, acpi, kworker, cpu, fix"
---

I recently upgraded my system and added 8 GB of RAM along with 250 GB of SSD (1TB HDD is in Optical Bay) to improve its performance. My laptop was total garbage, and it still is, but at least now, I can open 10 Chrome Tabs and 2 Text Editors without much pain in the ass.

After the fresh installation of Ubuntu, I noticed that while browsing, my laptop's fan was making way too much noise. It was odd, as it never happened before, especially while browsing.

To check what's going on, I did -

```sh
$ top
```

and got the following results - 

```
 PID  USER    PR  NI     VIRT     RES     SHR  S  %CPU  %MEM     TIME+  COMMAND                                     
 1053 root    20   0        0       0       0  S 100.0     0  95:13.18  kworker/0:1                                         
 2462 rishi   20   0  1460984  131336   70080  S   3.3   1.1  35:36.86  compiz                                      
```

To be honest, I had no idea about what __kworker__ was, but it was literally hogging the CPU usage, so I googled, and got the below answer from [here](https://askubuntu.com/questions/33640/kworker-what-is-it-and-why-is-it-hogging-so-much-cpu)

> __kworker__ is a placeholder process for kernel worker threads, which perform most of the actual processing for the kernel, especially in cases where there are interrupts, timers, I/O, etc. These typically correspond to the vast majority of any allocated "system" time to running processes.

Fortunately, a few days back, I was reading about the [ACPI Modules](https://wiki.archlinux.org/index.php/ACPI_modules) (Advanced Configuration and Power Interface), and it reminded me of the list of ACPI Kernel modules, in which, one of them was related to the __Bay (bay status)__.

> What is ACPI?
> <br><br>
> I don't know the exact definition, but it can be described as a way (bridge) through which an Operating System and Hardware communicates.

Anyway, [Wikipedia](https://en.wikipedia.org/wiki/Advanced_Configuration_and_Power_Interface) says - 

> ACPI provides an open standard that operating systems can use to discover and configure computer hardware components, to perform power management by (for example) putting unused components to sleep, and to perform status monitoring.

Since I was using the DVD slot (Optical Bay/Drive) for my Hard Disk, I thought the issue might be related to the same thing, so I took it out, and __BOOM__. The issue got resolved!

__Well, not actually.__

It was a temporary solution, as inserting the caddy back into the slot made kworker go crazy, and the CPU usage got back to 100%.

__Conclusion:__ Lots of ACPI interrupts.

__Possible reason:__ Hard drive being used through the Optical bay, and since the issue had everything to do with the ACPI, which means, GPE was telling OS (via ACPI) that something is happening. Probably transfer of events via Embedded Controller to GPE in huge number, per second.

Now, to understand the issue, I started digging further.

```sh
$ ls /sys/firmware/acpi/interrupts/
```
There were lots of files like `error`, `ff_gbl_lock`, `gpe01, gpe02, gpe03... gpe0n`. Separately `echoing` every file would have taken a lot of time, although I checked around 5-10 files manually. For this task, `grep` was what I needed.

```sh
$ cd /sys/firmware/acpi/interrupts/

$ grep '' -r 
```

```
gpe7B:       0         invalid    unmasked
gpe0F:       0   EN    enabled    unmasked
gpe03:   43946   STS   enabled    unmasked
gpe3D:       0         invalid    unmasked
```

The list was long, but I spotted that `gpe03` had the highest value. Since I'm not aware of these things, my best guess was that those numbers are the events which are and being sent through Embedded Controller (EC) to the Operating System. Yes, the numbers kept increasing. Anyway, I tried - 

```sh
$ echo "disable" > gpe03
```

And to my surprise, it fixed the problem. The fan started slowing down, and eventually, it stopped making any noise. 

__Big win?__

The changes made to `gpe03` wasn't permanent, so rebooting the system brought it back to its original state which ended up letting kworker eat the CPU.


I stumbled upon [this post on Ask Ubuntu](https://askubuntu.com/questions/176565/why-does-kworker-cpu-usage-get-so-high). The question was asked 7 years ago!

The solution was to add `@reboot echo "disable" > /sys/firmware/acpi/interrupts/gpe03` to `crontab`.

```sh
$ export EDITOR=vim
$ sudo crontab -e
```

I edited the crontab by adding -

```sh
@reboot echo "disable" > /sys/firmware/acpi/interrupts/gpe03
```

```sh
$ reboot
```

__NOTE__

- If you forget to add newline at the end of `crontab` file, it won't work.

- It's important to `sudo crontab -e` instead of just `crontab -e`. You need to be `root` to update changes in system files.

<br>

__Links__

- [ACPI Modules](https://wiki.archlinux.org/index.php/ACPI_modules)
- [Kworker - What is it, and why it's hogging so much CPU](https://askubuntu.com/questions/33640/kworker-what-is-it-and-why-is-it-hogging-so-much-cpu)
- [Origin of a kworker thread](https://stackoverflow.com/questions/10846747/origin-of-a-kworker-thread)
- [Is an ACPI GPE storm normal in Ubuntu session?](https://askubuntu.com/questions/149028/is-an-acpi-gpe-storm-normal-in-an-ubuntu-session?noredirect=1&lq=1)
- [Why kworker CPU usage so high?](https://askubuntu.com/questions/176565/why-does-kworker-cpu-usage-get-so-high)
- [Grep](https://www.gnu.org/savannah-checkouts/gnu/grep/manual/grep.html)