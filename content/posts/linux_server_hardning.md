---
title: 'Debian 12 Server Hardening'
description: 'Basic hardning of a new Debian 12 server'
date: '2024-07-10T13:03:56+02:00'
tags: ['debian', 'security', 'linux']
toc: true
url: linux-server-hardning
---

We take a look at some of the initial configuration you should do when spinning up a new Debian 12 server that is connected to the internet to harden it against common attacks. The list of actions is in no way exhaustive. Depending on what you are hosting there are further actions to take.

<!--more-->

## 1. Update and upgrade system packages

Make sure that your installed packages are up-to-date to reduce the risk of having vulnerable packages installed.

```console
apt update && apt upgrade -y
```

## 2. Create a dedicated non-root user

When you spin up a server you are provided with a root user that has access to everything. It's good practice to not use this for everyday tasks. Instead you create your own user and give it necessary permissions.

```console
useradd --create-home --shell /usr/bin/bash --groups sudo <user_name>
```

## 3. Setup SSH Keys

By default your are logging into SSH with passwords. It's recommended to use ssh keys instead. An attacker would then need to steel your private key in stead of just guessing a password. First switch to your newly created user `su <user_name>`.

```console
mkdir -p ~/.ssh
touch ~/.ssh/authorized_keys
chmod -R 0700 ~/.ssh
chmod 0600 ~/.ssh/authorized_keys
```

On your local machine (that you use to connect to the server with) you must generate a ssh keypair.

```console
ssh-keygen -b 521 -t ecdsa -f ~/.ssh/my_debian
```

This will create a private and public key in your `~/.ssh` folder with the name you specified.

```console
claw0ry@lnx:~$ ls ~/.ssh
my_debian           my_debian.pub
```

Next you must copy the contents of `my_debian.pub` and paste it into the `.ssh/authorized_keys` file on the server.


## 4. Disable SSH password auth and root login

Now that we have setup our ssh keys, we can go ahead and disable password login. We also want to disable direct login to `root`. If we need root we can login with our dedicated user and then become root, since we added ourselves to the `sudo` group.

```console
sudo vim /etc/ssh/sshd_config
```

Find the line where it says `PermitRootLogin` and change this to `no`. Next find `PasswordAuthentication` and also change this to `no`.

Restart **sshd**.

```console
sudo systemctl restart sshd
```

If everything was setup correctly you should be able to logout of the server and log back in without a password (only using your ssh key). You can also test it by moving your ssh keys out of the `~/.ssh` folder and try to connect to the server. It should tell you that it only accepts keypair.

## 5. Setup firewall with UFW

**ufw** stands for "uncomplicated firewall" and its pretty darn true.

**NOTE:** Make sure you allow ssh before enabling and starting ufw. Otherwise you might look yourself out for good!

```console
sudo apt install ufw
sudo ufw allow ssh
sudo ufw enable
sudo systemctl start ufw
```

You can see which rules are in play with `sudo ufw status`.

## 6. Setup fail2ban for SSH auth

If you have every deployed a server with internet access and looked at the logs you know that it will get hammered with suspicious login attemps. We can use fail2ban in conjuction with ufw to block such attempts. After x failed login attempts fail2ban will put the IP address in a timeout blocking list.

```console
sudo apt install fail2ban
```

Originally ssh authentication logs were stored in `/var/logs/auth.log`, but in Debian 12 these are now collected under the journalctl system. By default fail2ban will look for `/var/logs/auth.log`, so we need to tell it to use journalctl(systemctl) in stead.

```console
touch /etc/fail2ban/paths-debian.local
echo "[DEFAULT]" > /etc/fail2ban/paths-debian.local
echo "sshd_backend = systemd" >> /etc/fail2ban/paths-debian.local
```

After we edited the configuration we must restart fail2ban.

```console
sudo systemctl restart fail2ban
```

Here are some other usefull fail2ban commands.

```console
# see the overall status of jail <sshd>
sudo fail2ban-client status sshd

# get a list of currently banned ip
sudo fail2ban-client banned

# get a list of currently banned ip's for <sshd> jail
sudo fail2ban-client get sshd banip

# see ssh auth logs
sudo journalctl -u ssh
```
## 7. Get a list of connections and port on your system

You can see which ports and connections that are in use. On a fresh server install with sshd, this is a fairly normal state.

```console
claw0ry@localhost:~$ sudo ss -tulpn
Netid State  Recv-Q Send-Q Local Address:Port  Peer Address:Port Process
udp   UNCONN 0      0          127.0.0.1:323        0.0.0.0:*     users:(("chronyd",pid=568,fd=5))
udp   UNCONN 0      0              [::1]:323           [::]:*     users:(("chronyd",pid=568,fd=6))
tcp   LISTEN 0      128          0.0.0.0:22         0.0.0.0:*     users:(("sshd",pid=613,fd=3))
tcp   LISTEN 0      128             [::]:22            [::]:*     users:(("sshd",pid=613,fd=4))
```

- `-t`: services listening on TCP
- `-u`: services listening on UDP
- `-l`: services listening
- `-p`: listening process
- `-n`: use portnumber in stead of name
