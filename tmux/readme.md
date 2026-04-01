# How to use tmux on MSI systems

Todd Knutson

## Introduction

When connecting to remote systems using `ssh`, sometimes the connection can fail or you may need to shut down your computer and reconnect later. Using tmux will (1) allow your remote session to continue running even if your connection is broken, and (2) will allow you to reconnect to your still-running terminal session later. In addition, tmux has many other features that GNU Screen does not easily offer. Most importantly, reconnecting to a tmux session will update your environment variables (for example `DISPLAY` and `SSH_AUTH_SOCK`) in new windows. This makes reconnecting to X11 windows and using ssh key forwarding work after reconnecting.

### Disclaimer

This is how I do it. I am not a networking expert, so there might be better or more useful ways of accomplishing the same goal. Also, this example is focused on connecting a MacBook to MSI's Linux servers. Using a Windows computer might require some differences.

I am using some special flags in my `ssh` commands (for example `-Y -A -t -X -C`) or my `srun` commands (for example `--x11`) to allow for X11 forwarding.

## Connect to UMN network via VPN

To use MSI systems, you need to be on campus or connected to the UMN network using the VPN (split tunnel is fine). UMN OIT setup info:

- <https://it.umn.edu/service-details/virtual-private-network-vpn>

## Connect to HPC login node (`agate`)

Replace `USERNAME` with your MSI username.

```bash
ssh -Y -A USERNAME@agate.msi.umn.edu

# Alternative route staff need to use:
ssh -Y -A USERNAME@staff.msi.umn.edu
```

The direct `agate` login command routes your connection to one of the Agate login hosts (`ahl01`, `ahl02`, `ahl03`, or `ahl04`). We need to remember which exact hostname was used for our connection. Running `hostname` will tell us.

```bash
hostname
# ahl02
```

## Load the `tmux` software

After connecting to the MSI HPC login node, load tmux. MSI provides tmux version 2.7 at `/bin/tmux` by default.

I also have a personal build of `tmux 3.5`, but it is private and not shareable outside the `lmnp` project space. If you want to use a newer version, you can build and install your own copy somewhere accessible.

```bash
/bin/tmux -V
# tmux 2.7
```

If you maintain your own tmux build, add your install path to `PATH` so your preferred `tmux` is found first. For example:

```bash
export PATH="/projects/standard/PROJECT/path/to/tmux/bin:$PATH"
```


## Start a tmux session

Start a new session and provide it with a descriptive name. I tend to name the session the name of the group I am working with (for example `lmnp`) or the current date (for example `2020-09-09`).

```bash
tmux new -s lmnp
```

This clears your terminal window and you are now running inside a tmux session.

- Do any work as usual on the HPC nodes, including launching batch jobs.
- Or start an interactive job:

```bash
srun --pty --x11 --nodes=1 --ntasks-per-node=1 --tmp=16G --partition=interactive --mem=8gb --cpus-per-task=1 --time=1:00:00 bash -l
```

## Navigating the `tmux` interface

Tmux uses key bindings to navigate or manipulate the program through a variety of commands. All tmux commands are initiated with a `PREFIX` keystroke. The default tmux `PREFIX` is `C-b` (holding `Ctrl` and pressing `b`).

- To see a list of all possible commands, type `PREFIX ?`.
  - Verbose version: hold `Ctrl`, press `b`, release keys, then type `?`. Press `q` to leave the help screen.
- To create a new window (tab), type `PREFIX c`.
- To switch between windows, type `PREFIX` followed by the window number (for example `2`).
- To see all available sessions and windows, type `PREFIX w`.
- To exit a window (kill the window, not the session), type `PREFIX &`.
- To split a window into two vertical panes, type `PREFIX %`. Then switch panes with `PREFIX` and the left/right arrow keys.

### Great shortcuts and tricks

