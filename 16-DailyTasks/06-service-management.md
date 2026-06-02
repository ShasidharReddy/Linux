# Service Management

This guide covers systemd operations, unit troubleshooting, and service lifecycle tasks.

## 6.1 Basic service lifecycle with systemd

### Start service

```bash
sudo systemctl start nginx
```

### Stop service

```bash
sudo systemctl stop nginx
```

### Restart service

```bash
sudo systemctl restart nginx
```

### Reload configuration

```bash
sudo systemctl reload nginx
```

### Enable at boot

```bash
sudo systemctl enable nginx
```

### Disable at boot

```bash
sudo systemctl disable nginx
```

### Check status

```bash
systemctl status nginx --no-pager
systemctl is-active nginx
systemctl is-enabled nginx
```

## 6.2 Reading service logs

```bash
journalctl -u nginx --since '30 min ago' --no-pager
journalctl -u nginx -f
```

## 6.3 List failed units

```bash
systemctl --failed
```

## 6.4 View dependency chain

```bash
systemctl list-dependencies nginx
systemctl show nginx -p After -p Wants -p Requires
```

## 6.5 Reload systemd after unit change

```bash
sudo systemctl daemon-reload
```

## 6.6 Creating a custom systemd service

### Example unit file

```ini
[Unit]
Description=My Python App
After=network.target

[Service]
Type=simple
User=appuser
Group=appuser
WorkingDirectory=/opt/app
ExecStart=/usr/bin/python3 /opt/app/app.py
Restart=on-failure
RestartSec=5
Environment=APP_ENV=production

[Install]
WantedBy=multi-user.target
```

### Install and start it

```bash
sudo tee /etc/systemd/system/myapp.service >/dev/null <<'EOF'
[Unit]
Description=My Python App
After=network.target

[Service]
Type=simple
User=appuser
Group=appuser
WorkingDirectory=/opt/app
ExecStart=/usr/bin/python3 /opt/app/app.py
Restart=on-failure
RestartSec=5
Environment=APP_ENV=production

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable --now myapp
systemctl status myapp --no-pager
```

## 6.7 Override unit settings safely

```bash
sudo systemctl edit nginx
```

### Example override content

```ini
[Service]
LimitNOFILE=65535
Environment=ENVIRONMENT=production
```

### Show merged unit

```bash
systemctl cat nginx
```

## 6.8 Troubleshooting services

- Check unit file syntax.
- Check environment variables.
- Check file permissions.
- Check working directory.
- Check dependency readiness.
- Check port conflicts.
- Check SELinux or AppArmor denials.
- Check resource limits.

### Service debugging commands

```bash
sudo systemd-analyze verify /etc/systemd/system/myapp.service
```

```bash
ss -tulpn | grep ':80 '
```

```bash
sudo journalctl -u myapp -xe --no-pager
```

### Restart loop detection

```bash
systemctl show myapp -p NRestarts -p ExecMainStatus -p ExecMainCode
```

## 6.9 Service management one-liners

```bash
systemctl list-units --type=service --state=running | egrep 'nginx|httpd|sshd|docker'
```

```bash
for s in nginx sshd docker; do printf '%s: ' "$s"; systemctl is-active "$s"; done
```

---
