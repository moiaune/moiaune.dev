---
title: 'Linux Basics'
description: 'A basic introduction to Linux'
date: '2024-06-03'
tags: ['linux', 'cli']
toc: true
url: basic-linux
---

This is a very brief introduction to working with Linux on the command line. It's a collection of commands and tips&tricks that I have collected through out my Linux journey. Some of it spesific to Debian derivatives. Maybe you find something usefull too. I will probably update this from time to time. You can look at the history if you want to see whats changed.

<!--more-->

## Find OS information

Print information about your Linux distro.
```console
lsb_release -a
cat /etc/os-release
uname -a
```

See how long your machine has been running since last boot.

```console
moiaune@lnx:~$ uptime
 11:36:40 up  4:02,  1 user,  load average: 0.00, 0.00, 0.00
```

## Built-in documentation with manpages

In Linux we can use the **man** command to read built-in documentation.

```plaintext
man COMMAND
```

To list all available manpages.

```console
man -k .
```

You can combine it with grep to filter the result if you are not quite sure what the manpage entry is called.

```console
moiaune@lnx:~$ man -k . | grep ssh
ssh (1)              - OpenSSH remote login client
ssh-add (1)          - adds private key identities to the OpenSSH authentication agent
ssh-agent (1)        - OpenSSH authentication agent
ssh-argv0 (1)        - replaces the old ssh command-name as hostname handling
ssh-copy-id (1)      - use locally available keys to authorise logins on a remote machine
ssh-keygen (1)       - OpenSSH authentication key utility
ssh-keyscan (1)      - gather SSH public keys from servers
ssh-keysign (8)      - OpenSSH helper for host-based authentication
ssh-pkcs11-helper (8) - OpenSSH helper for PKCS#11 support
ssh-sk-helper (8)    - OpenSSH helper for FIDO authenticator support
ssh_config (5)       - OpenSSH client configuration file
sshd (8)             - OpenSSH daemon
sshd_config (5)      - OpenSSH daemon configuration file
~:$ man sshd
```

## Manage files and directories

### List files and directories

```console
# list name of files and directories under /var
ls /var

# show permissions, owners, size, modification date etc for each file and directory in /var
ls -l /var

# same as above, but also recursivly
ls -lR /var

# '-h' lists sizes in human readble format, '-F' will append a '/' to all directories
ls -lhF /var

# '-a' will also list hidden files and directories (that starts with '.')
ls -la $HOME
```

### Finding files

```console
# list all files and directories in /var
find /var

# list all files and directories in /var in 'ls' style
find /var -ls

# find all files that ends with '.log' in /var and execute 'type' on each file
find /var -name "*.log" -exec type {} \;
```

### Create files

```console
touch file.txt

vim file.txt

nano file.txt

echo "Some content" > file.txt

cat file.txt > another_file.txt

ls -lR /var > dir_list.txt
```

Write directly from **stdin** to file (also work with append).

```console
moiaune@lnx:~$ cat > file.txt
Write some
lines
of text
<CTRL-D>
moiaune@lnx:~$ cat file.txt
Write some
lines
of text
```


### Search content of files

We can use **grep** to search for spesific phrases/words or patterns in a file or files.

```console
grep 'Failed' /var/log/auth.log
```

### Search logfiles

Debian derivatives now use **journalctl** to display logs. Here are some basic filtering techniques.

- '--since': Filter on time. Can use 'yesterday', 'today' or a datetime '2024-07-09', '2024-07-09 18:00:00'
- '--grep': Filter using grep on the `MESSAGE=` field
- '--unit': Filter on the service, e.g ssh, nginx, apache2 etc

```console
journalctl --since yesterday --grep 'failed' --unit ssh
```

### Archiving files and directories

In Linux we use the **tar** command to create an archive. In most cases we also want to compress it with zip.

- 'c': Create an archive
- 'v': Verbose
- 'z': Compress the archive with zip
- 'f': Specify output file

```console
# backup my home directory to mybackup.tar.gz
tar -cvzf mybackup.tar.gz /home/moiaune
```

