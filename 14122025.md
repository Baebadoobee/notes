# DNS related problems

It seems like the steam client looks up the network address for all the download servers it connects to every time it makes a connection, so it may requests the same information many times a second from the DNS servers your network connection is using. A way around this, is to configure your own DNS server with <a href="https://wiki.archlinux.org/title/Dnsmasq">dnsmasq</a>.

First, install `dnsmasq` with

```sh
$ pacman -S dnsmasq
```

Next, create `/etc/dnsmasq.d` and `/etc/dnsmasq.d/dns_server_name`, then  adds the directory to the config file

```sh
$ mkdir -p /etc/dnsmasq.d
$ touch /etc/dnsmasq.d/dns_server_name
$ echo "conf-dir=/etc/dnsmasq.d" >> /etc/dnsmasq.conf
```

Check your internet interface with `ip a` (look for `wlp0`, `enp0` or `wlan0`). After, open `/etc/dnsmasq.d/dns_server_name` and let it looking like this

```sh
conf-file=/usr/share/dnsmasq/trust-anchors.conf
dnssec # stands for DNS security, check the arch wiki
no-dhcp-interface=[YOUR_INTERFACE]
interface=[YOUR_INTERFACE]
domain=[YOUR_DNS_DOMAIN]
local=/[YOUR_DNS_DOMAIN]/
listen-address=::1
listen-address=127.0.0.1
server=1.1.1.1
server=8.8.8.8
server=9.9.9.9
domain-needed
expand-hosts
bogus-priv
cache-size=1000 # this is really important
```

To finish, you have to edit the `/etc/resolv.conf`

```sh
nameserver ::1
nameserver 127.0.0.1
options trust-ad
```

And enable/start the dnsmasq.service with

```sh
$ systemctl enable --now dnsmasq
$ systemctl start dnsmasq
```

# Low disk usage related problems

## 1. The mount command

As <a href="https://man.archlinux.org/man/mount.8">mount(8)</a> describes, "the mount command serves to attach the filesystem found on some device to the big [file hierarchy]" and it's up to the user to decide the way these filesystems will behave when attached (e.g., read-only, read-write). 

This can be done either via the `mount -o` command 

```sh
$ mount -o [OPTIONS] [DEVICE] [MOUNT_POINT]
```

Or editing the `/etc/fstab` directly.

```sh
# UUID=[DEVICE-UUID]
/dev/your_device    /mount_point    fs_type    option1,option2     dump      fsck
```

You have to keep in mind that although many options are generic, some are specific to the file systemi (e.g., vfat, ext4, btrfs).

## 1.1. Options to take note

- Noatime: Disables updates on inode atimes on the desirable filesystem. 
- Relatime: Updates inode atimes relative to modify or change time.
- Strictatime: Allows to explicitly request full atime updates. This makes it possible for the kernel to default to relatime or noatime but still allow userspace to override it.
- Commit [ext4]: Defines the sync interval for data and metadata by changing the commit delay.

Seeing this list of `*atime` options may have made you think _"well what is atime?"_. In short terms, atime stands for access time, which is a timestamp that records the last time the file was read.

## 2. SSD TRIM commands

As solid-state drives cannot overwrite data directly. They must first erase chunks of unused data before writing over it. This way the TRIM command tells a SSD which blocks of data are no longer in use, so it can erase them internally.

It also can be configured in two very simple (I mean really) ways _Continuos TRIM_ or _Periodic TRIM_.

### 2.1. Continuos TRIM

It issues TRIM commands each time files are deleted. Although it may be seem nice at first, it can lead to some disk usage related problems.

To set it, you just need to add the `discard` (ext4 exclusive) mount option to the desired partition.

```sh
$ umount /your_device
$ mount -o rw,discard /dev/your_device /mount_point
```

or

```sh
# UUID=[DEVICE-UUID]
/dev/your_device    /mount_point    ext4    rw,discard     dump      fsck
```

### 2.2 Periodic TRIM

It issues TRIM commands from time to time (every week, by default). 

To set it, you will need to install the <a href="https://archlinux.org/packages/?name=util-linux">util-linux</a> package, as it provides the `fstrim.timer`. Then, do the following

```sh
$ systemctl enable --now fstrim.service
$ systemctl enable --now fstrim.timer
```

# References

1. Arch Linux Developers. (n.d.). *mount(8)*. Arch Linux Manual Pages. https://man.archlinux.org/man/mount.8
2. Arch Linux Contributors. (n.d.). *Solid state drive*. ArchWiki. https://wiki.archlinux.org/title/Solid_state_drive
3. Arch Linux Contributors. (n.d.). *Ext4*. ArchWiki. https://wiki.archlinux.org/title/Ext4
4. Arch Linux Contributors. (n.d.). *Fstab*. ArchWiki. https://wiki.archlinux.org/title/Fstab
5. Arch Linux Contributors. (n.d.). *Steam/Troubleshooting*. ArchWiki. https://wiki.archlinux.org/title/Steam/Troubleshooting
6. Arch Linux Contributors. (n.d.). *Dnsmasq*. ArchWiki. https://wiki.archlinux.org/title/Dnsmasq
7. Steam Community. (n.d.). *Discussion thread on Steam client network behavior*. Steam Community. https://steamcommunity.com/app/221410/discussions/2/616189106498372437/

