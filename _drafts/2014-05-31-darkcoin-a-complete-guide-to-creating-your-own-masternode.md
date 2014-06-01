---
layout: post
title: Darkcoin: A Complete Guide to Creating Your Own Masternode
---

![Darkcoin Logo](/_assets/darkcoinLogo001.png)

What's Darkcoin?
================

via [Darkcoin Whitepaper](http://www.darkcoin.io/downloads/DarkcoinWhitepaper.pdf)

> Darkcoin is the first privacy centric cryptographic currency based on Satoshi Nakamoto’s Bitcoin. DarkSend, a technology for sending anonymous block transactions is incorporated directly into the client using extensions to the core protocol. An improved proof­of­work using a chain of hashing algorithms replaces the SHA256 algorithm and will result in a slower encroachment of more advanced mining technologies (such as ASIC devices). DarkGravityWave is implemented to provide quick response to large mining power fluctuations.

What's a Masternode?
====================

via [wiki.darkcoin.eu](http://wiki.darkcoin.eu/wiki/FAQ#What_are_Master_Nodes.3F)

> Masternodes are network nodes, owned by the users of Darkcoin network. They perform the coin mixing service of DarkSend which anonymizes transactions. __Users get rewarded with 20% of the mining output per block (without actually performing any mining themselves)__ through a "voting mechanism".

Initial Setup
=============

To start off, you need to buy a server to host your Masternode on (if you don’t own one already). This guide will be for Linux based servers (I’m running Ubuntu x32). I shamelessly recommend [Digital Ocean](https://www.digitalocean.com/?refcode=a4ad65c140a2). I'm using the single CPU with 512MB of RAM, a 20GB SSD, and one TB of transfer for $5/month!!! /salespitch

Configure Your Server’s Security
================================

Before creating the server Droplet, you need to generate a SSH key pair and add the public key from your Digital Ocean dashboard.

First, we need to check for existing SSH keys on your computer. Open terminal and type:

``` bash
# Lists the files in your .ssh directory
cd ~/.ssh
ls -al
```

Check the directory listing to see if you have a file named `id_rsa.pub`.

If the file `id_rsa.pub` does not exist, you'll need to generate a new key pair by inputting:

``` bash
# Creates a new ssh key, using the provided email as a label
ssh-keygen -t rsa -C "your_email@email.com"
```

You're asked to enter a file for saving the key to. Simply press enter because defaults are preferred. Next, you'll be asked to enter a passphrase. Use a strong, unique password generated from __[KeePass](http://keepass.info/)__ or __[1Password](https://agilebits.com/onepassword)__ and store it for safe keeping. After your identification has been saved to `id_rsa.pub`, type:

``` bash
# Add your new key to the ssh-agent
ssh-add ~/.ssh/id_rsa
```

Followed by:

``` bash
# Copies the contents of the id_rsa.pub file to your clipboard
clip < ~/.ssh/id_rsa.pub
```

Goto your Digital Ocean dashboard, click the side bar link that says `SSH Keys`

![SSH Keys](/_assets/sshKeys000.png)

Click `Add SSH Key`, enter a name for the key, paste the key from your clipboard, and click `CREATE SSH KEY`.

![SSH Keys](/_assets/sshKeys001.png)


[Recommended Guide](https://darkcointalk.org/threads/how-to-set-up-ec2-t1-micro-ubuntu-for-masternode-part-1-3.240/)

[Darkcoin guides](https://darkcointalk.org/forums/guides.32/)

[Darkcoins on local machine](https://bitcointalk.org/index.php?topic=421615.msg6427569#msg6427569)
