# Open URL Listener for macOS

## Description
A lightweight background listener for macOS that opens URLs sent from another Mac. Runs automatically at login and survives reboots.

## Setup on Second Mac

Paste this in Terminal:

```bash
PYTHON_PATH=$(which python3) && \
mkdir -p ~/open_url_listener && cd ~/open_url_listener && \
cat << 'EOF' > open_url_listener.py
#!/usr/bin/env python3
import http.server
import socketserver
import os

PORT = 8765

class Handler(http.server.BaseHTTPRequestHandler):
    def do_POST(self):
        length = int(self.headers['Content-Length'])
        url = self.rfile.read(length).decode()
        os.system(f"open '{url}'")
        self.send_response(200)
        self.end_headers()
        self.wfile.write(b"URL opened")
    def log_message(self, format, *args):
        return

with socketserver.TCPServer(("0.0.0.0", PORT), Handler) as httpd:
    httpd.serve_forever()
EOF

chmod +x open_url_listener.py && \
cat << EOF > ~/Library/LaunchAgents/com.openurl.listener.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.openurl.listener</string>
    <key>ProgramArguments</key>
    <array>
        <string>$PYTHON_PATH</string>
        <string>/Users/$USER/open_url_listener/open_url_listener.py</string>
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
echo "Listener installed on port 8765 and running in background"
```

## Sending URLs from First Mac

On the second Mac, run `hostname` to get its name. Then from the first Mac:

```bash
curl -X POST http://<HOSTNAME>.local:8765 -d 'https://example.com'
```

Replace `<HOSTNAME>` with the result from the `hostname` command.

## Cleanup

```bash
launchctl unload ~/Library/LaunchAgents/com.openurl.listener.plist
rm ~/Library/LaunchAgents/com.openurl.listener.plist
rm -rf ~/open_url_listener
rm /tmp/openurl.log /tmp/openurl.err
```