If we omit the 'z' flag, tar would create an archive with the exact same size as my homefolder.

We can list the contents of a tar archive (or tarball).

```console
tar -tvf mybackup.tar.gz
```

To decompress or unpack an archive we also use **tar**.

- 'x': Extract an archive
- 'C': Use another directory than current working directory

```console
# extract to current working directory
tar -xf mybackup.tar.gz

# or to a specific directory
tar -xf mybackup.tar.gz -C /tmp/backup
```

### Transfer files

We can transfer files to and from other computers/servers with **scp** or **rsync**.

NOTE: If we have the same username on both ends, we dont need to specify the USER.

Using **scp**.

```plaintext
scp [src] [dest]

# from local to remote
scp [local-path] [USER@]HOST:[remote-path]

# from remote to local
scp [USER@]HOST:[remote-path] [local-path]
```

Using **rsync**.

```plaintext
rsync -aP [src] [dest]

# from local to remote
rsync -aP [local-path] [USER@]HOST:[remote-path]

# from remote to local
rsync -aP [USER@]HOST:[remote-path] [local-path]

# optionally we can specify -n to do a dry-run to see what would happen
rsync -naP [local-path] [USER@]HOST:[remote-path]
```

Real world examples.

```console
# copy my local '.bashrc' to a lab server
scp /home/moiaune/.bashrc lab_user@lab1.example.com:/home/lab_user/.bashrc

# sync my blog folder to webserver
rsync -aP /home/moiaune/code/blog web.moiaune.dev:/var/www
```

### Change permissions, owners and groups

Which files and directories can be accessed, modified etc by whom is determined by permissions and owners.

We can see the permissions and owners of a file/directory with **ls**.

```console
moiaune@lnx:~$ ls -la /home/cloud_user
total 20
drwxr-xr-x 3 cloud_user cloud_user 4096 Jul  9 08:25 .
drwxr-xr-x 4 root       root       4096 Aug 24  2023 ..
-rw-r--r-- 1 cloud_user cloud_user    1 Feb 29 19:38 .bash_history
-rw-r--r-- 1 cloud_user cloud_user    0 Feb 29 19:16 .cloud-locale-test.skip
-rw------- 1 cloud_user cloud_user   57 Jul  9 08:17 .lesshst
drwx------ 2 cloud_user cloud_user 4096 Aug 24  2023 .ssh
-rw-r--r-- 1 cloud_user cloud_user    0 Aug 28  2023 .sudo_as_admin_successful
```
The first character idicates what kind of item it is:

- '-': file
- 'd': directory
- 'l': symbolic link

Next we have the permissions divided into three groups. The first three are the owners permissions, then we have the groups permissions and lastly others permissions.

- '-': none
- 'r': read
- 'w': write
- 'x': execute

The owner of the file is specified in the first column that you see `cloud_user`, and the group is specified in the column next to it. In this example we can see that all files and directories is owned by the `cloud_user` user and `cloud_user` group execpt for the parent directory which is owned by `root` user and `root` group.

#### Change owners and groups

We can change the owner and group with the **chown** command.

```plaintext
chown [-R] [OWNER][:GROUP] FILE

# only change owner
chown OWNER FILE

# only change group
chown :GROUP FILE

# change both, but not the same
chown OWNER:GROUP FILE

# change both to the same
chown OWNER: FILE
```

- 'R': Recursive (will also change owner for all subfiles and directories)

```console
moiaune@lnx:~$ cd /home/cloud_user
moiaune@lnx:~$ touch file.txt
moiaune@lnx:~$ ls -l file.txt
-rw-r--r-- 1 cloud_user cloud_user 0 Jul  9 08:37 file.txt
moiaune@lnx:~$ chown another_user:another_user file.txt
moiaune@lnx:~$ ls -l file.txt
-rw-r--r-- 1 another_user another_user 0 Jul  9 08:37 file.txt
moiaune@lnx:~$ chown cloud_user file.txt
-rw-r--r-- 1 cloud_user another_user 0 Jul  9 08:37 file.txt
moiaune@lnx:~$ chown :cloud_user file.txt
-rw-r--r-- 1 cloud_user cloud_user 0 Jul  9 08:37 file.txt
```

