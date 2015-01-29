![Venmo Engineering Blog](images/banner.png)

## Running it locally

You run Jekyll to compile the static site for development in your terminal.

First install Bundler if you don't already have it (if the output from `which bundle` is not blank, you already have it). Bundler lets you keep the gems for the particular project isolated from your system, so it won't interfere with any other versions of Jekyll or other ruby gems you use elsewhere.

```sh
$ gem install bundler  # (may need sudo)
```

Then, install Jekyll and all dependencies through Bundler

```sh
$ bundle install --path ./vendor
```

Now you're ready to run it in development mode. With this running, every time you save the file Jekyll will re-compile the assets. Just refresh your browser to see the changes.

```sh
$ bundle exec jekyll serve
```

## Writing posts

Posts are written in markdown.  Be sure you include this at the top of each of your posts:

```
---
layout: post
title: "Your Post Title"
author: Your Name
---
```
