---
title: 'Using Go vanity URL with cgit'
description: 'How to setup Go vanity URLs with cgit to allow custom domain packages'
date: '2024-10-25T16:00:00+02:00'
tags: ['debian', 'cgit', 'nginx', 'git', 'go']
---

This year I wanted to experiment with moving away from Github (and other cloud based VCS). So I setup my own server with [git](https://git-scm.com/book/en/v2/Git-on-the-Server-Setting-Up-the-Server), [cgit](https://git.zx2c4.com/cgit/) as the interface and [nginx](https://nginx.org/) as the web server. The basic setup was fairly easy.

<!--more-->

I recently published one of my tools (snowy) and when I tried to install it from my repository I got an error message:

```
unrecognized import path "git.moiaune.dev/snowy" (parse https://git.moiaune.dev/snowy?go-get=1: no go-import meta tags ())
```

I went to the internet and did some research. Apparently, Go treats github.com (and other popular vcs) specially, so if we want to our custom domain and cgit hosted repository to work with `go install` and `go get` we need to do some extra work on our end. You can read the official documentation about the subject here: [https://pkg.go.dev/cmd/go#hdr-Remote_import_paths](https://pkg.go.dev/cmd/go#hdr-Remote_import_paths)

Basically what we need is an HTML meta-tag to tell go how to map a package name to a repository. For instance, go does not know that my package name `git.moiaune.dev/snowy` should resolve to `https://git.moiaune.dev/snowy` by itself.

This is the tag that go will look for.

```
<meta name="go-import" content="git.moiaune.dev/snowy git https://git.moiaune.dev/snowy">
```

To use this meta-tag cgit has a `repo.extra-head-content` option that we can use to inject the meta-tag. The downside of this is that it must be done pr repository. Instead we can use NGINX `sub_filter` module to inject this meta-tag on very repository.

So in your NGINX configuration for our domain we will replace a part of the HTML before returning it.

```conf
server {
    server_name git.moiaune.dev;
    # ...

    location / {
        # ...

        sub_filter '</head>' '<meta name="go-import" content="$host$uri git https://$host$uri"></head>';
        sub_filter_once on;
    }
}
```

The next time you do a `go install` it should work.
