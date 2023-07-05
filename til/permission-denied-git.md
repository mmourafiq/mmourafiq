---
layout: til
title: "Git : Permission denied (publickey)"
subtitle: "Permission denied (publickey). fatal: The remote end hung up unexpectedly"
cover_image: images/til/git.png
cover_image_caption: "Git."
tags: [ git ]
---

Sometimes it happens that you want to change your paraphrase for you public key, either because you
forget it or you want to enhance it. The thing is that you may encounter this problem :

`Permission denied (publickey). fatal: The remote end hung up unexpectedly`

After some googling, many answers were not yielding to any results, and because of that I am writing
this post to recapitulate the steps needed to solve this problem.

* Step 1:

Check for SSH keys `$ cd ~/.ssh` to see if there is a directory named `.ssh` in your user directory,
If it says “*No such file or directory*” skip to **Step 3**. Otherwise continue to **Step 2**.

* Step 2:

Backup and remove existing SSH keys.

```bash
$ mkdir key_backup

# Makes a subdirectory called "key_backup" in the current directory

cp id_rsa* key_backup

# Copies the id_rsa keypair into key_backup

rm id_rsa*

# Deletes the id_rsa keypair
```

* Step 3:

Generate a new SSH key

```bash
$ ssh-keygen -t rsa -C “your_email@youremail.com”
# Creates a new ssh key using the provided email

# Generating public/private rsa key pair.

# Enter file in which to save the key (/home/you/.ssh/id_rsa):

# Enter passphrase (empty for no passphrase): [Type a passphrase]

# Enter same passphrase again: [Type passphrase again] Your identification has been saved in /home/you/.ssh/id_rsa.

# Your public key has been saved in /home/you/.ssh/id_rsa.pub.

# The key fingerprint is:

# 01:0f:f4:3b:ca:85:d6:17:a1:7d:f0:68:9d:f0:a2:db your_email@youremail.com
```

* Step 4:

Add your SSH key to GitHub

```bash
$ sudo apt-get install xclip

# Downloads and installs xclip $ xclip -sel clip < ~/.ssh/id_rsa.pub

# Copies the contents of the id_rsa.pub file to your clipboard
```

Then, go to hithub, and do:

1. Go to your Account Settings
2. Click “SSH Keys” in the left sidebar
3. Click “Add SSH key”
4. Paste your key into the “Key” field
5. Click “Add key”
6. Confirm the action by entering your GitHub password

* Step 5:

Test everything out `$ ssh -T git@github.com # Attempts to ssh to github`

If ok, you’ll see Hi username! You’ve successfully authenticated, but GitHub does not # provide
shell access.

Otherwise (it happened with me), you will see Agent admitted failure to sign using the key. #
debug1: No more authentication methods to try. # Permission denied (publickey).

To solve this:

```bash
$ ssh-add # Enter passphrase for /home/you/.ssh/id_rsa:

# Identity added: /home/you/.ssh/id_rsa (/home/you/.ssh/id_rsa)
```

