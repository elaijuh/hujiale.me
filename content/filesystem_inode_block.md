---
title: "Linux filesystem(ext4), inode and blocks"
url: linux-filesystem-inode-block.html
date: 2025-02-22T14:51:39+02:00
type: post
draft: false
---

# Data structure

![filesystem_inode_block](filesystem_inode_block.excalidraw.png "filesystem_inode_block")

| Field            | Description                                                                                                                 |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------- |
| block group      | partition consists of multiple block groups                                                                                 |
| block group 0    | a little bit special. first 1024 bytes are reserved for boot sector                                                         |
| super block (sb) | is a record of the characteristics of a filesystem. primary superblock is in block group 0 whose number is power of 1,3,5,7 |
| group descriptor | records the location of block bitmap and inode bitmap. counter for free blocks and inodes. number of directories.           |
| block bitmap     | each bit represents block. max size is 8*4096(block size)                                                                   |
| inode            | index nodes                                                                                                                 |
| inode table      | array of inodes                                                                                                             |
| inode bitmap     | each bit represents a inode in inodetable. max size is 8*4096(block size)                                                   |

# Commands

- dumpe2fs(8) - retrieve filesystem information

```bash
$ sudo dumpe2fs -h /dev/nvme0n1p4
Filesystem OS type: Linux
Inode count: 24453120
Block count: 97792081
Reserved block count: 4889604
Overhead clusters: 1814658
Free blocks: 81687378
Free inodes: 23866982
First block: 0
Block size: 4096
Fragment size: 4096
Group descriptor size: 64
Reserved GDT blocks: 1024
Blocks per group: 32768
Fragments per group: 32768
Inodes per group: 8192
Inode blocks per group: 512
```

- dumpe2fs(8) - where is primary and backup superblocks (1,3,5,7,...)

```bash
$ sudo dumpe2fs /dev/nvme0n1p4 | grep -B 1 -i superblock
dumpe2fs 1.47.2 (1-Jan-2025)
Group 0: (Blocks 0-32767) csum 0x6bd4 [ITABLE_ZEROED]
  Primary superblock at 0, Group descriptors at 1-47
--
Group 1: (Blocks 32768-65535) csum 0xab1c [ITABLE_ZEROED]
  Backup superblock at 32768, Group descriptors at 32769-32815
--
Group 3: (Blocks 98304-131071) csum 0x992a [ITABLE_ZEROED]
  Backup superblock at 98304, Group descriptors at 98305-98351
--
Group 5: (Blocks 163840-196607) csum 0x8083 [ITABLE_ZEROED]
  Backup superblock at 163840, Group descriptors at 163841-163887
--
Group 7: (Blocks 229376-262143) csum 0x84c7 [ITABLE_ZEROED]
  Backup superblock at 229376, Group descriptors at 229377-229423
--
```

- tune2fs(8) - modify filesystem parameters

```bash
$ sudo tune2fs -l /dev/nvmen01p4
tune2fs 1.47.2 (1-Jan-2025)
Filesystem volume name:   endeavouros
Last mounted on:          /sysroot
Filesystem UUID:          9b6cd942-2a06-48b1-b9fb-cf00862cb5d0
Filesystem magic number:  0xEF53
Filesystem revision #:    1 (dynamic)
Filesystem features:      has_journal ext_attr resize_inode dir_index orphan_file filetype needs_recovery extent 64bit flex_bg metadata_csum_seed sparse_super large_file huge_file dir_nlink extra_isize metadata_csum orphan_present
Filesystem flags:         signed_directory_hash
Default mount options:    user_xattr acl
Filesystem state:         clean
Errors behavior:          Continue
Filesystem OS type:       Linux
Inode count:              24453120
Block count:              97792081
Reserved block count:     4889604
Overhead clusters:        1814658
Free blocks:              81687378
Free inodes:              23866982
First block:              0
Block size:               4096
Fragment size:            4096
Group descriptor size:    64
Reserved GDT blocks:      1024
Blocks per group:         32768
Fragments per group:      32768
Inodes per group:         8192
Inode blocks per group:   512
Flex block group size:    16
Filesystem created:       Mon Nov 27 02:07:25 2023
Last mount time:          Sun Feb 16 04:23:30 2025
Last write time:          Sun Feb 23 23:39:58 2025
Mount count:              32
Maximum mount count:      -1
Last checked:             Tue Dec 31 14:40:33 2024
Check interval:           0 (<none>)
Lifetime writes:          1030 GB
Reserved blocks uid:      0 (user root)
Reserved blocks gid:      0 (group root)
First inode:              11
Inode size:               256
Required extra isize:     32
Desired extra isize:      32
Journal inode:            8
Default directory hash:   half_md4
Directory Hash Seed:      de0781a2-b353-41ab-9a47-2cfdaefb0ec8
Journal backup:           inode blocks
Checksum type:            crc32c
Checksum:                 0xb125b965
Checksum seed:            0x0bc9aa60
Orphan file inode:        12
```

