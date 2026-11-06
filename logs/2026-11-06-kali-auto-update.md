# Auto-update Kali on boot

Run `apt update` and `full-upgrade` automatically once per boot using a systemd oneshot service. It runs as root, so there is no sudo password prompt, and it does not slow down opening terminals.

Putting the commands in `~/.zshrc` is not recommended. It would run every time a terminal opens and keep prompting for a password.

## 1. Create the script

```bash
sudo nano /usr/local/bin/kali-refresh.sh
```

```bash
#!/usr/bin/env bash
set -euo pipefail
export DEBIAN_FRONTEND=noninteractive

apt-get update
apt-get -y full-upgrade
apt-get -y autoremove
```

```bash
sudo chmod +x /usr/local/bin/kali-refresh.sh
```

`full-upgrade` is used instead of `upgrade` because Kali is a rolling release and this is the recommended command.

## 2. Create the service

```bash
sudo nano /etc/systemd/system/kali-refresh.service
```

```ini
[Unit]
Description=Kali update and upgrade
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/kali-refresh.sh

[Install]
WantedBy=multi-user.target
```

## 3. Enable it

```bash
sudo systemctl daemon-reload
sudo systemctl enable kali-refresh.service
```

## Test and check logs

```bash
sudo systemctl start kali-refresh.service
journalctl -u kali-refresh.service
```

## Notes

- It runs at boot, not at terminal launch. A session left up for days will not update again until the next restart. For long-running machines, use a systemd timer (daily or weekly) instead.
- Unattended upgrades can occasionally break something mid-lab. If that is a concern, keep only `apt-get update` in the script and run the upgrade manually.
- On a VM that gets reverted to a snapshot, the full update will repeat every boot. A weekly timer is a better fit there.