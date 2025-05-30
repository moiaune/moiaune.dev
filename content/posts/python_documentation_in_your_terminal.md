---
title: 'Python documentation in your terminal'
description: 'How to read/get Python documentation in your terminal using built-in tools'
date: '2025-05-30T16:00:00+02:00'
tags: ['python']
---

For the past few years I've been writing mostly Go and learned to use the `go doc` command to read documentation on how things work. For me this has been really effective for learning the language instead of Googling everything. A few weeks back I started to look into Python and first there wasnt an intuitive way to read the documentation in my terminal. Here's a few tips on how to do some of the same things in Python as in `go doc`.

<!--more-->

All of these are expecting you to run them in the Python REPL, but you can also just do `python3 -c "import <module>;<the_command>"`.

## Display the docstring

```python
import os

help(os)
help(os.getenv)
help(print)
```

## Show the source code

```python
import inspect
from datetime import datetime

print(inspect.getsource(datetime)
```
