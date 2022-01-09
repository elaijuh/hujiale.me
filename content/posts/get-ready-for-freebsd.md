---

date: 2017-09-12T00:21:39+08:00
draft: false
title: "Get ready for FreeBSD"
categories: ["tech"]
tags: ["freebsd"]

---

I would like to install both FreeBSD and Win10 onto my old workstation. FreeBSD offical manual is quite straight forward and smooth for beginner to complete installation. It just misses some pieces for dual boot with old BIOS, here it is.

I am installing version 11.1 and still using ufs with old BIOS, most parts should be compatible to the previous versions.

Now when you come to `Partitioning` step, choose `Manual`, you might probably see a divice like **ada0** with **MBR** label followed by **ada0s1** and **ada0s2** with **NTFS** label. I have Win10 already installed first, and this is the recommended way as I want FreeBSD to rewrite MBR later. Now create a new section under ada0 and choose freebsd type. From this newly created freebse MBR, create another freebsd-ufs (the size could be the rest of the disk) and freebsd-swap (I set it 4G same as my memory size). Now when reboot, the MBR will boot FreeBSD but not Win10 any longer. No worry, one more step after login: `sudo boot0cfg -B ada0`. This allow boot select from MBR, you will see Win options there.

If everything works just fine, kernel will load drivers for hardware and it's good to go. `pkg update` and `pkg install xorg`, `pkg install xf86-video-ati` (my video adapter is Radeon HD 6850). OK,  enjoy UNIX.
