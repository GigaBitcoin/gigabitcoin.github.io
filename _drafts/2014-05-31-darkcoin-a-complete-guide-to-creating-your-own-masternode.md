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

To start off, you need to buy a server to host your Masternode on (if you don’t own one already). This guide will be for Linux based servers (I’m running Ubuntu x32). I shamelessly recommend [Digital Ocean](https://www.digitalocean.com/?refcode=a4ad65c140a2). I'm using the single CPU with 512MB of RAM, a 20GB SSD, and one TB of transfer for $5/month!!! Send me a DM on twitter and I'll send you a $10 promo code. /salespitch

Configure Your Server’s Security
================================

Before creating the server Droplet, you need to generate a SSH key pair and add the public key from your Digital Ocean dashboard.

First, we need to check for existing SSH keys on your computer. Open terminal and type:

``` bash
# Lists the files in your .ssh directory
cd ~/.ssh
ls -al
```

Check the directory listing to see if you have a file named `id_rsa.pub`. If the file `id_rsa.pub` does not exist, you'll need to generate a new key pair by inputting:

``` bash
# Creates a new ssh key, using the provided email as a label
ssh-keygen -t rsa -C "your_email@email.com"
```

You're asked to enter a file for saving the key to. Simply press enter because defaults are preferred. Next, you'll be asked to enter a passphrase. Use a strong, unique password generated from __[KeePass](http://keepass.info/)__ or __[1Password](https://agilebits.com/onepassword)__ and store it and the generated key pair for safe keeping. After your identification has been saved to `id_rsa.pub`, type:

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

Now click green `CREATE` button from the side bar. Enter a `HOSTNAME`, defaults for `Select Size` and `Select Region`, Ubuntu 14.04 x32 for `Select Image`, select your SSH key(s) for `Add optional SSH Keys`, defaults for `Settings`, and click `Create Droplet`.

![Create Droplet](/_assets/createDroplet000.png)

Now that your server has been created, copy the server IP address from the `Droplets` side bar and SSH with terminal via:

``` bash
ssh root@YOUR.SERVER.IP.ADDRESS
```

Update/upgrade the software first.

``` bash
apt-get update
apt-get upgrade
```

Install [Fail2ban](http://www.fail2ban.org/wiki/index.php/Main_Page) which is a daemon that monitors login attempts to a server and blocks suspicious activity as it occurs.

``` bash
apt-get install fail2ban
```

Now, let’s set up your login user:

``` bash
useradd yourLoginUser
mkdir /home/yourLoginUser
mkdir /home/yourLoginUser/.ssh
chmod 700 /home/yourLoginUser/.ssh
```

Setup SSH for your login user:

``` bash
vim /home/yourLoginUser/.ssh/authorized_keys
```

Paste the contents of the `id_rsa.pub`. Press `ESCAPE`, type `:wq`, and then press `ENTER`. Next, you'll set a sudo password for your login user.

``` bash
passwd yourLoginUser
```

Store this password for late use. Now you'll configure the sudo file:

``` bash
visudo
```

Find the line:

``` bash
root    ALL=(ALL:ALL) ALL
```

Add the line:

``` bash
yourLoginUser    ALL=(ALL:ALL) ALL
```

Find the following lines:

``` bash
# Members of the admin group may gain root privileges
%admin ALL=(ALL) ALL

# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL
```

And change them to:

``` bash
# Members of the admin group may gain root privileges
# %admin ALL=(ALL) ALL

# Allow members of group sudo to execute any command
# %sudo   ALL=(ALL:ALL) ALL
```

Press `Control + X`, type `Y`, and hit `Enter`. Now you'll Configure SSH to prevent password logins and disable `root` with:

``` bash
vim /etc/ssh/sshd_config
```

Scroll down and look for the line:

``` bash
# Change to no to disable tunnelled clear text passwords
# PasswordAuthentication yes
```

Change it to look like:

``` bash
# Change to no to disable tunnelled clear text passwords
PasswordAuthentication no
```

Next find the line:

``` bash
PermitRootLogin yes
```

And change it to:

``` bash
PermitRootLogin no
```

Now hit `ESCAPE`, type `:wq`, and then press `ENTER`. After a server restart, `root` will be disabled and you'll need to SSH with `yourLoginUser`. No secure server is complete without a firewall. Ubuntu provides ufw, which makes firewall management easy. Run:

``` bash
# Configures the server firewall to accept traffic over port 22 for SSH and port 9999 for Darkcoind
ufw allow 22
ufw allow 9999
ufw enable
```

Enable automatic security updates which scare me a bit, but not as badly as unpatched security holes.

``` bash
apt-get install unattended-upgrades

vim /etc/apt/apt.conf.d/10periodic
```

Update the file to look like this:

``` bash
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::AutocleanInterval "7";
APT::Periodic::Unattended-Upgrade "1";
```

Then hit `ESCAPE`, type `:wq`, and then press `ENTER` to save. Next config file to edit:

``` bash
vim /etc/apt/apt.conf.d/50unattended-upgrades
```

Update the file to look like below. You should probably keep updates disabled and stick with security updates only:

``` bash
Unattended-Upgrade::Allowed-Origins {
        "${distro_id}:${distro_codename}-security";
//      "${distro_id}:${distro_codename}-updates";
//      "${distro_id}:${distro_codename}-proposed";
//      "${distro_id}:${distro_codename}-backports";
};
```

Again, hit `ESCAPE`, type `:wq`, and then press `ENTER` to save. Now shutdown your server and create an image for easy restoration or creating more Masternodes in the future.

``` bash
sudo poweroff
```

Goto the Digital Ocean dashboard and click `Droplets` from the side bar and select your Masternode droplet. Then select `Snapshots`, enter a snapshot name, and click `Take Snapshot`.

![Security Snapshot](/_assets/securitySnapshot000.png)

[Recommended Guide](https://darkcointalk.org/threads/how-to-set-up-ec2-t1-micro-ubuntu-for-masternode-part-1-3.240/)

[Darkcoin guides](https://darkcointalk.org/forums/guides.32/)

[Darkcoins on local machine](https://bitcointalk.org/index.php?topic=421615.msg6427569#msg6427569)
