# Using VS Code with MSI HPC Systems

This guide helps you set up and use Visual Studio Code (VS Code) to work on MSI's HPC systems (Agate).

## Table of Contents

- [Prerequisites](#prerequisites)
- [Step 1: Install VS Code](#step-1-install-vs-code)
- [Step 2: Set Up SSH Keys](#step-2-set-up-ssh-keys)
- [Step 3: Configure SSH Connection](#step-3-configure-ssh-connection)
- [Step 4: Install Remote-SSH Extension](#step-4-install-remote-ssh-extension)
- [Step 5: Connect to MSI](#step-5-connect-to-msi)
- [Step 6: Open Your Workspace](#step-6-open-your-workspace)
- [Step 7: Install Recommended Extensions](#step-7-install-recommended-extensions)
- [Step 8: Configure Git](#step-8-configure-git)
- [Step 9: Using tmux for Persistent Sessions](#step-9-using-tmux-for-persistent-sessions)
- [Tips and Best Practices](#tips-and-best-practices)
- [Troubleshooting](#troubleshooting)

---

## Prerequisites

- An active MSI account
- VS Code installed on your local computer
- Basic familiarity with terminal/command line
- Two-factor authentication (Duo) set up for MSI

---

## Step 1: Install VS Code

1. Download VS Code from: https://code.visualstudio.com/
2. Install it on your local computer (Windows, Mac, or Linux)
3. Launch VS Code

---

## Step 2: Set Up SSH Keys

SSH keys allow you to connect without entering your password every time.

### On Windows:

1. Open PowerShell or Command Prompt
2. Generate an SSH key:
   ```powershell
   ssh-keygen -t ed25519 -C "your_email@umn.edu"
   ```
3. Press Enter to accept the default location (`C:\Users\YourUsername\.ssh\id_ed25519`)
4. Set a passphrase (optional but recommended)

### On Mac/Linux:

1. Open Terminal
2. Generate an SSH key:
   ```bash
   ssh-keygen -t ed25519 -C "your_email@umn.edu"
   ```
3. Press Enter to accept the default location (`~/.ssh/id_ed25519`)
4. Set a passphrase (optional but recommended)

### Copy Your Public Key to MSI:

```bash
# Replace USERNAME with your MSI username
ssh-copy-id USERNAME@agate.msi.umn.edu
```

Or manually:
1. View your public key:
   - **Mac/Linux**: `cat ~/.ssh/id_ed25519.pub`
   - **Windows**: `type %USERPROFILE%\.ssh\id_ed25519.pub`
2. Copy the output
3. SSH into MSI: `ssh USERNAME@agate.msi.umn.edu`
4. Add to authorized keys:
   ```bash
   mkdir -p ~/.ssh
   chmod 700 ~/.ssh
   echo "PASTE_YOUR_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys
   chmod 600 ~/.ssh/authorized_keys
   ```

---

## Step 3: Configure SSH Connection

Create or edit your SSH config file to simplify connections.

### Location:
- **Mac/Linux**: `~/.ssh/config`
- **Windows**: `C:\Users\YourUsername\.ssh\config`

### Add MSI Configuration:

```
# MSI Agate Login Node
Host msi-agate
    HostName agate.msi.umn.edu
    User YOUR_MSI_USERNAME
    IdentityFile ~/.ssh/id_ed25519
    ForwardAgent yes
    ServerAliveInterval 60
    ServerAliveCountMax 5
```

**Important**: Replace `YOUR_MSI_USERNAME` with your actual MSI username.

### What these settings do:
- `HostName`: The actual server address
- `User`: Your MSI username
- `IdentityFile`: Path to your SSH private key
- `ForwardAgent`: Allows authentication forwarding (useful for git)
- `ServerAliveInterval/CountMax`: Keeps connection alive

---

## Step 4: Install Remote-SSH Extension

1. Open VS Code
2. Click the Extensions icon in the left sidebar (or press `Ctrl+Shift+X` / `Cmd+Shift+X`)
3. Search for "Remote - SSH"
4. Install the extension by Microsoft (look for the verified publisher badge)
5. You may also want to install:
   - **Remote - SSH: Editing Configuration Files** (for editing SSH config from VS Code)
   - **Remote Explorer** (usually installed automatically)

---

## Step 5: Connect to MSI

### Method 1: Using the Command Palette

1. Press `F1` or `Ctrl+Shift+P` (Windows/Linux) / `Cmd+Shift+P` (Mac)
2. Type "Remote-SSH: Connect to Host"
3. Select `msi-agate` (or the host name you configured)
4. A new VS Code window will open
5. You may be prompted:
   - To select the platform (choose **Linux**)
   - For your SSH key passphrase (if you set one)
   - For Duo two-factor authentication (approve on your device)

### Method 2: Using the Remote Explorer

1. Click the Remote Explorer icon in the left sidebar
2. Under "SSH Targets", you'll see `msi-agate`
3. Click the connect icon (arrow) next to it

### First Connection:

The first time you connect, VS Code will:
- Install VS Code Server on MSI (in `~/.vscode-server/`)
- This takes a few minutes
- You'll see a progress indicator in the bottom right

---

## Step 6: Open Your Workspace

Once connected:

1. Click **File > Open Folder** (or `Ctrl+K Ctrl+O`)
2. Navigate to your work directory, for example:
   - `/projects/regulated/GROUP/USERNAME/project/` (project directory)
   - `/scratch.global/USERNAME/` (scratch space)
3. Click **OK**
4. VS Code will reload with that folder as your workspace

### Create a Workspace File (Optional):

For projects you work on regularly:

1. Open the folder in VS Code
2. Go to **File > Save Workspace As...**
3. Save it with a `.code-workspace` extension
4. Next time, just open this workspace file

---

## Step 7: Install Recommended Extensions

Extensions add functionality to VS Code. Install them while connected to MSI.

1. Click the Extensions icon in the left sidebar (or press `Ctrl+Shift+X` / `Cmd+Shift+X`)
2. Search for the extension name
3. Click **Install** (make sure you're connected to MSI when installing)
4. Some extensions may require reloading VS Code

### Recommended Extensions by Language:

#### Python Extensions
- **Python** (Microsoft) - Essential for Python development, includes IntelliSense and debugging
- **Pylance** (Microsoft) - Advanced type checking and IntelliSense for Python
- **Jupyter** (Microsoft) - Run and edit Jupyter notebooks directly in VS Code
- **autoDocstring** - Automatically generate Python docstrings
- **Python Indent** - Correct Python indentation automatically

#### R Extensions
- **R** (REditorSupport) - R language support with syntax highlighting
- **R Debugger** - Debug R code interactively
- **R Markdown All in One** - Support for R Markdown files (.Rmd)

#### Bash/Shell Scripting Extensions
- **ShellCheck** - Linting and error detection for bash/shell scripts
- **Bash IDE** - IntelliSense and debugging for bash
- **shell-format** - Format shell scripts automatically
- **Even Better TOML** - For editing config files (common in HPC)

#### Git Extensions
- **GitLens** - Supercharged Git capabilities (blame, history, compare branches)
- **Git Graph** - View and interact with your Git repository graph
- **Git History** - View git log and file/line history

#### Markdown Extensions
- **Markdown All in One** - Keyboard shortcuts, table of contents, auto preview
- **Markdown Preview Enhanced** - Enhanced preview with diagrams and math support
- **markdownlint** - Linting and style checking for Markdown files

#### Data & File Viewers
- **Excel Viewer** - View CSV/TSV files as formatted tables
- **Rainbow CSV** - Highlight CSV/TSV columns in different colors
- **vscode-pdf** - View PDF files in VS Code

#### General HPC/Scientific Computing
- **YAML** - Syntax highlighting and validation for YAML files (Nextflow, Snakemake, etc.)
- **Todo Tree** - Highlights TODO, FIXME, and other comment tags
- **Path Intellisense** - Autocomplete file paths
- **Error Lens** - Show errors inline in your code
- **Remote - SSH** (already installed) - Connect to remote servers
- **Code Spell Checker** - Catch spelling mistakes in code and comments

#### Optional but Useful
- **Markdown Table Prettifier** - Format markdown tables
- **Nextflow** - Syntax highlighting for Nextflow workflows
- **Snakemake Language** - Support for Snakemake workflows

**Pro Tip**: Don't install all extensions at once. Start with the essentials for your language(s), then add others as needed. Too many extensions can slow down VS Code. **If your MSI sessions keep disconnecting, disable extensions that you're not actively using.**

---

## Step 8: Configure Git

VS Code has excellent built-in Git support that makes version control easy.

### Set Up Git Integration:

1. **View changes**: Click the Source Control icon in the left sidebar (or press `Ctrl+Shift+G` / `Cmd+Shift+G`)
2. **Stage files**: Click the `+` next to files you want to commit
3. **Commit changes**: Enter a commit message and click the checkmark
4. **Push/Pull**: Use the `...` menu for push, pull, and other Git operations
5. **View diffs**: Click any modified file to see side-by-side comparison

---

## Step 9: Using tmux for Persistent Sessions

**Why use tmux with VS Code:**
- Sessions persist even if VS Code disconnects or your laptop sleeps
- Long-running processes continue in the background
- **Critical for interactive SLURM sessions** - without tmux, interactive jobs are killed when you disconnect, close VS Code, or close the terminal
- The tmux session runs on the login node, and your interactive job runs inside it

**Note**: tmux is already installed on MSI systems - no installation needed!

**Basic tmux workflow in VS Code terminal:**

1. **Create a new tmux session** (in VS Code terminal):
   ```bash
   tmux new-session -s mysession
   ```

2. **Start your interactive SLURM job inside tmux**:
   ```bash
   srun -N 1 --ntasks-per-node=4 --mem-per-cpu=3900mb -t 8:00:00 -p interactive --pty bash
   ```

3. **Detach from tmux** (session keeps running):
   - Press `Ctrl+B`, then `D`
   - Or close VS Code - the session persists!

4. **List existing sessions**:
   ```bash
   tmux list-sessions
   ```

5. **Re-attach to a session** (in VS Code terminal):
   ```bash
   tmux attach -t mysession
   ```

6. **Kill a session when completely done**:
   ```bash
   tmux kill-session -t mysession
   ```

**Important - Login Node Consistency:**
- tmux sessions run on a specific login node (e.g., `ahl01`, `ahl02`)
- You must reconnect to the **same login node** where you started the session
- **Note**: VS Code typically connects to the same login node consistently (until VS Code is updated, then it may change)
- **To switch to a specific login node** (e.g., to reconnect to an existing tmux session):
  ```bash
  ssh ahl01
  # Or whatever login node your tmux session is on
  ```

**Learn more about tmux:**
- Official tmux documentation: https://github.com/tmux/tmux/wiki
- Quick reference cheat sheet: https://tmuxcheatsheet.com/

**Configure tmux for better VS Code experience:**

Create or edit `~/.tmux.conf` and add these settings:
```bash
# Enable mouse support (scroll, select panes, resize)
set -g mouse on

# Paste with Shift+Left-click (useful in VS Code terminal)
bind-key -T root S-MouseDown1Pane paste-buffer
```

After creating/editing the config, either:
- Reload in existing session: `tmux source-file ~/.tmux.conf`
- Or start a new tmux session to use the new settings

**Tips for VS Code + tmux:**
- Always start interactive sessions in tmux
- Use descriptive session names: `tmux new-session -s project-xyz-interactive-analysis`
- You can have multiple windows in one session (`Ctrl+B` then `C` to create new window, `Ctrl+B` then number to switch) - use different windows for different tasks
- If VS Code disconnects, your tmux session is still running - just reconnect to MSI and reattach
- For long-running jobs, tmux ensures your work continues even if you close your laptop

---

## Tips and Best Practices

### 1. Terminal Access

- Open an integrated terminal: **Terminal > New Terminal** or `` Ctrl+` ``
- This terminal runs directly on the HPC login node
- You can use all your usual commands: `sbatch`, `squeue`, `module load`, etc.

### 2. Running SLURM Jobs

You can submit jobs directly from VS Code's terminal:

```bash
# Submit a job
sbatch my_script.slurm

# Check job status
squeue --me

# View job output
tail -f my_script.slurm.o123456
```

### 3. Editing Files

- Just click any file in the Explorer to edit it
- Syntax highlighting works for most languages
- Use `Ctrl+P` to quickly open files by name
- Use `Ctrl+Shift+F` to search across all files

### 4. Working with Large Files

- VS Code may struggle with very large files (>50MB)
- For large data files, use command-line tools instead
- Consider using `head`, `tail`, or `less` for viewing

### 5. Preserving Your Session

- VS Code sessions persist across disconnections
- If you lose connection, just reconnect and your open files will still be there
- Running terminals will be restored (but not running processes - see section about tmux)

### 6. Using Multiple Terminal Tabs

- Create multiple terminals: Click the `+` in the terminal panel
- Switch between them using the dropdown
- Useful for monitoring jobs while working

---

## Troubleshooting

### Connection Issues

**Problem**: "Could not establish connection to host"

**Solutions**:
- Verify you can SSH from terminal: `ssh msi-agate`
- Check your internet connection
- If off-campus, make sure you're connected to MSI VPN
- Approve Duo notification if it appears

**Problem**: "Permission denied (publickey)"

**Solutions**:
- Verify your SSH key is set up: `ls ~/.ssh/id_ed25519*`
- Check that your public key is in `~/.ssh/authorized_keys` on MSI
- Verify the path in your SSH config matches your actual key location

### Performance Issues

**Problem**: VS Code is slow or unresponsive

**Solutions**:
- Close unnecessary files and tabs
- Disable extensions you're not using
- Exclude large directories from file watching (Settings > Files: Watcher Exclude)
- Don't open very large files in the editor

**Problem**: Git is slow

**Solutions**:
- Disable auto-fetch: Settings > Git: Autofetch = false
- Add large directories to `.gitignore` (data files, output directories, large binary files)
- For large repos, use command line git instead

### General Tips

- If something goes wrong, try disconnecting and reconnecting
- Check the Output panel (View > Output) and select "Remote - SSH" for logs
- Your `.bashrc` must not have interactive prompts or output (VS Code won't connect)
- Keep VS Code updated for bug fixes and improvements

---

## Getting Help

- **MSI Email**: help@msi.umn.edu
- **VS Code Remote-SSH Docs**: https://code.visualstudio.com/docs/remote/ssh
- **VS Code Issues**: https://github.com/microsoft/vscode-remote-release/issues

---

## Quick Reference Card

### Common Commands

| Action | Windows/Linux | Mac |
|--------|---------------|-----|
| Connect to remote | `F1` → "Remote-SSH: Connect to Host" | `F1` → "Remote-SSH: Connect to Host" |
| Open folder | `Ctrl+K Ctrl+O` | `Cmd+K Cmd+O` |
| New terminal | `` Ctrl+` `` | `` Cmd+` `` |
| Quick file open | `Ctrl+P` | `Cmd+P` |
| Search in files | `Ctrl+Shift+F` | `Cmd+Shift+F` |
| Command palette | `Ctrl+Shift+P` | `Cmd+Shift+P` |
| Save all | `Ctrl+K S` | `Cmd+K S` |
| Close folder | `Ctrl+K F` | `Cmd+K F` |

### Common File Locations

| Purpose | Path |
|---------|------|
| SSH config | `~/.ssh/config` (on your local machine) |
| Bash config | `~/.bashrc` (on MSI) |
| VS Code settings | `~/.vscode-server/` (on MSI) |

---

**Version**: 1.0  
**Last Updated**: May 2026  
**Maintained by**: MSI Users

Feel free to share this guide with colleagues or contribute improvements!
