---
title: How to create a blog with Quartz, GitHub, and Cloudflare
draft: false 
tags:
  - quartz
  - markdown
  - brew
  - node
  - npm
  - github
  - cloudflare
  - custom-domain
  - ssl
  - macOS
---

> If you don't want to use [Jekyll](https://jekyllrb.com/) as your static site generator for [GitHub Pages](https://pages.github.com/) and you want to have a [custom domain for your GitHub Pages](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/about-custom-domains-and-github-pages). This post is for you!

## 0. Dependencies

* Find a package manager for your current system and become familiar with it. I will use `brew` since I'm on `macOS`.
* `brew install node git gh` or something similar for your package manager.
* Using `gh` or the GitHub cli makes working with GitHub easier.

## 1. Fork [Quartz](https://github.com/jackyzha0/quartz)

* You can either use `Quartz` as a `Template` or `Fork` it.
  * `Template`: copy of `Quartz` as is, which might make keeping up with `Quartz` updates slightly difficult. Updates will probably need a copy and paste of `Quartz` at the root. Managing changes this way is difficult.
  * `Fork`: easier to sync changes from the source. Any changes or modifications you make to `Quartz` will become `merge` conflicts when an update happens, which are easier to manage with the right tools.
  * I recommend a `Fork`. After you click on `Fork`, **STOP**.

## 2. Name your [Quartz](https://github.com/jackyzha0/quartz) `Fork` to initialize your GitHub Pages

* Since you can have only one public `Fork` of [Quartz](https://github.com/jackyzha0/quartz) at a time, I used my `Fork` to create my GitHub Pages.
* Under the `repo` name after you click `Fork`, I entered `ababber.github.io`. When you name your [Quartz](https://github.com/jackyzha0/quartz) `Fork` `repo`, replace `ababber` with your GitHub `username`, which initializes your GitHub Pages.
* According to GitHub, you only have one GitHub Page per [username](https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages).
* Once the `Fork` is created. There are a few more steps before you're live.

## 3. Setup [Quartz](https://github.com/jackyzha0/quartz) locally

```sh
# clone with git
git clone https://github.com/username/username.github.io.git

# 0. if you get an error, then use the GitHub 
# cli to login to GitHub on your command line
# with the following command: `gh auth login`

# 1. clone with the github cli:
# gh repo clone username/username.github.io 

# change directory into repo root
cd username.github.io

# install quartz dependencies
npm i

# initialize quartz and create an index.md
npx quartz create 

# select: Empty Quartz
# select: Treat links as relative paths

# cd into content/, which is where quartz will 
# look to create static pages 
cd content 

# create a test markdown file
touch test.md

# return to the repo root
cd ..

# this will build a preview of your blog
npx quartz build --serve

# 0. Ensure your browser allows http traffic

# 1. Once the build is done, go to http://localhost:8080
# in your browser.

# 2. In the left hand Explorer column, you should
# see test or the file created earlier.
```

## 4. Setup Deploy in Repo

* In your favorite code editor, create a file called `deploy.yml`.
  * I recommend [vscode](https://code.visualstudio.com/).
* In the hidden directory `/username.github.io/.github/workflows/`, add `deploy.yml`
* The content of `deploy.yml` is below:

```yml
name: Deploy Quartz site to GitHub Pages
 
on:
  push:
    branches:
      - v4
 
permissions:
  contents: read
  pages: write
  id-token: write
 
concurrency:
  group: "pages"
  cancel-in-progress: false
 
jobs:
  build:
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0 # Fetch all history for git info
      - uses: actions/setup-node@v4
        with:
          node-version: 22
      - name: Install Dependencies
        run: npm ci
      - name: Build Quartz
        run: npx quartz build
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: public
 
  deploy:
    needs: build
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

## 5. Configure GitHub for [Quartz](https://github.com/jackyzha0/quartz) Deploy

* Go to your [Quartz](https://github.com/jackyzha0/quartz) `Fork`, which will be something like: `https://github.com/username/username.github.io`.
* Click on the `Settings` tab for the `repo`.
* In the left hand column, click on `Pages`.
* Under `Build and deployment` in the `Source` drop down menu, select `GitHub Actions`
* We've already setup the workflow with `deploy.yml`.
* At the bottom, select `Enforce HTTPS` if not selected.

## 6. Sync Content

* In `quartz.config.ts`, change the `baseUrl` property to your GitHub Pages root domain, which should be something like `username.github.io`.

```sh
# check if everything looks fine
npx quartz build --serve

# commit the changes and deploy your quartz site
npx quartz sync

# 0. it will take a few minutes for your page to 
# go live at username.github.io

# 1. in your username.github.io Actions tab, you 
# can check the status of your build and deploy, 
# any errors will be listed here
```

## 7. Setup Custom Domain with [cloudflare](https://www.cloudflare.com/)

* Go to [cloudflare](https://www.cloudflare.com/)
* Create an account.
* Search for a domain name.
* Register it (prices will vary depending on the domain name).
* From you `Account Home` go to the domain you just created.
* Click on `DNS` and ensure you're in the `Records` section.
* Select `Add record`:
  * Type: `CNAME`
  * Name: `@`
  * Target: `username.github.io`
  * Select `Save`
* Select `Add record`:
  * Type: `CNAME`
  * Name: `www`
  * Target: `username.github.io`
  * Select `Save`
* For all the records that were created, ensure the `Proxy status` is **OFF**, we will enable this later.
* On the left hand column of your custom domain page, click on `SSL/TLS`.
* Ensure you are on the `Overview` section of `SSL/TLS`.
* Click on `Configure`.
* By default [cloudflare](https://www.cloudflare.com/) sets `Custom SSL/TLS` to `Flexible`. Change this to `Full`.
  * We need to do this because `username.github.io` is being served to [cloudflare](https://www.cloudflare.com/) over `HTTPS` by GitHub.
  * If we don't do this then your GitHub page may become unreachable and report the following error: `ERR_TOO_MANY_REDIRECTS`. This is caused by a conlict with the `Custom SSL/TLS` `Flexible` setting and the Github Pages `Enfore HTTPS` setting which creates an unexpected feedback loop.

## 8. Set GitHub Pages Custom Domain

* [Verify your custom domain with the following instructions](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/verifying-your-custom-domain-for-github-pages).
* Go to `https://github.com/username/username.github.io`.
* Click on your repo `Settings` tab.
* In the left hand column, click on `Pages`.
* Under `Custom domain`, enter your `customdomain.com` root. Don't include the protocol or a subdomain.
* Click on `Save`.
* Wait on this page for GitHub to perform checks. If everything was setup correctly, `DNS check successful` will appear.

## 9. Final Things to Configure

* Go to your repo on your local machine.
* In `quartz.config.ts` change the `baseUrl` property to your `customdomain.com` root.
* Sync changes to GitHub with: `npx quartz sync`.
* Wait for the build and deploy to finish in your `username.github.io` repo.
* Check if your `customdomain.com` is live.
* Troubleshooting can be done in your `username.github.io` repo `Actions` and `Settings --> Pages` tabs.
* Once everything is live and working correctly, go back to your [cloudflare](https://www.cloudflare.com/) account.
  * Log in.
  * Go to your `customdomain --> DNS --> Records`
  * Set `Proxy status` to **ON** for the previously created `CNAME` records.

## 10. References

* [GitHub Pages Docs](https://docs.github.com/en/pages)
* [Quartz Docs](https://quartz.jzhao.xyz/)
* [Cloudflare Community](https://community.cloudflare.com/)
