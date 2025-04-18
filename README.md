# 🖥️ Headless Mac Setup (2025 Edition)

Turn any Mac into a 24x7 **headless machine** you can control remotely — with the lid closed and no display connected.

This guide walks you through making your Mac accessible via SSH, Screen Sharing, and keeps it awake using `caffeinate`, even when the lid is shut.

---

## 📋 Prerequisites

- A Mac with macOS Ventura or later
- Access to another Mac to control it remotely
- Local Wi-Fi or Ethernet network

---

## 🚫 Step 1: Prevent Mac From Sleeping (Lid Closed)

macOS sleeps when the lid closes. We'll override that with a background process.

### ✅ Use `caffeinate` (built-in, no install required)

```bash
nohup caffeinate -dimsu > /dev/null 2>&1 &
```

But this is temporary — it dies after logout/reboot.

---

## 🔁 Step 2: Auto-Start `caffeinate` on Boot (LaunchDaemon)

We'll create a system-wide LaunchDaemon so your Mac stays awake even without logging into the GUI or using VNC.

### Option A: Use the included `.plist` file

Download or copy `com.headless.caffeinate.plist` into `/Library/LaunchDaemons/` with the following:

```bash
sudo cp com.headless.caffeinate.plist /Library/LaunchDaemons/
sudo chown root:wheel /Library/LaunchDaemons/com.headless.caffeinate.plist
sudo chmod 644 /Library/LaunchDaemons/com.headless.caffeinate.plist
sudo launchctl load /Library/LaunchDaemons/com.headless.caffeinate.plist
```

### Option B: Create the file manually (if not using the repo copy)

```bash
sudo nano /Library/LaunchDaemons/com.headless.caffeinate.plist
```

Paste this content:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
 "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.headless.caffeinate</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/bin/caffeinate</string>
        <string>-dimsu</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
</dict>
</plist>
```

Then load it:

```bash
sudo chown root:wheel /Library/LaunchDaemons/com.headless.caffeinate.plist
sudo chmod 644 /Library/LaunchDaemons/com.headless.caffeinate.plist
sudo launchctl load /Library/LaunchDaemons/com.headless.caffeinate.plist
```

---

## 🌐 Step 3: Enable Remote Access

### A. Screen Sharing

- Go to `System Settings > General > Sharing`
- Enable **Screen Sharing**
- Connect from another Mac:
  ```
  vnc://your-mac-hostname.local
  ```
  or
  ```
  vnc://your-mac-ip
  ```

### B. Remote Login (SSH)

- Also in Sharing, enable **Remote Login**
- Connect from another device:
  ```bash
  ssh your-username@your-mac-hostname.local
  ```
  or
  ```bash
  ssh your-username@your-mac-ip
  ```

---

## 🗂️ Step 4: Enable File Sharing (Optional)

- Go to **System Settings > Sharing > File Sharing**
- Add folders you want accessible (e.g., `Downloads`, `Documents`)
- Access it from another Mac:
  ```
  smb://your-mac-hostname.local
  ```
  or
  ```
  smb://your-mac-ip
  ```

---

## ✅ Done! Your Mac is Now Headless & Remote-Ready

- ✅ Stays awake 24x7 with lid closed
- ✅ Accessible via Screen Sharing and SSH
- ✅ Ready for file transfers

---

## 🧠 Bonus (Advanced)

### 🔑 Set up SSH key login (from controller Mac to headless Mac)

To enable passwordless SSH login from your **controller Mac** to your **headless Mac**, follow these steps:

#### 1. Generate a new SSH key (on controller Mac)

```bash
ssh-keygen -t rsa -b 4096 -C "headless-mac" -f ~/.ssh/headless-mac
```

This creates:
- `~/.ssh/headless-mac` (private key)
- `~/.ssh/headless-mac.pub` (public key)

#### 2. Copy the key to the headless Mac

```bash
ssh-copy-id -i ~/.ssh/headless-mac.pub your-username@your-headless-mac-ip
```

Or with hostname:

```bash
ssh-copy-id -i ~/.ssh/headless-mac.pub your-username@your-headless-mac-hostname.local
```

> 💡 Replace `your-username` with your user account **on the headless Mac**.

#### 3. Test SSH login

```bash
ssh -i ~/.ssh/headless-mac your-username@your-headless-mac-ip
```

✅ You should be able to log in without entering a password.

#### 4. (Optional) Add to SSH config for convenience

```bash
nano ~/.ssh/config
```

Add this block:

```ssh
Host headless-mac
    HostName your-headless-mac-ip
    User your-username
    IdentityFile ~/.ssh/headless-mac
```

Now you can simply connect with:

```bash
ssh headless-mac
```

#### 5. (Optional) Store passphrase in macOS Keychain

If you entered a passphrase while generating the key and want macOS to remember it:

```bash
ssh-add --apple-use-keychain ~/.ssh/headless-mac
ssh-add -K ~/.ssh/headless-mac
```

## 🔗 License

MIT License — do whatever you want. Pull requests welcome!

