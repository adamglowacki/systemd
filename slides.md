---
author: ""
date: ""
paging: "%d / %d"
---
# Unix

* 1969 at Bell Labs
* 1983 -- System V (runlevels and init scripts)
* ...
* 1991 -- Linux
* 2010 -- systemd (Red Hat) was released
* now -- (almost) everybody uses systemd

## Runlevels

* 0 = off
* 1 = single-user mode
* 2 = multi-user mode
* 3 = multi-user mode with network
* 5 = multi-user mode with network and GUI
* 6 = reboot

---

# Daemon = service

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

---

# Long-running process

```
$ cat .config/systemd/user/nc.service 
[Unit]
Description=Listen and accept incoming connections

[Service]
Type=exec
ExecStart=nc -klv 12345
StandardOutput=append:/home/adas/nc.log
StandardError=inherit

[Install]
WantedBy=default.target
```

---

# Start/stop commands

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

# Use it

* `systemctl --user daemon-reload`
* `systemctl --user status postgresql.service`
* `systemctl --user enable --now postgresql.service`
* `systemctl --user stop postgresql.service`
* `journalctl --user -u postgresql.service`

---

# Other components

* systemd-boot
* systemd.mount (/etc/fstab)
* systemd-homed
* systemd-logind
* systemd-networkd
* systemd-resolved
* systemd-timesyncd
* systemd-tmpfiles (/run/user/1000)
* ...
