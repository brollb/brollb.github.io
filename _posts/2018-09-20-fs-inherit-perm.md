---
layout: post
title:  "Changing default file permissions (linux)"
date:   2018-9-20 20:06:00
categories: linux
---

Ever wanted to change the default file permissions in linux? Maybe there is a directory in which every file should be writable by members of the "users" group (not something I would recommend)? Well, now you can! (Ok, this wasn't meant to sound so much like an infomercial but I am kinda tired and I guess these things just kinda take a life of their own...)

With [Access Control Lists](https://wiki.archlinux.org/index.php/Access_Control_Lists), you can! All you have to do is use `setfacl` to set the permission to be inherited by files of the given directory like so:

    setfacl -dm "g:users:rwx" my-shared-directory

where `"g:users:rw"` is setting the file default permissions to be readable and writable to all members of the users group for the `my-shared-directory` directory. Alright, I am going to try to stop sounding like an infomercial... Access control lists seem like they could be useful and can be kinda tricky to find info on since many of the related questions on the forum are often more simple use cases such as changing the permissions or group of a file/directory. Another useful command for those who want really permissive files and dirs (it sets the permissions for all others):

    setfacl -dm "o:rwx" my-shared-directory

After changing the defaults with `setfacl`, the new defaults can be viewed with

    getfacl my-shared-directory

where `my-shared-directory` is the directory in question.

For more detailed info on ACL, check out the [Arch Wiki](https://wiki.archlinux.org/index.php/Access_Control_Lists).
