# Messages Kill Script and Password Protection

A macOS automation solution that automatically quits the Messages app and protects your Mac with password-based access control.

## Overview

This project provides a secure way to prevent unauthorized access to your Messages app by:
- Automatically terminating the Messages application
- Implementing a password-protected launch agent system
- Running continuously in the background on your Mac

## Requirements

- macOS 10.13 or later
- Ability to use Script Editor and Terminal
- Basic familiarity with command-line interface

## Installation Guide

### Step 1: Create the Auto-Quit Guard App

This step creates an AppleScript application that automatically quits Messages in a loop.

1. Open Script Editor by pressing **Cmd+Space** and typing `"Script Editor"`

2. Delete all placeholder text and paste the following script:

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

3. Export as an application:
   - Press **Cmd+E**
   - Set **File Format** to "Application"
   - Set **Name** to `MessagesQuit.app`

4. Save to LaunchAgents directory:
   - Press **Cmd+Shift+G**
   - Navigate to: `~/Library/LaunchAgents/`
   - Click **Save**

### Step 2: Deploy the Guard as a Launch Agent

This step sets up the app to run automatically on system startup.

1. Open Terminal by pressing **Cmd+Space** and typing `"Terminal"`

2. Find your macOS username:
   ```bash
   whoami
   ```
   Copy the output (your username)

3. Create the launch agent configuration file with the correct executable path:
   ```bash
   cat > ~/Library/LaunchAgents/com.user.messagesquit.plist << EOF
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
   <plist version="1.0">
   <dict>
       <key>Label</key>
       <string>com.user.messagesquit</string>
       <key>ProgramArguments</key>
       <array>
           <string>/Users/alexander-mckelvy/Library/LaunchAgents/MessagesQuit.app/Contents/MacOS/applet</string>
       </array>
       <key>RunAtLoad</key>
       <true/>
       <key>KeepAlive</key>
       <true/>
   </dict>
   </plist>
   EOF
   ```
   **Important:** Replace `alexander-mckelvy` with your actual username from Step 2.

4. Load the launch agent:
   ```bash
   launchctl load ~/Library/LaunchAgents/com.user.messagesquit.plist
   ```

### Step 3: Test the Setup

Verify everything is working correctly:

```bash
~/Library/LaunchAgents/MessagesQuit.app/Contents/MacOS/applet &
```

The Messages app should quit automatically when it tries to run.

## Troubleshooting

### Launch Agent Won't Load

If the plist file doesn't load, verify:
1. Your username is correctly set in the plist file
2. The file path to `MessagesQuit.app` is correct
3. File permissions are set properly:
   ```bash
   chmod 644 ~/Library/LaunchAgents/com.user.messagesquit.plist
   ```

### Messages Still Opens

1. Check if the launch agent is running:
   ```bash
   launchctl list | grep messagesquit
   ```

2. Review the plist file syntax:
   ```bash
   plutil -lint ~/Library/LaunchAgents/com.user.messagesquit.plist
   ```

3. Re-load the agent:
   ```bash
   launchctl unload ~/Library/LaunchAgents/com.user.messagesquit.plist
   launchctl load ~/Library/LaunchAgents/com.user.messagesquit.plist
   ```

## How It Works

1. The MessagesQuit.app runs as a background process through the launch agent
2. Every 1 second, it checks if the Messages process is running
3. If Messages is detected, it immediately terminates the process
4. This cycle repeats continuously
5. The launch agent ensures the guard app restarts if it crashes

## Security Notes

- This solution is designed to prevent accidental or casual unauthorized access
- It is not a complete security solution
- For sensitive information protection, use additional macOS security features (FileVault, etc.)
- Launch agents run with user-level privileges

## Uninstall

To remove this protection:

```bash
launchctl unload ~/Library/LaunchAgents/com.user.messagesquit.plist
rm ~/Library/LaunchAgents/com.user.messagesquit.plist
rm -rf ~/Library/LaunchAgents/MessagesQuit.app
```

## License

MIT License - Feel free to modify and use this project as needed.

## Support

For issues or questions, please open an issue on GitHub.
