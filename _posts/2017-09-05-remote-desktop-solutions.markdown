---
layout: post
title:  "Remote Desktop Solutions"
date:   2017-09-05 11:12:00 +0800
categories: Productivity
tags: Linux VNC x11vnc
---
Remote desktop is extremely useful when you want to have a continuous work experience, that is, work everywhere with the same environment. This post shares my own experience on remote desktop among various usage cases.

# Case 1: Desktop on Shared Server

I started working on a desktop server in the lab since 2016. During that time, the server was of the highest performance among the machines in the lab so everyone wanted to use it. The server was a Linux machine so SSH would be fine, however I found it insufficient when I wanted to handle some large projects in an IDE or a GUI text editor.

I chose to use a normal VNC server (`vnc4server` in this case) to solve the problem. `vnc4server` would open a new X server and log you into a new desktop (not the same with what is shown on the real monitor). This suffices because at that time I didn't need access to the server through the real monitor.

In `vnc4server` case, one can start a normal VNC server by

    $ vnc4server -depth 24 -geometry 1920x1080

The `-depth` option controls the pixel depth and the `-geometry` controls the resolution. You can see the output like

    New 'debian-yzy:2 (yzy)' desktop is debian-yzy:2

    Starting applications specified in /home/yzy/.vnc/xstartup
    Log file is /home/yzy/.vnc/debian-yzy:2.log

The number 2 in `debian-yzy:2` is the X display number assigned to the new desktop. Now you can connect to this desktop using any typical VNC client, using `<host>:<5900+display_number>` as the target.

If you want to kill the desktop, you have to specify the X display number, like

    $ vnc4server -kill :2

The `xstartup` file needs more configuration if you meet white screen or blank screen cases, which we won't mention here. Just Google it for more information.


# Case 2: Desktop on Personal PC

Several months later I got my own PC in the lab. This PC's performance was a little bit weaker than the lab server, however already enough for most of my work. So I moved all my code and data into my own PC, and did most of my work through its real monitor. The problem was that, if I wanted to work remotely on this machine, it was important to reuse the default desktop since the all my work context was there. The case 1 solution no longer works because `vnc4server` can only start new X servers.

A common solution here is to use `x11vnc`. **Note: `x11vnc` currently doesn't support Wayland. Please use Xorg here.** This software connects to an opened X display and forwards its content to a VNC client. Assume **you've logged into a desktop (which is a must in this case)** and the display number is :0 (typically), you can start the x11vnc by

    $ x11vnc -display :0

and the output might be like

    ...

    The VNC desktop is:      debian-yzy:0
    PORT=5900

and `x11vnc` will be listening to a VNC connection at port 5900. You can then connect to it, and enjoy the same output with the real monitor.


# Case 3: What if Personal PC Crashed

Since case 2 I've experienced several crashes of my PC, including X server crash, kernel panic (too bad), etc. After these crashes, there would be no opened X display since there is no user logged into X desktop yet, so we cannot start a shadow display using commands in case 2.

To solve the problem, one have to get access to the login greeter. `x11vnc` actually does have this functionality, but it's not well documented. I'll try to give a reasonable explanation here, in Gnome 3 case.

First, to access the login greeter, use

    $ sudo x11vnc -display :0 -auth /run/user/<gdm-uid>/gdm/Xauthority

assuming that the greeter is opened on display :0. the `gdm-uid` is the uid of user in which most of Gnome's operations are done, in my case it's 118. The `-auth` option specifies the authority cookie of opened X (see [`x11vnc`'s official FAQ](http://www.karlrunge.com/x11vnc/faq.html#faq)). After that connect to this x11vnc service and you should be able to login. However, after login, you might meet a black screen. This is because a new X display is opened. You can use

    $ ps aux | grep Xorg

to have a look. In my case, the output is like

    Debian-+  1309  0.0  0.3 313660 74920 tty1     Sl+  10:42   0:01 /usr/lib/xorg/Xorg vt1 -displayfd 3 -auth /run/user/118/gdm/Xauthority -background none -noreset -keeptty -verbose 3
    yzy       1763  4.0  0.6 434776 127408 tty2    Rl+  10:43   8:24 /usr/lib/xorg/Xorg vt2 -displayfd 3 -auth /run/user/1000/gdm/Xauthority -background none -noreset -keeptty -verbose 3

You can see that a new authority cookie for uid 1000 is generated. Now you can access your own desktop using

    $ x11vnc -display :1 -auth /run/user/<user-uid>/gdm/Xauthority

in my case `<user-uid>` is 1000. Note that the display number is :1, since the old display is still opened, though black screen.


# Conclusion

Remote desktop provides most complete and continuous experience for remote working, because you can reuse all the programs (like Android Studio) installed on, all code (like AOSP) and data (like Android Virtual Machine) stored in, and all the devices (like Android phones) connected to the remote PC.

<del>Only Android guys as lazy as you need your solution</del>

I'm now seeking a solution for case 4: **what if a personal PC suffers a power supply interruption**...maybe waking up it by Ethernet is a solution.

<del>So why not use cloud computing service</del>


# References
1. vnc4server Man Page
2. x11vnc Man Page
3. [x11vnc Official FAQ](http://www.karlrunge.com/x11vnc/faq.html#faq)
4. [x11vnc - Arch Wiki](https://wiki.archlinux.org/index.php/x11vnc)
5. [x11vnc - GitHub](https://github.com/LibVNC/x11vnc)
6. [Gnome Help: Security](https://help.gnome.org/admin/gdm/stable/security.html.en#gdmuser)