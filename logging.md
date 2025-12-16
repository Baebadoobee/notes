<div id="#top"></div>

#### Disclaimer

_As a biology teacher, I must say that although I am a Linux enthusiast, I may lack deeper technical knowledge related to operating systems, kernels, development, etc. Therefore, I ask you to view this article as a collaborative study. As you may notice, all references used are listed at the end._

# Logging on Arch Linux

As a Linux user, it is fundamental to learn how to check logs in order to identify the causes of different problems. Therefore, Linux systems (LS) usually provide multiple ways to accomplish this — either by accessing the `/var/log` directory directly or by using tools such as `dmesg` (included in the <a href="https://archlinux.org/packages/core/x86_64/util-linux/">util-linux</a> package) and `journalctl` (provided by <a href="https://man.archlinux.org/man/systemd.1">systemd</a>). With that in mind, there are some other commands that may prove helpful when you find yourself thinking about what went wrong, such as:
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

Any Linux system that uses systemd as its init system (such as Arch Linux) relies on the journal service as its primary logging mechanism. Thus, the `journalctl` command, as described in <a href="https://man.archlinux.org/man/journalctl.1">journalctl(1)</a>, is “used to print the log entries stored in the journal by `systemd-journald.service`”.

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
$ dmesg                               # outputs kernel messages since the last boot
$ dmesg -w                            # follows kernel messages in real time
$ dmesg -k                            # focuses only on kernel-related messages
$ dmesg --since "YYYY-MM-DD HH:MM:SS" # outputs kernel messages since a specific date and time
$ dmesg | grep -i [MATCH]             # filters kernel messages by match (device, driver, error)
$ dmesg -l err                        # outputs only error-level kernel messages
$ dmesg -l warn,err                   # outputs warnings and errors
$ dmesg -T                            # shows human-readable timestamps
$ dmesg | tail -n [N]                 # outputs the last N kernel messages
$ dmesg -c                            # outputs and clears the kernel ring buffer
```

### Coredumpctl

At this point, it must be clear that systemd plays an important role in system logging. Alongside its journal, it also provides the `systemd-coredump` service, which is responsible for processing core dumps — files that retain a process’s working memory state at a given moment, usually when it crashes. Since the job of systemd-coredump is to preserve this information, the main purpose of <a href="https://man.archlinux.org/man/coredumpctl.1">coredumpctl(1)</a> is to provide a tool for retrieving and processing previously saved core dumps and their associated metadata.

Some common usage examples include:

```sh
$ coredumpctl             # lists all core dumps recorded by systemd-coredump
$ coredumpctl info        # shows detailed information and metadata about recorded core dumps
$ coredumpctl debug [PID] # opens the core dump of the specified process in a debugger (usually gdb)
$ coredumpctl -1          # selects the most recent core dump entry
$ coredumpctl -S          # filters core dumps starting from a specific time (e.g., today, yesterday, or a date)
```

A careful reader might ask: _“Why can’t I just use journalctl to grep core dumps?”_

*Actually, you can!* However, the issue is that `journalctl` does not provide dedicated commands such as `info` or `debug`. Thus because it lacks some of the specialized utilities offered by `coredumpctl`, it is generally preferable to use `coredumpctl` when working with core dumps.

### The history command

To wrap things up, the `history` command is quite simple. It keeps track of recently executed commands, referred to as "events" (<a href="https://man.archlinux.org/man/history.n">history(n)</a>). It also provides several operations, such as:

```sh
$ history                     # displays the history of executed commands
$ history clear               # clears the command history
$ history keep [N]            # keeps the last N commands in history
$ history redo [EVENT_NUMBER] # re-executes a specific event by its number

# Event numbers are always negative.
```

### Conclusion

Mastering Linux is a challenging (and sometimes tedious) process that requires careful attention and a willingness to learn, one step at a time. Logging is not only one of the GNU/Linux fundamentals, but also an essential practice for properly understanding, managing, and troubleshooting an operating system. With that in mind, this article contextualized logging on Arch Linux by discussing and presenting several log management utilities, such as `journalctl`, `dmesg`, `coredumpctl`, and `history`.

### References

1. Arch Linux. (n.d.). Core dump. https://wiki.archlinux.org/title/Core_dump
2. Arch Linux. (n.d.). coredumpctl(1): retrive and process core dumps. https://man.archlinux.org/man/coredumpctl.1
3. Arch Linux. (n.d.). dmesg(1): Print or control the kernel ring buffer. https://man.archlinux.org/man/dmesg.1
4. Arch Linux. (n.d.). history(n): Manipulate the history list. https://man.archlinux.org/man/history.n
5. Arch Linux. (n.d.). journalctl(1): Query the systemd journal. https://man.archlinux.org/man/journalctl.1
6. Arch Linux. (n.d.). Rsyslog. https://wiki.archlinux.org/title/Rsyslog
7. Arch Linux. (n.d.). Syslog-ng. https://wiki.archlinux.org/title/Syslog-ng
8. Arch Linux. (n.d.). Systemd. https://wiki.archlinux.org/title/Systemd
9. Arch Linux. (n.d.). systemd(1): system and service manager. https://man.archlinux.org/man/systemd.1
10. Arch Linux. (n.d.). systemd-coredump(8): systemd core dump handler. https://man.archlinux.org/man/systemd-coredump.8
11. Arch Linux. (n.d.). Systemd/Journal. https://wiki.archlinux.org/title/Systemd/Journal
12. Arch Linux. (n.d.). util-linux (core/x86_64). https://archlinux.org/packages/core/x86_64/util-linux/
13. Boldrito, R., & Esteve, J. (2019). GNU/Linux advanced administration.
14. Linux Die. (n.d.). syslogd(8): System log daemon. https://linux.die.net/man/8/syslogd
15. Linux Die. (n.d.). rsyslogd(8): Reliable and extended system logging daemon. https://linux.die.net/man/8/rsyslogd
16. Schroder, C. (2021). Linux cookbook.
17. Ward, B. (2014). How Linux works: What every superuser should know.

##

<p align="right"><a href="#top">back to top</a></p>

### Contributors:

<a href="https://github.com/Baebadoobee/postInstallScript/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=Baebadoobee/postInstallScript" alt="contrib.rocks image" />
</a>

