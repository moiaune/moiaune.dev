---
title: 'Automatically release Go app with Goreleaser and Github'
description: "We're taking a look at how to automatically release Go applications with Goreleaser and Github"
tags: ['go', 'golang', 'github', 'automation']
date: 2021-08-24T01:02:53+02:00
draft: true
---

One of the biggest benefits of Go is that you can distribute your application as a single binary and it's pretty simple for Go programmers to install any Go app using `go install`. But most times you also want to provide a way for non-Go users to download your application, so in this post we're going to take a look at how you automatically can publish a new release on Github with GoReleaser that will include your binary.

## What is GoReleaser?

- brief introduction
- why we need goreleaser
- ease of use

## Setup our sample project

- create a new gorelaser-example repo with a small golang app
- add the finished project as another branch
- git clone repo

## Install GoReleaser locally

- use brew for macos

## Create GoReleaser config

- `goreleaser init`
- change archives format to binary

## Testing gorelease before activating in prod

- how to to use the --skip-publish and --snapshot params

## Add goreleaser to Github Actions workflow

- only trigger when a new tag is pushed
