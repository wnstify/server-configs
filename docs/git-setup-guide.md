# GitHub SSH Key Setup Guide

This guide explains how to set up SSH key authentication for GitHub on Linux servers, macOS, and Windows (including VS Code integration). Using SSH keys provides secure, password-free authentication when pushing and pulling from GitHub repositories.

## Table of Contents

- [Why SSH Keys?](#why-ssh-keys)
- [Prerequisites](#prerequisites)
- [Linux/Server Setup](#linuxserver-setup)
- [macOS Setup](#macos-setup)
- [Windows Setup](#windows-setup)
- [VS Code Integration](#vs-code-integration)
- [Adding the Key to GitHub](#adding-the-key-to-github)
- [Testing the Connection](#testing-the-connection)
- [Troubleshooting](#troubleshooting)
- [Quick Reference](#quick-reference)

---

## Why SSH Keys?

SSH keys provide several advantages over password authentication:

- **Security**: Keys are cryptographically secure and much harder to compromise than passwords
- **Convenience**: No need to enter credentials for every git operation
- **Automation**: Enables automated scripts and CI/CD pipelines to interact with GitHub
- **Revocability**: Individual keys can be revoked without affecting other access methods

We use **Ed25519** keys because they are:
- More secure than RSA keys of similar length
- Faster to generate and verify
- Shorter and easier to manage
- The current recommended standard

---

## Prerequisites

- Terminal access on your machine
- A GitHub account
- Git installed on your system
- For Windows: OpenSSH client (included in Windows 10/11)

---

## Linux/Server Setup

### Step 1: Generate the SSH Key

Open your terminal and run the following command:

```bash
ssh-keygen -t ed25519 -C "YOUR_EMAIL" -f ~/.ssh/github_ed25519 -N ""
```

**Explanation of flags:**
| Flag | Description |
|------|-------------|
| `-t ed25519` | Specifies the key type (Ed25519 algorithm) |
| `-C "YOUR_EMAIL"` | Adds a comment/label to identify the key (use your GitHub email) |
| `-f ~/.ssh/github_ed25519` | Specifies the output file path and name |
| `-N ""` | Sets an empty passphrase (no password required) |

This creates two files:
- `~/.ssh/github_ed25519` - Your **private key** (never share this!)
- `~/.ssh/github_ed25519.pub` - Your **public key** (this gets added to GitHub)

### Step 2: Create/Update SSH Config

The SSH config file tells your system which key to use when connecting to GitHub:

```bash
cat >> ~/.ssh/config << 'EOF'
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_ed25519
    IdentitiesOnly yes
EOF
```

**Explanation of config options:**
| Option | Description |
|--------|-------------|
| `Host github.com` | The alias for this configuration block |
| `HostName github.com` | The actual server hostname |
| `User git` | The username for SSH connections (always `git` for GitHub) |
| `IdentityFile` | Path to the private key to use |
| `IdentitiesOnly yes` | Only use the specified key, don't try others |

### Step 3: Set Correct Permissions

SSH requires strict permissions on config files:

```bash
chmod 600 ~/.ssh/config
chmod 600 ~/.ssh/github_ed25519
chmod 644 ~/.ssh/github_ed25519.pub
```

**Permission meanings:**
- `600` - Owner can read/write, no access for others (required for private files)
- `644` - Owner can read/write, others can only read (acceptable for public key)

### Step 4: Display Your Public Key

```bash
cat ~/.ssh/github_ed25519.pub
```

Copy the entire output - you'll need it for GitHub.

---

## macOS Setup

macOS has additional features like Keychain integration that make key management easier.

### Step 1: Generate the SSH Key

```bash
ssh-keygen -t ed25519 -C "YOUR_EMAIL" -f ~/.ssh/github_ed25519 -N ""
```

Same as Linux - this creates your key pair.

### Step 2: Add Key to macOS Keychain

The Keychain securely stores your key and automatically loads it:

```bash
ssh-add --apple-use-keychain ~/.ssh/github_ed25519
```

**Why use Keychain?**
- Key persists across reboots
- Secure storage managed by macOS
- Automatic key loading when needed

### Step 3: Create/Update SSH Config

```bash
cat >> ~/.ssh/config << 'EOF'
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_ed25519
    IdentitiesOnly yes
    AddKeysToAgent yes
    UseKeychain yes
EOF
```

**Additional macOS-specific options:**
| Option | Description |
|--------|-------------|
| `AddKeysToAgent yes` | Automatically add key to ssh-agent |
| `UseKeychain yes` | Use macOS Keychain to store/retrieve the key |

### Step 4: Set Correct Permissions

```bash
chmod 600 ~/.ssh/config
chmod 600 ~/.ssh/github_ed25519
chmod 644 ~/.ssh/github_ed25519.pub
```

### Step 5: Copy Public Key to Clipboard

macOS has a built-in command to copy directly to clipboard:

```bash
pbcopy < ~/.ssh/github_ed25519.pub
```

Your public key is now in your clipboard, ready to paste into GitHub.

---

## Windows Setup

This section covers the complete Windows setup process, including using Git Bash and PowerShell.

### Option A: Using Git Bash (Recommended)

Git Bash provides a Unix-like terminal experience on Windows and is the easiest way to set up SSH keys.

#### Step 1: Install Git for Windows

If you haven't already, download and install Git for Windows:
- Download from: https://git-scm.com/download/win
- During installation, ensure "Git Bash" is selected
- Keep default options for SSH (use bundled OpenSSH)

#### Step 2: Open Git Bash

- Press `Win` key, type "Git Bash", and open it
- Or right-click in any folder and select "Git Bash Here"

#### Step 3: Create .ssh Directory

```bash
mkdir -p ~/.ssh
```

#### Step 4: Generate the SSH Key

```bash
ssh-keygen -t ed25519 -C "YOUR_EMAIL" -f ~/.ssh/github_ed25519 -N ""
```

This creates your key pair in `C:\Users\YOUR_USERNAME\.ssh\`

#### Step 5: Start SSH Agent and Add Key

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/github_ed25519
```

#### Step 6: Create/Update SSH Config

```bash
cat >> ~/.ssh/config << 'EOF'
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_ed25519
    IdentitiesOnly yes
EOF
```

#### Step 7: Set Permissions

```bash
chmod 600 ~/.ssh/config
chmod 600 ~/.ssh/github_ed25519
chmod 644 ~/.ssh/github_ed25519.pub
```

#### Step 8: Copy Public Key to Clipboard

```bash
cat ~/.ssh/github_ed25519.pub | clip
echo "Public key copied to clipboard!"
```

---

### Option B: Using PowerShell

PowerShell is the native Windows terminal and works well with the built-in OpenSSH client.

#### Step 1: Verify OpenSSH is Installed

Open PowerShell and check if SSH is available:

```powershell
ssh -V
```

If not installed, enable it:
1. Open **Settings** → **Apps** → **Optional Features**
2. Click **Add a feature**
3. Search for "OpenSSH Client" and install it

#### Step 2: Open PowerShell

Press `Win + X` and select **Terminal** or **Windows PowerShell**.

#### Step 3: Create .ssh Directory

```powershell
if (!(Test-Path $env:USERPROFILE\.ssh)) {
    New-Item -ItemType Directory -Path $env:USERPROFILE\.ssh -Force
}
```

#### Step 4: Generate the SSH Key

```powershell
ssh-keygen -t ed25519 -C "YOUR_EMAIL" -f $env:USERPROFILE\.ssh\github_ed25519 -N '""'
```

**Note:** The `-N '""'` syntax sets an empty passphrase in PowerShell.

This creates:
- `C:\Users\YOUR_USERNAME\.ssh\github_ed25519` - Private key
- `C:\Users\YOUR_USERNAME\.ssh\github_ed25519.pub` - Public key

#### Step 5: Configure and Start SSH Agent Service

The SSH Agent needs to be enabled and started. **Run PowerShell as Administrator** for this step:

```powershell
# Check the current status of ssh-agent
Get-Service ssh-agent

# Set the service to start automatically
Set-Service -Name ssh-agent -StartupType Automatic

# Start the service
Start-Service ssh-agent

# Verify it's running
Get-Service ssh-agent
```

**Expected output after configuration:**
```
Status   Name               DisplayName
------   ----               -----------
Running  ssh-agent          OpenSSH Authentication Agent
```

#### Step 6: Add Your Key to the Agent

Back in regular PowerShell (doesn't need Administrator):

```powershell
ssh-add $env:USERPROFILE\.ssh\github_ed25519
```

Verify the key was added:

```powershell
ssh-add -l
```

You should see your key fingerprint listed.

#### Step 7: Create/Update SSH Config

```powershell
$configContent = @"
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_ed25519
    IdentitiesOnly yes
"@

Add-Content -Path $env:USERPROFILE\.ssh\config -Value $configContent
```

Or create the file manually:

1. Open Notepad or any text editor
2. Add the following content:
   ```
   Host github.com
       HostName github.com
       User git
       IdentityFile ~/.ssh/github_ed25519
       IdentitiesOnly yes
   ```
3. Save as `C:\Users\YOUR_USERNAME\.ssh\config` (no extension)

**Important:** Make sure the file is saved without a `.txt` extension.

#### Step 8: Copy Public Key to Clipboard

```powershell
Get-Content $env:USERPROFILE\.ssh\github_ed25519.pub | Set-Clipboard
Write-Host "Public key copied to clipboard!"
```

Or display it to copy manually:

```powershell
Get-Content $env:USERPROFILE\.ssh\github_ed25519.pub
```

---

### Option C: Using Windows Terminal (Windows 11)

Windows Terminal provides a modern terminal experience and can run PowerShell, CMD, or Git Bash.

#### Step 1: Open Windows Terminal

Press `Win + X` and select **Terminal**.

#### Step 2: Follow PowerShell or Git Bash Steps

- Select the **PowerShell** tab and follow [Option B](#option-b-using-powershell)
- Or select **Git Bash** tab (if installed) and follow [Option A](#option-a-using-git-bash-recommended)

---

### Windows: Persisting SSH Agent Across Reboots

By default, the Windows SSH Agent service will persist across reboots if configured correctly. To verify:

```powershell
# Check startup type (should be "Automatic")
Get-Service ssh-agent | Select-Object Name, Status, StartType
```

If your key isn't loaded after a reboot, re-add it:

```powershell
ssh-add $env:USERPROFILE\.ssh\github_ed25519
```

---

## VS Code Integration

VS Code uses your system's Git and SSH configuration, so once SSH is set up on your system, VS Code will automatically use it.

### Verify Git is Using SSH

1. Open VS Code
2. Open the integrated terminal: `` Ctrl + ` `` (backtick)
3. Test the SSH connection:
   ```bash
   ssh -T git@github.com
   ```

### Configure VS Code to Use SSH for Repositories

#### For New Clones

When cloning a repository, always use the SSH URL:

1. On GitHub, click the **Code** button
2. Select **SSH** tab
3. Copy the URL (format: `git@github.com:username/repo.git`)
4. In VS Code: `Ctrl + Shift + P` → "Git: Clone" → Paste SSH URL

#### Convert Existing Repository from HTTPS to SSH

If you have a repository using HTTPS, convert it to SSH:

**In VS Code Terminal:**

```bash
# Check current remote URL
git remote -v

# Change from HTTPS to SSH
git remote set-url origin git@github.com:USERNAME/REPOSITORY.git
```

**Example:**
```bash
# From this (HTTPS):
# origin  https://github.com/myuser/myrepo.git

# To this (SSH):
git remote set-url origin git@github.com:myuser/myrepo.git
```

### VS Code Settings for Git/SSH

Open VS Code settings (`Ctrl + ,`) and configure:

#### 1. Git Path (if Git isn't detected)

**Windows:**
```json
{
    "git.path": "C:\\Program Files\\Git\\bin\\git.exe"
}
```

**macOS/Linux:**
```json
{
    "git.path": "/usr/bin/git"
}
```

#### 2. Integrated Terminal Shell (Windows)

To use Git Bash in VS Code terminal (recommended for Windows):

```json
{
    "terminal.integrated.defaultProfile.windows": "Git Bash"
}
```

#### 3. Auto Fetch

Enable auto-fetch to keep your repository in sync:

```json
{
    "git.autofetch": true,
    "git.autofetchPeriod": 180
}
```

### VS Code Extensions for Git

Recommended extensions for better Git integration:

| Extension | Description |
|-----------|-------------|
| **GitLens** | Enhanced Git visualization and history |
| **Git Graph** | Visual git log and branch viewer |
| **GitHub Pull Requests** | Manage PRs directly in VS Code |

Install via Extensions panel (`Ctrl + Shift + X`) or command line:

```bash
code --install-extension eamodio.gitlens
code --install-extension mhutchie.git-graph
code --install-extension GitHub.vscode-pull-request-github
```

### Using Source Control in VS Code

Once SSH is configured, you can use VS Code's built-in Git features:

1. **Source Control Panel**: `Ctrl + Shift + G`
2. **Stage Changes**: Click `+` next to files
3. **Commit**: Enter message and click checkmark (or `Ctrl + Enter`)
4. **Push/Pull**: Click the sync icon in status bar or use commands:
   - `Ctrl + Shift + P` → "Git: Push"
   - `Ctrl + Shift + P` → "Git: Pull"

---

## Adding the Key to GitHub

### Step 1: Navigate to SSH Settings

1. Log into your GitHub account
2. Click your profile picture in the top-right corner
3. Select **Settings**
4. In the left sidebar, click **SSH and GPG keys**
5. Click the **New SSH key** button

Or go directly to: `https://github.com/settings/ssh/new`

### Step 2: Add Your Key

1. **Title**: Enter a descriptive name to identify this key
   - Examples: `production-server`, `macbook-pro`, `windows-desktop`
   - Use the machine/server name for easy identification later

2. **Key type**: Keep as "Authentication Key"

3. **Key**: Paste your public key
   - Linux: Copy output from `cat ~/.ssh/github_ed25519.pub`
   - macOS: Should be in clipboard from `pbcopy`
   - Windows: Should be in clipboard from `Set-Clipboard` or `clip`

4. Click **Add SSH key**

5. Confirm with your GitHub password if prompted

---

## Testing the Connection

Verify everything is working correctly:

```bash
ssh -T git@github.com
```

**Expected success response:**
```
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

If you see this message, your SSH key is properly configured.

---

## Troubleshooting

### Permission Denied (publickey)

**Symptoms:**
```
Permission denied (publickey).
```

**Solutions:**

1. Verify your key is added to GitHub:
   ```bash
   cat ~/.ssh/github_ed25519.pub
   ```
   Compare with keys listed at `https://github.com/settings/keys`

2. Check SSH is using the correct key:
   ```bash
   ssh -vT git@github.com
   ```
   Look for lines mentioning your key file.

3. Verify file permissions (Linux/macOS):
   ```bash
   ls -la ~/.ssh/
   ```
   Private key should be `600`, config should be `600`.

4. Windows - Verify SSH agent is running:
   ```powershell
   Get-Service ssh-agent
   ssh-add -l
   ```

### Could Not Open Connection

**Symptoms:**
```
ssh: Could not resolve hostname github.com
```

**Solutions:**
- Check your internet connection
- Verify DNS is working: `ping github.com`
- Check if firewall is blocking port 22

### Key Not Being Used

**Symptoms:** SSH tries different keys or asks for password

**Solutions:**

1. Verify config file syntax:
   ```bash
   cat ~/.ssh/config
   ```

2. Ensure no conflicting entries for github.com

3. Test with verbose output:
   ```bash
   ssh -vvv git@github.com
   ```

### macOS: Key Not Persisting After Reboot

**Solutions:**

1. Re-add to Keychain:
   ```bash
   ssh-add --apple-use-keychain ~/.ssh/github_ed25519
   ```

2. Verify `UseKeychain yes` is in your SSH config

### Windows: SSH Agent Not Running

**Solution (run PowerShell as Administrator):**

```powershell
Set-Service -Name ssh-agent -StartupType Automatic
Start-Service ssh-agent
```

Then add your key (regular PowerShell):

```powershell
ssh-add $env:USERPROFILE\.ssh\github_ed25519
```

### Windows: "ssh-add" Command Not Found

**Solution:**

1. Ensure OpenSSH Client is installed:
   - **Settings** → **Apps** → **Optional Features**
   - Look for "OpenSSH Client"
   - If not present, click "Add a feature" and install it

2. Or use Git Bash instead of PowerShell

### Windows: Config File Has .txt Extension

**Symptoms:** SSH ignores your config file

**Solution:**

1. Open File Explorer
2. Go to `C:\Users\YOUR_USERNAME\.ssh\`
3. Enable "File name extensions" in View menu
4. Rename `config.txt` to `config` (remove the extension)

### VS Code Not Using SSH

**Symptoms:** VS Code prompts for username/password

**Solutions:**

1. Check remote URL is SSH (not HTTPS):
   ```bash
   git remote -v
   ```

2. Convert to SSH if needed:
   ```bash
   git remote set-url origin git@github.com:USERNAME/REPO.git
   ```

3. Restart VS Code after SSH configuration changes

4. On Windows, ensure SSH agent is running and key is loaded:
   ```powershell
   ssh-add -l
   ```

---

## Quick Reference

### Linux One-Liner

```bash
ssh-keygen -t ed25519 -C "YOUR_EMAIL" -f ~/.ssh/github_ed25519 -N "" -q && \
cat >> ~/.ssh/config << 'EOF'
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_ed25519
    IdentitiesOnly yes
EOF
chmod 600 ~/.ssh/config && \
echo "" && \
echo "=== Add this key to GitHub ===" && \
cat ~/.ssh/github_ed25519.pub
```

### macOS One-Liner

```bash
ssh-keygen -t ed25519 -C "YOUR_EMAIL" -f ~/.ssh/github_ed25519 -N "" -q && \
ssh-add --apple-use-keychain ~/.ssh/github_ed25519 && \
cat >> ~/.ssh/config << 'EOF'
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_ed25519
    IdentitiesOnly yes
    AddKeysToAgent yes
    UseKeychain yes
EOF
chmod 600 ~/.ssh/config && \
pbcopy < ~/.ssh/github_ed25519.pub && \
echo "Public key copied to clipboard! Add it at: https://github.com/settings/ssh/new"
```

### Windows One-Liner (Git Bash)

```bash
ssh-keygen -t ed25519 -C "YOUR_EMAIL" -f ~/.ssh/github_ed25519 -N "" -q && \
eval "$(ssh-agent -s)" && \
ssh-add ~/.ssh/github_ed25519 && \
cat >> ~/.ssh/config << 'EOF'
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_ed25519
    IdentitiesOnly yes
EOF
chmod 600 ~/.ssh/config && \
cat ~/.ssh/github_ed25519.pub | clip && \
echo "Public key copied to clipboard! Add it at: https://github.com/settings/ssh/new"
```

### Windows One-Liner (PowerShell)

```powershell
ssh-keygen -t ed25519 -C "YOUR_EMAIL" -f $env:USERPROFILE\.ssh\github_ed25519 -N '""'; `
ssh-add $env:USERPROFILE\.ssh\github_ed25519; `
$config = "Host github.com`n    HostName github.com`n    User git`n    IdentityFile ~/.ssh/github_ed25519`n    IdentitiesOnly yes"; `
Add-Content -Path $env:USERPROFILE\.ssh\config -Value $config; `
Get-Content $env:USERPROFILE\.ssh\github_ed25519.pub | Set-Clipboard; `
Write-Host "Public key copied to clipboard! Add it at: https://github.com/settings/ssh/new"
```

### Command Summary

| Task | Linux/macOS | Windows (Git Bash) | Windows (PowerShell) |
|------|-------------|--------------------|-----------------------|
| Generate key | `ssh-keygen -t ed25519 -C "email" -f ~/.ssh/github_ed25519 -N ""` | Same | `ssh-keygen -t ed25519 -C "email" -f $env:USERPROFILE\.ssh\github_ed25519 -N '""'` |
| View public key | `cat ~/.ssh/github_ed25519.pub` | Same | `Get-Content $env:USERPROFILE\.ssh\github_ed25519.pub` |
| Copy to clipboard | `pbcopy < file` (macOS) | `cat file \| clip` | `Get-Content file \| Set-Clipboard` |
| Start SSH agent | `eval "$(ssh-agent -s)"` | Same | `Start-Service ssh-agent` |
| Add key to agent | `ssh-add ~/.ssh/github_ed25519` | Same | `ssh-add $env:USERPROFILE\.ssh\github_ed25519` |
| Test connection | `ssh -T git@github.com` | Same | Same |
| Debug connection | `ssh -vT git@github.com` | Same | Same |
| List loaded keys | `ssh-add -l` | Same | Same |
| Convert repo to SSH | `git remote set-url origin git@github.com:USER/REPO.git` | Same | Same |

---

## Security Notes

- **Never share your private key** (`github_ed25519` without `.pub`)
- **Keep backups** of your keys in a secure location
- **Use unique keys** for different purposes/machines when possible
- **Revoke compromised keys** immediately at `https://github.com/settings/keys`
- **Review your keys periodically** and remove any that are no longer needed
