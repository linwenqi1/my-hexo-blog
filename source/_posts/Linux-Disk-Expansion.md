---
title: Linux Disk Expansion Guide
date: 2026-01-08 21:51:25
tags:
  - Linux
categories:
  - Computer Science
---

Increasing the disk size in your VM settings (VMware/VirtualBox) is only half the battle. The OS won't automatically use that extra space until you manually update the partition table and the filesystem.

Here is a quick guide on how I expanded my `/dev/sda2` partition from 30GB to 40GB without losing any data.

## The Problem

After increasing the virtual disk to 40GB, running `lsblk` showed that the physical disk (`sda`) was 40GB, but the primary partition (`sda2`) was still stuck at 30GB.
```bash
# Output snippet
sda      8:0    0    40G  0 disk 
└─sda2   8:2    0    30G  0 part /
```

<!-- more -->

## Step 1: Install the Growth Tools

We need `growpart`, a handy utility that can safely extend a partition while the system is running.
```bash
sudo apt update
sudo apt install cloud-guest-utils -y
```

## Step 2: Extend the Partition

We tell the system to expand the 2nd partition of the sda disk. Note the space between the disk name and the partition number:
```bash
sudo growpart /dev/sda 2
```

## Step 3: Resize the Filesystem

Now that the "room" (partition) is bigger, we need to stretch the "carpet" (filesystem) to cover the new floor space.

First, check your filesystem type:
```bash
df -T /
```

Depending on the output (ext4 or xfs), run the corresponding command:

### For ext4 (Common in Ubuntu)
```bash
sudo resize2fs /dev/sda2
```

### For xfs
```bash
sudo xfs_growfs /
```

## Conclusion

Check the final result with `df -h /`. You should see the full capacity reflected in your root directory!
```bash
# Final check
/dev/sda2        40G   21G   17G  55% /
```

## Key Takeaway

Always remember the three-layer hierarchy: **Physical Disk → Partition Table → Filesystem**. You must expand them in that specific order.