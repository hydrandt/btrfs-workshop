%title: btrfs - modern linux CoW filesystem
%author: @hydrandt|lukas@aiya.cz
%date: 2017-05-12

-> # btrfs - modern linux CoW filesystem

-> *https://btrfs.wiki.kernel.org/*

* presentation created using [mdp](https://github.com/visit1985/mdp)

## Agenda
1. Introduction: What is btrfs,how it works, history (20 min)
2. Start of workshop: Distributing the virtualbox VM image (10 min)
3. Create btrfs filesystem, put some data on, add second block device (10 min)
4. Enable compression, how it works, available algorithms (10 min)
5. Enable mirroring (raid1), how it works, available raid levels (10 min)
6. Rebalance the filesystem: when is it necessary, what does it do (10 min)
7. Create snapshots / subvolumes: how it works, read only and writable, automatic tools (20 min)
8. Transferring a subvolume to another machine: btrfs send / receive (15 min)
9. Set up a simple backup system with rsync and snapshots (20 min)

---

# $ whoami
lukas

* free software and culture enthusiast
* Chinese studies graduate
* interested in censorship, media control, freedom, security
* freelance sysadmin


## My workflow:

-> ▛▀▀▀▀▀▀▀▀▀▀▀▀▀▜          ▛▀▀▀▀▀▀▀▀▀▀▀▀▀▀▜  
-> ▌broken server▐   ----->  ▌working server ▐  
-> ▙▄▄▄▄▄▄▄▄▄▄▄▄▄▟          ▙▄▄▄▄▄▄▄▄▄▄▄▄▄▄▟


---

# What is btrfs

* modern filesystem for Linux, based on copy-on-write principle
* development started in 2007, in kernel from 2009, automatic scrubbing and defragmentation from 2011, on-disk format stable from 2014

## Main features

* 2^64 byte == 16 EiB maximum file size (practical limit is 8 EiB due to Linux VFS) 
* Checksums on data and metadata (crc32c)  -> avoiding silent data corruption
* Cheap read only // writable snapshots
* Subvolumes (separate internal filesystem roots)
* Compression (zlib, lzo)
* Multiple device support (=raid) - raid0, raid1 (raid5 experimental, eats your data)
* Background scrub, online defragmentation
* Seed devices
* Quota support
* Send/receive of subvolume changes -> incremental filesystem mirroring
* out-of-band deduplication



---

# What is copy-on-write (CoW)

---


# Start of workshop: Distributing the virtualbox VM image (10 min)

# Useful guide: https://btrfs.wiki.kernel.org/index.php/SysadminGuide

man btrfs
man btrfs-filesystem

---

# Create btrfs filesystem, put some data on, add second block device (10 min)

mkfs.btrfs /dev/sdXX
mkdir /mnt/workshop
mount /dev/sdXX /mnt/workshop

vim /etc/fstab

btrfs filesystem show

https://btrfs.wiki.kernel.org/index.php/Using_Btrfs_with_Multiple_Devices

btrfs device add /dev/sdXX /mnt/workshop
btrfs balance start -dconvert=raid1 -mconvert=raid1 /mnt/workshop

btrfs device delete /dev/sdX /mnt/workshop

---

# Disk usage

btrfs filesystem usage /[montpoint] 


# Enable compression, how it works, available algorithms (10 min)

mount -o remount,compress=lzo /mnt/workshop

btrfs filesystem defrag -v -r -czlib /mnt/workshop


# Create snapshots / subvolumes: how it works, read only and writable, automatic tools (20 min)

btrfs subvolume create /mnt/workshop/new_subvol

mkdir /mnt/workshop/snapshots

btrfs subvolume snapshot /mnt/workshop /mnt/workshop/snapshots/20170512-test-snapshot

# Transferring a subvolume to another machine: btrfs send / receive (15 min)

https://btrfs.wiki.kernel.org/index.php/Incremental_Backup

btrfs subvolume snapshot -r /mnt/workshop/data /mnt/workshop/snapshots/backup-latest
sync  
btrfs subvolume create /mnt/workshop/backup  
btrfs send /mnt/workshop/snapshots/170512-backup | btrfs receive /mnt/workshop/backup/latest  
btrfs subvolume snapshot /mnt/workshop/backup/latest /mnt/workshop/backup/170512

### next day:  
btrfs subvolume snapshot -r /mnt/workshop/data /mnt/workshop/snapshots/170513-backup
btrfs send -p /mnt/workshop/snapshots/170512-backup /mnt/workshop/snapshots/170513-backup | btrfs receive /mnt/workshop/backup/latest
btrfs subvolume snapshot /mnt/workshop/backup/latest /mnt/workshop/backup/170513



# Set up a simple backup system with rsync and snapshots (20 min)

