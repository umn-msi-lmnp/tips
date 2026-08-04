# Software tips and tricks

This repo contains various scripts or "help" files that might be useful for setting up software configurations or basic usage examples. Let me know if you see any errors or have better ideas!

The local repo is located here:

    /project/standard/lmnp/tips



## MSI access and connection

Connecting to MSI resources can be pretty quick and easy. Click the link for my basic introduction and favorite connection tools.


[MSI access](access_msi/readme.md)


## OpenCode and local AI models

You can run OpenCode on MSI Agate while sending prompts to a local Ollama model running on your MacBook through an SSH reverse tunnel.

[OpenCode with local Ollama models](opencode_local_models/readme.md)


## OpenCode and UMN Gemini models

You can run OpenCode on MSI Agate with the UMN-provided Google Gemini models.

[OpenCode with UMN Gemini models](opencode_gemini/readme.md)


## VS Code on MSI

VS Code can be used as a local editor while connecting to MSI systems for remote development.

[VS Code HPC guide](vscode_guide/vscode-hpc-guide.md)




## Rclone

`rclone` is like `rsync` for transferring data between cloud platforms (e.g. panasas, ceph, Google Drive, etc.).


[Rclone example](rclone/readme.md)


## Terminal multiplexers (GNU Screen or tmux) 

I use *GNU Screen* or *tmux* to manage my ssh connections to MSI. This way, if my network connection breaks, or I need to bring my laptop to a meeting, my MSI interactive session can continue running in the background. I can even reconnect to that session. Screen is fairly simple to get running, but tmux is more powerful and worth the extra work setting up. 

* [GNU Screen example](gnu_screen/readme.md)  
* [tmux example](tmux/readme.md)


## R Markdown

I like to create reports using R Markdown syntax. These files can be rendered using R and other software on MSI systems. 

[R Markdown example](rmarkdown/readme.md)


## Python environments

Python environments help keep analysis software and package versions organized.

[Python environments example](python_envs/readme.md)


## Sharing data

These notes describe ways to share data with collaborators.

[Share data example](share_data/readme.md)





## SSH Keys

This is how I set up `ssh` keys to connect to MSI systems.

[ssh keys example](ssh_keys/readme.md)






## Viewing PDFs in X11 windows

I use a tool called `mupdf-x11` to view PDFs saved on MSI tier1 Panasas.

[mupdf example](mupdf/readme.md)



## How to find disk usage using Krona

I use a tool called `Krona` to examine disk usage in a fun way!

[Krona disk usage example](disk_usage/readme.md)



## How to get information from your scripts

I use `trap` statements and a few other commands at the beginning of my shell scripts for easy reporting or monitoring purposes.

[Script reporting example](script_reporting/readme.md)

