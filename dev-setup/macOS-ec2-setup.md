# macOS EC2 instance setup runbook

**Create an AWS EC2 Mac instance that looks and feels exactly like a real Mac computer**

For course recordings, development environments, and professional macOS setups in the cloud.

-----

## Table of contents

- [What you are building](#what-you-are-building)
- [Before you start: requirements and costs](#before-you-start-requirements-and-costs)
- [Part 1: Launch the macOS EC2 instance](#part-1-launch-the-macos-ec2-instance)
- [Part 2: Connect for the first time](#part-2-connect-for-the-first-time)
- [Part 3: Make it look exactly like a real Mac](#part-3-make-it-look-exactly-like-a-real-mac)
- [Part 4: Install the full developer toolchain](#part-4-install-the-full-developer-toolchain)
- [Part 5: Connect via screen sharing for a full desktop](#part-5-connect-via-screen-sharing-for-a-full-desktop)
- [Part 6: Make it feel native](#part-6-make-it-feel-native)
- [Daily workflow](#daily-workflow)
- [Cost control](#cost-control)
- [Common errors and fixes](#common-errors-and-fixes)

-----

## What you are building

AWS offers real Mac hardware as EC2 instances. These are actual Mac mini or Mac Studio machines running in an AWS data center. You get a full macOS desktop, complete with Finder, Dock, Safari, the App Store, and every native Mac application. It connects to your laptop through Screen Sharing or any VNC client and looks indistinguishable from sitting in front of a real Mac.

What you will have at the end of this runbook:

- A real macOS desktop accessible from any computer on the internet
- macOS Ventura or Sonoma with all system UI intact
- Homebrew, Python 3.12, Node.js, VS Code, Docker, kubectl, and Helm installed
- The Dock, menu bar, Mission Control, and Spotlight configured exactly as on a local Mac
- A custom wallpaper and system font to match your personal or course branding
- Screen sharing that opens full-screen and feels like a local machine

-----

## Before you start: requirements and costs

### AWS account requirements

- An AWS account with a payment method attached
- IAM permissions for EC2 (or root account access)
- A key pair created in the AWS region you will use

### Cost warning: read this before proceeding

Mac EC2 instances are billed differently from Linux instances. AWS allocates a dedicated Mac mini or Mac Studio to you and charges by the hour even when the instance is stopped. There is also a **minimum allocation period of 24 hours** regardless of when you stop or terminate.

Approximate costs as of 2026:

|Instance type     |Hardware               |vCPU|RAM  |Hourly cost|
|------------------|-----------------------|----|-----|-----------|
|`mac1.metal`      |Mac mini (Intel, 2018) |12  |32 GB|~$1.08/hr  |
|`mac2.metal`      |Mac mini (M1, 2020)    |8   |16 GB|~$0.65/hr  |
|`mac2-m2.metal`   |Mac mini (M2, 2023)    |8   |24 GB|~$0.65/hr  |
|`mac2-m2pro.metal`|Mac mini (M2 Pro, 2023)|12  |32 GB|~$1.00/hr  |

**Recommendation:** Use `mac2.metal` (M1) for development and course recording. It runs macOS 12 Monterey through macOS 14 Sonoma, supports Apple Silicon natively, and is the most cost-effective.

A 40-hour recording week costs approximately $26. A month of full-time use (160 hours) costs approximately $104.

### Dedicated host requirement

Mac instances require a **Dedicated Host**, not a standard EC2 instance. This is because AWS must comply with Apple’s macOS license, which requires single-tenant hardware. You allocate the host first, then launch an instance on it.

-----

## Part 1: Launch the macOS EC2 instance

### Step 1: Allocate a Dedicated Host

Open the AWS console. Go to **EC2 > Dedicated Hosts > Allocate Dedicated Host**.

Fill in the form:

```
Name tag:          mac-host-01
Instance family:   mac2
Instance type:     mac2.metal
Availability Zone: choose one (e.g. us-east-1a)
Quantity:          1
```

Click **Allocate**. Wait for the status to show **Available**. This takes 2 to 5 minutes.

> The dedicated host starts billing immediately. The 24-hour minimum allocation clock starts now.

### Step 2: Create a security group for Mac access

Go to **EC2 > Security Groups > Create security group**.

```
Name:        mac-access-sg
Description: Security group for macOS EC2 access
VPC:         your default VPC
```

Add these inbound rules:

|Type      |Protocol|Port|Source|Purpose            |
|----------|--------|----|------|-------------------|
|SSH       |TCP     |22  |My IP |SSH terminal access|
|Custom TCP|TCP     |5900|My IP |VNC screen sharing |
|Custom TCP|TCP     |5901|My IP |VNC alternate port |


> Always set Source to **My IP**, not `0.0.0.0/0`. Your Mac desktop will be fully visible to anyone who reaches port 5900.

### Step 3: Launch the macOS EC2 instance

Go to **EC2 > Instances > Launch Instances**.

**Name:** `mac-dev-01`

**Application and OS images:**

- Click **macOS** in the Quick Start tab
- Select **macOS Sonoma 14** (or Ventura 13 if Sonoma is unavailable in your region)
- Architecture: **Apple Silicon (arm64)** for mac2, **x86_64** for mac1

**Instance type:** `mac2.metal`

**Key pair:** Select your existing key pair, or create one now. Download the `.pem` file and keep it safe.

**Network settings:**

- Select your default VPC and a public subnet
- Enable **Auto-assign public IP**
- Select the security group you created: `mac-access-sg`

**Configure storage:**

- Root volume: increase from the default 60 GB to **200 GB** (gp3)
- The default 60 GB fills up quickly with Xcode, Docker images, and course materials

**Advanced details:**

- Host: select the dedicated host you allocated in Step 1
- Tenancy: **Dedicated Host**

Click **Launch Instance**. Wait for the instance state to show **Running** and the status checks to show **2/2 checks passed**. This takes 5 to 10 minutes on first boot.

### Step 4: Note your connection details

In EC2 > Instances, click your instance and note:

```
Public IPv4 address:   copy this
Public IPv4 DNS:       copy this (use this for connections, it stays stable during reboots)
Instance ID:           i-xxxxxxxxxxxx
```

-----

## Part 2: Connect for the first time

### Step 5: SSH into the instance

On your local machine, open Terminal.

```bash
# Fix key permissions (required every time you use a new key)
chmod 400 /path/to/your-key.pem

# Connect via SSH
ssh -i /path/to/your-key.pem ec2-user@YOUR_PUBLIC_DNS
```

> The default user for macOS EC2 instances is `ec2-user`, not `ubuntu` or `root`.

You will see the macOS terminal welcome message. You are now inside the Mac instance.

### Step 6: Resize the root volume to use all 200 GB

AWS allocates the volume at 200 GB but macOS only sees 60 GB until you tell it to use the rest.

```bash
# Check current disk size (you will see ~60 GB used, ~200 GB allocated)
df -h /

# Resize the APFS container to fill the full volume
sudo diskutil apfs resizeContainer disk0s2 0
```

Verify:

```bash
df -h /
# Expected: Size column now shows ~190 GB or larger
```

### Step 7: Set the ec2-user password

The screen sharing system requires a password. Set one now.

```bash
sudo passwd ec2-user
```

Type a strong password. You will need it every time you connect via VNC.

-----

## Part 3: Make it look exactly like a real Mac

All commands in this part run over SSH in the terminal. They configure the macOS graphical environment so that when you connect via screen sharing it looks and behaves identically to a physical Mac.

### Step 8: Enable screen sharing (VNC)

```bash
# Enable the built-in macOS VNC server
sudo /System/Library/CoreServices/RemoteManagement/ARDAgent.app/Contents/Resources/kickstart \
  -activate \
  -configure \
  -access -on \
  -clientopts -setvnclegacy -vnclegacy yes \
  -clientopts -setvncpw -vncpw YOUR_VNC_PASSWORD \
  -restart -agent \
  -privs -all
```

Replace `YOUR_VNC_PASSWORD` with a password of 8 characters or fewer (VNC has an 8-character limit).

Enable the sharing preference as well:

```bash
sudo defaults write /var/db/launchd.db/com.apple.launchd/overrides.plist \
  com.apple.screensharing -dict Disabled -bool false

sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.screensharing.plist
```

### Step 9: Set the screen resolution

By default, the EC2 Mac has a very low virtual resolution. Set it to a resolution that matches your laptop screen so content renders at the right size.

```bash
# Install the displayplacer tool to control virtual displays
brew install jakehilborn/jakehilborn/displayplacer

# List current displays
displayplacer list

# Set resolution to 2560x1600 at 60Hz (matches most Mac laptop screens)
displayplacer "id:<DISPLAY_ID> res:2560x1600 hz:60 color_depth:8 scaling:on origin:(0,0) degree:0"
```

> Replace `<DISPLAY_ID>` with the ID shown by `displayplacer list`. On a fresh EC2 Mac it is usually `37D8832A-2D66-02CA-B9F7-8F30A301B230`.

For a 1080p setting (lighter on bandwidth):

```bash
displayplacer "id:<DISPLAY_ID> res:1920x1080 hz:60 color_depth:8 scaling:off origin:(0,0) degree:0"
```

### Step 10: Set the Mac computer name

Make the instance appear with a real Mac name so the menu bar and system dialogs look authentic.

```bash
# Set all three name properties that macOS uses
sudo scutil --set ComputerName "Emmanuel-MacStudio"
sudo scutil --set HostName "Emmanuel-MacStudio"
sudo scutil --set LocalHostName "Emmanuel-MacStudio"

# Flush the DNS cache so the name takes effect
sudo dscacheutil -flushcache
```

Replace `Emmanuel-MacStudio` with any name you prefer. This name appears in Finder, System Settings, and the menu bar sharing menu.

### Step 11: Set the wallpaper

```bash
# Download a macOS-style wallpaper (Sonoma default)
curl -L -o ~/Desktop/wallpaper.jpg \
  "https://512pixels.net/downloads/macos-wallpapers/14-0-Dynamic.heic" 2>/dev/null || \
  echo "Download manually and place at ~/Desktop/wallpaper.jpg"

# Set it as the desktop wallpaper via osascript
osascript -e 'tell application "Finder" to set desktop picture to POSIX file "/Users/ec2-user/Desktop/wallpaper.jpg"'
```

Alternatively, set any solid color:

```bash
# Set a dark solid color desktop (looks clean for screen recordings)
osascript << 'EOF'
tell application "System Events"
  tell every desktop
    set picture to ""
    set picture rotation to 0
    set random order to false
    set change interval to 0
  end tell
end tell
EOF

# Use the macOS built-in Space Gray equivalent
osascript -e 'tell app "Finder" to set desktop picture to POSIX file "/System/Library/Desktop Pictures/Solid Colors/Stone.png"'
```

### Step 12: Configure the Dock

Set up the Dock to look like a clean professional Mac install.

```bash
# Remove all default Dock items
defaults write com.apple.dock persistent-apps -array
defaults write com.apple.dock persistent-others -array

# Set Dock size (48 is the macOS default)
defaults write com.apple.dock tilesize -int 48

# Enable magnification and set magnification size
defaults write com.apple.dock magnification -bool true
defaults write com.apple.dock largesize -int 72

# Position: bottom (options: bottom, left, right)
defaults write com.apple.dock orientation -string "bottom"

# Auto-hide the Dock
defaults write com.apple.dock autohide -bool true
defaults write com.apple.dock autohide-delay -float 0
defaults write com.apple.dock autohide-time-modifier -float 0.5

# Show only open applications in the Dock
defaults write com.apple.dock static-only -bool false

# Minimize windows using the scale effect (faster than genie)
defaults write com.apple.dock mineffect -string "scale"

# Add your most-used apps to the Dock
# Format: add <app-bundle-path> to persistent-apps
defaults write com.apple.dock persistent-apps -array-add \
  '<dict><key>tile-data</key><dict><key>file-data</key><dict><key>_CFURLString</key><string>/Applications/Finder.app</string><key>_CFURLStringType</key><integer>0</integer></dict></dict></dict>'

defaults write com.apple.dock persistent-apps -array-add \
  '<dict><key>tile-data</key><dict><key>file-data</key><dict><key>_CFURLString</key><string>/System/Applications/Launchpad.app</string><key>_CFURLStringType</key><integer>0</integer></dict></dict></dict>'

defaults write com.apple.dock persistent-apps -array-add \
  '<dict><key>tile-data</key><dict><key>file-data</key><dict><key>_CFURLString</key><string>/Applications/Safari.app</string><key>_CFURLStringType</key><integer>0</integer></dict></dict></dict>'

defaults write com.apple.dock persistent-apps -array-add \
  '<dict><key>tile-data</key><dict><key>file-data</key><dict><key>_CFURLString</key><string>/Applications/Visual Studio Code.app</string><key>_CFURLStringType</key><integer>0</integer></dict></dict></dict>'

defaults write com.apple.dock persistent-apps -array-add \
  '<dict><key>tile-data</key><dict><key>file-data</key><dict><key>_CFURLString</key><string>/Applications/Utilities/Terminal.app</string><key>_CFURLStringType</key><integer>0</integer></dict></dict></dict>'

# Apply all Dock changes
killall Dock
```

### Step 13: Configure the menu bar

```bash
# Show battery percentage in menu bar (if applicable)
defaults write com.apple.menuextra.battery ShowPercent -string "YES"

# Show date and time in menu bar
defaults write com.apple.menuextra.clock ShowDate -int 1
defaults write com.apple.menuextra.clock ShowDayOfWeek -bool true
defaults write com.apple.menuextra.clock DateFormat -string "EEE d MMM  HH:mm"

# Show all menu bar items (no auto-hiding)
defaults write com.apple.controlcenter "NSStatusItem Visible WiFi" -bool true
defaults write com.apple.controlcenter "NSStatusItem Visible Bluetooth" -bool true

# Restart menu bar to apply
killall SystemUIServer
```

### Step 14: Configure Finder

```bash
# Show filename extensions in Finder
defaults write NSGlobalDomain AppleShowAllExtensions -bool true

# Show hidden files in Finder
defaults write com.apple.finder AppleShowAllFiles -bool true

# Show path bar at the bottom of Finder windows
defaults write com.apple.finder ShowPathbar -bool true

# Show status bar in Finder
defaults write com.apple.finder ShowStatusBar -bool true

# Set the default Finder view to columns (most professional for recordings)
defaults write com.apple.finder FXPreferredViewStyle -string "clmv"

# Keep folders on top when sorting by name
defaults write com.apple.finder _FXSortFoldersFirst -bool true

# Search the current folder by default (not the whole Mac)
defaults write com.apple.finder FXDefaultSearchScope -string "SCcf"

# Remove items from Trash after 30 days
defaults write com.apple.finder FXRemoveOldTrashItems -bool true

# Show the home folder in the Finder sidebar
chflags nohidden ~/Library

# Restart Finder to apply
killall Finder
```

### Step 15: Configure system appearance

```bash
# Set appearance to Dark mode (or Light mode: change "Dark" to "Light")
osascript -e 'tell app "System Events" to tell appearance preferences to set dark mode to true'

# Set accent color to Blue (the macOS default)
defaults write NSGlobalDomain AppleAccentColor -int 4

# Set highlight color to Blue
defaults write NSGlobalDomain AppleHighlightColor -string "0.698039 0.843137 1.000000 Blue"

# Enable font smoothing (makes text look crisp on VNC)
defaults write NSGlobalDomain AppleFontSmoothing -int 1
defaults write -g CGFontRenderingFontSmoothingDisabled -bool NO

# Set sidebar icon size to medium
defaults write NSGlobalDomain NSTableViewDefaultSizeMode -int 2

# Disable the "Are you sure you want to open this application?" dialog
defaults write com.apple.LaunchServices LSQuarantine -bool false
```

### Step 16: Configure the keyboard and trackpad behavior

```bash
# Enable full keyboard access for all controls (Tab moves between all UI elements)
defaults write NSGlobalDomain AppleKeyboardUIMode -int 3

# Disable press-and-hold for keys (enables key repeat instead)
defaults write NSGlobalDomain ApplePressAndHoldEnabled -bool false

# Set fast key repeat rate
defaults write NSGlobalDomain KeyRepeat -int 2
defaults write NSGlobalDomain InitialKeyRepeat -int 15

# Disable autocorrect
defaults write NSGlobalDomain NSAutomaticSpellingCorrectionEnabled -bool false

# Disable smart quotes (important for coding)
defaults write NSGlobalDomain NSAutomaticQuoteSubstitutionEnabled -bool false

# Disable smart dashes (important for coding)
defaults write NSGlobalDomain NSAutomaticDashSubstitutionEnabled -bool false
```

### Step 17: Apply all system changes

Many macOS defaults require a full logout and login to apply. On EC2, restart the WindowServer instead:

```bash
# Apply all defaults changes
/System/Library/PrivateFrameworks/SystemAdministration.framework/Resources/activateSettings -u

# Restart the graphics system (this disconnects any active VNC session briefly)
sudo killall -HUP WindowServer
```

Wait 10 to 15 seconds after this command before reconnecting via VNC.

-----

## Part 4: Install the full developer toolchain

All commands in this part run in the SSH terminal session.

### Step 18: Install Homebrew

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Add Homebrew to PATH for Apple Silicon (mac2.metal):

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zshrc
eval "$(/opt/homebrew/bin/brew shellenv)"
```

Verify:

```bash
brew --version
# Expected: Homebrew 4.x.x
```

### Step 19: Install Python 3.12

```bash
brew install python@3.12

echo 'export PATH="/opt/homebrew/opt/python@3.12/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

python3 --version
# Expected: Python 3.12.x
```

### Step 20: Install Node.js and npm

```bash
brew install node

node --version
# Expected: v20.x.x or v22.x.x

npm --version
# Expected: 10.x.x
```

### Step 21: Install VS Code

```bash
brew install --cask visual-studio-code
```

Enable the `code` shell command:

```bash
cat >> ~/.zshrc << 'EOF'
export PATH="$PATH:/Applications/Visual Studio Code.app/Contents/Resources/app/bin"
EOF
source ~/.zshrc

code --version
# Expected: 1.9x.x
```

Install extensions from the command line:

```bash
code --install-extension ms-python.python
code --install-extension eamodio.gitlens
code --install-extension ms-azuretools.vscode-docker
code --install-extension ms-kubernetes-tools.vscode-kubernetes-tools
code --install-extension dbaeumer.vscode-eslint
code --install-extension esbenp.prettier-vscode
```

### Step 22: Install Docker Desktop

```bash
brew install --cask docker

# Open Docker Desktop for the first time (it needs to initialize)
open /Applications/Docker.app
```

Wait for the whale icon to appear in the menu bar and stop animating. Then verify:

```bash
docker --version
# Expected: Docker version 27.x.x

docker run hello-world
# Expected: Hello from Docker!
```

### Step 23: Install kubectl and Helm

```bash
brew install kubectl
kubectl version --client
# Expected: Client Version: v1.3x.x

brew install helm
helm version
# Expected: version.BuildInfo{Version:"v3.1x.x"}
```

### Step 24: Install Git and configure it

Git ships with macOS developer tools. Configure your identity:

```bash
git config --global user.name "Your Full Name"
git config --global user.email "you@email.com"
git config --global init.defaultBranch main

git --version
# Expected: git version 2.x.x
```

-----

## Part 5: Connect via screen sharing for a full desktop

### Step 25: Connect from a Mac (native Screen Sharing app)

On your local Mac, open **Finder > Go > Connect to Server** (`Cmd+K`). Type:

```
vnc://YOUR_EC2_PUBLIC_DNS
```

Click **Connect**. Enter your `ec2-user` password when prompted. The EC2 Mac desktop opens in a window.

To go full-screen: press `Cmd+Shift+F` inside the Screen Sharing window. The EC2 Mac fills your entire display and is indistinguishable from sitting in front of a physical Mac.

### Step 26: Connect from Windows or Linux (VNC client)

Download and install **RealVNC Viewer** (free) from `realvnc.com/en/connect/download/viewer/`.

Create a new connection:

```
VNC Server:  YOUR_EC2_PUBLIC_DNS:5900
Name:        Mac EC2
```

Connect and enter your VNC password when prompted.

### Step 27: Connect from a browser (noVNC)

For browser-based access without installing any client:

```bash
# Install noVNC on the EC2 instance
brew install novnc

# Start noVNC on port 6080, forwarding to the VNC server on 5900
nohup novnc --listen 6080 --vnc localhost:5900 > ~/novnc.log 2>&1 &
```

Add port 6080 to your security group inbound rules (Custom TCP, port 6080, My IP). Then open in any browser:

```
http://YOUR_EC2_PUBLIC_DNS:6080/vnc.html
```

Click Connect and enter your VNC password.

-----

## Part 6: Make it feel native

These steps make the remote session behave as if you are using a local Mac.

### Step 28: Enable SSH tunneling for secure VNC (recommended)

Instead of exposing port 5900 directly, tunnel VNC through SSH. This means you only need port 22 open in the security group.

```bash
# Run this on your LOCAL machine (not the EC2 instance)
ssh -i /path/to/your-key.pem -L 5900:localhost:5900 -N ec2-user@YOUR_EC2_PUBLIC_DNS &
```

Then connect your VNC client to `localhost:5900` instead of the public DNS. All traffic is encrypted and you can remove the port 5900 inbound rule from the security group.

### Step 29: Set up SSH config for one-command connection

On your local machine, open `~/.ssh/config` and add:

```
Host mac-ec2
    HostName       YOUR_EC2_PUBLIC_DNS
    User           ec2-user
    IdentityFile   /path/to/your-key.pem
    ServerAliveInterval 60
    ServerAliveCountMax 3
    LocalForward   5900 localhost:5900
```

Now connect with just:

```bash
ssh mac-ec2
```

And your VNC tunnel is set up automatically.

### Step 30: Install Oh My Zsh for a professional terminal

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

Set a clean theme:

```bash
# Open ~/.zshrc and find the ZSH_THEME line
sed -i '' 's/ZSH_THEME="robbyrussell"/ZSH_THEME="agnoster"/' ~/.zshrc
source ~/.zshrc
```

### Step 31: Configure Terminal colors and font

Open **Terminal > Preferences** (`Cmd+,`) in the VNC session. Under **Profiles**:

- Select the **Pro** profile
- Set font to **SF Mono Regular 14pt** (this is Apple’s own coding font, installed with Xcode)
- Set background color to black with 90% opacity
- Set text color to white

To install SF Mono if it is not available:

```bash
# SF Mono ships with Xcode Command Line Tools
xcode-select --install
```

### Step 32: Set up a startup script to auto-start services on reboot

```bash
cat > ~/startup.sh << 'EOF'
#!/bin/bash

# Start VNC screen sharing
sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.screensharing.plist

# Start Docker Desktop
open /Applications/Docker.app

echo "Startup complete: $(date)"
EOF

chmod +x ~/startup.sh
```

Add it to the login items via the command line:

```bash
osascript -e 'tell application "System Events" to make login item at end with properties {path:"/Users/ec2-user/startup.sh", hidden:false}'
```

-----

## Daily workflow

### Starting a session

```bash
# Step 1: On your local machine, open the SSH tunnel
ssh mac-ec2

# Step 2: Connect your VNC client to localhost:5900
# (the tunnel forwards it to the EC2 instance automatically)

# Step 3: Work in the full Mac desktop
```

### Stopping the instance to save cost

**Critical:** Stopping a Mac EC2 instance does not stop billing immediately. The dedicated host continues to accrue charges. To stop billing you must **release the dedicated host**.

```bash
# From the AWS console:
# EC2 > Dedicated Hosts > select your host > Actions > Release hosts
# This terminates the instance as well
```

If you need to pause but keep your data:

```bash
# Stop the instance (keeps your data, but the dedicated host still bills at full rate)
# EC2 > Instances > select instance > Instance State > Stop
```

### Creating an AMI to resume later

Before releasing the dedicated host, create an AMI (Amazon Machine Image) so you can restore your exact setup later:

```bash
# From the AWS console:
# EC2 > Instances > select your instance > Actions > Image and templates > Create image
# Name: mac-dev-snapshot-YYYY-MM-DD
# No reboot: checked (keeps the instance running while the snapshot is taken)
```

Next time you need the environment:

1. Allocate a new dedicated host
1. Launch an instance using your saved AMI
1. All tools, settings, and files are exactly as you left them

-----

## Cost control

### Strategy 1: Use Elastic IP to keep a stable address across restarts

Without an Elastic IP, your public DNS changes every time you stop and start the instance. An Elastic IP is static and free while attached to a running instance.

```bash
# From the AWS console:
# EC2 > Elastic IPs > Allocate Elastic IP address > Allocate
# Then: Actions > Associate Elastic IP address > select your Mac instance
```

### Strategy 2: Set a billing alarm

```bash
# From the AWS console:
# Billing > Budgets > Create Budget
# Type: Cost budget
# Amount: $50/month (adjust to your comfort level)
# Alert at: 80% of budget
# Email: your email address
```

### Strategy 3: Schedule automatic stop with Lambda

For a course recording environment used only during business hours, you can auto-stop the instance with an AWS Lambda function triggered on a schedule. This keeps the dedicated host but stops unnecessary Docker and server processes.

The simplest approach: use **AWS Instance Scheduler**, available in the AWS Solutions Library at `aws.amazon.com/solutions/implementations/instance-scheduler-on-aws/`.

-----

## Common errors and fixes

### SSH: `Permission denied (publickey)`

```bash
# Fix key file permissions on your local machine
chmod 400 /path/to/your-key.pem

# Verify you are using the correct username
ssh -i /path/to/your-key.pem ec2-user@YOUR_EC2_PUBLIC_DNS
# The user is ec2-user, not ubuntu, not root, not admin
```

### SSH: `Connection timed out`

Port 22 is not open in the security group. Go to EC2 > Security Groups > your security group > Inbound Rules and confirm SSH (port 22) is set to My IP.

### VNC: `Connection refused` on port 5900

Screen sharing is not running on the instance. SSH in and run:

```bash
sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.screensharing.plist

# Confirm it is listening on port 5900
sudo lsof -i :5900
# Expected: output shows screensharingd listening
```

### VNC: blank black screen after connecting

The WindowServer restarted and the display session needs to be re-initialized. SSH in and run:

```bash
# Restart the screen sharing service
sudo launchctl unload /System/Library/LaunchDaemons/com.apple.screensharing.plist
sleep 3
sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.screensharing.plist
```

Wait 15 seconds and reconnect your VNC client.

### VNC: very slow or laggy desktop

The default VNC server sends uncompressed frames. Use a VNC client that supports compression, or reduce resolution:

```bash
# Set a lower resolution to reduce bandwidth
displayplacer "id:<DISPLAY_ID> res:1920x1080 hz:60 color_depth:8 scaling:off origin:(0,0) degree:0"
```

RealVNC Viewer automatically negotiates compression. If using the macOS Screen Sharing app, go to View > Connection Quality and select **Adaptive**.

### Homebrew: `command not found: brew` after SSH reconnect

The PATH was set for your current session but not saved permanently.

```bash
# Add to your shell profile permanently
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zshrc
source ~/.zshrc
```

### displayplacer: `No displays found`

The VNC server must be running and at least one VNC client must have connected at least once before `displayplacer` can see the virtual display.

1. Connect via VNC from your client
1. Disconnect the VNC session
1. Now run `displayplacer list` over SSH

### Docker Desktop: does not start after instance reboot

Docker Desktop requires user interaction to accept the license on first launch. Do this once through the VNC desktop session, not SSH.

For subsequent reboots, add Docker to the startup script from Step 32 and ensure it is also in System Settings > General > Login Items.

### Instance type not available in your region

Not all AWS regions offer Mac instances. Available regions as of 2026:

```
us-east-1       (US East, N. Virginia)
us-east-2       (US East, Ohio)
us-west-2       (US West, Oregon)
eu-west-1       (Europe, Ireland)
ap-southeast-1  (Asia Pacific, Singapore)
ap-northeast-1  (Asia Pacific, Tokyo)
```

If `mac2.metal` is unavailable in your preferred region, switch to `us-east-1` which has the most Mac instance capacity.

-----

## Verify the full setup

Run this after completing all parts to confirm everything is working:

```bash
#!/bin/bash
echo "========================================"
echo " macOS EC2 environment verification"
echo "========================================"
echo ""
echo "System:"
sw_vers
echo ""
echo "Disk:"
df -h / | tail -1
echo ""
echo "Tools:"
brew --version | head -1
python3 --version
node --version
npm --version
code --version | head -1
docker --version
kubectl version --client 2>&1 | head -1
helm version --short
git --version
echo ""
echo "Screen sharing:"
sudo lsof -i :5900 | grep LISTEN | head -1 && echo "VNC: listening on port 5900" || echo "VNC: NOT listening"
echo ""
echo "========================================"
echo " All checks complete"
echo "========================================"
```

Save this as `verify_mac_ec2.sh` and run it with `bash verify_mac_ec2.sh`.

-----

*Runbook by Emmanuel Naweji · 2026 · emmanuelnaweji.com*

*AWS pricing and instance availability change frequently. Verify current prices at aws.amazon.com/ec2/pricing/on-demand/ before allocating a dedicated host.*