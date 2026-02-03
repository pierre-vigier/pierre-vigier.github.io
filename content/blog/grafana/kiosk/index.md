---
title: "[Grafana] Dual screen dashboard setup, on Raspberry Pi"
date: 2026-02-01T12:00:00+08:00
summary: Display of multi org dashboard on dual screen setup
draft: false
tags: [ "grafana", "kiosk", "raspberry" ]
---
Displaying Grafana dashboards, using a Raspberry Pi as a kiosk device sounds like it should be straightforward enough, however, this proved to be much more challenging than expected.

I had to resinstall a Raspberry Pi to use as a kiosk device. The Raspberry Pi was a Pi 4, connected to two TV via HDMI. The goal was to display dashboards to monitor different services. While this seems possible, here is a list of challenges I faced, where any amount of googling and AI assistance did not work.

The first setup, before reinstallagion was using an old version of Raspberry Pi OS, with Chromium browser in kiosk mode. Issue on the SD card made me reinstall the OS, and I went for the latest Raspberry Pi OS, which is based on Debian Bookworm.

I wanted to try something as light as possible, and tries to use Cog, a lightweight browser for kiosk usage, wihtout a full desktop environment. After a lot of trial and error, I did not manage to make it work with multiple screens, it seems that the packaged version of Cog, with wayland on Raspberry Pi OS is not a good mix.

After too much wasted time, I decided to go back to use a full desktop environment, this simplified a lot of things, using LXDE as desktop environment. 
To stay light, i tries several browsers, including Midori, Falkon, and while the setup was working, I face issue with authentication.

The Raspberry is behind a TV, without easy access, and not mouse/keyboard connected, I wanted a setup that i could control purely though SSH, a setup with chromium browser in kiosk mode, the issue with this setup being that access to keyboard input is needed to login to Grafana.

After more research, i stumbled upon [grafana-kiosk](https://github.com/grafana/grafana-kiosk), a utility to launch grafana in kiosk mode, using chromium browser. This utility also handles authentication, by storing the credentials in a config file.

However, a specific challenge with my setup, is that I wanted to display dashboards from multiple organizations, and with 2 instances of grafana-kiosk using login-method local, the second instance would override the first one, and both instances would end up logged in the same organization, so one window was not working.

Another option of grafana-kiosk is to use API key for authentication, which is per organization, so using one instance per organization, with API key authentication would work, but the documentation was not very clear on how to pass the API key to grafana-kiosk.

Final setup:
- in Grafana, i created one service account with viewer role per organization, and created a token for each service account.
- on the Raspberry Pi, I created a config file for each grafana-kiosk, and a startup script to launch both instances.
- i configured auto login to `pi` user on LXDE, and added the startup script to autostart.

kiosk.sh:

```bash
#!/bin/bash

# Define log file locations
LOG1="/home/pi/kiosk1.log"
LOG2="/home/pi/kiosk2.log"

# Pre-cleanup: clear old logs and kill existing sessions
rm -f "$LOG1" "$LOG2"
killall grafana-kiosk 2>/dev/null
export DISPLAY=:0
sleep 5

killall grafana-kiosk

echo "$(date): Starting Kiosk 1 on Monitor 1..."
grafana-kiosk -c kiosk1.yaml > "$LOG1" 2>&1 &
PID1=$!

sleep 60

if ps -p $PID1 > /dev/null; then
    echo "Kiosk 1 is running (PID: $PID1)."
else
    echo "ERROR: Kiosk 1 failed. Check $LOG1"
fi

echo "$(date): Starting Kiosk 2 on Monitor 2..."
grafana-kiosk -c kiosk2.yaml > "$LOG2" 2>&1 2>&1 &

PID2=$!

sleep 15


if ps -p $PID1 > /dev/null; then
    echo "Kiosk 2 is running (PID: $PID2)."
else
    echo "ERROR: Kiosk 2 failed. Check $LOG2"
fi
```

kiosk1.yaml :

```yaml
general:
  kiosk-mode: full
  autofit: true
  lxde: true
  lxde-home: /home/pi
  scale-factor: 1.0
apikey:
  apikey: API_KEY_FOR_ORG_1
target:
  login-method: apikey
  URL: https://grafana.mydomain/dashboard_1?orgId=1&from=now-6h&to=now&timezone=utc&refresh=5m
```

kiosk2.yaml :

```yaml
general:
  kiosk-mode: full
  autofit: true
  window-position: 1920,0
  lxde: true
  lxde-home: /home/pi
  scale-factor: 1.0
apikey:
  apikey: API_KEY_FOR_ORG_2
target:
  login-method: apikey
  URL: https://grafana.mydomain/dashboard_2?orgId=1&from=now-6h&to=now&timezone=utc&refresh=5m
  window-position: "1920,0"
```

.config/autostart/kiosk.desktop :

```ini
[Desktop Entry]
Type=Application
Name=Kiosk
Exec=/home/pi/kiosk.sh
X-GNOME-Autostart-enabled=true
```
