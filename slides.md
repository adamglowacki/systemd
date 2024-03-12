---
author: ""
date: ""
paging: "%d / %d"
---

# Unix

* 1969 at Bell Labs
* 1983 -- System V (init scripts)
* ...
* 1991 -- Linux
* 2010 -- systemd (Red Hat) was released
* now -- (almost) everybody uses systemd

```
init
 │
 ├─logind──bash
 │
 ├─sshd
 │
 ├─mysqld
 │
 └─httpd
 ```

---

# Classic approach
* init starts a process at switch of runlevel (on boot)
* use runlevels to keep the right order of running
* redirect output to file
* write PID to file
* sequential launch

# Our approach
* CRON starts a process at boot
* user starts a process at deployment/reconfiguration
  * detach from terminal
  * env vars

---

# systemd approach
* describe the "service" in file
* use `systemctl` to start now or on boot, stop, restart, ...
* journald for logs

# Files
## For us
* `~/.config/systemd/user/`

## If we have `root`
* `/etc/systemd/system/`
* `/etc/systemd/user/`
* ...

---

# nc.service
```
[Unit]
Description=Listen and accept incoming connections

[Service]
Type=exec
ExecStart=nc -klv 12345
StandardOutput=append:/home/adas/nc.log
StandardError=inherit
Restart=always
RestartSec=3

[Install]
WantedBy=default.target
```

---

# postgresql.service
```
[Unit]
Description=PostgreSQL database server

[Service]
Type=oneshot
ExecStart=pg_ctl -D /home/adas/db/data -l /home/adas/db/log -w -t 600 start
ExecStop=pg_ctl -D /home/adas/db/data -l /home/adas/db/log -w -t 600 stop
RemainAfterExit=yes

[Install]
WantedBy=default.target
```

---

```
$ loginctl enable-linger strats
$ loginctl show-user strats | grep Linger
Linger=yes

```

---

```
$ systemctl --user daemon-reload
$ systemctl --user start nc
$ systemctl --user status nc
● nc.service - Listen and accept incoming connections
     Loaded: loaded (/home/adas/.config/systemd/user/nc.service; enabled; preset: disabled)
     Active: active (running) since Tue 2024-03-05 11:52:43 CET; 4s ago
   Main PID: 1625 (nc)
      Tasks: 1 (limit: 11018)
     Memory: 788.0K
        CPU: 4ms
     CGroup: /user.slice/user-1000.slice/user@1000.service/app.slice/nc.service
             └─1625 nc -klv 12345

Mar 05 11:52:43 andromeda systemd[1415]: Starting Listen and accept incoming connections...
Mar 05 11:52:43 andromeda systemd[1415]: Started Listen and accept incoming connections.
```

---

```
$ systemctl --user stop nc
$ systemctl --user enable --now nc

---

# Activation
* when a client connects to a socket (`inetd`)
* over `D-Bus`
* when a file appeared
* when a device is connected

---

# Units
* .service
* .mount
* .device
* .socket
* .swap
* .timer
* .snapshot
* ...

---

# Other components
* systemd-boot
* systemd.mount
* systemd-homed
* systemd-logind
* systemd-networkd
* systemd-resolved
* systemd-timesyncd
* systemd-tmpfiles (/run/user/1000)
* ...

---

```
$ hostnamectl
 Static hostname: andromeda
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: 42bc1f055d0443f799b120629373bcae
         Boot ID: 4968b5522884413c943f622c5ad34ee8
  Virtualization: kvm
Operating System: Rocky Linux 9.3 (Blue Onyx)
     CPE OS Name: cpe:/o:rocky:rocky:9::baseos
          Kernel: Linux 5.14.0-362.18.1.el9_3.0.1.x86_64
    Architecture: x86-64
 Hardware Vendor: Red Hat
  Hardware Model: KVM
Firmware Version: 1.16.1-1.el9
```

---

```
$ loginctl list-sessions
SESSION  UID USER SEAT TTY   STATE  IDLE SINCE
      1 1000 adas      pts/0 active yes  1h 21min ago
      3 1000 adas      pts/1 active no
      4 1000 adas      pts/2 active no

3 sessions listed.
$ loginctl kill-session 4
$ loginctl list-sessions
SESSION  UID USER SEAT TTY   STATE  IDLE SINCE
      1 1000 adas      pts/0 active yes  1h 21min ago
      3 1000 adas      pts/1 active no

2 sessions listed.
```

---

```
$ timedatectl
               Local time: Tue 2024-03-05 11:58:23 CET
           Universal time: Tue 2024-03-05 10:58:23 UTC
                 RTC time: Tue 2024-03-05 10:58:23
                Time zone: Europe/Warsaw (CET, +0100)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

---

```
$ journalctl | head -n 5
Mar 05 10:31:10 andromeda kernel: Linux version 5.14.0-362.18.1.el9_3.0.1.x86_64 (mockbuild@iad1-prod-build001.bld.equ.rockylinux.org) (gcc (GCC) 11.4.1 20230605 (Red Hat 11.4.1-2), GNU ld version 2.35.2-42.el9) #1 SMP PREEMPT_DYNAMIC Sun Feb 11 13:49:23 UTC 2024
Mar 05 10:31:10 andromeda kernel: The list of certified hardware and cloud instances for Enterprise Linux 9 can be viewed at the Red Hat Ecosystem Catalog, https://catalog.redhat.com.
Mar 05 10:31:10 andromeda kernel: Command line: BOOT_IMAGE=(hd0,msdos1)/vmlinuz-5.14.0-362.18.1.el9_3.0.1.x86_64 root=/dev/mapper/rl_andromeda-root ro crashkernel=1G-4G:192M,4G-64G:256M,64G-:512M resume=/dev/mapper/rl_andromeda-swap rd.lvm.lv=rl_andromeda/root rd.lvm.lv=rl_andromeda/swap rhgb quiet
Mar 05 10:31:10 andromeda kernel: x86/fpu: Supporting XSAVE feature 0x001: 'x87 floating point registers'
Mar 05 10:31:10 andromeda kernel: x86/fpu: Supporting XSAVE feature 0x002: 'SSE registers'
```

---

```
$ journalctl -u sshd.service
Mar 05 10:31:13 andromeda systemd[1]: Starting OpenSSH server daemon...
Mar 05 10:31:13 andromeda sshd[727]: Server listening on 0.0.0.0 port 22.
Mar 05 10:31:13 andromeda sshd[727]: Server listening on :: port 22.
Mar 05 10:31:13 andromeda systemd[1]: Started OpenSSH server daemon.
Mar 05 10:35:49 andromeda sshd[1411]: Accepted publickey for adas from 192.168.17.11 port 44316 ssh2: RSA SHA256:fzBdCFeq/Rp5zM+HYmP6udGjU3boRcg1L8ydXzSO++U
Mar 05 10:35:49 andromeda sshd[1411]: pam_unix(sshd:session): session opened for user adas(uid=1000) by (uid=0)
```

---

```
$ journalctl --disk-usage
Archived and active journals take up 4.4M in the file system.
```

* `--vacuum-size=1GB`
* `--vacuum-time=24h`
* `--vacuum-files=64`

---

```
$ systemd-analyze
Startup finished in 1.457s (kernel) + 1.966s (initrd) + 3.451s (userspace) = 6.875s
multi-user.target reached after 2.197s in userspace.
```

```
$ systemd-analyze
Startup finished in 11.371s (firmware) + 2.449s (loader) + 1.011s (kernel) + 1.718s (initrd) + 20.636s (userspace) = 37.187s
multi-user.target reached after 8.700s in userspace.
```
