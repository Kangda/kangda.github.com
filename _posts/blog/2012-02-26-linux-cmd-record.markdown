---
date: 2012-02-26 07:01:29+00:00
layout: post
title: Linux操作积累
categories: blog
tags:
- gnome
- linux
description: 一些Linux上的常用操作
---

## GNOME


alt+F2

gnome-session-properties   开机启动项设置



Q:Gnome3.4.1 的gnome-control-center（system setting）无法启动

A：

This regression occurs with clutter-gtk-1.2.0 or cheese-3.4.0. You can fix it by installing cheese from the extra repsitory:




    
    pacman -S extra/cheese





Or downgrading clutter:




    
    pacman -U /var/cache/pacman/pkg/clutter-gtk-1.0.4-1-x86_64.pkg.tar.xz







Q:Gnome3.4.1里设置快捷键win键无效

A：（临时，这样的修改后，需要按两次字母键才能调用，还得等这个bug修复..）

The dconf path is `org/gnome/settings-daemon/plugins/media-keys`. The predefined shortcuts live there. Custom shortcuts live further down under `custom-keybindings/custom0` (or custom1, and so on).

Changing `<Super>` to `<Mod4>` in my shortcuts fixed the problem.




## Fedora


Q：开机启动慢，总是卡在Starting LSB:Start and Stop sendmail...之类的提示上

A：关闭sendmail服务，这个貌似是做邮件服务器用的。

    
    /sbin/chkconfig --level 0123456 sendmail off





## MPLAYER


Q：如何实现双字幕？

A：首先，在~/.mplayer/config中添加设置

    
    overlapsub=1


然后，将要使用的两个字幕作如下合成操作：

    
    cat subA.srt subB.srt > subC.srt


然后播放subC即可


## KChmViewer


chm阅览器，即使在Gnome环境下也能非常好的完成任务，比仅限于Gnome环境的Gnochm好很多，GnoChm对某些chm文件会出现崩溃或者目录失效的情况
