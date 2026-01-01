# Messages Kill Script and Password-Protection

## Step 1: Create Auto-Quit Guard App

1. Cmd+Space → "Script Editor"

2. Delete all → Paste exactly:
```text
repeat
    try
        tell application "System Events"
            if exists process "Messages" then
                tell application "Messages" to quit
            end if
        end tell
        delay 1
    end try
end repeat
```

3. Cmd+E → File Format: Application → Name: MessagesQuit.app

4. Cmd+Shift+G → ~/Library/LaunchAgents/ → Save

## Step 2: Deploy Guard as Launch Agent
1. Terminal (Cmd+Space → "Terminal")

2. Copy your username. You can use `whoami` in the terminal.
```bash
whoami
```

3. Create the plist file:

```bash
nano ~/Library/LaunchAgents/com.user.messagesquit.plist
```

4. Paste (replace john with your username):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.user.messagesquit</string>
    <key>ProgramArguments</key>
    <array>
        <string>/Users/john/Library/LaunchAgents/MessagesQuit.app/Contents/MacOS/MessagesQuit</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
</dict>
</plist>
```

5. Ctrl+X → Y → Enter

6. Load:
```bash
launchctl load ~/Library/LaunchAgents/com.user.messagesquit.plist
```

## Step 3: Test Guard

1. Cmd+Space → "Messages" → Quits instantly ✓

## Step 4: Create Password Launcher Shortcut

1. Cmd+Space → "Shortcuts" → +

2. Name: "Messages"

3. Add actions:

    - Ask for Input → Text → "Enter password:" → Hidden Input ✓

    - If → equals → `your_password`

        - Run Shell Script:  
        `launchctl unload ~/Library/LaunchAgents/com.user.messagesquit.plist`

        - Open App → Messages  

        - Wait → 3 seconds  

        - Run Shell Script:  
        `launchctl load ~/Library/LaunchAgents/com.user.messagesquit.plist`

    - Otherwise → Show Alert "Wrong password!", etc.

## Step 5: Messages Icon & Deploy

1. Shortcuts → Messages shortcut → i → Edit Icon

2. Right-click `/Applications/Messages.app` → Show Package Contents → Resources → `AppIcon.icns` → copy

3. Paste into Shortcuts icon editor

4. `i` → Add to Dock

## Final Setup

1. Hide original: mv /Applications/Messages.app ~/HiddenMessages.app

2. Use Dock "Messages" shortcut only

3. Test: Dock Messages → password → opens. Spotlight Messages → quits.

## IF FAILS

Run:
```Bash
# Unload first
launchctl unload ~/Library/LaunchAgents/com.user.messagesquit.plist 2>/dev/null

# Create plist with correct "applet" executable
cat > ~/Library/LaunchAgents/com.user.messagesquit.plist << EOF
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.user.messagesquit</string>
    <key>ProgramArguments</key>
    <array>
        <string>/Users/alexander-mckelvey/Library/LaunchAgents/MessagesQuit.app/Contents/MacOS/applet</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
</dict>
</plist>
EOF

# Test executable directly
~/Library/LaunchAgents/MessagesQuit.app/Contents/MacOS/applet &

# Verify & load
plutil ~/Library/LaunchAgents/com.user.messagesquit.plist
launchctl load ~/Library/LaunchAgents/com.user.messagesquit.plist

```
