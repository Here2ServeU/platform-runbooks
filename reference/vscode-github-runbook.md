# VS Code And GitHub Setup Runbook

Author: Emmanuel Naweji
Role: PhD Candidate at National University; Infrastructure, Cloud, AI, and DevOps Engineer; Mentor

## Purpose

Use this runbook to install Visual Studio Code on macOS or Windows and configure it to work with GitHub for OpenShift platform engineering tasks.

## Scope

This runbook covers:

- installing VS Code on macOS or Windows
- signing in with GitHub
- configuring Git identity and authentication
- adding useful extensions for YAML, Git, Markdown, and OpenShift workflows
- validating the setup from the terminal and editor

## Prerequisites

- macOS or Windows workstation
- GitHub account
- permission to install local applications
- internet access to download VS Code and extensions

## Install VS Code

### macOS Option 1. Download From Microsoft

1. Open the Visual Studio Code download page.
2. Download the macOS build for your processor architecture.
3. Move Visual Studio Code into the `Applications` folder.
4. Launch VS Code.

### macOS Option 2. Install With Homebrew

```bash
brew install --cask visual-studio-code
```

### Windows Option 1. Download From Microsoft

1. Open the Visual Studio Code download page.
2. Download the Windows User Installer unless your team requires the System Installer.
3. Run the installer.
4. Select the options to add VS Code to `PATH` and register supported file types if your team allows it.
5. Launch VS Code after installation completes.

### Windows Option 2. Install With Winget

```powershell
winget install --id Microsoft.VisualStudioCode --exact
```

### Windows Option 3. Install With Chocolatey

```powershell
choco install vscode -y
```

## Enable The `code` Command

### macOS

In VS Code, open the command palette and run:

```text
Shell Command: Install 'code' command in PATH
```

Validate:

```bash
code --version
```

### Windows

The VS Code installer usually adds the `code` command to `PATH` automatically.

If `code` is not available:

1. Close and reopen PowerShell or Command Prompt.
2. Confirm VS Code was installed with the PATH option enabled.
3. If required, add the VS Code `bin` directory to the user PATH.

Typical user install path:

```text
C:\Users\<username>\AppData\Local\Programs\Microsoft VS Code\bin
```

Validate:

```powershell
code --version
```

## Configure Git Identity

```bash
git config --global user.name "<your-name>"
git config --global user.email "<your-company-email>"
git config --global init.defaultBranch main
git config --global pull.ff only
```

Validate:

```bash
git config --global --list
```

## Sign In To GitHub In VS Code

1. Open VS Code.
2. Select the Accounts icon.
3. Choose `Sign in with GitHub`.
4. Complete the browser-based authentication flow.
5. Return to VS Code and approve access.

## Configure Git With GitHub Tokens

Use token-based authentication when your environment does not allow password authentication for Git over HTTPS, or when your team requires PAT-based access.

### Token Types

#### Fine-Grained Personal Access Token

Use a fine-grained token when possible.

Benefits:

- repository-specific access
- narrower permission scope
- better control over expiration and approval

Typical use case:

- you only need access to one repository or one organization scope

#### Classic Personal Access Token

Use a classic token only if your workflow or GitHub organization still requires it.

Typical use case:

- older automation or tools that do not yet support fine-grained tokens cleanly
- legacy workflows that expect classic scopes such as `repo`

### Recommended GitHub Token Guidance

- prefer fine-grained tokens for interactive workstation use
- use the shortest practical expiration period
- grant only the minimum repository permissions required
- do not paste tokens into notes, screenshots, or markdown files
- if your organization uses SSO, authorize the token for the organization after creation

### Create A Token

Create the token in GitHub account settings under developer settings and personal access tokens.

When creating a token:

- set a clear token name, for example `vscode-macbook-git`
- set an expiration date
- limit repository access to only the repositories you need
- grant read and write access only where required

### Configure Git Credential Storage

Avoid embedding a token directly in the remote URL. Prefer a credential helper or GitHub CLI.

#### macOS Keychain

```bash
git config --global credential.helper osxkeychain
```

#### Windows Credential Manager

```powershell
git config --global credential.helper manager
```

If Git Credential Manager is not installed in your environment, install Git for Windows with credential manager support or use GitHub CLI authentication.

### Authenticate Using GitHub CLI

This is the cleanest option for many workstation setups.

```bash
gh auth login
```

Choose:

- `GitHub.com` or your approved GitHub host
- `HTTPS`
- authenticate with a browser or paste a token when prompted

After successful sign-in, Git operations over HTTPS can reuse the stored authentication.

### Authenticate By Prompting For The Token

If a credential helper is configured, Git can prompt you the first time you clone, fetch, or push over HTTPS.

