---
layout: post
title: My GitHub Flow
date: 2021-10-01 12:00:00 +0200
img: github_auto.jpg # Add image post (optional)
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
tags: [InfoSec, Technical]
---

> GitHub, Inc. is a provider of Internet hosting for software development and version control using Git. It offers the distributed version control and source code management functionality of Git, plus its own features.

I've been using [GitHub](https://github.com/) for quite a while now with Visual Studio Code and the GitHub Desktop client for Windows. I wanted to create a setup for my Linux machine so that I had a semi-automated setup to push changes for my pentesting notes and scripts to GitHub to ensure that I always had an up-to-date backup of my files.

I didn't want to install a thick client and instead, I opted to figure out SSH-based user access and git commands via the commandline. This blog post describes my configuration settings and current workflow. 

### GitHub Access
In order for this to work, you first need a GitHub account. With an account created, I was able to create a SSH key in order to push and pull changes from both my public and private repositories without needing to input my password within a terminal. If you are unsure of SSH access, this [blog post](https://docs.github.com/en/authentication/connecting-to-github-with-ssh) describes key creation and usage for GitHub. 

Before creating a SSH key, I created a new directory which I would use as a parent folder for my GitHub repositories. With the new folder created, I navigated into the folder and created a new `.ssh` directory which I used to store my SSH keys. To create a new Key, I used the following command:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

You can decide where to save the file, I used the `.ssh` directory in the folder I created for ease of use. With a SSH key created, I needed to edit the SSH config file to point GitHub to my SSH key's directory. My SSH config file setup is provided below: 

```bash
kyhle@computername:~/Documents/Github$ cat ~/.ssh/config
Host github.com
  User git
  Hostname github.com
  IdentityFile /home/kyhle/Documents/Github/.ssh/id_ed25519
```

Now that the SSH keys are created, you can start cloning your repositories into the specified directory. In order to accomplish this, simply navigate to the repository using the GitHub web application and copy the SSH cloning command:

<p class="imgMiddle">
<img src="/assets/img/GitHub/cloning.png"  style="width: 80%" />
</p>

With the command copied to your clipboard, open a terminal in your directory and use the `git clone` command with the copied URL.

### Basic Usage

Now that you have access to the repository on your own workstation with your associated SSH keys, the next step to push changes to GitHub is to set your local git variables. If you have email privacy enabled (configured by default), Your personal email address will be used for account-related notifications as well as password resets. However, GitHub will provide you with a `@users.noreply.github.com` email address which you will need to use for Git operations, e.g. edits and merges.

To get your GitHub email address, you can navigate to the [Email address](https://github.com/settings/emails) settings page. Once you have the email address, you can configure your GitHub settings as follows using your terminal:

```bash
git config user.email "Your GitHub email address"
git config user.name "Your User Name"
```

If you want it to apply to all GitHub folders, you can add the `--global` argument to the aforementioned commands. Since it's my own repository and I am going to be using it to push new content, I don't need branches. I would suggest becoming familiar with the following commands if you intend to use this setup:

* `git add .` - Add all files to the new commit tree
* `git status` - list file changes
* `git pull` - If I added anything to the web app / if I use across multiple devices
* `git commit -am "Message Here"` - New commits for all files
* `git push` - Push new files to Master Branch

### Semi-Automation

With all the building blocks in place, I wanted to automate some of the commands to make my life a bit easier. In order to do this, I created a bash script which looks for changes within my notes directory and pushes any changes to my GitHub repository. The script is provided below:

```bash
#!/bin/bash
cd Pentesting-Notes
git add .
set +e  # Grep succeeds with nonzero exit codes to show results.
git status | grep modified
if [ $? -eq 0 ]
then
    set -e
    git commit -am "Updated on - $(date)"
    git push
else
    set -e
    echo "No changes since last run"
fi
```

With that created, I wanted to be able to call it from anywhere, so I also created an entry within `~/.bash_aliases`:

```bash
alias github='bash /home/kyhle/Documents/Github/Git_Auto.sh'
```

If you want to confirm that it works, you can refresh your source using: `source ~/.bashrc`. Now, every time that I update my notes and want to keep an updated copy backed up on GitHub, I simply type `github` into my console and it's done for me.

If you want to fully automate this process, you can include a cron job which runs on a predefined schedule, but I'm happy doing the updates manually at the end of each day. This will probably require a bit of modification the more I use it, but hopefully it helps someone that is looking for a basic flow with semi-automated backups.