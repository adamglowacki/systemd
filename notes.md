It all started with Unix, more than 50 years ago. Our Linux shares the
philosophy, is based on Unix. Or Unixes, to be more precise.

At some point in time, there came a Unix named System V. In this specific Unix,
appeared a thing called init scripts. It was a way to boot the system and
manage all its components.

Very early at the boot time, a process called "init" is created. It gets PID 1
always. It's a grandfather of every other process in the system. This process
must know how to start all the other system components --- how to set up
network, how to run webserver, logging daemon, X11 server, logind and so on. It
is provided with just a bunch of shell scripts and it runs them at certain
points of time. Namely, when a "runlevel" is being changed. It's like a stage
of the system's life.

The runlevel is just a number. It may look like this.

Having a shell script that is just run is very flexible, one can do virtually
anything but it has a few disadvantages:
* with great power comes great responsibility -- you have to be careful what
  you are doing
* simple things have to be "coded" from scratch
* if init knows nothing about what's going on, it can't perform any
  tricky/aggressive parallelization/optimization
* it takes time to spawn shell for each script
* only root can modify the scripts (though systemd also had this limitation for
  a long time)

So guys from Red Hat came up with an idea of a new implementation of init
process with a more declarative configuration. And it became widely popular
although there are strong opponents of this approach. Some Linux distributions
split based on that --- see Debian vs Devuan.

---

What was traditionally called a daemon in Unix world -- a long running
background process -- is called a service in systemd.

---

How a system daemon was started in the past?

What we can do since we have no root? We can't use this machinery. So we use
CRON to run our app at certain moment (on boot). We need to somehow cope with
running/restarting our app during deployment -- we either use `nohup` or create
another CRON item for a moment in the very near future. The first option is not
necessarily perfect, for example different env vars. It also looks different in
the list of processes. Effectively, CRON line is not tested until reboot.

---

So now we can do differently. We can just shortly describe our service and ask
systemd to run it now or on boot (or both). In exactly the same way.

A few years ago, using as non-root wasn't yet supported. If I recall correctly,
it wasn't supported when I discovered it in my previous job. I had to install
the services in the system directories and start/stop them as root. But one can
specify in the service description to be rather run under the name of another
user (drop root permissions).

Now we can do it without being root.

---

This is how we can specify a simple long-running background service.

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
