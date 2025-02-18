---
title: 'TIL: Initiate Config class in Python'
description: 'Today I Learned a neat trick to initiate a Config class from a JSON file in Python'
date: '2024-11-27T14:36:00+02:00'
tags: ['python']
---

I came across [anthonywritescode's](https://github.com/anthonywritescode) Github repository for his [twitch-chat-bot](https://github.com/anthonywritescode/twitch-chat-bot) which is written in Python. I have not written much Python in my life so maybe this is trivial, but I found it to be a neat little trick anyways.

<!--more-->

Consider you have the following JSON config file.

```jsonc
// file: config.json
{
    "username": "moiaune",
    "token": "636DDA4D-A395-43C5-A2B1-7A0401DE51AB"

}
```

In the past I would probably have loaded it like this:

```python
import json

class Config():
    username: str
    token: str

    def validate_username(self) -> bool:
        return True if len(self.username) > 0 else False

with open('./config.json') as f:
    d = json.load(f)
    cfg = Config(username=d.username, token=d.token)

print(cfg.validate_username())
```

What we essentially are doing is that we load the `config.json` into a variable as dictionary and then we initiate a new Config class and assign each class property by arguments. With this little trick we can make Python do it for us.

```python
import json
from typing import NamedTuple

class Config(NamedTuple):
    username: str
    token: str

    def validate_username(self) -> bool:
        return True if len(self.username) > 0 else False

with open('./config.json') as f:
    cfg = Config(**json.load(f))

print(cfg.validate_username())
```

The first thing to notice is that our `Config` class now takes arguments in the form of a `NamedTuple`. Then we have combined the `json.load()` with `**` when initiating our Config class. The `json.load()` method will give us a Python dictionary from the `config.json` file. We then unpack the dictionary (with `**`) into named arguments for the Config class.

It's not much, but a neat little shortcut and in my opinion it reads a little better too.