#### Change permissions

To set permissions we use the **chmod** command.

There are two modes to set permissions with; symbolic and octal.

##### Octal mode

```plaintext
chmod OCTAL FILE
```

- read(r): 4
- write(w): 2
- execute(x): 1

To calculate the permissions bits we just need to add the permissions together. So for read and write access it will be `4+2=6`. For all permissions it will be `4+2+1=7`.

Let's say we have `file.txt` and we want the owner to have full permissions, the group to have read and write and others to have none. This would calculate to `760`. We can set these permissions with the following command.

```console
moiaune@lnx:~$ chmod 760 file.txt
moiaune@lnx:~$ ls -l file.txt
-rwxrw---- 1 cloud_user cloud_user 0 Jul  9 08:37 file.txt
```

These are the most common permissions used:

- 777 (everyone has full permissions)
- 644 (owner has read+write, and everyone else has read)
- 750 (owner has full permissions, groups has read+execute, and everyone else has none)
- 600 (owner is the only user that has access)

##### symbolic mode

```plaintext
chmod MODE FILE
```

- '+': add permission
- '-': remove permission
- '=': set permisions
- 'u': owner (user)
- 'g': group
- 'o': others

```console
# add read permissions for the owner
chmod u+r file.txt

# remove execute for owner
chmod u-x file.txt

# set owners permissions to read, write, execute
chmod u=rwx file.txt

```

We can combine permissions with `,`.

```console
moiaune@lnx:~$ ls -l file.txt
-rwxrw---- 1 cloud_user cloud_user 0 Jul  9 08:37 file.txt
moiaune@lnx:~$ chmod u=rwx,g=rw,o=r file.txt
moiaune@lnx:~$ ls -l file.txt
-rwxrw-r-- 1 cloud_user cloud_user 0 Jul  9 08:37 file.txt
```

## Input/output redirection

In Linux shells there is a concept of three streams.

- The standard input (stdin), which takes the users input.
- The standard out (stdout), which is the output of a command. It is usually displayed in your terminal.
- The standard error (stderr), which is the error messages from a command. This is also usually displayed in your terminal alongside stdout.

Write errors to file instead of console.

```console
# no redirection
moiaune@lnx:~$ ls -l nonexisting
ls: nonexisting: No such file or directory

# with redirection of stderr
moiaune@lnx:~$ ls -l nonexisting 2> ls_error.txt
moiaune@lnx:~$ cat ls_error.txt
ls: nonexisting: No such file or directory
```

Redirect **stdout** to a file and only show errors in the console.

```console
# these two are equivalent
find /root 1> dirs_i_can_read.txt
find /root > dirs_i_can_read.txt
```

Redirect both **stdout** and **stderr** to the same destination.

```console
find /root 2>&1 root_dirs.txt
```

This tells Linux to redirect **stderr** to **stdout** and then **stdout** to a file.


## Reboot and PowerOff

```plaintext
shutdown [OPTIONS] [TIME] [WALL...]
```

```console
# poweroff
poweroff
shutdown -P
shutdown -P 20:00
shutdown -P +5
shutdown -P +5

# reboot
reboot
shutdown -r [TIME]
shutdown -r 20:00
shutdown -r +5 'Server will be rebooted for maintenance!'
```

## Manage users and groups

```console
# Add a new user
adduser LOGIN

# add user to group
gpasswd -a USER GROUP

# give sudo access
cat /etc/sudoers
gpasswd -a USER sudo
```

## Disk space

List filesystem space usage.

```console
moiaune@lnx:~$ df -h
Filesystem       Size  Used Avail Use% Mounted on
udev             467M     0  467M   0% /dev
tmpfs             96M  484K   95M   1% /run
/dev/nvme0n1p1    20G  2.1G   17G  11% /
tmpfs            477M     0  477M   0% /dev/shm
tmpfs            5.0M     0  5.0M   0% /run/lock
/dev/nvme0n1p15  124M   12M  113M  10% /boot/efi
tmpfs             96M     0   96M   0% /run/user/1001
tmpfs             96M     0   96M   0% /run/user/1002
```

