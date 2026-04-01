# How to use SSH Keys on MSI systems

_By Todd Knutson_

## Introduction

Connecting to remote systems with an `ssh` key (instead of a password) is more secure and makes connecting easier after setup. Everything in this document was adapted from various online sources. Thus, the content here is specific to how I set up my keys to work with MSI systems.

### What computer are you talking about?

When working on multiple computers/hosts, it can be confusing to know where these instructions are designed to be entered. In your terminal, typing `hostname -d` will reveal your current hostname (e.g. `local` for your laptop and `agate.msi.umn.edu` on the MSI HPC cluster). The following text will precede each command to indicate where you should enter the commands:

Below indicates the command prompt for your local laptop (e.g. running macOS):

```bash
(local)$
```

Below indicates the command prompt for an MSI HPC login node:

```bash
(agate.msi.umn.edu)$
```

## Connect to UMN network via VPN

To use MSI systems, you'll need to be on campus or connected to the UMN network using the VPN (split tunnel is fine). Below is a link to information about setting up a VPN connection from UMN OIT. The "AnyConnect" software method seems to work well.

- <https://it.umn.edu/service-details/virtual-private-network-vpn>

## Create a key-pair

This only needs to be done once -- alternatively, you can use an existing key-pair that you control. I use keys stored on my local Mac computer (in the `~/.ssh` directory). It is often helpful to keep separate keys for different services (for example one key for MSI and one key for GitHub) so access can be managed independently. In this guide, we use `id_ed25519_msi` and `id_ed25519_github`.

> Note: since these are custom filenames (not the default `id_ed25519`), you must provide the path to your key via the command line (`-i`) or your ssh config (`IdentityFile`).

- Create two new key-pairs with `ssh-keygen`:

  ```bash
  (local)$ ssh-keygen -t ed25519 -a 100 -f ~/.ssh/id_ed25519_msi
  (local)$ ssh-keygen -t ed25519 -a 100 -f ~/.ssh/id_ed25519_github
  ```

- Enter passphrase: Use a long passphrase (it can contain spaces and ideally would not use standard dictionary words). We'll set this up so you will NOT need to type the passphrase every time you ssh to a host. If you're using a Mac, we'll add your passphrase to your macOS-Keychain, so you will not need to type the passphrase after reboots either! You should still remember or save this passphrase in private place.

  ```bash
  (local)$ XXXXXXXXXXXXXXXXXXXXXXXXXXX
  ```

- Add a comment to the key (which will help you find it later). I just use the key name:

  ```bash
  (local)$ ssh-keygen -f ~/.ssh/id_ed25519_msi -o -c -C "id_ed25519_msi"
  (local)$ ssh-keygen -f ~/.ssh/id_ed25519_github -o -c -C "id_ed25519_github"
  ```

## Copy your public key to MSI login host

We need to add our public key to the `~/.ssh/authorized_keys` file that is located on the MSI server in your home `.ssh` directory. There are multiple ways to do this. Below are two options:

- Copy/paste the text from your public key on your local Mac to the `authorized_keys` file on the MSI host.

  ```bash
  # On your local Mac:
  # print the public key using cat function
  # highlight text
  # copy the entire line to your clipboard
  (local)$ cat ~/.ssh/id_ed25519_msi.pub
  
  # Log into MSI host
  (local)$ ssh USERNAME@agate.msi.umn.edu

  # Open the authorized keys file and edit
  (agate.msi.umn.edu)$ vim ~/.ssh/authorized_keys
  # Paste your clipboard text to a new line at the end of this file.
  # Press `i` to enter INSERT mode
  # CMD-v to paste the text into the file
  # Press ESC to exit INSERT mode
  # Press `:wq` to write and quit vim
  ```

- Or use the ssh copy tool. This approach will ask for your UMN password on the command line, and then use DUO. Running `ssh -s` will use SFTP mode, which avoids the fragile shell/stdin path and often fixes hangs.

  ```bash
  (local)$ ssh-copy-id -s -i ~/.ssh/id_ed25519_msi.pub USERNAME@agate.msi.umn.edu
  ```

## Optimize your `ssh` connection

### Your local Mac ssh config

