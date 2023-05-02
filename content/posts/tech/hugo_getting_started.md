+++
title = "hugo_getting_started"
date = "2023-04-30T16:18:12-07:00"
author = "Daniel Firu"
authorTwitter = "danfiru" #do not include @
cover = ""
tags = ["technology", "tutorial"]
keywords = ["", ""]
description = "Hugo is an excellent choice if you're looking for a static site generator framework that is fast, flexible, and easy to use. It is written in Go and designed to be incredibly fast, making it an ideal choice for building large and complex websites. In this blog post, we'll explore why you should choose Hugo as your static site generator framework and, if you do, how to set it up from scratch. "
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

# Hugo is the Best Static Site Generator
Hugo is an excellent choice if you're looking for a static site generator framework that is fast, flexible, and easy to use. It is written in Go and designed to be incredibly fast, making it an ideal choice for building large and complex websites. In this blog post, we'll explore why you should choose Hugo as your static site generator framework and, if you do, how to set it up from scratch. 

## Intro
I've settled on Hugo as my static site generator of choice. In addition to that, I've chosen terminal as my base theme to get started. No frills -- just a good place to write and share what I want to share. Keep reading if you want to set up a blog on hugo, track it with git, and ultimately auto-publish using GitHub pages.

## Materials and Requirements
- `go`: Hugo is an SSG based on `go`. So, you're going to need `go`. 
> To set up `go` on your system, visit the official Go website [[golang.org](https://golang.org)] and download the installation package for your operating system. 
- `git`: We will use `git` to track versions of the blog with the added benefit of using [GitHub](https://github.com) to host the blog via `Github` Actions`.  
- A [GitHub](https://github.com) account
- `hugo`: [Install Hugo for your OS](https://gohugo.io/installation/)
- `terminal`: It is super simple and has a retro / tech vibe. I decided to install the terminal theme as a submodule in my `hugo` project, as I will likely want to tweak it over time. 
>[terminal installation instructions](https://github.com/panr/hugo-theme-terminal) 

## Procedure
Initialize a new hugo project
``` shell
hugo new site quickstart

```
Intialize a new `git` repository with the default branch: `main` (to conform with new GitHub defaults).
```shell
cd quickstart
git init -b main
```

Submodule the `terminal` theme inside of the new git repo. And copy the demo site from inside of the terminal project. And replace builtin theme reference with local one (since we are submoduling the theme into our repo).
```shell
git submodule add -f https://github.com/panr/hugo-theme-terminal.git themes/terminal
cp themes/terminal/exampleSite/config.toml .
sed -i 's/hugo-theme-terminal/terminal/' config.toml 
```

Now start the server with:
```shell
hugo server --buildDrafts
```

## Deployment

Deploy to GitHub