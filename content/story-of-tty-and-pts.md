---
title: "Story of TTY and PTS"
url: story-of-tty-and-pts.html
date: 2025-02-09T01:45:35+08:00
type: post
draft: false
---

# A little bit backgroud

As a Linux user, I am using terminal every day. One day I find the term "terminal" should be precisely named as "terminal emulator", I decide to dig more into
the mystery of TTY. TTY stands for teletype and it has a long history all the way back to 1800s even before ASCII was invented. It was introduced into UNIX in
1900s for multiple users to talk to the operating system. By then the physical teletype is equipped with ticker tape (for input) and the paper printer (for
output). It was gradually evolved to hardware emulated teletype. CRT and keyboard was made to replace paper printer and ticker tape. Now days keyboard and
monitor are equipped with every Laptop and PC. People use software emulated terminal to talk to kernel or applications. But the legacy from the old cast-iron is
till lurking beneath the surface. How kernel treats physical teletype and software emulated teletype doesn't vary as much as hardware evolves.

# Physical teletype

Ken Thompson developped UNIX on PDP7 with it.

Teletype Model 33
![Teletype model 33](https://upload.wikimedia.org/wikipedia/commons/thumb/3/33/Teletype-IMG_7287.jpg/1920px-Teletype-IMG_7287.jpg "teletype model 33")

# Hardware emulated teletype

DEC VT05\
![DEV VT05](https://terminals-wiki.org/wiki/images/thumb/9/9e/VT05.jpg/1920px-VT05.jpg "vt05")

DEC VT100\
![DEC VT100](https://upload.wikimedia.org/wikipedia/commons/6/6f/Terminal-dec-vt100.jpg "vt100")

# Software emulated teletype

Now days physical and hardware emulated teletypes are practically kept in computer museum. People install software emulated terminal in graphical desktop
environment to manage day to day jobs. Some well known software terminal emulators: urxvt, xterm, konsole, iterm2, st, kitty. In modern UNIX or Linux distros,
mutiple software emulated terminal processes can run concurrently by a single user or multiple users, but all of them are attached to same tty (`who(1)`) which
usually assigned for graphical desktop environment. How to distinguish different terminal processes as a logical tty? Here comes PTS - pseudo-teletypes. In
Linux there is a master multiplexor `ptmx(4) /dev/ptmx` manages all master PTM slave PTS `/dev/pts/*` pair. The slave PTS number can be checked with `tty`
command.

# IO control commands

`termios(3)`, `tty_ioctl(4)`, `stty(1)`

Print TTY device setting

```bash
$ stty -a
speed 38400 baud; rows 75; columns 254; line = 0;
intr = ^C; quit = ^\; erase = ^?; kill = ^U; eof = ^D; eol = <undef>; eol2 = <undef>; swtch = <undef>; start = ^Q; stop = ^S; susp = ^Z; rprnt = ^R; werase = ^W; lnext = ^V; discard = ^O; min = 1; time = 0;
-parenb -parodd -cmspar cs8 -hupcl -cstopb cread -clocal -crtscts
-ignbrk -brkint -ignpar -parmrk -inpck -istrip -inlcr -igncr icrnl ixon -ixoff -iuclc -ixany -imaxbel iutf8
opost -olcuc -ocrnl onlcr -onocr -onlret -ofill -ofdel nl0 cr0 tab0 bs0 vt0 ff0
isig icanon iexten echo echoe echok -echonl -noflsh -xcase -tostop -echoprt echoctl echoke -flusho -extproc
```

Current PTS

```bash
$ tty
/dev/pts/1

$ who
hjl      tty1         2025-02-09 13:18

$ tty -s && echo "In a tty"
In a tty
```

# Interesting reading

- https://www.linusakesson.net/programming/tty/
