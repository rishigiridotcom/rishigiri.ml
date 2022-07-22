---
layout: post
title: "Keyboard Issue"
comments: true
keywords: "keyboard, xmodmap, xev"
---

I use three keyboards, and just now, after spending almost 3 hours, I figured out the issue and fixed it. The problem was only in one of them, but it used to appear so randomly - it was difficult for me to find out which keyboard is causing the whole matter.

Since the last couple of months, on random days, I used to face this problem as the pages automatically get scrolled up to the top. Lately, this started to happen daily. At first, I thought,  the mouse is causing the problem, but after smashing it on the ground, I realised that the mouse is completely fine.

Anyway, the problem used to appear randomly, and I didn't pay much of the attention towards it, as whenever it happened, I ended up poking keys with my index finger, and everything used to work fine after that.

Today, things went out of control. I had no idea what was happening, and hitting the keys wasn't doing the job. I was reading the source code of `dmidecode` and trying to understand things, but in the middle of everything, when I opened the terminal to build the program from source, all the previous commands started to appear in the reverse order. Somehow I was able to `make`, but one of the keys were causing so much of the issue that the whole terminal got filled with __`^[[5~`__. I wasn't able to do anything.

I searched for __[Key binding table in Linux](https://unix.stackexchange.com/questions/116562/key-bindings-table)__ and from here, I found about `^[[5~`. It's a "key-binding" for `Page Up`, I guess. I'm not completely sure if I'm using the right term as I don't know much about it. I'll learn more about these things later.

As now I was aware of the problematic key, I just had to disable it to fix the issue, and I did.

With the help of `xev`, I found the `keycode` of `Page Up`, and then I used `xmodmap` to disable the key.

- `$ xmodmap - 'keycode 112='`

- `$ echo 'xmodmap -e "keycode 112="' >> ~/.profile`

It's been almost an hour and everything is working fine.
