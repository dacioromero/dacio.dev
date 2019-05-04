---
layout: post
title: Hosting a Jekyll Website with GitLab Pages, a Custom Domain, and HTTPS
---

To get this blog up and running I had the issue of needing to serve it through HTTPS because of HSTS property on .dev domains.

This tutorial assumes that you're on a Mac with Homebrew installed

# Getting the environment set up {#environment}

1. Install rbenv with `brew install rbenv`
1. Add `if which rbenv > /dev/null; then eval "$(rbenv init -)"; fi` to your terminal profile and reload it
    ```sh
    echo 'if which rbenv > /dev/null; then eval "$(rbenv init -)"; fi' >> ~/.zshrc
    source ~/.zshrc
    ```
1. Install the latest Ruby (2.6.2 at time of writing) with `rbenv install $(rbenv install -l | grep -v - | tail -1)` [credit](https://stackoverflow.com/a/30191850)
1. Set the global version of Ruby with `rbenv global [version]` where version is whatever it installed previously (e.g. `rbenv global 2.6.2`)
1. Run `rbenv rehash` to use it in your terminal
1. Install Jekyll and bundler with `gem install jekyll bundler` confirming when necessary
1. Navigate to where you want your new project and run `jekyll new [your-project]`
1. Run `bundle install` and then `bundle exec jekyll serve`

# Automatically building and publishing with GitLab CI {#gitlab-ci}

One of my favorite things about GitLab is the similar but often more functionality than GitHub such as having a tool like GitLab CI which is similar to Travis CI or AppVeyor in the same environment

1. Create a file at the root of your new project with the following
    ```yaml
    image: ruby:2.6.2

    include:
      remote: 'https://gitlab.com/pages/jekyll/raw/master/.gitlab-ci.yml'

    before_script:
      - gem install bundler:2.0.1
      - bundle install
    ```
1. Replace `2.6.2` with the version from step 3 of the [last section](#environment) and `2.0.1` with the version at the bottom of your Gemfile.lock in the `BUNDLE WITH` block

What this does is extends GitLab's template CI to use our version of Ruby and bundler

The original confiuration runs a test build and then a full build which is stored in the [artifacts](https://docs.gitlab.com/ee/user/project/pipelines/job_artifacts.html). `public` artifacts are then taken to serve using GitLab Pages.

Read more on
- [GitLab CI](https://docs.gitlab.com/ee/ci/)
- [GitLab Pages](https://docs.gitlab.com/ee/user/project/pages/)

# It's your domain {#custom-domain}

You're likely not going to want to send people to gitlab.io link, so lets get your domain set up.

1. Add your domain under the pages settings tab<br/>
![GitLab Page Settings](https://i.imgur.com/GFVM8sW.png){: height="300px"}
1. Uncheck "Force HTTPS" and click save, then click on "New Domain"
1. Add the CNAME and TXT fields to your DNS settings and click verify

# Time to get secure {#letsencrypt}

***THIS SECTION IS A SHORTENED VERISON OF [GITLAB'S](https://about.gitlab.com/2016/04/11/tutorial-securing-your-gitlab-pages-with-tls-and-letsencrypt/)***

If you bought a .dev or .app domain like me you'll run into the issue. Where you can't access you're newly minted page due to pesky [HSTS](https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security)

Regardless you should encrypt your site for peace of mind

1. Install certbot with `brew install certbot`
1. Run `sudo certbot certonly --manual` following the directions up to "Create a file containing just this data"
1. Create a file called letsencrypt-setup.html in your project's root with the following
    ```
    ---
    layout: null
    permalink: /.well-known/acme-challenge/[path]
    ---

    [data]
    ```
    Replace `[path]` with the string of numbers and letters in the path and `[data]` with the data they want
1. Test locally with `bundle exec jekyll serve` and `curl http://127.0.0.1:4000/.well-known/acme-challenge/[path]` and make sure its the same as `[data]`
1. `git push` and wait for GitLab to finish the build
1. Make sure that `curl http://[your-domain]/.well-known/acme-challenge/[path]` gives the expected output and finish the setup
1. Open the edit screen for the domain you previously added to your GitLab Pages settings
1. Run `sudo cat /etc/letsencrypt/live/fullchain.pem | pbcopy` and paste to "Certificate (PEM)"
1. Run `sudo cat /etc/letsencrypt/live/privkey.pem | pbcopy` and paste it to "Key (PEM)"
1. Save your domain and wait a few minutes before navigating to `https://[your-domain]`

If all has gone well you now have an encrypted, static, Jekyll website which will be automatically updated every time you push!
