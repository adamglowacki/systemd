Why init scripts are not perfect:
* everything has to be written manually, from scratch
* start script: usually it's not a single binary to be run (redirect output to
  files, logs should be rotated, PID has to be stored somewhere, it's nice to
  inform the admin about crash, maybe automatic rerun but not too many, ...)
* stop script: locate the process and send a signal
* we can't use them since we are not `root`; so CRON + our own not-init scripts

What it does:
* PID 1 and everything else
* socket & D-Bus activation for starting services (what is that?)
* Linux cgroups to keep track of processes
* mounting
* journalctl
* systemd-boot
* hostname, locale, date
* containers
* virtual machines
* user accounts
* network configuration
* NTP
* name resolution (how?)
* systemd-logind

# systemctl
The most interesting part for us.

`systemctl -H <user>@<host> status sshd.service`
`systemctl --system` vs `systemctl --user`
`loginctl enable-linger <user>`
`systemctl --user daemon-reload`
`systemctl --user enable --now <service>`
`systemctl --user disable --now <service>`
`systemctl --user status <service>`

Get rid of cron and tricky Bash commands to detach from terminal, collect the
logs, find and kill the process. It looks like we are circumventing what the
system was designed for.

At the beginning for `root` only. But with optional switch to another user.
Luckily for us now we can also run as non-`root` user.

## Units

* `.service`
* `.mount`
* `.device`
* `.socket`
* `.timer`
* ...

## Service


## Power management
Together with polkit (kind of more sophisticated `sudo`).

`systemctl reboot`, `systemctl poweroff`, ...

# loginctl

# TODO
* learn about D-Bus
* how to make a service self-restart
* how to notify somebody on crash
* what it means WantedBy=