#### Disclaimer

_As a biology teacher, I must say that although I am a Linux enthusiast, I may lack deeper technical knowledge related to operating systems, kernels, development, etc. Therefore, I ask you to view this article as a collaborative study. As you may notice, all references used are listed at the end._

# Logging on Arch Linux

As a Linux user, it is fundamental to learn how to check logs in order to identify the causes of different problems. Therefore, Linux systems (LS) usually provide multiple ways to accomplish this — either by accessing the `/var/log` directory directly or by using tools such as `dmesg` (included in the <a href="https://archlinux.org/packages/core/x86_64/util-linux/">util-linux</a> package) and `journalctl` (provided by systemd). With that in mind, there are some other commands that may prove helpful when you find yourself thinking about what went wrong, such as:
- `history`
- `coredumpctl`

In this article, we discuss each of these commands, focusing, primarily, on the `journalctl` command, as it is the main way of logging in LS nowadays.

### The logger command

Before we really begin, it is useful to keep in mind that you can use the `logger` command to generate custom log entries. This is a convenient way to verify whether logging is working properly. Try:

```sh
$ logger -p daemon.info "How would I check this entry if I had no logging daemon?"
```

### The old way of doing things

First proposed in the 1980s, `syslog` is a standard/protocol for message logging. It defines how log entries must be formatted and transported.

With regard to its implementation, syslog was originally provided through the syslog daemon, which, according to <a href="https://linux.die.net/man/8/syslogd">syslogd(8)</a>, offered two main utilities: (i) support for system logging and (ii) kernel message trapping.

When discussing syslog, the concepts of _facility_ — a general category of message (e.g., kern, user, root) — and _severity_ — the order of event priorities (e.g. err, crit, emerg) — are fundamental to recognize the omnipresence of this standard. 

As user and system log management needs changed over time, so did syslog implementations, resulting in many variations. Two of them, `rsyslogd` and `syslog-ng`, deserve particular attention. In short:
- `syslog-ng`: released almost two decades after the original syslog implementation, it extended the syslog model with advanced filtering, parsing, and flexible log destinations;
- `rsyslogd`: a modern and highly performant implementation of syslogd, capable of advanced filtering and processing; it supports flexible and customizable output format definitions.


### Journalctl

Any Linux system that uses systemd as its init system (such as Arch Linux) relies on the journal service as its primary logging mechanism. Thus, the `journalctl` command, as described in <a href="https://man.archlinux.org/man/journalctl.1">journalctl(1)</a>, is “used to print the log entries stored in the journal by systemd-journald.service”.

As explained above, `journalctl` provides information about events occurring throughout the system. This can be done either statically, by outputting past log entries, or continuously, by following logs in real time.

The key to diagnosing problems with `journalctl` lies in understanding how to properly filter its output. For this reason, it is important to be familiar with some matching and output parameters. A few useful examples are shown below:

```sh
$ journalctl -x                            # outputs log entries alongside some explanation data
$ journalctl -f                            # follows the log entries happening in the machine
$ journalctl -u [UNIT_NAME]                # focus on a specific unit
$ journalctl --since "YYYY-MM-DD HH:MM:SS" # outputs log entries since a specific date
$ journalctl _UID=[USER_UID]               # outputs log entries specific to a user
$ journalctl _PID=[PROCESS]                # outputs log entries specific to a process
$ journalctl --vacuum-size=[SIZE]          # defines a maximum size for log outputs (used to clear excess log files)
$ journalctl -p err..alert                 # outputs all errors, critical and alerts entries
$ journalctl -b [DESIRED_BOOT]             # defines a specific boot log to look up
$ journalctl --grep="[MATCH]"              # searches entries by match
$ journalctl -n [N]                        # outputs a number N of events
```

### Dmesg

As stated in <a href="https://man.archlinux.org/man/dmesg.1">dmesg(1)</a>, its main purpose is to examine or control the kernel ring buffer — a region of memory reserved for recording kernel activity, which allows the system to log events that occurred early in the boot process.

_Keep in mind that, since the information read by `dmesg` is stored in RAM, it is cleared when the system is powered off. In other words, it is not possible to inspect logs from previous boots using `dmesg`._

Although the system journal also provides kernel-related information (with output similar to `dmesg`, via the `journalctl -k` command), it is still useful to know how to use `dmesg` directly when necessary. Below are some usage examples:

```sh
$ dmesg                                    # outputs kernel messages since the last boot
$ dmesg -w                                 # follows kernel messages in real time
$ dmesg -k                                 # focuses only on kernel-related messages
$ dmesg --since "YYYY-MM-DD HH:MM:SS"      # outputs kernel messages since a specific date and time
$ dmesg | grep -i [MATCH]                  # filters kernel messages by match (device, driver, error)
$ dmesg -l err                             # outputs only error-level kernel messages
$ dmesg -l warn,err                        # outputs warnings and errors
$ dmesg -T                                 # shows human-readable timestamps
$ dmesg | tail -n [N]                      # outputs the last N kernel messages
$ dmesg -c                                 # outputs and clears the kernel ring buffer
```

### Coredumpctl

_Soon to be written_

```sh
$ coredumpctl
$ coredumpctl info
$ coredumpctl debug [PID]
```

### The history command

_Soon to be written_


<!-- Check 
All that information are stored in `/var/log/messages`, `/var/log/dmesg`, and `journalctl`.*
### Systemctl Basics

```sh
$ systemctl                                     # displays all unit running
$ systemctl status [UNIT_NAME]                  # checks status for a specific unit
$ systemctl enable [UNIT_NAME]                  # enables a specific unit
$ systemctl disable [UNIT_NAME]                 # disables a specific unit
$ systemctl start [UNIT_NAME]                   # starts a specific unit
$ systemctl stop [UNIT_NAME]                    # stops a specific unit
$ systemctl enable [UNIT_NAME]@[USER_NAME]      # enables a unit under a specific user
```

Look up `/usr/lib/systemd/system/` to check and edit unit files.

### Checking things up with Atop

- `atop`
_Soon to be written_


Falar sobre monitoramento de sistema com /proc /sys
-->

### References

1. Arch Linux. (n.d.-a). util-linux (core/x86_64). Retrieved December 16, 2025, from https://archlinux.org/packages/core/x86_64/util-linux/
2. Arch Linux. (n.d.-b). journalctl(1): Query the systemd journal. Retrieved December 16, 2025, from https://man.archlinux.org/man/journalctl.1
3. Arch Linux. (n.d.-c). dmesg(1): Print or control the kernel ring buffer. Retrieved December 16, 2025, from https://man.archlinux.org/man/dmesg.1
4. Arch Linux Wiki. (n.d.-a). Syslog-ng. Retrieved December 16, 2025, from https://wiki.archlinux.org/title/Syslog-ng
5. Arch Linux Wiki. (n.d.-b). Systemd/Journal. Retrieved December 16, 2025, from https://wiki.archlinux.org/title/Systemd/Journal
6. Boldrito, R., & Esteve, J. (2019). GNU/Linux advanced administration.
7. Linux Die. (n.d.-a). syslogd(8): System log daemon. Retrieved December 16, 2025, from https://linux.die.net/man/8/syslogd
8. Linux Die. (n.d.-b). rsyslogd(8): Reliable and extended system logging daemon. Retrieved December 16, 2025, from https://linux.die.net/man/8/rsyslogd
9. Ward, B. (2014). How Linux works: What every superuser should know.
