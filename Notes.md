# Notes
A collection of notes and snippets

## Install full dependencies for NextCloud on Debian

### Required Packages

```
apt install default-mysql-server default-mysql-client apache2 php-curl php-fdomdocument php-gd php-json php-xml php-xmlrpc php-zip php-bz2 php-mysql php-imap php-intl php-ldap php-bcmath php-fpm php-gmp php-imagick libmagickcore-6.q16-6-extra php-apcu php-mbstring ffmpeg 
```

### Enabling Required Apache Modules
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

`-device virtio-vga,virgl=on -display gtk,gl=on`

### Enable Full Virtualzation and SMP.

To make QEMU use full hardware virtualization and use all your CPU cores use the following aruments.

`-smp $(nproc) -enable-kvm -cpu host`