You can specify additional `ssh` settings for your connection using an ssh config file. Most of these settings can also be used on the command line with flags.

Create an ssh config file on your Mac:

```bash
(local)$ vim ~/.ssh/config
```

Copy and paste the following parameters inside this file. The GitHub host block is recommended to avoid sending unnecessary X11/TTY options to GitHub. The MSI block ensures MSI hosts use your MSI key. Find your Mac-specific `xauth` path program by running `which xauth` -- then use that path in your config file. 

```ssh
Host *github*
    ForwardX11 no
    RequestTTY no
    IdentityFile ~/.ssh/id_ed25519_github

Host *.msi.umn.edu
    IdentityFile ~/.ssh/id_ed25519_msi

Host *
    GatewayPorts no
    StrictHostKeyChecking ask
    ForwardAgent yes
    ForwardX11 yes
    ForwardX11Trusted yes
    ServerAliveInterval 30
    ServerAliveCountMax 60
    NoHostAuthenticationForLocalhost yes
    XAuthLocation /usr/bin/xauth
    Compression yes
    TCPKeepAlive no
```

### Add your private key to your Mac's "Keychain" app

- This allows you to avoid entering your ssh key passphrase every time you connect (including after reboots).

  ```bash
  ssh-add -K ~/.ssh/id_ed25519_msi
  ssh-add -K ~/.ssh/id_ed25519_github
  ssh-add -A
  ```

- You can do this automatically every time you reboot by entering the following to your Mac's `~/.bashrc` file:

  ```bash
  # ---------------------------------------------------------------------
  # Start the ssh agent
  # ---------------------------------------------------------------------

  # Make sure we are attached to a tty
  if /usr/bin/tty > /dev/null
  then
      # Check the output of "ssh-add -l" for identities
      ssh-add -l | grep 'no identities' > /dev/null
      if [ $? -eq 0 ]
      then
          # Load your identities.
          echo "Adding IDs to ssh-agent"
          ssh-add -K ~/.ssh/id_ed25519_msi
          ssh-add -K ~/.ssh/id_ed25519_github
          ssh-add -A
      else
          # echo "Not adding keys to ssh-agent"
          :
      fi
  fi
  ```

### Your MSI ssh config

Keep your MSI-side `~/.ssh/config` minimal: you usually do not need to specify any `IdentityFile` entries on MSI because your local `ssh-agent` is forwarded after you connect.

```sshconfig
Host *github*
    ForwardX11 no
    RequestTTY no

Host *
    ForwardAgent yes
    ForwardX11 yes
    ForwardX11Trusted yes
    ServerAliveInterval 30
    ServerAliveCountMax 60
```

## Add your GitHUb key to github.com

In order to connect to github.com, you need to add your public key to the website. 

- Visit [https://github.com/settings/keys](https://github.com/settings/keys)
- Click the button to add a New SSH key
- Provide the name of the key (e.g. id_ed25519_github)
- Key type should be: Authentication Key
- Copy your public key to the web form and click Add SSH Key

To find your public key:

```bash
(local)$ cat ~/.ssh/id_ed25519_github.pub
```

## Test the connection using your key

- If you specified your ssh private key (IdentityFile) in your ssh config file, you do not need to specify it again on the command line. In addition, since we loaded your private key (IdentityFile) into the `ssh-agent` using `ssh-add`, _you should not need to specify your private key on the command line or type your passphrase._

  ```bash
  (local)$ ssh USERNAME@agate.msi.umn.edu
  ```

- Hop to other MSI nodes. Since we are using the `ssh-agent` to forward your key, you should not need to enter your passphrase when connecting to other nodes at MSI.

  ```bash
  (agate.msi.umn.edu)$ ssh agate
  ```

- Test your GitHub SSH authentication from your local machine.

  ```bash
  (local)$ ssh -T git@github.com
  ```

- Test your GitHub SSH authentication from Agate (using forwarded agent).

  ```bash
  (agate.msi.umn.edu)$ ssh -T git@github.com
  ```

## References

- [MSI - How to set up ssh keys](https://www.msi.umn.edu/support/faq/how-do-i-setup-ssh-keys)
- [Lean Crew Blog](https://leancrew.com/all-this/2017/02/ssh-keys/)
