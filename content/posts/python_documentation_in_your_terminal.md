---
title: 'Python documentation in your terminal'
description: 'How to read/get Python documentation in your terminal using built-in tools'
date: '2025-05-30T16:00:00+02:00'
tags: ['python']
---

For the past few years I've been writing mostly Go and learned to use the `go doc` command to read documentation on how things work. For me this has been really effective for learning the language instead of Googling everything. A few weeks back I started to look into Python and first I didnt think there was an intuitive way to read the documentation in my terminal. But after some digging I found a few methods. Here's a few tips on how to do some of the same things in Python as in `go doc`.

<!--more-->

## help

Pythons `help` function will return the function or class docstring.

```python
import os

help(os)
help(os.getenv)
help(print)
```

## inspect

We can use the inspect module to get the source code of a function or class.

```python
import inspect
from datetime import datetime

print(inspect.getsource(datetime)
```

## pydoc

Pydoc behaves more like `go doc` and also have a feature to start a local HTTP server to serve you the docs.

```console
# find function and modules that contains the search_term
pydoc3 -k <search_term>

# display doc for a module or function
pydoc3 <module/function>

# start a local HTTP server
pydoc3 -b
```

I find the `pydoc` command very useful and I especially like that you can run a local HTTP server for offline documentation.