- [tmuxcheatsheet](https://tmuxcheatsheet.com/?q=&hPP=100&idx=tmux_cheats&p=0&is_v=1)
- [tmux-cheats](https://gist.github.com/Starefossen/5955406) (this page uses `C-a` as the `PREFIX`, so adjust commands accordingly)

## Disconnect from your tmux session

- If you lose your network connection for any reason, your session is still running on MSI. Disconnecting from tmux can happen by accident (for example VPN timeout).
- You can also manually disconnect/detach with `PREFIX d`.

## Reconnect to your tmux session

If you lose your internet connection, you can reconnect/reattach to your still-running session on the HPC login node.

- Close your broken terminal window/tab (Terminal.app, WezTerm, etc.).
- Open a new terminal window/tab.
- You must reconnect to the same Agate login hostname (`ahl01` to `ahl04`) that was running your tmux session.
- Reconnect using one of these patterns:

```bash
# Pattern 1: go to agate, then move to the original login host
ssh -Y -A USERNAME@agate.msi.umn.edu
hostname
# if this is not your original host, hop to it directly:
ssh ahl02

# Pattern 2: staff gateway first, then target the original host
ssh -Y -A USERNAME@staff.msi.umn.edu
# hop to the original login host, from staff node
ssh ahl02.agate.msi.umn.edu
```

- Reattach the tmux session currently running on that host:

```bash
# verify tmux is available, then inspect and attach
tmux ls
tmux attach
```

- If you had multiple sessions/windows running, you can browse any of them with `PREFIX w`.

### Restore important environment variables

When you connect to a server (for example MSI) using `ssh-agent` forwarding and/or X11 enabled, certain environment variables get set, including `DISPLAY` and `SSH_AUTH_SOCK`. These variables are needed to properly display an X11 window (for example interactive R graphics) or access keys provided by `ssh-agent` (for example push/pull to GitHub).

If you reattach to an old tmux session, these environment variables are not updated in your old windows. However, any new windows you create inherit the latest values.

You can manually update these environment variables inside old windows:

```bash
eval $(tmux showenv -s SSH_CONNECTION); eval $(tmux showenv -s SSH_AUTH_SOCK); eval $(tmux showenv -s DISPLAY)
```

#### Special note for R users

If you reattach to a session running an R console, your X11 `DISPLAY` will be old and X11 windows may not work properly. I have not solved this for R consoles running on a compute node. However, the procedure below works for R consoles running on Stratus:

- Open a new tmux window (tab). This inherits the proper `DISPLAY` in bash: `echo $DISPLAY`.
- Copy the value (for example `localhost:13.0`).
- Go back to your old window still running R. Set `DISPLAY` in R: `Sys.setenv(DISPLAY = "localhost:13.0")`.
- Test an X11 window by creating a simple plot: `plot(1:10, 1:10)`.
- If that does not work, quit and reopen your terminal app and repeat.

## Delete all tmux sessions

You can delete all tmux sessions from the HPC login host with:

```bash
tmux kill-session
```

## Best practices

When using `tmux`, all MSI best practices still apply. In particular, do not use login nodes (`ahl01` to `ahl04`) for CPU- or memory-intensive tasks. Your tmux sessions persist until you kill them (or until login nodes are rebooted on maintenance day).

When using `tmux` on a Stratus VM, you are connected to your full compute resources. In this case, leaving active CPU-intensive processes running in a tmux session can make sense.

### Ideal use-cases for `tmux` on MSI

- Starting an interactive SLURM job on a compute node.
- Launching and monitoring batch SLURM jobs.
- Browsing the filesystem.
- Viewing or editing scripts (split panes are great for this).
- Pushing/pulling from GitHub.
- Using multiple terminal windows at one time without opening separate ssh connections.

## `.tmux.conf` advanced (helpful)

Tmux can be heavily modified using a configuration text file saved as `~/.tmux.conf` in your home directory. I have provided a bare-bones example in this repo with some changes I find helpful.

For example, scrolling behavior can be improved, tab bar colors can be changed, and window numbering can be altered.

- Example file provided here: [`~/.tmux.conf`](./.tmux.conf)

## Alternatives

Tmux is not the only way to solve this problem.

- GNU Screen: an early terminal multiplexer with fewer features than `tmux`, but it gets the job done. <https://www.gnu.org/software/screen>
- MSI NICE Linux desktop sessions with compute resources: <https://www.msi.umn.edu/support/faq/how-do-i-obtain-graphical-connection-using-nice-system>
- `nohup command &`: adding `&` runs your command in a subshell. `nohup` changes `SIGHUP` behavior to `ignore` and then executes the command.
  - <https://stackoverflow.com/questions/15595374/whats-the-difference-between-nohup-and-ampersand>
  - <https://www.quora.com/What-is-the-difference-between-running-a-process-with-nohup-screen-and-using-upstart>
- Use built-in bash job-control commands: `Ctrl+Z`, `bg`, `disown`, `jobs`, `fg`.
  - <https://www.shell-tips.com/linux/disown-a-running-shell-process-and-reattach-it-to-a-new-screen>

## References

- [LinkedIn Learning videos (free for UMN)](https://www.linkedin.com/learning/linux-multitasking-at-the-command-line/introduction-to-tmux?resume=false&u=42740356)
- <https://leanpub.com/the-tao-of-tmux/read>
- <https://readthedocs.org/projects/tmuxguide/downloads/pdf/latest/>
- <https://thoughtbot.com/blog/a-tmux-crash-course>
- <https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/>
- <https://thoughtbot.com/upcase/tmux>
- <https://phoenixnap.com/kb/tmux-tutorial-install-commands>
