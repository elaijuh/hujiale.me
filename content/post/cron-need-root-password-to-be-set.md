---

date: 2017-09-30T14:51:31+08:00
draft: false
title: "Cron need root password to be set"
slug: ""
categories: ["tech"]
tags: ["unix, cron"]

---


I am using Digital Ocean with password login disabled. I provision new machine using ansible userdata, so that every team member can login by ssh.
Everything works fine until one day I find cron job doesn't work correctly under root.

The error log is
`pam_unix(cron:account): expired password for user root (root enforced)`

The tricky part is, when creating a new droplet, Digital Ocean needs you to login by initial password and reset it immediately. As I am using terraform to create new droplet and ansible to provision it, I miss the reset password part. But how immediate reset works? The file `/etc/shadow` is the key to it. It saves the user and password information. It looks like this:

`root:<hashed_password>:0:0:14600:14:::`

Each field in the shadow file is also separated with ":" colon characters, and are as follows:

Username, up to 8 characters. Case-sensitive, usually all lowercase. A direct match to the username in the /etc/passwd file.

Password, 13 character encrypted. A blank entry (eg. ::) indicates a password is not required to log in (usually a bad idea), and a "*" entry (eg. :*:) indicates the account has been disabled.

The number of days (since January 1, 1970) since the password was last changed.

The number of days before password may be changed (0 indicates it may be changed at any time)

The number of days after which password must be changed (99999 indicates user can keep his or her password unchanged for many, many years)

The number of days to warn user of an expiring password (7 for a full week)

The number of days after password expires that account is disabled

The number of days since January 1, 1970 that an account has been disabled

A reserved field for possible future use

There is also a easy command `chage` to check these information with human readable format, `sudo chage -l root`

So to solve the cron problem, instead of update the VPS set initial root password, I can just update number of days since the password was last changed.

`sudo chage -d 2017-09-30 root`

Now cron should work as expected.


