# Fail2Ban Configuration for WordPress Security

This guide provides the necessary configurations to secure a WordPress installation using Fail2Ban. It includes jails for blocking brute-force attacks on `wp-login.php`, `xmlrpc.php`, and WordPress scanners.

---

## 📌 1️⃣ Install Fail2Ban (if not installed)

```bash
sudo apt update && sudo apt install fail2ban -y
```

---

## 📌 2️⃣ Configure Fail2Ban Jails

Create and edit the Fail2Ban jail configuration file:

```bash
sudo nano /etc/fail2ban/jail.d/wordpress.conf
```

Add the following content:

```ini
[wordpress-wplogin]
enabled = true
filter = wordpress-wplogin
logpath = /var/log/nginx/access.log
maxretry = 3
findtime = 300
bantime = 3600
action = iptables-multiport[name=wordpress-wplogin, port="http,https", protocol=tcp]

[wordpress-xmlrpc]
enabled = true
filter = wordpress-xmlrpc
logpath = /var/log/nginx/access.log
maxretry = 2
findtime = 300
bantime = 3600
action = iptables-multiport[name=wordpress-xmlrpc, port="http,https", protocol=tcp]

[block-wp-scanners]
enabled = true
filter = block-wp-scanners
logpath = /var/log/nginx/access.log
maxretry = 5
findtime = 3600
bantime = 86400
action = iptables-multiport[name=block-wp-scanners, port="http,https", protocol=tcp]
```

## For VestaCP

```ini
[wordpress-xmlrpc]
enabled = true
filter = wordpress-xmlrpc
logpath = /var/log/nginx/access.log
maxretry = 2
findtime = 300
bantime = 86400
action  = vesta[name=WEB]
port    = http,https

[wordpress-wplogin]
enabled = true
filter = wordpress-wplogin
logpath = /var/log/nginx/access.log
maxretry = 5
findtime = 300
bantime = 86400
action  = vesta[name=WEB]
port    = http,https

[block-wp-scanners]
enabled = true
filter = block-wp-scanners
logpath = /var/log/nginx/access.log
maxretry = 5
findtime = 3600
bantime = 86400
action  = vesta[name=WEB]
port    = http,https
```

Save and exit (`CTRL + X`, `Y`, `ENTER`).

---

## 📌 3️⃣ Create Filter Files

### 🔹 WordPress Login (`wordpress-wplogin.conf`)

```bash
sudo nano /etc/fail2ban/filter.d/wordpress-wplogin.conf
```

Add:

```ini
[Definition]
failregex = ^<HOST> - - \[.*\] (?:GET|POST) .*wp-login\.php.* HTTP/.*
```

Save and exit.

### 🔹 WordPress XML-RPC (`wordpress-xmlrpc.conf`)

```bash
sudo nano /etc/fail2ban/filter.d/wordpress-xmlrpc.conf
```

Add:

```ini
[Definition]
failregex = ^<HOST> - - \[.*\] (?:GET|POST) .*xmlrpc\.php.* HTTP/.*
```

Save and exit.

### 🔹 WordPress Scanners (`block-wp-scanners.conf`)

```bash
sudo nano /etc/fail2ban/filter.d/block-wp-scanners.conf
```

Add:

```ini
[Definition]
failregex = ^<HOST> - - \[.*\] (?:GET|POST) .*(wp-content|wp-includes|wp-admin|wp-config|wp-cron|wp-json).* HTTP/.*
```

Save and exit.

---

## 📌 4️⃣ Restart Fail2Ban & Apply Changes

```bash
sudo systemctl restart fail2ban
sudo fail2ban-client reload
```

---

## 📌 5️⃣ Verify Jail Status

Check if Fail2Ban is banning IPs:

```bash
sudo fail2ban-client status wordpress-wplogin
sudo fail2ban-client status wordpress-xmlrpc
sudo fail2ban-client status block-wp-scanners
```

Expected output should show `Currently banned: X` if it detects malicious activity.

---

## 📌 6️⃣ Test the Regex Filters

Run the following commands to verify each filter against Nginx logs:

```bash
sudo fail2ban-regex /var/log/nginx/access.log /etc/fail2ban/filter.d/wordpress-wplogin.conf --print-all-matched
sudo fail2ban-regex /var/log/nginx/access.log /etc/fail2ban/filter.d/wordpress-xmlrpc.conf --print-all-matched
sudo fail2ban-regex /var/log/nginx/access.log /etc/fail2ban/filter.d/block-wp-scanners.conf --print-all-matched
```

If the `Failregex` count is greater than `0`, then the filters are working.

---

## 📌 7️⃣ Unbanning an IP (If Needed)

```bash
sudo fail2ban-client set wordpress-wplogin unbanip <IP_ADDRESS>
sudo fail2ban-client set wordpress-xmlrpc unbanip <IP_ADDRESS>
sudo fail2ban-client set block-wp-scanners unbanip <IP_ADDRESS>
```

Replace `<IP_ADDRESS>` with the actual IP you want to unban.

---

## ✅ Conclusion

With this setup, Fail2Ban will automatically block brute-force login attempts, XML-RPC abuse, and WordPress scanners, improving security significantly.

---

### 🚀 Contribute & Report Issues

If you have improvements or issues, feel free to open a PR or issue on GitHub.
