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
