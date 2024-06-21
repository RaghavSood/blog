---
layout: post
title: Installing NixOS on an OVH/Kimsufi/SoYouStart server
---

I've been running NixOS servers for a few years now, largely hosting them across a mix of Mini PCs, DigitalOcean, Hetzner, and more recently Contabo.

While migrating [burned.money](https://burned.money) off fly.io (their 500GB volume limit started to become a problem), I settled on OVH's Kimsufi offering as it offered the best price for the storage I needed, and was located in a region close to my BTC node.

Unfortunately, as it turns out, a mix of OVH weirdness and Nix flakiness (distinct from Nix flakes) made this a long and cumbersome install.

Hopefully, this post will help you save some sanity and days if you are ever in the same boat.

## Getting a server

This part is pretty straightforward - head over to OVH or one of their subsidiaries (Kimsufi, SoYouStart) and pick a server that suits your needs. Buy it. Wait a few minutes to a few hours for the server to be provisioned. Done.

## Preparing OVH

Before we get started, let me save you some time - open up the KVM. This isn't available on all servers, but most with datacenter-level hardware (sorry, no ATOMs).

OVH offers a KVM console, accessible by downloading a `jnlp` file through their web console. Unfortunately, in 2024, `jnlp` files are a relic of the past. M-series Macs aren't supported by the `iKVM64.jar`. Debugging this cost me a few hours of my life.

Instead, you can fallback to sane options. Using a Linux machine with Nix (or, you know, NixOS), or a VM if you're on a Mac, you can simply run:

```bash
$ nix-shell -p adoptopenjdk-icedtea-web
$ javaws /path/to/your/downloaded.jnlp
```

Don't waste your time with CheerpJ or other wrappers that claim to be able to deal with this. They can't. Use [IcedTea-Web](https://github.com/AdoptOpenJDK/IcedTea-Web){:target='blank'} and be done with it.

By now, you should have a KVM interface - it should be showing you the graphical output as well as piping through your keyboard. By default, you're probably looking at the rescue mode login or some other OVH banner.

To get NixOS installed, we're going to use the `kexec` method to temporarily boot into a NixOS system from the rescue system, and then use that to install NixOS onto the disk.

## Installing NixOS

First, we're going to use the OVH console to boot into rescue mode (use a Debian systems - Windows will not help), with an SSH key added - this is important, as our kexec NixOS is going to copy existing SSH keys.

To do this, alter the Boot configuration using the OVH console and select the rescue system. Provide your SSH key and click through the confirmation prompt. Then, reboot the server from the OVH console.

A minute or so later, you should be able to SSH into the server (tip: run `ping <ip>` to have an indication of when the server has finished restarting).

You should now see a prompt like this:

```bash
root@rescue-customer-eu (ns326340.ip-<ip>.eu) ~ #
```

The Nix community offers weekly kexec images over at [nixos-image](https://github.com/nix-community/nixos-images){:target='blank'}. Unfortunately, recent images don't quite seem to work[^1] - they hang during execution, rendering the server inaccessible without a new reboot into rescue.

Instead, we'll use an older image for NixOS-22.11.

```bash
curl --location https://github.com/nix-community/nixos-images/releases/download/nixos-22.11/nixos-kexec-installer-noninteractive-x86_64-linux.tar.gz | tar -C /root -xvzf-
/root/kexec/run
```

The machine will restart after a 6 second delay. As before, you can use `ping` to watch for the server to come back online. Once it's up, SSHing again gives us a lovely NixOS prompt.

```bash
root@nixos ~ #
```

With everything in place now, we can install and configure the actual NixOS system. To start, we need to partition the disks. OVH has a preculiar constraint requiring the boot partition to be on `/dev/sda` (I'm unsure if this translates over to NVME disks as well).

The rest of your disks can be partitioned and merged as needed, but this constrained means it is not possible to have a mirrored boot drive or a mdadm RAID1 boot drive.

In the case of this demo server, we only have one disk, we let's get that set up:

```bash
[root@nixos:~]# lsblk
NAME  MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0   7:0    0 341.3M  0 loop /nix/.ro-store
sda     8:0    0   1.8T  0 disk
[root@nixos:~]# parted /dev/sda -- mklabel msdos
Information: You may need to update /etc/fstab.
[root@nixos:~]# parted /dev/sda -- mkpart primary 1MiB 512MiB
Information: You may need to update /etc/fstab.
[root@nixos:~]# parted /dev/sda -- mkpart primary 512MiB 100%
Information: You may need to update /etc/fstab.
[root@nixos:~]# lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0    7:0    0 341.3M  0 loop /nix/.ro-store
sda      8:0    0   1.8T  0 disk
├─sda1   8:1    0   511M  0 part
└─sda2   8:2    0   1.8T  0 part
[root@nixos:~]# mkfs.ext4 -L boot /dev/sda1
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 523264 1k blocks and 130560 inodes
Filesystem UUID: 53e7efcf-73ba-4145-b503-aab590481958
Superblock backups stored on blocks:
	8193, 24577, 40961, 57345, 73729, 204801, 221185, 401409

Allocating group tables: done
Writing inode tables: done
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done
[root@nixos:~]# zpool create -f -O mountpoint=none rpool /dev/sda2
[root@nixos:~]# zpool list
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
rpool  1.81T   165K  1.81T        -         -     0%     0%  1.00x    ONLINE  -
```

I'm using ext4 for a BIOS boot parition, and ZFS for the main partition. If multiple disks are present, the above can be tweaked to set up a mirrored ZFS pool.

Next, we're going to mount the partitions and generate the NixOS configuration.

```bash
[root@nixos:~]# zfs create -o mountpoint=legacy rpool/root
zfs create -o mountpoint=legacy rpool/root/nixos
zfs create -o mountpoint=legacy rpool/home
[root@nixos:~]# mount -t zfs rpool/root/nixos /mnt
[root@nixos:~]# mkdir /mnt/home
[root@nixos:~]# mount -t zfs rpool/home /mnt/home
[root@nixos:~]# mkdir /mnt/boot
[root@nixos:~]# mount /dev/sda1 /mnt/boot
[root@nixos:~]# findmnt
TARGET                       SOURCE     FSTYPE    OPTIONS
/                            tmpfs      tmpfs     rw,relatime,mode=755
├─/dev                       devtmpfs   devtmpfs  rw,nosuid,size=200432k,nr_inodes=455291,mode=755
│ ├─/dev/pts                 devpts     devpts    rw,nosuid,noexec,relatime,gid=3,mode=620,ptmxmode=666
│ ├─/dev/shm                 tmpfs      tmpfs     rw,nosuid,nodev
│ ├─/dev/mqueue              mqueue     mqueue    rw,nosuid,nodev,noexec,relatime
│ └─/dev/hugepages           hugetlbfs  hugetlbfs rw,relatime,pagesize=2M
├─/proc                      proc       proc      rw,nosuid,nodev,noexec,relatime
├─/run                       tmpfs      tmpfs     rw,nosuid,nodev,size=1002152k,mode=755
│ ├─/run/keys                ramfs      ramfs     rw,nosuid,nodev,relatime,mode=750
│ ├─/run/wrappers            tmpfs      tmpfs     rw,nodev,relatime,mode=755
│ ├─/run/user/1000           tmpfs      tmpfs     rw,nosuid,nodev,relatime,size=400860k,nr_inodes=100215,mode
│ └─/run/user/0              tmpfs      tmpfs     rw,nosuid,nodev,relatime,size=400860k,nr_inodes=100215,mode
├─/sys                       sysfs      sysfs     rw,nosuid,nodev,noexec,relatime
│ ├─/sys/kernel/security     securityfs securityf rw,nosuid,nodev,noexec,relatime
│ ├─/sys/fs/cgroup           cgroup2    cgroup2   rw,nosuid,nodev,noexec,relatime,nsdelegate,memory_recursive
│ ├─/sys/fs/bpf              bpf        bpf       rw,nosuid,nodev,noexec,relatime,mode=700
│ ├─/sys/kernel/debug        debugfs    debugfs   rw,nosuid,nodev,noexec,relatime
│ ├─/sys/fs/fuse/connections fusectl    fusectl   rw,nosuid,nodev,noexec,relatime
│ ├─/sys/kernel/config       configfs   configfs  rw,nosuid,nodev,noexec,relatime
│ └─/sys/fs/pstore           pstore     pstore    rw,nosuid,nodev,noexec,relatime
├─/nix/.ro-store             /dev/loop0 squashfs  ro,relatime,errors=continue
├─/nix/.rw-store             tmpfs      tmpfs     rw,relatime,mode=755
├─/nix/store                 overlay    overlay   rw,relatime,lowerdir=/mnt-root/nix/.ro-store,upperdir=/mnt-
│ └─/nix/store               overlay    overlay   ro,relatime,lowerdir=/mnt-root/nix/.ro-store,upperdir=/mnt-
└─/mnt                       rpool/root/nixos
                                        zfs       rw,relatime,xattr,noacl
  ├─/mnt/home                rpool/home zfs       rw,relatime,xattr,noacl
  └─/mnt/boot                /dev/sda1  ext4      rw,relatime
[root@nixos:~]# nixos-generate-config --root /mnt
writing /mnt/etc/nixos/hardware-configuration.nix...
writing /mnt/etc/nixos/configuration.nix...
For more hardware-specific settings, see https://github.com/NixOS/nixos-hardware.
[root@nixos:~]#
```

After mounting the required ZFS datasets and bootdisk, I confirm everything looks right with `findmnt`, and then generate the NixOS configuration.

We're going to make a few changes to the auto-generated configuration. Let's get `vim` by entering into a nix-shell:

```bash
[root@nixos:~]# nix-shell -p vim
```

Now, we can edit the configuration:

```diff
diff --git a/configuration.nix b/configuration.nix
index b476787..c926f3f 100644
--- a/configuration.nix
+++ b/configuration.nix
@@ -17,13 +17,23 @@
   # boot.loader.grub.efiInstallAsRemovable = true;
   # boot.loader.efi.efiSysMountPoint = "/boot/efi";
   # Define on which hard drive you want to install Grub.
-  # boot.loader.grub.device = "/dev/sda"; # or "nodev" for efi only
+  boot.loader.grub.device = "/dev/sda"; # or "nodev" for efi only

-  # networking.hostName = "nixos"; # Define your hostname.
+  networking.hostName = "blog-demo";
+  networking.hostId = "12345678";
   # Pick only one of the below networking options.
   # networking.wireless.enable = true;  # Enables wireless support via wpa_supplicant.
   # networking.networkmanager.enable = true;  # Easiest to use and most distros use this by default.

+  networking.usePredictableInterfaceNames = false;
+  networking.interfaces.eth0.ipv4.addresses = [{
+    address = "<ip from console>";
+    prefixLength = 24;
+  }];
+
+  networking.defaultGateway = "<gateway from console>";
+  networking.nameservers = [ "8.8.8.8" ];
+
   # Set your time zone.
   # time.timeZone = "Europe/Amsterdam";

@@ -90,7 +100,11 @@
   # List services that you want to enable:

   # Enable the OpenSSH daemon.
-  # services.openssh.enable = true;
+  services.openssh.enable = true;
+  users.users.root.openssh.authorizedKeys.keys = [
+    "key1..."
+    "key2..."
+  ];

   # Open ports in the firewall.
   # networking.firewall.allowedTCPPorts = [ ... ];
@@ -109,7 +123,7 @@
   # this value at the release version of the first install of this system.
   # Before changing this value read the documentation for this option
   # (e.g. man configuration.nix or on https://nixos.org/nixos/options.html).
-  system.stateVersion = "22.11"; # Did you read the comment?
+  system.stateVersion = "24.05"; # Did you read the comment?

 }
 ```

 Our changes are to:

 1. Install grub to `/dev/sda`
 2. Configure IP address, gateway, and DNS servers based on what OVH provides
 3. Enable SSH and add keys to the root user
 4. Set the hostname and a random 8 hex character host ID (this is required for ZFS - you can skip `hostId` for non-ZFS systems)
 5. Update the state version to the latest NixOS release - Since we're using an old image to work around `kexec` issues, we will manually bump up the install to use the latest release.

Lastly, we need to update the `nixos` channel to point to the new release:

```bash
[root@nixos:~]# nix-channel --list
nixos https://nixos.org/channels/nixos-22.11
[root@nixos:~]# nix-channel --add https://nixos.org/channels/nixos-24.05 nixos
[root@nixos:~]# nix-channel --update
unpacking channels...
[root@nixos:~]#
```

And with that, we're ready to install NixOS:

```bash
[root@nixos:~]# nixos-install --root /mnt
```

This will take a few minutes. Once done, we need to unmount our ZFS pool so it can be cleanly imported by the new system.

```bash
[root@nixos:~]# umount /mnt/boot
[root@nixos:~]# umount /mnt/home
[root@nixos:~]# umount /mnt
```

Now all that's left is to reboot the server and hope for the best. In the OVH console, change the boot configuration to boot from the hard disk, and then reboot the server.

```bash
[root@nixos:~]# reboot
```

Once rebooted, the SSH host key will have changed, so you'll need to remove it from `~/.ssh/known_hosts`, then accept the new key when SSHing in.

```bash
$ ssh-keygen -R <ip>
$ ssh root@<ip>
```

And that's it! You should now have a working NixOS system on your OVH server.

## All for naught

In the end, the OVH system didn't actually being useful for [burned.money](https://burned.money) - the spinning rust was just too slow, estimating ~80 days to import the SQLite backup. Nevertheless, at least I know that NixOS can be run on OVH with a bearable amount of pain.


## What didn't work

Before arriving at this recipe, I tried a number of other methods that didn't work. For posterity, here they are:

1. Using `nixos-infect`[^2] - This _might_ work if you first install a regular Linux system, but with the rescue system being a temporary FS without any disk partitioning, `nixos-infect` couldn't seem to figure out how to install.
2. Using `nixos-anywhere`[^3] - This relies on `kexec`, and as newer `kexec` images are broken, at least on OVH, it hung in much the same way as `kexec`. That said, it is possible to override[^4] the `kexec` image used, so it might be possible to use an older image. When trying this out, I didn't yet know that the `kexec` images were unusable, so I didn't explore that.
3. Using `nixos-generators`[^5] - In theory, it should be possible to generate an ISO containing my system config, and using OVH's Custom ISO boot option to install from it. However, it is not currently possible to generate linux ISOs from a MacOS system, and I didn't go to the effort to trying it via a VM or other server.

# Notes/references 

[^1]: https://discourse.nixos.org/t/kexec-hangs-on-install/40969
[^2]: https://github.com/elitak/nixos-infect
[^3]: https://github.com/nix-community/nixos-anywhere
[^4]: https://github.com/nix-community/nixos-anywhere/blob/ce18c086d8ca143d43ab20b3db20ab1e3e62c519/docs/howtos/custom-kexec.md
[^5]: https://github.com/nix-community/nixos-generators
