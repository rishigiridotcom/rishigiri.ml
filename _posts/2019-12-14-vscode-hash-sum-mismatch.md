---
layout: post
title: "Visual Studio Code - Fix Hash Sum Mismatch"
comments: true
keywords: "vscode, apt-update, ubuntu, apt, fix, Hash Sum Mismatch, error, fix"
---

Yesterday, I did `$ sudo apt update` and I ended up getting the following error - 

```sh
Err:25 http://packages.microsoft.com/repos/vscode stable/main amd64 Packages
Hash Sum mismatch
```

To be honest, I had no idea what does this error even stands for, so I searched for __`ubuntu apt-update hash sum mismatch fix`__ on google, and got the following results - 

- [Trouble downloading packages list due to a “Hash sum mismatch” error](https://askubuntu.com/questions/41605/trouble-downloading-packages-list-due-to-a-hash-sum-mismatch-error)
- [How do you fix apt-get update “Hash Sum mismatch”](https://unix.stackexchange.com/questions/116641/how-do-you-fix-apt-get-update-hash-sum-mismatch)

One of the users wrote that the easiest fix would be -

```sh
$ sudo apt clean
$ sudo apt update
```

Another [fix](https://askubuntu.com/a/41618/806290), according to a user, was to remove the contents of `/var/lib/apt/lists` and run `sudo apt update`. 

A comment on that answer suggested - 

> If you remove all files, you have to download them again. You can just remove the invalid file to make this process faster.


## __Part 1__

Now, I had two options to fix the issue, but for some reasons, I didn't want to remove the contents of `/var/lib/apt/lists`, so I started with -

```sh
sudo apt clean
sudo apt autoclean
sudo apt update
```

Unfortunately, the solution didn't work. I was still getting the `hash sum mismatch` error. 



## __Part 2__

Removing the contents of `/var/lib/apt/lists`.

Since the source of problem was Visual Studio Code, it was completely unnecessary to remove all the files. So, I just had to delete the files which were related to vscode -

- __Search for files__

```sh
cd /var/lib/apt/lists
ls | grep vs
```
```sh
packages.microsoft.com_repos_vscode_dists_stable_InRelease
packages.microsoft.com_repos_vscode_dists_stable_main_binary-amd64_Packages
```

- __Enter superuser mode and delete files__

```sh
sudo su # or just su
cd /var/lib/apt/lists
rm packages.microsoft.com_repos_vscode_dists_stable_InRelease
rm packages.microsoft.com_repos_vscode_dists_stable_main_binary-amd64_Packages
```

- __Clean, remove, and update__

```sh
sudo apt clean && sudo apt autoclean && sudo apt remove && 
sudo apt autoremove && sudo apt update # I know. Please don't judge.
```

__ISSUE__

> The problem didn't disappear.

It was still getting the same error while updating. Only this time, I read the error carefully.


- __Failed to fetch http://packages.microsoft.com/repos/vscode/dists/stable/main/binary-amd64/Packages.bz2__
- Hash Sum mismatch
- E: Some index files failed to download. They have been ignored, or old ones used instead.


## __Part 3__

This part is all about hit and trial, and trusting my instincts alongside!

```sh
Failed to fetch http://packages.microsoft.com/repos/vscode/dists/stable/main/binary-amd64/Packages.bz2
```

The above error made me check if `Packages.bz2` file exists in `/var/lib/apt/lists`. So, I checked - 

```sh
ls /var/lib/apt/lists | grep Packages.bz2
```

__Result -__ Nothing. __Packages.bz2 was missing!__ Good News?


__My initial thought -__ Add `Packages.bz2` to `/var/lib/apt/lists` and run `sudo apt update`.


__Problem -__ I had already deleted the two files. - 

```sh
packages.microsoft.com_repos_vscode_dists_stable_InRelease
packages.microsoft.com_repos_vscode_dists_stable_main_binary-amd64_Packages
```

__How to get those files back?__

The files available in `/var/lib/apt/lists` are in the form of `site.com_foo_bar_buzz`. Now, if you replace __`_`__ with __`/`__, you'll get - 

```
site.com/foo/bar/buzz
```

So,

```
packages.microsoft.com_repos_vscode_dists_stable_InRelease
```
will become - 
```
packages.microsoft.com/repos/vscode/dists/stable/InRelease
```

Since two of the files related to VSCode were already gone, I had to download them and put them back to their place. Kudos to `history`.

```sh
history | grep packages.microsoft
```

I also needed the `Packages.bz2` file, so I grabbed the downloadable link from the `error message`.

```
http://packages.microsoft.com/repos/vscode/dists/stable/InRelease
http://packages.microsoft.com/repos/vscode/dists/stable/main_binary-amd64_Packages
http://packages.microsoft.com/repos/vscode/dists/stable/main/binary-amd64/Packages.bz2
```

- __Step 1__ - Download the files.

- __Step 2__ - Rename files. The name of the files should be similar to the url they were obtained form, except `/` will be replaced by `_`. __InRelease__, __Packages__, and __Packages.bz2__ will become -

```
packages.microsoft.com_repos_vscode_dists_stable_InRelease
packages.microsoft.com_repos_vscode_dists_stable_main_binary-amd64_Packages
packages.microsoft.com_repos_vscode_dists_stable_main_binary-amd64_Packages.bz2
```

Now, move those files to `/var/lib/apt/lists`

```sh
sudo su
mv packages.microsoft.com_repos_vscode_dists_stable_InRelease /var/lib/apt/lists
mv packages.microsoft.com_repos_vscode_dists_stable_main_binary-amd64_Packages /var/lib/apt/lists
mv packages.microsoft.com_repos_vscode_dists_stable_main_binary-amd64_Packages.bz2 /var/lib/apt/lists
```

- You can move them all together. It doesn't matter.

Run `sudo apt update`.

This fixed the whole `Hash Sum Mismatch` error for me!.

I had no idea if it would work or not. I attempted to solve a problem, and everything went fine.

__Links__

- [Why hash sum match error occurs?](https://unix.stackexchange.com/a/273441/367425)
- [What's the difference between su and sudo?](https://www.howtogeek.com/111479/htg-explains-whats-the-difference-between-sudo-su/)