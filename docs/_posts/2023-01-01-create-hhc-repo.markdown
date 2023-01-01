---
layout: single
title:  "Creating Git Repo and Jekyll Documentation for HHC"
date:   2023-01-01 12:00:00 +1000
toc: true
categories: hhc sans git github aws amplify jekyll
---

This guide can be used to create a Git repo for the SANS Holiday Hacking Challenge or similar where code and static website need to be used.

# Steps

## Create Git Repo

In github create the repo.  Namcing scheme can be:

    hhc<yyyy>

## Clone the repo to local machine

Use vscode or similar to clone the repo to the local machine.

## Setup Documentation

Inside the parent directory of the repo.  Create `docs` directory to store the static website.

    mkdir docs

Now create `jekyll` static website content in the `docs` directory.

    cd docs
    jekyll new hhcReport

Create additional folders for the collection of challenges (page per HHC challenge).  Create `data` directory.  Cleanup unused content.

    cd hhcReport
    mkdir _challenges
    mkdir _data
    rm _posts/*

Update `_config.yml`.  Update values in `<...>` as required.

```yml
title: SANS Holiday Hacking  - KringleCon <year> Report
email: <email>
description: >- # this means to ignore newlines until "baseurl:"
<name>'s report for the SANS Holiday Hacking challenge (KringleCon <year>).
baseurl: "" # the subpath of your site, e.g. /blog
url: "" # the base hostname & protocol for your site, e.g. http://example.com
twitter_username: jekyllrb
github_username:  jekyll

# Build settings
#theme: minima
theme: minimal-mistakes-jekyll
#plugins:
#  - jekyll-feed

collections:
challenges:
    output : true
    permalink: /:collection/:path/

defaults:
# _challenges
- scope:
    path: ""
    type: challenges
    values:
    layout: single
    sidebar:
        nav: "docs"
```

The above will use `minimal-mistakes-jekyll` theme.  Add the `gem` to `Gemfile`:

    gem "minimal-mistakes-jekyll"

 To install the theme use `bundle` to get required depencies.

    bundle update    

Setup navigation for sidebar in `_data/navigation.yml`.  In the below, `main` will be displayed in the header of the site while `docs` will be used for the sidebar.  The `challenge1` & `challenge2` in the `url` below should match the filename of the page to display in the collection with the extension dropped.

```yml
main:
  - title: "Home"
    url: /
  - title: "Challenges"
    url: /challenges/index/
  - title: "About"
    url: /about/

docs:
  - title: Main Challenge Section 1
    children:
    - title: "Challenge 1"
      url: /challenges/challenge1/
    - title: "Challenge 2"
      url: /challenges/challenge2/
```

Update `index.markdown` as required.  Basic template as below:

    ---
    layout: single    
    sidebar: 
        nav: "docs"
    ---

    <index content>

Update `about.markdown` as required.  Basic template as below:

    ---
    layout: single
    title: About
    permalink: /about/
    ---

    These are my solutions for the <year> SANS Holiday Hacking Challenge (KringleCon <version>).

    Author: <name>

    Email: <email>




Basic template for content of a challenge page is below.  Put in `_challenges` directory.

    ---
    name: Buy a Hat
    title: Buy a Hat
    ---

    ## Problem

    <problem statement>

    ## Hints

    1. Hint 1
    1. Hint 2

    ## Solution

    <steps to solve the challenge>

    ## Appendix

    <additional information if required>