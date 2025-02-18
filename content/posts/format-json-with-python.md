---
title: "Format JSON with Python (oneliner)"
description: "Use python to format JSON data"
date: 2024-12-19T16:00:00+02:00
tags: ['python', 'json', 'cli']
---

I'm experimenting with getting by without Language Server Protocol (LSP), and today I came a accross a large JSON file that was compressed into a single line. As I usually do I typed `<leader>+f` to make the LSP format the file.. I soon realized that I had to find another way without my LSP.

<!--more-->

So I came across this python3 module `json.tool` that can format JSON input.

To format a file:

```bash
python3 -m json.tool unformatted.json > formatted.json
```

To format from stdin:

```bash
cat unformatted.json | python3 -m json.tool -- > formatted.json
```

We can also do it from within vim thanks to vim's support of external commands. Let's say we have the following line in our editor.

```json
{ "name": "moiaune", "website": "https://moiaune.dev"}
```

Let's select the line with `V` and then type

```
:'<,'>!python3 -m json.tool --
```

And VOILA, the data is formatted.

```json
{
    "name": "moiaune",
    "website": "https://moiaune.dev"
}
```

## Alternative

Another alternative is to use `jq` which is way less characters to type, but it requires you to have the tool installed and the python module is built-in.
