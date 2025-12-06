# 🛡️ MikroTik Parental Control via Telegram Bot

A collection of MikroTik RouterOS scripts for remote network access management, designed for parental control and network device management through Telegram bot integration.

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Setup Instructions](#setup-instructions)
  - [1. Telegram Bot Setup](#1-telegram-bot-setup)
  - [2. MikroTik Router Configuration](#2-mikrotik-router-configuration)
  - [3. Script Installation](#3-script-installation)
- [Available Scripts](#available-scripts)
- [Usage](#usage)
- [Troubleshooting](#troubleshooting)
- [License](#license)

## 🎯 Overview

This project provides a Telegram-controlled interface for managing network access on a MikroTik router. It's specifically designed for parents who want to remotely control their child's internet access, reboot network devices, and manage WiFi access points - all through simple Telegram messages.

## ✨ Features

- **🔐 Remote Parental Control**
  - Enable/disable internet access completely for specific devices
  - Whitelist mode: allow access ONLY to educational sites (e.g., Stepik) via firewall rules
  - Real-time status notifications via Telegram

- **⚡ Device Management**
  - Reboot USB devices (modems, storage, etc.)
  - Enable/disable PoE-powered access points
  - Remote power control for network equipment

- **📱 Telegram Integration**
  - Secure command execution through Telegram bot
  - Automatic status notifications with timestamps
  - Simple text commands for all operations

## 📦 Prerequisites

Before you begin, ensure you have:

- **MikroTik Router** with RouterOS (tested on RouterOS 6.x)
  - Support for scripting
  - Internet connectivity
  - PoE support (if using PoE scripts)
  - USB port (if using USB reboot scripts)

- **Telegram Account**
  - Access to create a bot via [@BotFather](https://t.me/botfather)
  - Your personal Chat ID

- **Network Setup**
  - MikroTik Kid Control configured with device profiles
  - Firewall rules (if using selective access features)

## 🚀 Setup Instructions

### 1. Telegram Bot Setup

#### Step 1.1: Create Your Telegram Bot

1. Open Telegram and search for [@BotFather](https://t.me/botfather)
2. Start a chat and send `/newbot`
3. Follow the prompts:
   - Choose a name for your bot (e.g., "My Router Control Bot")
   - Choose a username for your bot (must end with 'bot', e.g., "my_router_control_bot")
4. BotFather will provide you with a **bot token** (format: `123456789:ABCdefGHIjklMNOpqrsTUVwxyz`)
5. **Save this token** - you'll need it for configuration

#### Step 1.2: Get Your Chat ID

1. **Method 1: Using a Bot**
   - Search for [@userinfobot](https://t.me/userinfobot) in Telegram
   - Start a chat with it
   - It will display your Chat ID (a number like `123456789`)

2. **Method 2: Using Web Browser**
   - Send any message to your newly created bot
   - Open in browser: `https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates`
   - Replace `<YOUR_BOT_TOKEN>` with your actual bot token
   - Look for `"chat":{"id":123456789}` in the response
   - The number is your Chat ID

3. **Save your Chat ID** - you'll need it for configuration

#### Step 1.3: Prepare Bot Credentials

You now have two important values:
- **Bot Token**: `123456789:ABCdefGHIjklMNOpqrsTUVwxyz`
- **Chat ID**: `123456789`

You'll configure these in the format: `botID:bot_token` for scripts (e.g., `123456789:ABCdefGHIjklMNOpqrsTUVwxyz`)

### 2. MikroTik Router Configuration

#### Step 2.1: Access Router Terminal

1. Connect to your MikroTik router:
   - **Via Winbox**: Click "New Terminal"
   - **Via SSH**: `ssh admin@<router_ip>`
   - **Via Web Interface**: Go to Terminal tab

#### Step 2.2: Configure Kid Control (Optional)

If you're using parental control features, set up Kid Control:

```routeros
# Create a Kid Control profile
/ip kid-control add name=Yana disabled=no

# Add devices to the profile (example with MAC addresses)
/ip kid-control device add name="Child-Phone" mac-address=AA:BB:CC:DD:EE:FF parent=Yana
/ip kid-control device add name="Child-Laptop" mac-address=11:22:33:44:55:66 parent=Yana

# Set time restrictions (optional)
/ip kid-control set Yana mon="0s-1d" tue="0s-1d" wed="0s-1d" thu="0s-1d" fri="0s-1d" sat="0s-1d" sun="0s-1d"
```

Replace:
- `Yana` with your child's name
- MAC addresses with actual device MAC addresses

#### Step 2.3: Configure Firewall Rule (For Selective Access)

If you want to enable "whitelist only Stepik" feature, create a firewall rule:

```routeros
# Create address list for allowed sites
/ip firewall address-list add list=allowed address=stepik.org

# Create firewall filter rule (rule #1)
/ip firewall filter add chain=forward src-address-list=kid-devices dst-address-list=!allowed action=drop comment="Block all except whitelisted" disabled=yes place-before=0
```

**Note:** Adjust the rule according to your specific needs and firewall configuration.

### 3. Script Installation

#### Step 3.1: Upload Scripts to Router

For each script in this repository, follow these steps:

1. **Access Router Scripts**:
   - In Winbox: Go to `System → Scripts`
   - In Web Interface: Go to `System → Scripts`

2. **Create New Script**:
   - Click "+" to add a new script
   - Name it **exactly** as shown in the script filename (e.g., `Kid Control Off (Internet Access Turn On)`)
   - Set Policy: `read`, `write`, `policy`, `test`

3. **Configure Script**:
   - Copy the script content
   - Replace placeholders:
     - `bot_ID:bot_token` → Your actual `123456789:ABCdefGHIjklMNOpqrsTUVwxyz`
     - `chat_ID` → Your actual Chat ID (e.g., `123456789`)

4. **Save the Script**

#### Step 3.2: Configure the Telegram Watcher Script

The **Telegram Watcher** script is the main bot listener. It requires special setup:

1. Create a new script named `Telegram Watcher`
2. Copy the content from the `Telegram Watcher` file
3. Replace placeholders:
   - Line 3: `bot_ID:bot_token` → Your bot credentials
   - Line 4: `chat_ID` → Your Chat ID
4. **Policies Required**: `read`, `write`, `policy`, `test`

#### Step 3.3: Auto-Start Telegram Watcher

The Telegram Watcher needs to run continuously. You have two options:

**Option 1: Run on Startup (Recommended)**

Make the Telegram Watcher start automatically when the router boots:

```routeros
# Create a startup script
/system script add name=StartTelegramWatcher source="/system script run \"Telegram Watcher\"" policy=read,write,policy,test

# Add to scheduler to run on startup
/system scheduler add name=TelegramWatcherAutoStart on-event=StartTelegramWatcher start-time=startup
```

**Option 2: Run Every Minute with Auto-Restart (Most Reliable)**

This ensures the Telegram Watcher is always running, even if it crashes:

```routeros
# Add to scheduler to run every minute
# The script itself checks if it's already running and won't create duplicates
/system scheduler add name=TelegramWatcherMonitor interval=1m on-event="/system script run \"Telegram Watcher\"" policy=read,write,policy,test
```

**Manual start:**

```routeros
/system script run "Telegram Watcher"
```

**Important:** The Telegram Watcher runs as an infinite loop and keeps checking for messages. You'll see it running in the background. The every-minute scheduler ensures it restarts automatically if it ever stops.

#### Step 3.4: Verify Installation

Test that everything works:

1. Run the test script manually:
   ```routeros
   /system script run "Test Message to Telegram"
   ```

2. You should receive a test message in your Telegram chat
3. If you receive the message, the setup is successful! ✅

## 📜 Available Scripts

### 1. **Telegram Watcher**
- **Purpose**: Main bot listener that processes incoming Telegram commands
- **Function**: Continuously polls Telegram API for new messages and executes corresponding scripts
- **Auto-run**: Should be started automatically or run on router startup
- **Usage**: Runs in the background, no manual execution needed

### 2. **Kid Control Off (Internet Access Turn On)**
- **Purpose**: Enable full internet access for the child's devices
- **Command**: Send this script name via Telegram to your bot
- **Action**: Disables Kid Control, disables firewall filter rule #1
- **Result**: Full internet access (blacklist filtering still active if configured)
- **Use Case**: Normal internet usage with parental blacklist protection
- **Notification**: Confirms "Full Access to Inet for Yana Enabled" with timestamp

### 3. **Kid Control On (Internet Access Turn Off)**
- **Purpose**: Completely block internet access for the child's devices
- **Command**: Send this script name via Telegram to your bot
- **Action**: Enables Kid Control (blocks all internet), disables firewall filter rule #1
- **Result**: No internet access - all websites and services are blocked
- **Use Case**: Bedtime, punishment, or forced offline time
- **Notification**: Confirms "Inet for Yana Disabled" with timestamp

### 4. **Enable Access to only Stepik on Yana's Devices**
- **Purpose**: Whitelist mode - allow access ONLY to Stepik.org for studying
- **Command**: Send this script name via Telegram to your bot
- **Action**: Disables Kid Control (enables internet), enables firewall filter rule #1 (whitelist)
- **Result**: Only Stepik.org is accessible, all other sites and services are blocked
- **Use Case**: Focused study time - child can access only Stepik.org for homework/learning
- **How it works**: Firewall rule blocks everything except whitelisted domains (Stepik.org)
- **Notification**: Confirms "Access to Stepik Only for Yana Enabled" with timestamp

### 5. **USB-port reboot**
- **Purpose**: Reboot USB device connected to the router
- **Command**: Send this script name via Telegram to your bot
- **Action**: Power cycles the USB port (10 seconds off)
- **Use Case**: Restart USB modem, storage, or other USB devices
- **Notification**: Confirms action with timestamp

### 6. **PoE Enable**
- **Purpose**: Turn on PoE-powered device (e.g., WiFi Access Point)
- **Command**: Send this script name via Telegram to your bot
- **Action**: Enables PoE output on ether5
- **Use Case**: Enable WiFi access point
- **Notification**: Confirms WiFi enabled with timestamp

### 7. **PoE Disable**
- **Purpose**: Turn off PoE-powered device (e.g., WiFi Access Point)
- **Command**: Send this script name via Telegram to your bot
- **Action**: Disables PoE output on ether5
- **Use Case**: Disable WiFi access point
- **Notification**: Confirms WiFi disabled with timestamp

### 8. **Test Message to Telegram**
- **Purpose**: Test Telegram notifications
- **Command**: Send this script name via Telegram to your bot
- **Action**: Sends a test message back to you
- **Use Case**: Verify bot configuration and connectivity

## 📱 Usage

### Basic Operation

1. **Open Telegram** and start a chat with your bot
2. **Send a command** by typing the exact script name:
   ```
   Kid Control Off (Internet Access Turn On)
   ```
3. **Receive confirmation** with router name and timestamp:
   ```
   Dec/06/2021 17:46:00 MyRouter Full Access to Inet for Yana Enabled
   ```

### Example Commands

| Command | Action |
|---------|--------|
| `Kid Control On (Internet Access Turn Off)` | Block internet completely - no access at all |
| `Kid Control Off (Internet Access Turn On)` | Enable full internet access (with blacklist filtering) |
| `Enable Access to only Stepik on Yana's Devices` | Whitelist mode: allow ONLY Stepik.org, block everything else |
| `USB-port reboot` | Restart USB device |
| `PoE Enable` | Turn on WiFi access point |
| `PoE Disable` | Turn off WiFi access point |
| `Test Message to Telegram` | Test bot connectivity |

### Security Notes

⚠️ **Important Security Considerations:**

1. **Chat ID Verification**: The Telegram Watcher only accepts commands from your specific Chat ID, preventing unauthorized access
2. **Bot Token**: Keep your bot token secret - anyone with this token can send messages as your bot
3. **Script Names**: Commands must match script names exactly (case-sensitive)
4. **Unknown Commands**: If you send an unknown command, the bot will respond with "Unknown command"

## 🔧 Troubleshooting

### Bot Not Responding

**Problem**: Sending commands to the bot, but nothing happens

**Solutions**:
1. Check if Telegram Watcher is running:
   ```routeros
   /system script job print
   ```
   You should see "Telegram Watcher" in the list

2. Restart Telegram Watcher:
   ```routeros
   /system script job remove [find script="Telegram Watcher"]
   /system script run "Telegram Watcher"
   ```

3. Verify internet connectivity:
   ```routeros
   /ping 8.8.8.8
   ```

4. Check bot credentials:
   - Open each script and verify botID and chatID are correct
   - Test with: `/system script run "Test Message to Telegram"`

### Wrong Chat ID

**Problem**: Bot responds with "Unknown command" to all messages

**Solution**: Your Chat ID might be incorrect. Verify it using [@userinfobot](https://t.me/userinfobot) and update all scripts.

### Script Errors

**Problem**: Script execution fails

**Solutions**:
1. Check logs:
   ```routeros
   /log print where topics~"error"
   ```

2. Verify script policies:
   - Ensure all scripts have: `read`, `write`, `policy`, `test`

3. Check script syntax:
   - Copy and paste scripts carefully
   - Ensure no extra spaces or formatting issues

### PoE Not Working

**Problem**: PoE Enable/Disable doesn't work

**Solutions**:
1. Verify your router supports PoE
2. Check the correct ethernet port:
   ```routeros
   /interface ethernet print
   ```
3. Update script with correct port name (default is `ether5`)

### Firewall Rule Issues

**Problem**: "Enable Access to only Stepik" doesn't work correctly

**Solutions**:
1. Verify firewall rule exists:
   ```routeros
   /ip firewall filter print
   ```
2. Check rule number (default is `1`)
3. Adjust the script if your rule number is different

## 🛠️ Customization

### Changing Child Name

To adapt for a different child name (instead of "Yana"):

1. Update Kid Control profile name:
   ```routeros
   /ip kid-control add name=YourChildName
   ```

2. Update all scripts - replace `Yana` with your child's name

### Changing PoE Port

If your access point is on a different port:

1. Find your ethernet ports:
   ```routeros
   /interface ethernet poe print
   ```

2. Update PoE scripts - replace `ether5` with your port name

### Adding New Commands

You can create custom scripts:

1. Create a new script in `System → Scripts`
2. Name it with a descriptive name
3. Add your bot credentials and desired actions
4. Send the script name via Telegram to execute

## 📄 License

This project is open source and available for personal use. Feel free to modify and adapt to your needs.

## 🤝 Contributing

Contributions, issues, and feature requests are welcome!

## ⚠️ Disclaimer

- Use these scripts at your own risk
- Always test in a safe environment before production use
- Keep backups of your router configuration
- Ensure you understand MikroTik scripting before deployment

---

**Made with ❤️ for responsible parenting and network management**
