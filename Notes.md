# Notes
A collection of notes and snippets

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
```
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -I FORWARD -i tun+ -j ACCEPT
iptables -I FORWARD -o tun+ -j ACCEPT
iptables -I INPUT -i tun+ -j ACCEPT
iptables -t nat -I POSTROUTING -o EXTERNAL_INTERFACE -j MASQUERADE
```
