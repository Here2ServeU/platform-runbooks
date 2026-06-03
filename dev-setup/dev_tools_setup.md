# Developer tools setup guide

**Complete installation guide for VS Code, Python, Helm, kubectl, Node.js, npm, JavaScript, and Docker Desktop**

For complete beginners on Windows, macOS, and Linux.

---

## Table of contents

* [Before you start](#before-you-start)
* [What you are installing](#what-you-are-installing)
* [Windows setup](#windows-setup)
* [macOS setup](#macos-setup)
* [Linux setup](#linux-setup)
* [Verify everything works](#verify-everything-works)
* [Common errors and fixes](#common-errors-and-fixes)
* [Quick reference card](#quick-reference-card)

---

## Before you start

**What is a terminal?**
A terminal (also called command prompt, shell, or console) is a text window where you type commands. Every step in this guide uses one. Do not be intimidated: you will only ever type what is written here.

* **Windows:** Press `Win+R`, type `cmd`, press Enter. Or search for "PowerShell" in the Start menu.
* **macOS:** Press `Cmd+Space`, type `Terminal`, press Enter.
* **Linux:** Press `Ctrl+Alt+T` or search for "Terminal" in your app menu.

**What is PATH?**
PATH is a list your computer keeps of folders that contain programs. When you type `python` in a terminal, your computer looks through PATH to find it. If an install says "add to PATH": always say yes. If something "is not recognized" after installing: PATH is usually why.

**How to read this guide:**
* Lines inside grey boxes are commands to type in your terminal
* Lines starting with `#` are comments: they explain what the next line does, you do not type them
* Lines starting with `Expected:` or `#` after a command show what success looks like

---

## What you are installing

| Tool | What it is | Why you need it |
|------|-----------|-----------------|
| **VS Code** | Code editor | Where you write all your code |
| **Python** | Programming language | Runs Python scripts and AI agent code |
| **Node.js** | JavaScript runtime | Runs JavaScript outside a browser |
| **npm** | Node package manager | Installs JavaScript libraries |
| **JavaScript** | Language (runs via Node.js) | Needed for web frontends and many tools |
| **Docker Desktop** | Container platform | Packages and runs apps in isolated containers |
| **kubectl** | Kubernetes CLI | Controls Kubernetes clusters from your terminal |
| **Helm** | Kubernetes package manager | Installs pre-packaged apps onto Kubernetes |

**Install order matters.** Follow the steps in the order written. Some tools depend on others (Helm requires kubectl; kubectl works better after Docker is installed).

---

## Windows setup

> All commands below run in **PowerShell**. Search "PowerShell" in the Start menu and open it. Run it as Administrator where noted.

---

### Step 1: Install a package manager (winget or Chocolatey)

Windows 11 and recent Windows 10 versions include **winget** built in. Check if you have it:

```powershell
winget --version
```

If you see a version number, skip to Step 2.

If winget is not found, install **Chocolatey** instead: it does the same job:

```powershell
# Run PowerShell as Administrator for this command
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

Close and reopen PowerShell after this. Verify:

```powershell
choco --version
# Expected: Chocolatey v2.x.x
```

> For the rest of the Windows steps, commands show both winget and choco versions. Use whichever you have.

---

### Step 2: Install VS Code

**Option A: winget**
```powershell
winget install Microsoft.VisualStudioCode
```

**Option B: Chocolatey**
```powershell
choco install vscode -y
```

**Option C: Manual**
Go to **code.visualstudio.com**, download the Windows installer, run it. Check both boxes:
* "Add 'Open with Code' action to Windows Explorer file context menu"
* "Add to PATH"

Close and reopen PowerShell. Verify:

```powershell
code --version
# Expected: 1.9x.x
```

Open VS Code. Press `Ctrl+Shift+X` to open Extensions. Install these:

```
Python            (publisher: Microsoft)
GitLens           (publisher: GitKraken)
Docker            (publisher: Microsoft)
Kubernetes        (publisher: Microsoft)
ESLint            (publisher: Microsoft)
Prettier          (publisher: Prettier)
```

---

### Step 3: Install Python

> **Always check "Add Python to PATH"** during install. This is the single most common beginner mistake on Windows.

**Option A: winget**
```powershell
winget install Python.Python.3.12
```

**Option B: Chocolatey**
```powershell
choco install python312 -y
```

**Option C: Manual**
Go to **python.org/downloads**, download Python 3.12 Windows 64-bit installer, run it.
On the first screen: check **"Add Python 3.12 to PATH"** before clicking Install Now.

Close and reopen PowerShell. Verify:

```powershell
python --version
# Expected: Python 3.12.x

pip --version
# Expected: pip 24.x from ...Python312...
```

> **If `python` is not recognized after installing:** Open Start → search "Edit the system environment variables" → Environment Variables → under System Variables, find Path → Edit → New → add these two lines:
> ```
> C:\Users\YourUsername\AppData\Local\Programs\Python\Python312
> C:\Users\YourUsername\AppData\Local\Programs\Python\Python312\Scripts
> ```
> Close and reopen all terminals.

---

### Step 4: Install Node.js and npm

npm is bundled with Node.js: installing Node.js gives you both.

**Option A: winget**
```powershell
winget install OpenJS.NodeJS.LTS
```

**Option B: Chocolatey**
```powershell
choco install nodejs-lts -y
```

**Option C: Manual**
Go to **nodejs.org**, click the **LTS** version (Long Term Support: the stable one), download the Windows installer, run it. Accept all defaults.

Close and reopen PowerShell. Verify:

```powershell
node --version
# Expected: v20.x.x or v22.x.x (LTS)

npm --version
# Expected: 10.x.x
```

---

### Step 5: Install Docker Desktop

Docker Desktop is the largest install. It requires Windows 10 version 2004 or higher, and WSL2 (Windows Subsystem for Linux 2).

**First, enable WSL2:**
```powershell
# Run PowerShell as Administrator
wsl --install
```

Restart your computer when prompted.

**Then install Docker Desktop:**

**Option A: winget**
```powershell
winget install Docker.DockerDesktop
```

**Option B: Manual**
Go to **docker.com/products/docker-desktop**, download Docker Desktop for Windows, run the installer. Check "Use WSL2 instead of Hyper-V".

After install, launch Docker Desktop from the Start menu. Wait for the whale icon in the system tray to stop animating (takes 1–2 minutes). Then verify:

```powershell
docker --version
# Expected: Docker version 27.x.x

docker run hello-world
# Expected: "Hello from Docker!" message
```

> **If Docker Desktop asks to update WSL2 kernel:** Click the link it provides and follow the instructions, then restart Docker Desktop.

---

### Step 6: Install kubectl

kubectl is the command-line tool for controlling Kubernetes clusters.

**Option A: winget**
```powershell
winget install Kubernetes.kubectl
```

**Option B: Chocolatey**
```powershell
choco install kubernetes-cli -y
```

**Option C: Manual**
```powershell
# Download the latest stable release
$version = (Invoke-WebRequest -Uri "https://dl.k8s.io/release/stable.txt" -UseBasicParsing).Content.Trim()
Invoke-WebRequest -Uri "https://dl.k8s.io/release/$version/bin/windows/amd64/kubectl.exe" -OutFile "$HOME\kubectl.exe"

# Move to a folder in PATH
Move-Item "$HOME\kubectl.exe" "C:\Windows\System32\kubectl.exe"
```

Verify:

```powershell
kubectl version --client
# Expected: Client Version: v1.3x.x
```

---

### Step 7: Install Helm

Helm is the package manager for Kubernetes. kubectl must be installed first.

**Option A: Chocolatey**
```powershell
choco install kubernetes-helm -y
```

**Option B: Manual**
Go to **github.com/helm/helm/releases**, download the latest `helm-v3.x.x-windows-amd64.zip`, extract it, move `helm.exe` to `C:\Windows\System32\`.

Verify:

```powershell
helm version
# Expected: version.BuildInfo{Version:"v3.1x.x", ...}
```

---

### Step 8: Confirm JavaScript works

JavaScript runs through Node.js: no separate install is needed. Verify it works:

```powershell
node -e "console.log('JavaScript is working. Node version: ' + process.version)"
# Expected: JavaScript is working. Node version: v20.x.x
```

Create a quick test file:

```powershell
echo "console.log('Hello from JavaScript')" > test.js
node test.js
# Expected: Hello from JavaScript
```

---

## macOS setup

> macOS requires Homebrew as a package manager. Every step uses it. If you already have Homebrew, skip Step 1.

---

### Step 1: Install Homebrew

Open **Terminal** (`Cmd+Space` → type `Terminal` → `Enter`).

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

The installer will ask for your Mac password: type it (nothing appears while typing, that is normal). Takes 3–5 minutes.

When it finishes, it prints two lines starting with `echo` and `eval`. **Run both of those lines.** They add Homebrew to your PATH.

**Apple Silicon Macs (M1/M2/M3):**
```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zshrc
eval "$(/opt/homebrew/bin/brew shellenv)"
```

**Intel Macs:**
```bash
echo 'eval "$(/usr/local/bin/brew shellenv)"' >> ~/.zshrc
eval "$(/usr/local/bin/brew shellenv)"
```

Verify:

```bash
brew --version
# Expected: Homebrew 4.x.x
```

---

### Step 2: Install VS Code

```bash
brew install --cask visual-studio-code
```

Enable the `code` command in terminal: open VS Code → press `Cmd+Shift+P` → type `shell command` → click **Shell Command: Install 'code' command in PATH**.

Open VS Code. Press `Cmd+Shift+X`. Install these extensions:

```
Python            (publisher: Microsoft)
GitLens           (publisher: GitKraken)
Docker            (publisher: Microsoft)
Kubernetes        (publisher: Microsoft)
ESLint            (publisher: Microsoft)
Prettier          (publisher: Prettier)
```

Verify:

```bash
code --version
# Expected: 1.9x.x
```

---

### Step 3: Install Python

> **Do not use the Python that came with macOS.** It is old and managed by the system. Install a fresh one with Homebrew.

```bash
brew install python@3.12
```

Add it to PATH permanently:

```bash
# Apple Silicon
echo 'export PATH="/opt/homebrew/opt/python@3.12/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

# Intel Mac (if the above does not work)
echo 'export PATH="/usr/local/opt/python@3.12/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

> Check your prefix with: `brew --prefix python@3.12`

Verify:

```bash
python3 --version
# Expected: Python 3.12.x

pip3 --version
# Expected: pip 24.x from .../python3.12/...
```

---

### Step 4: Install Node.js and npm

```bash
brew install node
```

Verify:

```bash
node --version
# Expected: v20.x.x or v22.x.x

npm --version
# Expected: 10.x.x
```

---

### Step 5: Install Docker Desktop

```bash
brew install --cask docker
```

After install, open Docker from Applications. Wait for the whale icon in the menu bar to stop animating (1–2 minutes first launch).

Verify:

```bash
docker --version
# Expected: Docker version 27.x.x

docker run hello-world
# Expected: "Hello from Docker!" message
```

> **If Docker asks for your Mac password on first launch:** allow it: it needs to install helper tools.

---

### Step 6: Install kubectl

```bash
brew install kubectl
```

Verify:

```bash
kubectl version --client
# Expected: Client Version: v1.3x.x
```

---

### Step 7: Install Helm

```bash
brew install helm
```

Verify:

```bash
helm version
# Expected: version.BuildInfo{Version:"v3.1x.x", ...}
```

---

### Step 8: Confirm JavaScript works

```bash
node -e "console.log('JavaScript is working. Node version: ' + process.version)"
# Expected: JavaScript is working. Node version: v20.x.x
```

---

## Linux setup

> Tested on **Ubuntu 22.04 and 24.04 LTS**. For other distributions, see the notes at each step. Commands that differ on Fedora/RHEL are marked `[Fedora]`.

---

### Step 1: Update the system

Always start with a full system update. Many install failures on Linux come from an out-of-date package index.

```bash
sudo apt update && sudo apt upgrade -y

# [Fedora/RHEL]
# sudo dnf update -y
```

Install core build tools that many packages need:

```bash
sudo apt install -y \
  build-essential \
  curl \
  wget \
  git \
  unzip \
  apt-transport-https \
  ca-certificates \
  gnupg \
  lsb-release \
  software-properties-common

# [Fedora/RHEL]
# sudo dnf install -y gcc gcc-c++ make curl wget git unzip gnupg
```

---

### Step 2: Install VS Code

VS Code on Linux is installed via Microsoft's official apt repository. Do not install via snap: the apt version has better terminal integration.

```bash
# Add Microsoft's GPG key
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
sudo install -D -o root -g root -m 644 packages.microsoft.gpg /etc/apt/keyrings/packages.microsoft.gpg

# Add the VS Code repository
sudo sh -c 'echo "deb [arch=amd64,arm64,armhf signed-by=/etc/apt/keyrings/packages.microsoft.gpg] \
  https://packages.microsoft.com/repos/code stable main" \
  > /etc/apt/sources.list.d/vscode.list'

# Install
sudo apt update
sudo apt install -y code

# [Fedora/RHEL]
# sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
# sudo sh -c 'echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/vscode.repo'
# sudo dnf install code -y
```

Open VS Code (`code .`) and install extensions:

```
Python            (publisher: Microsoft)
GitLens           (publisher: GitKraken)
Docker            (publisher: Microsoft)
Kubernetes        (publisher: Microsoft)
ESLint            (publisher: Microsoft)
Prettier          (publisher: Prettier)
```

Verify:

```bash
code --version
# Expected: 1.9x.x
```

---

### Step 3: Install Python 3.12

Ubuntu 22.04 ships with Python 3.10. The deadsnakes PPA provides 3.12 without breaking system Python.

```bash
# Add the deadsnakes PPA (Ubuntu)
sudo add-apt-repository ppa:deadsnakes/ppa -y
sudo apt update
sudo apt install -y python3.12 python3.12-venv python3.12-dev python3.12-distutils

# [Fedora/RHEL]
# sudo dnf install -y python3.12 python3.12-pip
```

Set 3.12 as the default `python3`:

```bash
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.12 1
sudo update-alternatives --config python3
# Type 1 and press Enter to select python3.12
```

Install pip for Python 3.12:

```bash
curl -sS https://bootstrap.pypa.io/get-pip.py | python3.12
```

Verify:

```bash
python3 --version
# Expected: Python 3.12.x

python3 -m pip --version
# Expected: pip 24.x from .../python3.12/...
```

---

### Step 4: Install Node.js and npm

Use the NodeSource repository to get a current LTS version. The Node.js in the default Ubuntu apt repo is outdated.

```bash
# Download and run the NodeSource setup script for Node.js 20 LTS
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# [Fedora/RHEL]
# curl -fsSL https://rpm.nodesource.com/setup_20.x | sudo bash -
# sudo dnf install -y nodejs
```

Verify:

```bash
node --version
# Expected: v20.x.x

npm --version
# Expected: 10.x.x
```

---

### Step 5: Install Docker Engine (Linux does not have Docker Desktop)

On Linux, Docker Desktop is not available. Instead you install **Docker Engine** directly. It is the same underlying technology: just without the graphical interface.

```bash
# Remove any old Docker packages
sudo apt remove -y docker docker-engine docker.io containerd runc 2>/dev/null || true

# Add Docker's official GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add Docker's repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# [Fedora/RHEL]
# sudo dnf install -y dnf-plugins-core
# sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
# sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Start Docker and enable it to run on boot:

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

Allow your user to run Docker without `sudo`:

```bash
sudo usermod -aG docker $USER
```

> **You must log out and log back in** for this group change to take effect. Or run `newgrp docker` to apply it in the current session without logging out.

Verify:

```bash
docker --version
# Expected: Docker version 27.x.x

docker run hello-world
# Expected: "Hello from Docker!" message
```

---

### Step 6: Install kubectl

```bash
# Download the latest stable version
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Make it executable and move it to PATH
chmod +x kubectl
sudo mv kubectl /usr/local/bin/kubectl

# [Fedora/RHEL: alternative via repo]
# cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
# [kubernetes]
# name=Kubernetes
# baseurl=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/
# enabled=1
# gpgcheck=1
# gpgkey=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/repodata/repomd.xml.key
# EOF
# sudo dnf install -y kubectl
```

Verify:

```bash
kubectl version --client
# Expected: Client Version: v1.3x.x
```

---

### Step 7: Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

Verify:

```bash
helm version
# Expected: version.BuildInfo{Version:"v3.1x.x", ...}
```

---

### Step 8: Confirm JavaScript works

```bash
node -e "console.log('JavaScript is working. Node version: ' + process.version)"
# Expected: JavaScript is working. Node version: v20.x.x
```

---

## Verify everything works

Run this script on any platform after completing your setup. It checks every tool in one pass.

Create a file called `verify_setup.sh` (macOS/Linux) or `verify_setup.ps1` (Windows).

### macOS / Linux: `verify_setup.sh`

```bash
#!/bin/bash

echo "================================================"
echo " Developer tools verification"
echo "================================================"
echo ""

check() {
  if command -v $1 &>/dev/null; then
    echo "  ✓  $1 — $($1 $2 2>&1 | head -1)"
  else
    echo "  ✗  $1 — NOT FOUND"
  fi
}

check code    "--version"
check python3 "--version"
check pip3    "--version"
check node    "--version"
check npm     "--version"
check docker  "--version"
check kubectl "version --client --short 2>/dev/null || kubectl version --client"
check helm    "version --short"

echo ""
echo "================================================"
echo " Docker smoke test"
echo "================================================"
docker run --rm hello-world 2>&1 | grep "Hello from Docker" && echo "  ✓  Docker can run containers" || echo "  ✗  Docker container run failed"

echo ""
echo "================================================"
echo " Node.js smoke test"
echo "================================================"
node -e "console.log('  ✓  JavaScript running on Node ' + process.version)"

echo ""
echo "================================================"
echo " Python smoke test"
echo "================================================"
python3 -c "import sys; print('  ✓  Python', sys.version.split()[0], 'ready')"
```

Run it:

```bash
chmod +x verify_setup.sh
./verify_setup.sh
```

### Windows: `verify_setup.ps1`

```powershell
Write-Host "================================================"
Write-Host " Developer tools verification"
Write-Host "================================================"
Write-Host ""

function Check-Tool {
  param($name, $args)
  if (Get-Command $name -ErrorAction SilentlyContinue) {
    $ver = & $name $args 2>&1 | Select-Object -First 1
    Write-Host "  OK  $name -- $ver" -ForegroundColor Green
  } else {
    Write-Host "  MISSING  $name -- NOT FOUND" -ForegroundColor Red
  }
}

Check-Tool "code"    "--version"
Check-Tool "python"  "--version"
Check-Tool "pip"     "--version"
Check-Tool "node"    "--version"
Check-Tool "npm"     "--version"
Check-Tool "docker"  "--version"
Check-Tool "kubectl" "version --client"
Check-Tool "helm"    "version --short"

Write-Host ""
Write-Host "Docker smoke test:"
docker run --rm hello-world 2>&1 | Select-String "Hello from Docker" | ForEach-Object { Write-Host "  OK  Docker can run containers" -ForegroundColor Green }

Write-Host ""
Write-Host "Node.js smoke test:"
node -e "console.log('  OK  JavaScript running on Node ' + process.version)"

Write-Host ""
Write-Host "Python smoke test:"
python -c "import sys; print('  OK  Python', sys.version.split()[0], 'ready')"
```

Run it:

```powershell
.\verify_setup.ps1
```

**Every tool should show a version number.** If any show NOT FOUND, go back to that tool's step and check the PATH instructions.

---

## Common errors and fixes

### VS Code: `code: command not found`

**macOS:** Open VS Code → `Cmd+Shift+P` → type `shell command` → click **Install 'code' command in PATH**.

**Linux:** The apt install puts `code` in PATH automatically. If it is missing: `sudo ln -s /usr/share/code/code /usr/local/bin/code`

**Windows:** Reinstall VS Code and check "Add to PATH" during install.

---

### Python: `python: command not found` on macOS/Linux

Use `python3` instead of `python`. The command `python` (without the 3) no longer exists on modern systems.

```bash
# Create a permanent alias (macOS/Linux)
echo "alias python=python3" >> ~/.zshrc   # macOS
echo "alias python=python3" >> ~/.bashrc  # Linux
source ~/.zshrc    # or source ~/.bashrc
```

---

### Python: `pip: command not found`

```bash
# macOS / Linux
python3 -m pip --version

# If that also fails: reinstall pip
curl -sS https://bootstrap.pypa.io/get-pip.py | python3.12
```

---

### Python on Windows: `python is not recognized`

Python was installed without "Add to PATH". Uninstall Python from Settings → Apps, rerun the installer, and check **"Add Python 3.12 to PATH"** on the very first screen before clicking Install Now.

---

### Node.js: outdated version from system package manager

Never install Node.js with `sudo apt install nodejs` on Ubuntu: it gives a years-old version. Always use the NodeSource script or `brew install node` on macOS.

```bash
# Check your version
node --version
# If it shows v10, v12, or v14: remove it and reinstall using the guide above

# Ubuntu: remove the old version
sudo apt remove -y nodejs npm
# Then follow Step 4 in the Linux section above
```

---

### Docker: `permission denied` on Linux

Your user is not in the docker group yet. After running `sudo usermod -aG docker $USER` you must log out and log back in for the change to apply.

Quick fix without logging out:

```bash
newgrp docker
docker run hello-world
```

---

### Docker: `Cannot connect to the Docker daemon`

Docker is not running.

```bash
# Linux
sudo systemctl start docker
sudo systemctl status docker    # look for "active (running)"

# macOS / Windows
# Open Docker Desktop from Applications / Start menu and wait for the whale icon to stop animating
```

---

### Docker on Windows: WSL2 error during install

```powershell
# Run as Administrator
wsl --install
wsl --update
```

Restart your computer. Then launch Docker Desktop again.

---

### kubectl: `The connection to the server was refused`

This is normal if you do not have a Kubernetes cluster running. kubectl installed correctly: it just has nothing to connect to yet.

```bash
# Confirm kubectl itself is fine
kubectl version --client
# Expected: shows client version: no server version needed at this stage
```

---

### Helm: `Error: Kubernetes cluster unreachable`

Same as above: Helm needs a running cluster to deploy to. The install is correct.

```bash
# Confirm Helm works on its own
helm version
helm env
```

---

### PowerShell: `scripts is disabled on this system`

```powershell
# Run once as Administrator
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

---

### Any tool not found after installing on Windows

The tool installed but the terminal session does not know about it yet. Close all PowerShell and Command Prompt windows and open a fresh one. PATH updates only apply to new terminal sessions.

---

## Quick reference card

### Version check: run any time

```bash
# macOS / Linux
code --version
python3 --version
pip3 --version
node --version
npm --version
docker --version
kubectl version --client
helm version

# Windows
python --version
pip --version
# (everything else is the same)
```

### Update all tools

```bash
# macOS: update everything via Homebrew
brew update && brew upgrade

# Linux: update system packages
sudo apt update && sudo apt upgrade -y

# Windows: update via winget
winget upgrade --all

# npm: update npm itself
npm install -g npm@latest

# Helm: macOS/Linux
brew upgrade helm   # macOS
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash   # Linux

# kubectl: download the latest binary (same install command as initial install)
```

### Create a new Python virtual environment

```bash
# Always use a virtual environment for Python projects
python3 -m venv venv
source venv/bin/activate    # macOS / Linux
venv\Scripts\Activate.ps1   # Windows

# Install packages inside the venv
pip install package-name

# Save dependencies
pip freeze > requirements.txt

# Restore from requirements.txt on a new machine
pip install -r requirements.txt

# Deactivate when done
deactivate
```

### Create a new Node.js project

```bash
mkdir my-project
cd my-project
npm init -y                  # creates package.json
npm install package-name     # install a package
npm run start                # run the start script
```

### Basic Docker commands

```bash
docker run hello-world                          # test Docker works
docker ps                                       # list running containers
docker ps -a                                    # list all containers including stopped
docker images                                   # list downloaded images
docker pull ubuntu:22.04                        # download an image
docker run -it ubuntu:22.04 bash                # run interactively
docker stop container-id                        # stop a running container
docker rm container-id                          # remove a stopped container
docker rmi image-name                           # remove an image
```

### Basic kubectl commands

```bash
kubectl get nodes                               # list cluster nodes
kubectl get pods                                # list pods in default namespace
kubectl get pods -n namespace-name              # list pods in a specific namespace
kubectl get all                                 # list everything in default namespace
kubectl apply -f manifest.yaml                  # deploy from a YAML file
kubectl delete -f manifest.yaml                 # remove what was deployed
kubectl logs pod-name                           # view pod logs
kubectl describe pod pod-name                   # detailed pod info
kubectl exec -it pod-name -- bash               # open a shell in a pod
```

### Basic Helm commands

```bash
helm repo add stable https://charts.helm.sh/stable    # add a chart repository
helm repo update                                       # update repo index
helm search repo nginx                                 # search for a chart
helm install my-release chart-name                    # install a chart
helm list                                              # list installed releases
helm upgrade my-release chart-name                    # upgrade a release
helm uninstall my-release                             # remove a release
helm status my-release                                # check release status
```

---

*Guide maintained by Emmanuel Naweji · T2S · emlinkapp.com*

*Last updated: 2026 · Versions current as of date above: check official docs for latest releases*
