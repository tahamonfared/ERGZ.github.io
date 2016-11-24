---
layout: post
title: "How I Created My Website"
---

In this short article I will show the steps I took to set up this Website.
There are several tools we need to get started, shown in the Prerequisite section,
once we have those installed the process is straightforward.

## Prerequisites

#### 1. Install Ruby

The static site generator (Jekyll) is a ruby gem. So first things first, we need
to install ruby. This is platform dependent process but the steps are documented
on the [Go Rails](https://gorails.com/setup/ubuntu/16.04) website.

#### 2. Install Jekyll

Jekyll is a static site generator, installing it is breeze,

~~~bash
gem install Jekyll
~~~

It will take care of converting our static (markdown) pages to html and css.

*Note if you are using a linux distro, it may be required you install ruby headers
separately*

~~~bash
sudo apt-get install ruby-dev # on ubuntu/debian
sudo dnf install ruby-devel # on fedora/centos
~~~

*Note ubuntu repos have an old version of ruby (1.9.2), but it is recommended
to use > 2.0, use the guide linked above for the latest version.*

#### 3. Other Gems

Other gems we need to install,

~~~bash
gem install jekyll-paginate
gem install bundler
~~~

#### 4. Git and github

We will be using Github to host our static site, and of course git to push to Github.
Once again this is platform dependent, but Github documents the process here:
[Set Up Git](https://help.github.com/articles/set-up-git/#platform-mac).


## Setting up Github

The first thing we will take care of is creating a repo in Github to host our site.
This is done usual way, but the name of repository **must** be of the form:

__*username.github.io*__

Where *username* is the one registered with Github. My site for example lives under
the repo *ergz.github.io*.

This is pretty much all you have to do.

## Creating A Site

There are a few ways we can get started creating a site. The first way is to start
fresh using jekyll, using the command,

~~~bash
jekyll new my-website
~~~

This will create a set of directories and files in a new folder called **my-website**.
It produces a ready to use template website. However there are some really good
themes we can use to get started instead. We will focus on doing this.

A set of themes I highly recommended are those produced by **mdo** called Hyde,
Lanyon and Poole, the latter being the most basic and simple of the three. My
webiste is itself the Hyde theme with some minor tweaks. You can learn more
about them [here](http://getpoole.com/).

You can obtain these by simply using `git clone`, as follows:

~~~bash
# choose one
git clone https://github.com/poole/lanyon.git # get the lanyon theme
git clone https://github.com/poole/hyde.git # get the hyde theme
~~~

The above will create a new directory called `lanyon` (or hyde), a ready to use
website. 
