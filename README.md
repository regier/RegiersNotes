# Regier's Notes
Just a collection of notes of things I find interesting, code snippets, instructions, usefull commands or things I need to remember for later.

## Terminal Manipulation

### Escape Sequences

To be used with `printf` or `echo -e` for example.

```sh
# Show the terminal cursor             = \033[?25h
# Hide the terminal cursor             = \033[?25l
# Repeat a character n times           = seq 1 n
# Save current cursor position         = \033[s
# Restore previous cursor position     = \033[u
# Move cursor to the specified coords  = \033[1;1H
# Carriage Return                      = \r
# Return cursor to the home position   = \033[H
# Clear line                           = \033[K
# Clear screen                         = \033[J
```

### Terminal Parameters

Consulting terminal capabilities and status.

```sh
echo "Terminal type and supported colours: ${TERM}."
echo "Current terminal size is Columns x Lines: ${COLUMNS}x${LINES}."
```

If for some reason the variables $COLUMNS and $LINES are not available, then the following could be tried.

Assuming you have the `stty` command available.

```sh
COLUMNS="$(stty size | cut -d ' ' -f 2)"
LINES=$(stty size | cut -d ' ' -f 1)
```

## Auto Scaling/Adjusting Resolution KVM Virtio Virtual Machine Window

In virt-manager using spice

### Required Packages (Guest)

- [Spice VDA Agent/Spice Guest Tools](https://www.spice-space.org/download.html#guest)

#### Linux

```
spice-vdagent
```

#### Windows

```
spice-guest-tools
```

### Automatically Setting Resolution

- Note: Works better/Only works with X11.

```
xrandr --output Virtual-1 --auto
```

## Install full dependencies for NextCloud on Debian

### Required Packages

- NextCloud Official Doc [Prerequisites](https://docs.nextcloud.com/server/latest/admin_manual/installation/source_installation.html#prerequisites-for-manual-installation)
- NextCloud Official Doc [Required PHP Modules](https://docs.nextcloud.com/server/latest/admin_manual/installation/php_configuration.html)

```
apt install default-mysql-server default-mysql-client apache2 php-curl php-fdomdocument php-gd php-json php-xml php-xmlrpc php-zip php-bz2 php-mysql php-imap php-intl php-ldap php-bcmath php-fpm php-gmp php-imagick libmagickcore-6.q16-6-extra php-apcu php-mbstring ffmpeg 
```

### Enabling Required Apache Modules

- NextCloud Official Doc [Apache Configuration](https://docs.nextcloud.com/server/latest/admin_manual/installation/source_installation.html#apache-configuration-label)

```
a2enmod rewrite
a2enmod headers
a2enmod env
a2enmod dir
a2enmod mime
systemctl restart apache2
```

## Sharing Internet on Linux

### Internet Forwarding

1. Uncomment `#net.ipv4.ip_forward=1` in the file `/etc/sysctl.conf`
2. Enable ipv4 forwarding without having to reboot. `echo "1" > /proc/sys/net/ipv4/ip_forward`
3. Enable forwarding with `iptables` with the command `interface=enp0d1 iptables -t nat -A POSTROUTING -o $interface -j MASQUERADE`
4. Save `iptables` rule so it can persist when the system is rebooted. `iptables-save | tee /etc/iptables.conf`

### DHCP Server

1. You can use a `dnsmasq` as a dhcp server, simply install it and uncomment dhcp range.
2. Restart `dnsmasq` with the command `systemctl restart dnsmasq`

## Debian Linux simple internet configuration

`/etc/network/interfaces`

1. Replace `eth0` with `enp0s1` for example. Omit address and gateway, set `inet dhcp` for dhcp.
2. Adjust `address` and `gateway` according to your configuration.

```
auto eth0
iface eth0 inet static
  address 192.0.2.7/24
  gateway 192.0.2.254
```

## Fix for common issues

### When servers use a TLS/SSL cypher that is too simple.

#### Error message.

`tls_process_ske_dhe:dh key too small`

#### Workaround

The workaround for this issue is to make your SSL accept less secure cyphers.

Edit the file `/etc/ssl/openssl.cnf` as root. Change it to match the lines below.

```
[ default_conf ]
ssl_conf = ssl_sect
[ssl_sect]
system_default = ssl_default_sect
[ssl_default_sect]
MinProtocol = TLSv1.2
CipherString = DEFAULT:@SECLEVEL=1
```

## Seting up SSH VPN Server-side

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -I FORWARD -i tun+ -j ACCEPT
iptables -I FORWARD -o tun+ -j ACCEPT
iptables -I INPUT -i tun+ -j ACCEPT
iptables -t nat -I POSTROUTING -o EXTERNAL_INTERFACE -j MASQUERADE
```

## Making RAID 0/1 on Debian

### 1. Install mdadm

```bash
apt-get install mdadm
update-initramfs -u
```

### 2. Adding sda and sdb devices as RAID members and setting RAID 1 in them,

```bash
mdadm --zero-superblock /dev/sda /dev/sdc
mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sda /dev/sdc
mdadm --detail --scan /dev/md0 >> /etc/mdadm/mdadm.conf
```

### 3. Making an XFS filesystem in the new RAID device.

```bash
mkfs.xfs -L Raid1 /dev/md0
```

## QEMU KVM with 3D acceleration for Linux and Android Guests

### 3D Acceleration

The following arguments enable a fully accelerated display on QEMU.

`-device virtio-vga-gl -display gtk,gl=on`

### Enable Full Virtualzation and SMP.

To make QEMU use full hardware virtualization and use all your CPU cores use the following aruments.

`-smp $(nproc) -enable-kvm -cpu host`

## Ext4 and F2FS Filesystem Level Encryption

### Enabling Encryption

#### Create new FS

* F2FS `mkfs.f2fs -O encrypt /dev/sdX`
* Ext4 `mkfs.ext4 -O encrypt /dev/sdX`

#### Or Enable for existing FS

* F2FS `fsck.f2fs -O encrypt /dev/sdX`
* Ext4 `tune2fs -O encrypt /dev/sdX`

#### Configure Encryption For a Directory

* `# fscrypt setup`
* `#/$ fscrypt setup /path/mountpoint`

* `$ fscrypt encrypt /path/mountpoint/directory`

#### Lock/Unlock Dir

* `$ fscrypt unlock fscrypt unlock`
* `$ fscrypt lock fscrypt unlock`
