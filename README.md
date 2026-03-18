# Open URL Listener for macOS

## Description
A lightweight background listener for macOS that opens URLs sent from another Mac via [ntfy.sh](https://ntfy.sh). Works on any network (no direct connection needed). Runs automatically at login and survives reboots.

## Setup on Second Mac

Replace `YOUR_SECRET_TOPIC` below with a long random string (e.g. `myMacs_a8f3k2x9`). This acts as your private channel. Then paste in Terminal:

```bash
TOPIC=YOUR_SECRET_TOPIC && \
mkdir -p ~/open_url_listener && \
cat << SCRIPT > ~/open_url_listener/open_url_listener.sh
#!/bin/bash
while true; do
  curl -sN https://ntfy.sh/$TOPIC/raw | while read -r url; do
    [ -n "\$url" ] && open "\$url"
  done
  sleep 2
done
SCRIPT

chmod +x ~/open_url_listener/open_url_listener.sh && \
cat << EOF > ~/Library/LaunchAgents/com.openurl.listener.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.openurl.listener</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>$HOME/open_url_listener/open_url_listener.sh</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/tmp/openurl.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/openurl.err</string>
</dict>
</plist>
EOF

launchctl unload ~/Library/LaunchAgents/com.openurl.listener.plist 2>/dev/null
launchctl load ~/Library/LaunchAgents/com.openurl.listener.plist
echo "Listener installed and running in background"
```

## Sending URLs from First Mac

```bash
curl -d 'https://example.com' https://ntfy.sh/YOUR_SECRET_TOPIC
```

Use the same topic string as above.

## Cleanup

```bash
launchctl unload ~/Library/LaunchAgents/com.openurl.listener.plist
rm ~/Library/LaunchAgents/com.openurl.listener.plist
rm -rf ~/open_url_listener
rm /tmp/openurl.log /tmp/openurl.err
```
