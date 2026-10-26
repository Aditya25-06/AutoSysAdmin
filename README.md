<div align="center">

# 🐧 AutoSysAdmin

### Linux System Administration & Monitoring Toolkit

## 👤 Author

**Aditya Mangalam** · UID: 26MCC20078
Master of Computer Applications (MCA), Chandigarh University
Supervisor: Mr. Ajay Yadav (E13382)
#----
A menu-driven **Bash** toolkit for **CentOS Stream 9** that automates everyday admin tasks:
storage & LVM, users & ACLs, log analysis, SELinux checks, process control and cron scheduling.

![Bash](https://img.shields.io/badge/Bash-5.x-4EAA25?logo=gnubash&logoColor=white)
![CentOS](https://img.shields.io/badge/CentOS_Stream-10-262577?logo=centos&logoColor=white)
![Platform](https://img.shields.io/badge/RHEL--compatible-yes-EE0000?logo=redhat&logoColor=white)
![Lines](https://img.shields.io/badge/code-~630_lines-blue)

</div>

---

## 📖 About

AutoSysAdmin was built as an end-semester **Linux Administration Laboratory** project.
It turns the six lab experiments into six reusable modules that share one menu, one logging
system and one safety layer, so the commands learned in class become a single, repeatable tool.

| Exp | Topic | Toolkit module |
|-----|-------|----------------|
| 1 | Introduction, boot process, installation, `dnf`/`yum` repos | System snapshot, setup guide |
| 2 | Partitions, LVM, swap, file systems | `modules/disk_lvm.sh` |
| 3 | Users, groups, `chmod`/`chgrp`/`chown`, ACLs | `modules/user_mgmt.sh` |
| 4 | `grep`, `egrep`, `fgrep`, `sed`, `sort`, `uniq` pipelines | `modules/log_analyzer.sh` |
| 5 | SELinux modes and contexts, root password recovery | `modules/selinux_check.sh` |
| 6 | `ps`, `kill`, `jobs`, `fg`, `bg`, `cron` | `modules/process_monitor.sh`, `modules/scheduler.sh` |

## ✨ Features

- 💽 **Storage:** disk overview, usage alerts, `fdisk` helper, LVM wizard (PV → VG → LV → XFS → mount → `fstab`), LV extend, swap file
- 👥 **Users & access:** create users with custom home, shell and **expiry date**; groups; shared folder with **setgid + ACL**; owner/group/mode fixer; user report
- 🔍 **Log analyzer:** failed-login summary, **top attacking IPs**, most-targeted usernames, failed logins per hour, regex search, IP anonymiser
- 🛡️ **SELinux:** mode status, temporary mode switch, explain a file's context, `restorecon`, recent AVC denials
- ⚙️ **Processes:** system snapshot, top CPU/memory processes, zombie check, resource alerts, polite `kill` (SIGTERM, then optional SIGKILL), background-job demo
- ⏰ **Automation:** schedule a daily health report and weekly `/etc` backup with `cron`; list or remove toolkit jobs
- 📝 **Audit trail:** every action is logged to `/var/log/sysadmin-toolkit.log`

## 📂 Project structure

```
linux-sysadmin-toolkit/
├── toolkit.sh                  # controller: menus + --report / --backup flags
├── lib/
│   └── common.sh               # colours, logging, validation, confirm(), require_root()
├── modules/
│   ├── disk_lvm.sh             # Experiments 1-2
│   ├── user_mgmt.sh            # Experiment 3
│   ├── log_analyzer.sh         # Experiment 4
│   ├── selinux_check.sh        # Experiment 5
│   ├── process_monitor.sh      # Experiment 6
│   └── scheduler.sh            # Experiment 6
├── sample/
│   ├── generate_sample_log.sh  # creates a reproducible test log
│   └── secure_sample.log       # 745-line sample SSH log
└── README.md
```

## 🧰 Requirements

| Item | Requirement |
|------|-------------|
| OS | CentOS Stream 9 (also works on RHEL 9, Rocky, AlmaLinux, Fedora) |
| Shell | Bash 4+ |
| Privileges | `sudo` / root for storage, user, SELinux and cron features |
| Packages | `unzip lvm2 acl policycoreutils audit cronie git` |
| Spare disk | A second 5 GB virtual disk (only for the LVM test) |

## 🚀 Installation

```bash
# 1. Install the packages the toolkit uses
sudo dnf install -y unzip lvm2 acl policycoreutils audit cronie git
sudo systemctl enable --now crond

# 2. Get the project (choose one)
git clone https://github.com/<your-username>/linux-sysadmin-toolkit.git
#   or: unzip linux-sysadmin-toolkit-centos.zip

# 3. Make the scripts executable
cd linux-sysadmin-toolkit
chmod +x toolkit.sh sample/generate_sample_log.sh
```

## ▶️ Usage

```bash
sudo ./toolkit.sh                    # interactive menu
sudo ./toolkit.sh --report           # one-shot health report (used by cron)
sudo ./toolkit.sh --backup           # one-shot /etc backup   (used by cron)
./toolkit.sh --help                  # show usage
```

**Try the log analyzer safely, with no root and no real logs:**

```bash
LOG_FILE=sample/secure_sample.log ./toolkit.sh      # then choose 3) Log Analyzer
```

### Main menu

```
  ==============================================
     AutoSysAdmin  |  Linux Admin Toolkit v1.0
  ==============================================
  Host: centos9   User: root   SELinux: Enforcing

  1) Storage, LVM and Swap        (Experiments 1-2)
  2) Users, Groups and ACLs       (Experiment 3)
  3) Log Analyzer                 (Experiment 4)
  4) SELinux Security             (Experiment 5)
  5) Processes and Scheduling     (Experiment 6)
  6) Generate full health report
  0) Exit
```

### Example: the log-analyzer pipeline

```bash
grep -E "Failed password" /var/log/secure \
  | sed -E 's/.* from ([0-9]{1,3}(\.[0-9]{1,3}){3}) .*/\1/' \
  | sort | uniq -c | sort -rn | head -5
```

| Stage | Command | Purpose |
|-------|---------|---------|
| 1 | `grep -E` | keep only failed-password lines |
| 2 | `sed -E 's/.. \1/'` | extract just the IP address (regex capture group) |
| 3 | `sort` | group identical IPs together |
| 4 | `uniq -c` | count each IP |
| 5 | `sort -rn \| head -5` | rank and keep the top five |

Sample result on `sample/secure_sample.log`: `203.0.113.45` is the top source with **161** of **420** failed attempts.

## 💽 LVM walk-through (Experiments 1-2)

1. Shut the VM down and add a **5 GB virtual disk** in VMware/VirtualBox, then power on.
2. Run `lsblk`: the new empty disk is usually `/dev/sdb` (your system disk is `/dev/sda`, **never use it**).
3. `sudo ./toolkit.sh` → **1) Storage** → **5) Create LVM volume**:

   | Prompt | Example |
   |--------|---------|
   | Disk | `/dev/sdb` |
   | Volume group | `vg_data` |
   | Logical volume | `lv_data` |
   | Size | `2G` |
   | Mount point | `/data` |

4. Verify: `lvs` and `df -h /data`.
5. Grow it later: **6) Extend logical volume** → `+500M` (`-r` resizes the XFS file system too).

## ⏰ Scheduling

| Cron expression | Meaning | Job |
|-----------------|---------|-----|
| `0 6 * * *` | every day, 06:00 | `toolkit.sh --report` |
| `0 * * * *` | every hour | `toolkit.sh --report` (optional) |
| `30 2 * * 0` | Sunday, 02:30 | `toolkit.sh --backup` (tar.gz of `/etc`, 14-day retention) |

Reports are saved in `/var/log/toolkit-reports/`; backups in `/var/backups/toolkit/`.
Toolkit jobs are tagged `# sysadmin-toolkit` in `crontab -l`, so they can be removed safely.

## 🔐 Recovering a forgotten root password (Experiment 5)

On **your own** CentOS Stream 9 VM:

1. Reboot; at the GRUB menu press **`e`** on the first entry.
2. Add `rd.break` at the end of the line that starts with `linux`, then press **Ctrl+X**.
3. At the emergency shell:
   ```bash
   mount -o remount,rw /sysroot
   chroot /sysroot
   passwd root
   touch /.autorelabel      # SELinux relabels files after the change
   exit
   exit
   ```
4. The system relabels and reboots; log in with the new password.

> Use this only on machines you own or administer. A GRUB password and physical-access
> control are the standard protection against this method.

## 🛡️ Safety features

- Destructive actions (format, delete, kill) always ask for **confirmation**.
- Usernames, group names, dates, PIDs and device paths are **validated** before use.
- System accounts (UID < 1000) and **PID 1** cannot be deleted or killed.
- Privileged functions stop politely when not run as root.
- `/etc/fstab` is backed up before it is modified.
- **Test storage features only on a virtual machine** with a spare virtual disk.

## 🧪 Testing

```bash
for f in toolkit.sh lib/*.sh modules/*.sh; do bash -n "$f" && echo "OK $f"; done   # syntax check
bash sample/generate_sample_log.sh                                                 # rebuild sample log
LOG_FILE=sample/secure_sample.log ./toolkit.sh                                     # try the analyzer
```

Expected on the sample log: 420 failed attempts, 44 successful logins, top IP `203.0.113.45`.

## 🩺 Troubleshooting

| Problem | Fix |
|---------|-----|
| `sudo: ./toolkit.sh: command not found` | You are in the wrong folder. Run `cd ~/linux-sysadmin-toolkit` and `ls` to confirm `toolkit.sh` is there |
| `Permission denied` | `chmod +x toolkit.sh` |
| `bad interpreter` or `^M` errors | Windows line endings: `sudo dnf install -y dos2unix` then `dos2unix toolkit.sh lib/*.sh modules/*.sh sample/*.sh` |
| "needs root privileges" | Start with `sudo ./toolkit.sh` |
| "No readable log found" | Run as root, or use `LOG_FILE=sample/secure_sample.log` |
| `pvcreate` / `lvs` not found | `sudo dnf install -y lvm2` |
| `setfacl` not found | `sudo dnf install -y acl` |
| Cron jobs never run | `sudo systemctl enable --now crond` |

## 🔭 Future scope

- Collect logs from several servers over SSH or `rsyslog`
- Automatic blocking of repeat attackers with `firewalld` or `fail2ban`
- E-mail alerts and Ansible playbooks for multi-server use
- Web dashboard or `dialog`-based interface

## 📄 License

Released for educational use. Add a `LICENSE` file (for example MIT) if you publish it publicly.
