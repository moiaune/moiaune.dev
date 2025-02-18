---
title: 'Make git work with multiple accounts'
date: 2022-08-11T13:38:35+02:00
draft: false
---

In this article we're going to look at how you can setup git to work with multiple Github accounts and SSH.

<!--more-->

### Generate SSH keys

Github only allows you to use the same SSH key for one account, therefor if you have multiple accounts (e.g personal and work) you must generate two different SSH key pairs.

```bash
ssh-keygen -f ~/.ssh/gh_personal -t rsa -b 4096
ssh-keygen -f ~/.ssh/gh_work -t rsa -b 4096
```

### Folderstructure

My personal preference is to use dedicated folder pr Github account. So my folderstructure looks something like this:

```
~/code
└── github.com
    ├── personal
    └── work
```

Based on this, we can put a `.gitconfig` in each of folders, so it becomes this:

```
~/code
└── github.com
    ├── personal
    │   └── .gitconfig
    └── work
        └── .gitconfig
```

We're going to edit these files soon to add account specific configurations.

### Conditional configuration includes

git version 2.13 and newer supports [conditional includes](https://git-scm.com/docs/git-config#_includes), which means that we can include different `.gitconfigs` based on a condition.

We're going to use the `gitdir` keyword to include a specific `.gitconfig` based on where our git project is located.

In our main `.gitconfig` (usually under `~/.gitconfig`) we need to add these conditionals at the top.

```
[includeIf "gitdir:~/code/github.com/personal/"]
    path = ~/code/github.com/personal/.gitconfig
[includeIf "gitdir:~/code/github.com/work/"]
    path = ~/code/github.com/work/.gitconfig

...
```

**NOTE:** The trailing slash of folder path is neccessary or else it wont work

### Tell git which ssh-key to use

Now that we have set our conditionals we can edit each of the `.gitconfig` files to tell git which ssh-key to use, and set other account specific configs like name and email.

*~/code/github.com/personal/.gitconfig*

```
[user]
    name = "John Doe"
    email = "johndoe@example.com"
[core]
    sshCommand = "ssh -i ~/.ssh/gh_personal
```

*~/code/github.com/work/.gitconfig*

```
[user]
    name = "John Doe"
    email = "johndoe@company.com"
[core]
    sshCommand = "ssh -i ~/.ssh/gh_work
```

Now whenever you interact with git in a project that is located under `~/code/github.com/work` it will use the `.gitconfig` for work and associated ssh key, and vice-versa for your personal projects.

### Additional providers

This setup is not specific to Github, so if you for example also have a Bitbucket (or any other git provider that supports ssh) account which uses a different SSH key you would use the same technique.

```
ssh-keygen -f ~/.ssh/bitbucket -t rsa -b 4096
```

```
# file: ~/.gitconfig

...

[includeIf "gitdir:~/code/bitbucket/"]
    path = ~/code/bitbucket/.gitconfig

...
```

```
# file: ~/code/bitbucket/.gitconfig

[user]
    name = "John Doe"
    email = "johndoe@company.com"
[core]
    sshCommand = "ssh -i ~/.ssh/bitbucket"
```
