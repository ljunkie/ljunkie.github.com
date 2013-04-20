---
title: ssh-add on demand
author: rob
layout: post
categories:
  - Linux
  - Ubuntu
comments: true
---
# 

**Solution**: bash alias - `alias ssh="( ssh-add -l > /dev/null || ssh-add ) && ssh"`

Normally one would put this in their ~/.bash\_profile. However with Ubuntu 11.10, at least with my install,~/.bash\_profile is not included. The file to add this line to is:

**~/.bash_aliases**
```
alias ssh="( ssh-add -l > /dev/null || ssh-add ) && ssh"
```

**More Info:**
Ubuntu does state if ~/.bash_profile does exists, then ~/.profile will not be read, so I am using what has been setup to work by default.

<!-- more -->

```
# cat ~/.profile
# ~/.profile: executed by the command interpreter for login shells.
# This file is not read by bash(1), if ~/.bash_profile or ~/.bash_login
# exists.
...
if [ -n "$BASH_VERSION" ]; then
    # include .bashrc if it exists
    if [ -f "$HOME/.bashrc" ]; then
        . "$HOME/.bashrc"
    fi
fi
...
```

~/.profile is set to include ~/.bash_rc which is set (at least on my system) to include ~/.bash_aliases
```
# cat ~/.bashrc
...
if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi
...
```

