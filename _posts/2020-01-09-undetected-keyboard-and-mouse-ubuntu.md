---
layout: post
title: "Dead Keyboard and Mouse - Xorg Issue"
comments: true
keywords: "keyboard, mouse, ubuntu, xorg"
---

I've been seeing `/dev/sda1: clean` followed by `{x} files, {n} blocks` from a long time, and every time I boot my system, the message appears.

There was a time when the screen used to get stuck at this point and I had to restart my system and press `Enter` key number of times so that the message can be skipped, and I can login. I got used to this habit of pressing Enter key. Never tried fixing the problem. Never bothered googling what's the matter. It didn't seem important to me. I'm lazy.

I cleaned up my system recently, but before that, I also had an issue and it used to appear along with the above message. The screen used to get filled with `^[[5~`

```sh
/dev/sda1: clean {x} files, {n} blocks

^[[5~^[[5~^[[5~^[[5~^[[5~^[[5~^[[5~^[[5~^[[5~^[[5~^[[5~^[[5~
^[[5~^[[5~^[[5~^[[5~^[[5~^[[5~^[[5~^[[5~^[[5~^[[5~^[[5~^[[5~
^[[5~^[[5~^[[5~^[[5~^[[5~^[[5~^[[5~^[[5~^[[5~^[[5~^[[5~^[[5~
```

- Two things

    - Even after using Ubuntu for such a long time, I had no idea what was happening.

    - After the fresh installation, the message didn't disappear.

- Note: I solved the keypress issue [by disabling the PageUp key, forever.](2019-11-10-keyboard-issue.md)

I think my laptop's keyboard has some fault, but I'm not totally sure about it.

__Coming back to the real issue__ - I googled the problem and stumbled upon [this question on Ask Ubuntu](https://askubuntu.com/questions/882385/dev-sda1-clean-this-message-appears-after-i-startup-my-laptop-then-it-w). The OP had the same issue. 

One of the comments was - 

> This is a normal startup message. It lets you know that there are no filesystem errors.

There were some answers which pointed out that the issue is caused by the graphics driver, in their case - it was nvidia. 

__My system has Intel HD Graphics!__ 

After a couple of days, my system crashed after logging in, and when I was checking the `logs`, I found out the `Xorg` had some issue. I don't exactly remember what the error was, but after checking the crash reports, I saw 

```sh
Xorg crashed with SIGABRT in OsAbort()?
```

I tried to fix it by doing (reinstalling the input drivers) - 

```sh
sudo apt-get install --reinstall xserver-xorg-input-all
```

Next day, when I switched on the laptop, I wasn't able to type the password. At first, I thought the problem is in my external keyboard, so I unplugged it, but this didn't solve the problem. 

I switched on and off my laptop several times hoping that somehow the problem will get fixed (hello?), but there was no luck!

### Attempt 1

- Press `Ctrl + Alt + F1`

- Run `sudo apt update && sudo apt upgrade`

__Problem__

Login screen was completely blank. No prompt to enter `username` and `password`.

### Attempt 2

I stumbled upon [this blog post](https://techwiser.com/fix-keyboard-not-working-in-ubuntu-18-04/), which said

- Go to the GRUB menu

- Edit the boot option

- Insert `/bin/bash` before `$vt_handoff` in `linux /boot/vmlinuz-4.18.0-25-generic root=UUID=a98c605-2ac4-4ee3-8070-2560255293fe ro quiet splash $vt_handoff`

- Run `sudo apt-get install xserver-xorg-input-all`

__Problem__

The solution didn't work. Pressing `Ctrl + Alt + F1` was useless, because again, nothing appeared!

### Attempt 3 

> This worked for me!

I didn't try to read other answers. Wanted to try my luck.

- Booted into __Recovery Mode__

- Enabled `networking`

- Selected `repair broken packages`

- Resume normal boot.

__Issue Fixed!__

I don't exactly remember, but packages related to `Xorg` were broken. `dpkg` fixed it.
