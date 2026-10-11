# Optional legacy password SSH branch

Observed on one rooted Classic. Default to the owner's own key authentication. Enabling a configuration flag is insufficient when the daemon's crypt implementation and QNX account database disagree. This reference is an advanced reconstruction guide, not a universal account replacement script.

## Actual successful components

- Original, unmodified **OpenSSH 7.1p2** executable extracted locally from BerryCore's `packages/openssh-7.1p2.zip`.
- Private executable path `/var/rootdata/sshd-password`, chmod 700.
- `/proc/boot/pathtrust !/var/rootdata/sshd-password` applied before execution on this device.
- Writable copy of the existing passwd database in `/var/etc/passwd`; changed only root's hash field, preserving every other field and entry.
- QNX overlay: `/bin/ln -sPf /var/etc/passwd /etc/passwd`.
- Traditional **DES crypt** hash generated locally with a compatible old OpenSSL `passwd -crypt`; attempted MD5 hashes did not authenticate in this build.
- Original shadow view retained; no published or replacement shadow database.

Use a freshly chosen password; feed it through a local secure prompt/stdin supported by the chosen tool. Do not put a password literal in a command, shell history, log, script or repository. Do not publish the hash. DES considers only the first eight password bytes and provides weak protection; use this only as a local legacy compatibility experiment and prefer keys. If the required crypt utility is unavailable, do not invent a hash or replace it with an unverified algorithm.

## Service config used here

```text
Protocol=2
AuthorizedKeysFile=/var/etc/ssh/authorized_keys2
AllowUsers=root
PermitRootLogin=yes
PasswordAuthentication=yes
PermitEmptyPasswords=no
HostKey=/var/etc/ssh/ssh_host_rsa_key
UsePrivilegeSeparation=no
PidFile=/var/run/sshd.pid
Subsystem=sftp /usr/libexec/sftp-server
```

Inspect owner/modes and existing config; use the daemon's config-check option if supported. Keep the working key session open while changing authentication. Stop only the known old listener, then start the alternative and authenticate from a separate client. Do not count a returned shell prompt on the phone as proof of remote authentication.

```sh
/proc/boot/pathtrust !/var/rootdata/sshd-password
/var/rootdata/sshd-password -e -f /var/etc/ssh/sshd_config
```

The modified `btool` branch on this phone reapplied the passwd overlay, pathtrust exemption and alternative launch under the existing `use.sshd` condition. It stopped both known native/alternative daemon names before starting. Read the current full btool, preserve all root/Runtime initialization logic and apply the smallest targeted change; a device-specific copy is intentionally not shipped here. Ensure the old native daemon is not started by a second service.

This session completed remote password authentication with public-key authentication disabled. It did not establish that every QNX OpenSSH build supports this mechanism or that this is appropriate for internet exposure. SSH host private keys remain on the phone; client private keys remain on the owner's computer.