List directory space usage.

```console
moiaune@lnx:~$ du -hs /home/cloud_user
36K     /home/cloud_user
moiaune@lnx:~$ sudo du -hs /var/log
55M     /var/log
```

If we omit the `-s` we will also see subdirectories.

```console
moiaune@lnx:~$ sudo du -h /var/log
4.0K    /var/log/runit/ssh
8.0K    /var/log/runit
52K     /var/log/apt
4.0K    /var/log/private
53M     /var/log/journal/ec228e8f22cbefcdced18ace3b891949
53M     /var/log/journal
16K     /var/log/unattended-upgrades
32K     /var/log/amazon/ssm/audits
852K    /var/log/amazon/ssm
856K    /var/log/amazon
55M     /var/log
```

You can also sort the result to find what takes the most space. Since we are using the `-h` flag on **du** to get human readable sizes, we also provide the same flag to **sort** so that I will sort correctly.

```console
moiaune@lnx:~$ sudo du -h /var/log | sort -h
4.0K    /var/log/chrony
4.0K    /var/log/private
4.0K    /var/log/runit/ssh
8.0K    /var/log/runit
36K     /var/log/apt
136K    /var/log/nginx
440K    /var/log/sysstat
25M     /var/log
25M     /var/log/journal
25M     /var/log/journal/b8d53a40a48e4a9baaf67b1d19735980
```

To get the size of a file(s), you can use the **ls** command with `-lh` to get the size in human readable format.

Another alternative is **ncdu** which is an interactive version of `du` built with ncurses. It does not come pre-installed.

```console
sudo apt install ncdu

# replace /home with whatevery starting path you want
ncdu /home
```

Now you can navigate and see the size of files and directories interactivly.

## Processes

Dashboard overview of your processes, CPU and memory usage.
```console
htop
```

Snapshot of the current process state.
```console
ps aux
```

Kill a process.

```console
pkill -9 PID
```

## Schedule tasks

In Linux we use **cron** with a `crontab` file to scedule runs of commands or scripts.

The crontab template looks like this.

```plaintext
# +----------- minute (0 - 59)
# | +--------- hour (0 - 23)
# | | +------- day of month (1 - 31)
# | | | +----- month (1 - 12)
# | | | | +--- day of week (0 - 6) (starts at sunday)
# | | | | |
# * * * * * COMMAND
#
# NOTES
# The 'month' and 'day of week' can be represented by either a number, name or shortname.
# e.g 1, January, Jan
# e.g 1, Monday, Mon
#
# Command can be either a script or a command. You can seperate them with ';' to run
# multiple commands.
#
# Visit https://crontab.guru for a visual representation of cron schedule
```

To edit your crontab you use the **crontab** command.

```console
crontab -e
```

You can also specify the user to edit crontab for if you want it to run as another user.

```console
crontab -u USER -e
```

> NOTE: When adding commands to your crontab, make sure you use full paths for both commands and scripts.

You can check the cron logs using **journalctl**.

```console
journalctl --unit cron
```

## Manage and update packages

### dpkg

```console
# install .deb package from file
dpkg -i FILE.deb

# list installed packages
dpkg -l

# remove package
dpkg -r NAME

# search installed packages
dpkg -S PATTERN
```

NOTE: When intalling packages with `dpkg -i` it does not perform any dependency checks. If a dependency is missing it will fail.

### apt

```console
# update package list
apt update

# upgrade installed packages
apt upgrade

# install package(s)
apt install NAME

# remove package
apt remove NAME

# remove package and config
apt purge NAME

# remove unwanted packages
apt autoremove

# search packages
apt search PATTERN

# show package details
apt show NAME

# list installed packages
apt list --installed

# list packages that can be upgraded
apt list --upgradeable
```
