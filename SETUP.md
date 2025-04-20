# NFTables Setup Guide (for Debian-based systems)

This guide explains how to set up a secure firewall configuration using **nftables** on Debian-based Linux distributions. It also includes tips for hardening your system and monitoring active SSH sessions.

> ⚠️ Tested on Debian-based distros. File paths and package names may differ on RedHat-based systems.

---

## 🔥 Preliminary Cleanup: Remove Conflicting Firewalls

To prevent conflicts with nftables, remove or disable the following tools:

```bash
sudo systemctl stop firewalld
sudo systemctl disable firewalld
sudo apt remove --purge ufw gufw iptables iptables-persistent
sudo rm -rf /etc/ufw /etc/iptables
```

---

## ✅ Required Services and Tools
Ensure the following services are installed and active:

- **ssh** (OpenSSH server)
- **nftables**
- **fail2ban**
- **cron**

Check SSH port (default is 22):
```bash
sudo netstat -tulnp | grep :22
```

If you modify your SSH configuration, it's good practice to back up the file first:
```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```

> 📍 For Debian-based distros: SSH config is usually in `/etc/ssh/sshd_config`.
> On RedHat-based systems, check your distro's documentation.

---

## 🛠 Cloning and Using the nftables Firewall

1. **Clone this repository**:
```bash
git clone https://github.com/Tahriyan/......(add correct URL)
cd nftables-project
```

2. **List existing rules:**
```bash
sudo nft -a list ruleset
```

3. **Apply your own ruleset:**
```bash
sudo nft -f firewall.nft
```

4. **Monitor active rules:**
```bash
sudo nft list ruleset
sudo nft list counters  # Useful to monitor traffic stats
```

5. **Test your configuration:**
```bash
nc -wl -vz 127.0.0.1 22
```
Then check updated ruleset and counters.

6. **Make nftables persistent:**
Edit the config file (for Debian-based):
```bash
sudo nano /etc/nftables.conf
```
Paste your ruleset or use `include "/path/to/firewall.nft"`

For RedHat-based systems, the file might be:
```
/etc/sysconfig/nftables.conf
```

---

## 🌐 IPv6 Check

Check if IPv6 is enabled:
```bash
cat /proc/sys/net/ipv6/conf/all/disable_ipv6
```
- `0`: IPv6 is enabled
- `1`: IPv6 is disabled

Modify nftables rules accordingly.

---

## 👁 SSH Connection Monitor (Auto-Kick after 3h)

Create a script to monitor SSH connections longer than 3 hours and disconnect them:
```bash
#!/usr/bin/env bash
MAX_TIME=10800

who | grep "pts/" | while read user tty date time rest; do
    login_time=$(who -u | grep "$tty" | awk '{print $3" "$4}')
    login_timestamp=$(date -d "$login_time" +%s)
    current_timestamp=$(date +%s)
    duration=$((current_timestamp - login_timestamp))

    if ((duration > MAX_TIME)); then
        echo "Disconnecting user $user on $tty after $((duration/3600)) hours."
        pid=$(who -u | grep "$tty" | awk '{print $6}')
        kill -HUP $pid
    fi
done
```
Save this as `/usr/local/bin/kick_idle_ssh.sh` and make it executable:
```bash
chmod +x /usr/local/bin/kick_idle_ssh.sh
```
Schedule it with cron (every 5 minutes):
```bash
sudo crontab -e
```
Add:
```cron
*/5 * * * * /usr/local/bin/kick_idle_ssh.sh
```

---

## 🔐 SSH Login Warning Message

Set up a banner to warn users:

1. Edit `/etc/issue.net` and write something like:
```
This system is under monitoring. All access attempts are logged. Unauthorized access is prohibited and may be prosecuted.
```

2. Then enable it in your SSH config:
```bash
sudo nano /etc/ssh/sshd_config
```
Uncomment or add:
```bash
Banner /etc/issue.net
```
Restart SSH:
```bash
sudo systemctl restart ssh
```
Test the login message with a new SSH session.

---

## 🛡 Fail2Ban Configuration

Fail2Ban protects against brute-force attacks. Basic config:

In your jail file (usually `/etc/fail2ban/jail.local` or `/etc/fail2ban/jail.d/defaults-debian.conf`):
```ini
[sshd]
enabled = true
port    = ssh
filter  = sshd
maxretry = 4
ignoreip = 127.0.0.1/8 ::1
bantime  = 480m
findtime = 30s
```

Check the jail status:
```bash
sudo fail2ban-client status sshd
```
Unban an IP if needed:
```bash
sudo fail2ban-client set sshd unbanip 192.168.x.x
```

---

## 👤 SSH Least Privilege Principle

You may create a dedicated SSH user that does **not** belong to the `sudo` group, to reduce the risk of privilege escalation.

---

## ✅ Summary of Key Commands

```bash
sudo nft list ruleset
sudo nft -f firewall.nft
sudo nft list counters
nc -wl -vz 127.0.0.1 22
sudo fail2ban-client status sshd
cat /proc/sys/net/ipv6/conf/all/disable_ipv6
```

---

> 🔒 All configurations tested on Debian-based systems. Please adapt file paths for your specific Linux distribution.

