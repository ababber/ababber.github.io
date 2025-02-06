---
title: How to create a blog with Quartz, GitHub, and Cloudflare
draft: true 
tags:
  - quartz
  - brew
  - node
  - npm
  - github
  - cloudflare
  - custom-domain
  - ssl
  - macOS
---

> If you don't want to use [Jekyll](https://jekyllrb.com/) as your static site generator for [GitHub Pages](https://pages.github.com/) and you want to have a custom domain for your [GitHub Pages](https://pages.github.com/). This post is for you!

## 0. Dependencies

* Find a package manager for your current system and become familiar with it. I will use `brew` since I'm on `macOS`.
* `brew install node git` or something similar for your package manager.

## 1. Fork [Quartz](https://github.com/jackyzha0/quartz)

* You can either use `Quartz` as a `Template` or `Fork` it.
  * `Template`: copy of `Quartz` as is, which might make keeping up with `Quartz` updates slightly difficult. Updates will probably need a copy and paste of `Quartz` at the root. Managing changes this way is difficult.
  * `Fork`: easier to sync changes from the source. Any changes or modifications you make to `Quartz` will become `merge` conflicts when an update happens, which are easier to manage with the right tools.
  * I recommend a `Fork`. After you click on `Fork`, **STOP**.

## 2. Setup `username.github.io` to initialize your `GitHub Pages`

* Since you can have only one public `Fork` of `Quartz` at a time, I used my `Fork` to create my `GitHub Pages`.
* Under the `repo` name after you click `Fork`, I entered `ababber.github.io`. When you name your `repo`, replace `ababber` with your `GitHub` username, which initializes your `GitHub Pages`.
* Once the `Fork` is created. There are a few more steps before you're live.

## 3. 