Example clone:

```bash
git clone https://github.com/<org>/<repo>.git
```

When prompted:

- username: your GitHub username
- password: paste the personal access token, not your GitHub password

The credential helper should store it securely after the first successful operation.

### Verify Token-Based Git Access

Run:

```bash
git ls-remote https://github.com/<org>/<repo>.git
```

If authentication is working, Git returns the remote refs instead of prompting repeatedly or failing with `Authentication failed`.

### Avoid This Pattern

Do not use URLs like this in documentation or shell history:

```text
https://<username>:<token>@github.com/<org>/<repo>.git
```

This can leak credentials into shell history, process lists, screenshots, and logs.

## Optional GitHub CLI Setup

Install GitHub CLI if your team uses it.

```bash
brew install gh
gh auth login
```

On Windows:

```powershell
winget install --id GitHub.cli --exact
gh auth login
```

## Recommended VS Code Extensions

Install these extensions for OpenShift-oriented work:

- `GitHub.copilot`
- `GitHub.copilot-chat`
- `eamodio.gitlens`
- `redhat.vscode-yaml`
- `redhat.ansible`
- `redhat.vscode-openshift-connector`
- `yzhang.markdown-all-in-one`

Install from the terminal if preferred:

```bash
code --install-extension GitHub.copilot
code --install-extension GitHub.copilot-chat
code --install-extension eamodio.gitlens
code --install-extension redhat.vscode-yaml
code --install-extension redhat.ansible
code --install-extension redhat.vscode-openshift-connector
code --install-extension yzhang.markdown-all-in-one
```

## Recommended VS Code Settings

Create or update user settings with values similar to the following:

```json
{
  "editor.formatOnSave": true,
  "files.trimTrailingWhitespace": true,
  "files.insertFinalNewline": true,
  "yaml.format.enable": true,
  "yaml.validate": true,
  "git.autofetch": true,
  "markdown.preview.breaks": true
}
```

## Clone A GitHub Repository In VS Code

### macOS

```bash
cd ~/git
git clone https://github.com/<org>/<repo>.git
cd <repo>
code .
```

### Windows PowerShell

```powershell
New-Item -ItemType Directory -Path "$HOME\git" -Force
Set-Location "$HOME\git"
git clone https://github.com/<org>/<repo>.git
Set-Location .\<repo>
code .
```

## Validation Checklist

Confirm the following:

- `code --version` works in the terminal
- `git config --global --list` shows the expected identity
- GitHub sign-in is visible in VS Code Accounts
- token-based Git authentication works for HTTPS operations if your team uses PATs
- you can clone a repository
- source control view shows branch and file status correctly
- YAML files validate without parser errors
- the integrated terminal opens in your expected shell on macOS or Windows

## Best Practices

- use your company-managed GitHub identity for work repositories
- configure Git before making your first commit
- prefer PAT, GitHub CLI, or SSO-backed authentication over password-based flows
- prefer fine-grained personal access tokens over classic tokens unless tooling requires classic scopes
- store tokens in Keychain, Credential Manager, or GitHub CLI rather than in remotes or scripts
- install only extensions your team actually uses in production workflows
- keep VS Code and extensions updated to avoid schema and auth bugs
- on Windows, prefer PowerShell or Windows Terminal for Git and OpenShift CLI work

## Troubleshooting

### `code` Command Is Not Found

- on macOS, rerun `Shell Command: Install 'code' command in PATH`
- restart the terminal session
- on Windows, verify the VS Code installer added the `bin` directory to `PATH`
- check whether your shell profile or enterprise management tooling overrides `PATH`

### GitHub Sign-In Loops Or Fails

- confirm the browser completed the auth flow
- verify any corporate proxy or VPN restrictions
- sign out and retry from the Accounts menu
- if your org uses SSO, ensure the token or app has been authorized for the organization

### Git Push Fails After Cloning

- verify repository permissions
- confirm the remote URL is correct with `git remote -v`
- re-authenticate with GitHub CLI or your configured credential helper
- if using a personal access token, verify it is not expired and has the required repository permissions

### Token Works In Browser But Git Still Fails

- confirm the token was authorized for your organization if SSO is required
- verify the remote uses `https` if you intend to use PAT authentication
- clear old cached credentials and retry authentication with the correct token
- confirm you used the token in place of the password, not in place of the username

### YAML Files Show Schema Errors

- confirm the Red Hat YAML extension is installed
- verify the file is valid YAML before debugging schema issues
- ensure tabs are not used for indentation

### Escalation Data To Capture

Capture:

- operating system and version
- VS Code version
- Git version
- extension names involved
- exact error text or screenshot with sensitive data removed