- stat(1) - print file or file system status

stat calculates "Blocks" based on virtual block(512 bytes), that's why you see 24 blocks below.

````bash
$ stat -fc "%n type=%T freeblk=%f totalblk=%b blksz=%S" /root
/root type=ext2/ext3 freeblk=81921201 totalblk=95977423 blksz=4096

$ dd if=/dev/urandom of=a count=3 bs=4096

$ stat a
  File: a
  Size: 12288       Blocks: 24         IO Block: 4096   regular file
Device: 259,4   Inode: 21644642    Links: 1
Access: (0644/-rw-r--r--)  Uid: ( 1000/     hjl)   Gid: ( 1000/     hjl)
Access: 2025-04-19 23:19:07.283020996 +0800
Modify: 2025-04-19 23:20:00.730320206 +0800
Change: 2025-04-19 23:20:00.730320206 +0800
 Birth: 2025-04-19 23:19:07.283020996 +0800```
````

- ls(1) - list file and directory inode

```bash
$ dd if=/dev/urandom of=a count=3 bs=4096
$ ls -i a
21644642 a

$ mkdir b

$ ls -id .
21657519 .

$ ls -id b
21774445 b

$ ls -lia b
total 8
21774445 drwxr-xr-x 2 hjl hjl 4096 Apr 20 03:18 .
21657519 drwxr-xr-x 3 hjl hjl 4096 Apr 20 03:18 ..
```

- df(1) - filesystem space usage

```bash
$ df /dev/nvme0n1p4
Filesystem     1K-blocks     Used Available Use% Mounted on
/dev/nvme0n1p4 383909692 59225448 305109444  17% /

$ df -B 4096 /dev/nvme0n1p4
Filesystem     4K-blocks     Used Available Use% Mounted on
/dev/nvme0n1p4  95977423 14806327  76277396  17% /
```

- blockdev(8) - block device ioctls

from "EXTENTS" we can see file a occupies 3 blocks

```bash
$ sudo blockdev --getbsz /dev/nvme0n1p4
4096

$ dd if=/dev/urandom of=a count=3 bs=4096

$ ls -i a
21644642 a

$ sudo debugfs -R "stat <21644642>" /dev/nvme0n1p4
debugfs 1.47.2 (1-Jan-2025)
Inode: 21644642   Type: regular    Mode:  0644   Flags: 0x80000
Generation: 1360945917    Version: 0x00000000:00000009
User:  1000   Group:  1000   Project:     0   Size: 12288
File ACL: 0
Links: 1   Blockcount: 24
Fragment:  Address: 0    Number: 0    Size: 0
 ctime: 0x6803bf20:ae1f3538 -- Sat Apr 19 23:20:00 2025
 atime: 0x6803beeb:437a3b10 -- Sat Apr 19 23:19:07 2025
 mtime: 0x6803bf20:ae1f3538 -- Sat Apr 19 23:20:00 2025
crtime: 0x6803beeb:437a3b10 -- Sat Apr 19 23:19:07 2025
Size of extra inode fields: 32
Inode checksum: 0x44c771d9
EXTENTS:
(0-2):5850527-5850529
```

# Reference

- https://www.kernel.org/doc/html/v6.13/filesystems/ext4/overview.html#layout

- https://www.kernel.org/doc/html/v6.13/filesystems/ext4/dynamic.html#dynamic-structures
