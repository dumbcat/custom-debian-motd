# Customize motd for Raspberry Pi

![Customize motd for Raspberry Pi](/2020-04-21_101211.png)

---
## Install Step

**Step 1.** Create file `/usr/sbin/update-motd`, and change the access permissions to 755.

```Shell
$ touch /usr/sbin/update-motd
$ chmod 755 /usr/sbin/update-motd
```

**Step 2.** Edit `usr/sbin/update-motd`, Enter the following script.

```Bash
#!/bin/sh
#
#    update-motd - update the dynamic MOTD immediately
#
#    Copyright (C) 2008-2014 Dustin Kirkland <dustin.kirkland@gmail.com>
#
#    Authors: Dustin Kirkland <dustin.kirkland@gmail.com>
#    [ comments edited out by ownyourbits ]
set -e
 
if ! touch /var/run/motd.new 2>/dev/null; then
        echo "ERROR: Permission denied, try:" 1>&2
        echo "  sudo $0" 1>&2
        exit 1
fi
 
if run-parts --lsbsysinit /etc/update-motd.d > /var/run/motd.new; then
        if mv -f /var/run/motd.new /var/run/motd; then
                cat /var/run/motd
                exit 0
        else
                echo "ERROR: could not install new MOTD" 1>&2
                exit 1
        fi
fi
echo "ERROR: could not generate new MOTD" 1>&2
exit 2
```

**Step 3.** Confirm the following content in `/etc/pam.d/sshd`.

```Bash
session    optional     pam_motd.so  motd=/run/motd.dynamic`
```

**Step 4.** Confirm the following content in `/etc/init.d/motd`, skip if the system does not have the file.

```Bash
uname -snrvm > /var/run/motd.dynamic
```

**Step 5.** Disable automatically generate motd file.

```Shell
$ systemctl disable motd
```

**Step 6.** Remove default motd.

```Shell
$ rm -f /etc/motd
```

**Step 7.** Create symbolic link for customize motd to /etc/motd.

```Shell
$ ln -s /var/run/motd /etc/motd
```

**Step 8.** Create folder for svae motd Script.

```Shell
$ mkdir /etc/update-motd.d
```

**Step 9.** Upload motd scripts to `/etc/update-motd.d`, and change the access permissions to 755. Scripts will be load by order of filename, i.e., 10-logo will be load before 20-sysinfo.

**Step 10.** Re-connect system by ssh should see customize motd.

## Troubleshooting

* If load motd fail, check error message by run `/usr/sbin/update-motd`. If the message is `ERROR: could not generate new MOTD`, it's may syntax error in your motd scripts.

+ If motd show twice when you login, try comment out `session    optional     pam_motd.so  noupdate` in `/etc/pam.d/sshd`. **(It may not working.)**
