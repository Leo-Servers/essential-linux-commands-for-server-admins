# essential-linux-commands-for-server-admins
Beginner to Advanced — A Practical Field Guide for server administrators. Master essential Linux commands, bash scripting, systemd, and server troubleshooting.
# Essential Linux Commands for Server Administrators
### Beginner to Advanced — A Practical Field Guide | [Leo Servers](https://www.leoservers.com)

> If you manage servers for a living, you already know that clicking through a web UI will only take you so far. What you reach for is the terminal.

📖 **Full tutorial:** [leoservers.com/tutorials/essential-linux-commands-for-server-admins](https://www.leoservers.com/tutorials/essential-linux-commands-for-server-admins/)

---

## Table of Contents

1. [Navigating and Managing the Filesystem](#1-navigating-and-managing-the-filesystem)
2. [User Accounts, Groups, and File Permissions](#2-user-accounts-groups-and-file-permissions)
3. [Process Management and System Monitoring](#3-process-management-and-system-monitoring)
4. [Service Management with systemd](#4-service-management-with-systemd)
5. [Networking — Diagnostics and Configuration](#5-networking--diagnostics-and-configuration)
6. [Disk Management and Storage](#6-disk-management-and-storage)
7. [Package Management](#7-package-management)
8. [Bash Scripting for Server Automation](#8-bash-scripting-for-server-automation)
9. [Advanced Commands for Power Users](#9-advanced-commands-for-power-users)
10. [A Practical Troubleshooting Framework](#10-a-practical-troubleshooting-framework)
11. [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)

---

## Introduction

Linux powers the vast majority of the world's web servers, cloud infrastructure, containers, and embedded systems. The command-line interface is not a relic from the past — it is the primary interface for serious server work.

Scripts written in Bash run on a schedule and keep systems clean. SSH sessions give you full control over remote machines from anywhere. A single `grep` pipeline can sift through gigabytes of logs in seconds.

Every example in this guide was tested on real servers running **Ubuntu, Debian, AlmaLinux, and Rocky Linux**.

---

## 1. Navigating and Managing the Filesystem

### 1.1 Directory Navigation

| Command | What It Does |
|---|---|
| `pwd` | Print the full path of your current working directory |
| `cd /var/log` | Change into the /var/log directory |
| `cd ..` | Move up one directory level |
| `cd -` | Jump back to the previous directory |
| `ls` | List files and directories |
| `ls -lah` | Long format, human-readable sizes, including hidden files |
| `ls -lt` | List sorted by modification time — newest first |

```bash
# Where are we right now?
pwd

# List all files including hidden ones with sizes
ls -lah

# Find the largest files in /var to identify disk issues
ls -lhS /var/log | head -20
```

> **Tip:** Add `alias ll='ls -lah'` to your `~/.bashrc` and run `source ~/.bashrc`. Now just type `ll`.

### 1.2 File and Directory Operations

| Command | What It Does |
|---|---|
| `mkdir -p /opt/app/logs` | Create nested directories in one command |
| `cp -r /source /dest` | Copy an entire directory tree recursively |
| `cp -p file.conf file.conf.bak` | Copy preserving permissions and timestamps |
| `mv old.conf new.conf` | Rename or move a file |
| `rm file.txt` | Delete a file — permanent, no trash |
| `rm -rf /old/dir/` | Force-delete a directory and everything inside |
| `touch /tmp/testfile` | Create an empty file |
| `ln -s /opt/app/current /opt/app/live` | Create a symbolic link |

> ⚠️ **Warning:** `rm -rf` has no undo. Preview with `echo rm -rf /path` before running.

### 1.3 Finding Files and Searching Content

```bash
# Find all .conf files modified in the last 7 days
find /etc -name '*.conf' -mtime -7

# Find files larger than 100MB
find / -size +100M -type f 2>/dev/null

# Find files owned by a specific user
find /var/www -user www-data -type f

# Search inside files
grep -i 'error' /var/log/nginx/error.log

# Search recursively with filename and line number
grep -rn 'database_host' /etc/myapp/

# Find temp files older than 3 days and delete them
find /tmp -name '*.tmp' -mtime +3 | xargs rm -f
```

### 1.4 Viewing and Editing Files

| Command | What It Does |
|---|---|
| `cat /etc/hostname` | Print entire file contents |
| `less /var/log/syslog` | View a file page by page (q to quit) |
| `head -20 file.log` | Show first 20 lines |
| `tail -50 file.log` | Show last 50 lines |
| `tail -f /var/log/syslog` | Follow a log file in real time |
| `nano /etc/hosts` | Simple terminal editor |
| `vim /etc/nginx/nginx.conf` | Powerful editor (i=insert, :wq=save & quit) |

### 1.5 Archiving and Compressing Files

```bash
# Create a compressed archive
tar -czvf etc-backup-$(date +%F).tar.gz /etc

# Extract an archive
tar -xzvf myapp-v2.tar.gz -C /opt/

# List contents without extracting
tar -tzvf backup.tar.gz

# Compress / decompress a single file
gzip largefile.log
gunzip largefile.log.gz
```

---

## 2. User Accounts, Groups, and File Permissions

### 2.1 Managing User Accounts

| Command | What It Does |
|---|---|
| `useradd -m -s /bin/bash deploy` | Create user with home directory and bash shell |
| `passwd deploy` | Set or change the password for a user |
| `usermod -aG sudo deploy` | Add user to sudo group (Ubuntu/Debian) |
| `usermod -aG wheel deploy` | Add user to wheel group (RHEL/AlmaLinux) |
| `userdel -r olduser` | Delete user and their home directory |
| `id deploy` | Show user ID and group memberships |
| `whoami` | Display the current logged-in username |
| `last` | Show login history |
| `w` | Show who is currently logged in |

```bash
# Create a service account with no login shell
useradd -r -s /sbin/nologin -d /opt/myapp myapp

# Lock / unlock an account
usermod -L username
usermod -U username

# Show all users on the system
cut -d: -f1 /etc/passwd

# Check sudo privileges for a user
sudo -l -U username
```

### 2.2 File Permissions and Ownership

| Command | What It Does |
|---|---|
| `chmod 755 script.sh` | Owner: rwx, Group: r-x, Others: r-x |
| `chmod 644 index.html` | Owner: rw-, Group: r--, Others: r-- |
| `chmod 600 ~/.ssh/id_rsa` | Readable only by owner (required for SSH keys) |
| `chmod -R 750 /opt/myapp/` | Apply permissions recursively |
| `chown www-data:www-data /var/www/` | Change owner and group |
| `chown -R deploy:deploy /opt/app/` | Change ownership recursively |

```
chmod 755  =  owner:7(rwx)  group:5(r-x)  others:5(r-x)
chmod 644  =  owner:6(rw-)  group:4(r--)  others:4(r--)
chmod 600  =  owner:6(rw-)  group:0(---)  others:0(---)
```

### 2.3 Privilege Escalation and sudo

```bash
# Run a command with elevated privileges
sudo apt update

# Run a command as a specific user
sudo -u www-data php artisan migrate

# Open a root shell
sudo -i

# Safely edit the sudoers file
visudo
```

---

## 3. Process Management and System Monitoring

### 3.1 Viewing Processes

| Command | What It Does |
|---|---|
| `ps aux` | Show all running processes |
| `ps aux \| grep nginx` | Filter process list by name |
| `pgrep nginx` | Return the PID for a named process |
| `top` | Real-time interactive process monitor |
| `htop` | Enhanced interactive viewer |
| `pstree` | Show processes as a parent-child tree |

```bash
# Top 10 memory consumers
ps aux --sort=-%mem | head -10

# Top 10 CPU consumers
ps aux --sort=-%cpu | head -10

# Find which process is using port 80
ss -tlnp | grep ':80'
lsof -i :80
```

### 3.2 Controlling and Killing Processes

| Command | What It Does |
|---|---|
| `kill 1234` | Graceful shutdown request (SIGTERM) |
| `kill -9 1234` | Force kill (SIGKILL) |
| `killall nginx` | Kill all processes named 'nginx' |
| `pkill -f 'python worker'` | Kill by command pattern |
| `nice -n 10 ./backup.sh` | Start with lower CPU priority |
| `renice 15 -p 1234` | Change priority of running process |

### 3.3 Keeping Processes Running After Logout

```bash
# Run a process that survives SSH disconnection
nohup ./long-script.sh > /tmp/script.log 2>&1 &

# Better approach: tmux
tmux new -s deploy
# Detach: Ctrl+b, then d
# Reattach:
tmux attach -t deploy

# List all sessions
tmux ls
```

### 3.4 System Resource Monitoring

```bash
uptime          # Load averages: last 1, 5, and 15 minutes
free -h         # Memory usage summary
df -h           # Disk usage on all mounted filesystems
du -sh /var/log/* | sort -rh | head -20
vmstat 2 5      # CPU/memory/IO stats every 2 seconds, 5 samples
iostat -xz 1    # Per-disk I/O statistics in real time
```

---

## 4. Service Management with systemd

### 4.1 Controlling Services

| Command | What It Does |
|---|---|
| `systemctl start nginx` | Start the service |
| `systemctl stop nginx` | Stop the service |
| `systemctl restart nginx` | Restart the service |
| `systemctl reload nginx` | Reload config without full restart |
| `systemctl status nginx` | Check status and recent log lines |
| `systemctl enable nginx` | Start automatically on boot |
| `systemctl disable nginx` | Prevent starting on boot |
| `systemctl list-units --type=service` | List all loaded services |
| `systemctl --failed` | Show all failed services |

```bash
# Full workflow: install, enable, start, verify
sudo apt install nginx
sudo systemctl enable nginx
sudo systemctl start nginx
sudo systemctl status nginx

# Read why a service failed
journalctl -u nginx.service -n 50 --no-pager

# After editing a unit file
sudo systemctl daemon-reload
```

### 4.2 Reading Logs with journalctl

```bash
# Follow logs live
journalctl -u nginx.service -f

# Logs since a specific time
journalctl -u nginx.service --since '2026-03-20 08:00:00'

# Only error-level messages from last boot
journalctl -p err -b

# Logs from previous boot (useful after a crash)
journalctl -b -1

# Check journal disk usage
journalctl --disk-usage

# Trim old journals
sudo journalctl --vacuum-size=500M
```

---

## 5. Networking — Diagnostics and Configuration

### 5.1 Checking Network Interfaces and Routes

```bash
ip addr show    # Show all interfaces with IPs
ip a            # Shorthand
ip route show   # Show the routing table
ip -s link show eth0   # Packet and error statistics
```

> **Note:** `ifconfig` and `netstat` are deprecated. Use `ip addr` and `ss` instead.

### 5.2 Connectivity and DNS Testing

```bash
ping -c 4 google.com
traceroute google.com
mtr google.com                  # Live combined traceroute + ping
dig google.com +short           # DNS resolution
dig -x 8.8.8.8                  # Reverse DNS lookup
nc -zv 10.0.1.50 3306           # Test if a port is reachable
curl -I https://yourdomain.com  # Test an HTTP endpoint
```

### 5.3 Port and Socket Inspection

```bash
ss -tlnp                        # All listening TCP ports with process names
ss -tnp state established       # All established connections
ss -tlnp | grep ':443'          # Find process using port 443
lsof -i :443
```

### 5.4 Firewall Management

**UFW — Ubuntu and Debian**
```bash
sudo ufw allow 22/tcp           # Always allow SSH first!
sudo ufw enable
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow from 203.0.113.10 to any port 22
sudo ufw delete allow 8080
sudo ufw status verbose
```

**Firewalld — RHEL, AlmaLinux, Rocky Linux**
```bash
sudo firewall-cmd --state
sudo firewall-cmd --list-all
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```

### 5.5 Secure Remote Access with SSH

```bash
ssh user@192.168.1.100
ssh -p 2222 user@192.168.1.100
ssh -i ~/.ssh/deploy_key user@192.168.1.100
scp localfile.tar.gz user@192.168.1.100:/opt/app/
rsync -avz --progress /local/dir/ user@remote:/remote/dir/
ssh -L 3307:127.0.0.1:3306 user@192.168.1.100 -N    # Port forwarding
ssh-keygen -t ed25519 -C 'deploy@leoservers.com'
ssh-copy-id user@192.168.1.100
```

---

## 6. Disk Management and Storage

### 6.1 Disk Usage and Space Monitoring

```bash
df -h
du -sh /var/log/* | sort -rh | head -20
find / -xdev -size +500M -exec ls -lh {} + 2>/dev/null
df -i           # Check inode usage
iostat -xz 2
iotop -o        # Processes actively doing I/O
```

### 6.2 Partitioning, Formatting, and Mounting

```bash
lsblk                           # List all block devices
sudo fdisk -l /dev/sdb          # Partition info for a disk
sudo mkfs.ext4 /dev/sdb1        # Format as ext4
sudo mkfs.xfs /dev/sdb1         # Format as XFS
sudo mount /dev/sdb1 /mnt/data  # Mount temporarily
sudo blkid /dev/sdb1            # Get UUID for /etc/fstab
```

### 6.3 LVM — Logical Volume Management

```bash
pvs && vgs && lvs               # Show all LVM info
sudo lvextend -L +20G /dev/mapper/ubuntu--vg-ubuntu--lv
sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv   # ext4
sudo xfs_growfs /mnt/data                          # XFS
```

---

## 7. Package Management

### APT — Debian and Ubuntu

```bash
sudo apt update
sudo apt upgrade
sudo apt install nginx php8.3-fpm mariadb-server
sudo apt remove nginx
sudo apt purge nginx
sudo apt autoremove
apt search 'web server'
apt show nginx
dpkg -l | grep '^ii'
```

### DNF — RHEL, AlmaLinux, Rocky Linux

```bash
sudo dnf update
sudo dnf install nginx
sudo dnf remove nginx
dnf search php
dnf info nginx
dnf list installed
sudo dnf install epel-release
```

---

## 8. Bash Scripting for Server Automation

### 8.1 Script Structure and Best Practices

```bash
#!/bin/bash
set -euo pipefail
# -e = exit on error | -u = error on unset vars | -o pipefail = catch pipe failures

THRESHOLD=85
HOSTNAME=$(hostname -f)

df -h | awk 'NR>1 {gsub(/%/,"",$5); if ($5+0 >= THRESHOLD+0) print $0}' \
  THRESHOLD=$THRESHOLD | while read -r line; do
    echo "DISK ALERT on $HOSTNAME: $line"
  done
```

### 8.2 Variables, Conditionals, and Loops

```bash
APP_DIR='/opt/myapp'

if [ -d "$APP_DIR" ]; then
    echo "App directory exists"
else
    mkdir -p "$APP_DIR"
fi

if systemctl restart nginx; then
    echo 'nginx restarted successfully'
else
    echo 'ERROR: nginx failed to restart' >&2
    exit 1
fi

for config in /etc/nginx/sites-enabled/*; do
    nginx -t -c "$config" && echo "$config: OK"
done
```

### 8.3 Scheduling Tasks with Cron

```bash
crontab -e

# .──────── minute (0–59)
# | .────── hour (0–23)
# | | .──── day of month (1–31)
# | | | .── month (1–12)
# | | | | .─ day of week (0=Sunday)
# * * * * * command

0 2 * * *     /opt/scripts/backup.sh >> /var/log/backup.log 2>&1
0 */12 * * *  /usr/bin/certbot renew --quiet
0 3 * * 0     find /tmp -mtime +7 -delete
```

---

## 9. Advanced Commands for Power Users

### 9.1 Text Processing Pipelines

```bash
# Count HTTP response codes
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c | sort -rn

# Top 10 IP addresses hitting your server
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10

# Replace a string in a config file
sed -i 's/old_hostname/new_hostname/g' /etc/hosts

# Show only lines 100–200 of a large log file
sed -n '100,200p' /var/log/syslog
```

### 9.2 Deep System Diagnostics

```bash
strace -p 1234              # Trace system calls of a running process
lsof -p 1234                # List all open files for a process
lsof /mnt/data              # What's preventing unmount?
journalctl -k | grep -i 'oom\|killed process'
dmesg | grep -i 'out of memory'
dd if=/dev/zero of=/tmp/testfile bs=1G count=1 oflag=dsync   # Disk benchmark
```

### 9.3 Security Checks

```bash
w                           # Who is currently logged in
last -20                    # Recent logins
lastb | head -20            # Recent failed login attempts
grep 'Failed password' /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn | head
find / -perm -4000 -type f 2>/dev/null    # Find SUID files
echo | openssl s_client -connect yourdomain.com:443 2>/dev/null | openssl x509 -noout -dates
openssl rand -base64 24     # Generate a strong random password
```

### 9.4 Useful One-Liners

| Command | What It Does |
|---|---|
| `history \| grep rsync` | Search command history |
| `sudo !!` | Re-run last command with sudo |
| `Ctrl+R` | Reverse search through history |
| `watch -n 5 'df -h'` | Re-run df -h every 5 seconds |
| `curl -s ifconfig.me` | Get the public IP of your server |
| `openssl rand -base64 24` | Generate a strong random password |
| `column -t file.txt` | Format text into aligned columns |

---

## 10. A Practical Troubleshooting Framework

### The Layered Diagnostic Approach

1. Is the server reachable at all? → `ping`, `ssh`
2. Is the service actually running? → `systemctl status`
3. What do the service logs say? → `journalctl -u service -n 100`
4. Are system resources exhausted? → `df -h`, `free -h`, `uptime`, `top`
5. Is the process listening on the correct port? → `ss -tlnp`
6. Is the firewall blocking the connection? → `ufw status`
7. Is DNS resolving correctly? → `dig domain.com +short`
8. Did anything change recently? → `journalctl --since '1 hour ago'`

### Scenario: Website Returning 502 Bad Gateway

```bash
sudo systemctl status nginx
sudo systemctl status php8.3-fpm
sudo tail -50 /var/log/nginx/error.log
ss -tlnp | grep php
ls -la /run/php/
df -h
```

### Scenario: Server is Slow — High Load Average

```bash
uptime
nproc                       # How many CPU cores?
ps aux --sort=-%cpu | head -10
iostat -xz 1 5
iotop -o
free -h
vmstat 1 5                  # Watch 'si' and 'so' columns for swap
```

---

## Quick Reference Cheat Sheet

### Filesystem
| Command | What It Does |
|---|---|
| `ls -lah` | List with permissions, sizes, hidden files |
| `find / -name 'file.txt'` | Find a file by name |
| `grep -rn 'pattern' /dir` | Search file contents recursively |
| `tail -f /var/log/syslog` | Follow a log file in real time |
| `tar -czvf archive.tar.gz /dir` | Create a compressed archive |
| `tar -xzvf archive.tar.gz -C /dest` | Extract archive |

### Users and Permissions
| Command | What It Does |
|---|---|
| `useradd -m -s /bin/bash user` | Create a user with home directory |
| `usermod -aG sudo user` | Add user to the sudo group |
| `chmod 755 file` | Standard permissions for executables |
| `chown user:group file` | Change file owner and group |
| `visudo` | Safely edit the sudoers file |

### Processes and Services
| Command | What It Does |
|---|---|
| `ps aux --sort=-%cpu \| head` | Top CPU-consuming processes |
| `kill -9 PID` | Force kill a process |
| `systemctl status service` | Check if a service is running |
| `systemctl restart service` | Restart a service |
| `journalctl -u service -f` | Follow live service logs |

### Networking
| Command | What It Does |
|---|---|
| `ip a` | Show all network interfaces |
| `ss -tlnp` | Show listening ports with process names |
| `dig domain.com +short` | Quick DNS lookup |
| `nc -zv host port` | Test if a TCP port is open |
| `rsync -avz src/ user@host:/dest/` | Efficient file sync |

### Disk and Storage
| Command | What It Does |
|---|---|
| `df -h` | Disk space on all mounted filesystems |
| `du -sh /path/*` | Space used by each item in a directory |
| `lsblk` | List all block devices |
| `iostat -xz 1` | Real-time per-disk I/O statistics |
| `df -i` | Check inode usage |

---

## About Leo Servers

High-performance dedicated, GPU & storage servers with DDoS protection, unmetered bandwidth & top security across 150+ global locations.

🌐 [leoservers.com](https://www.leoservers.com) | 📧 [marketing@leoservers.com](mailto:marketing@leoservers.com)

**Server Locations:**
[North America](https://www.leoservers.com/north-america-dedicated-servers/) • [South America](https://www.leoservers.com/south-america-dedicated-servers/) • [Europe](https://www.leoservers.com/europe-dedicated-servers/) • [Asia](https://www.leoservers.com/asia-dedicated-servers/) • [Australia](https://www.leoservers.com/australia-dedicated-servers/) • [Africa](https://www.leoservers.com/africa-dedicated-servers/)

