---
layout: post
title: Setting up my macOS terminal.
---

This post outlines how I recently configured a new macOS terminal. 

## Ingredients

The following components will be installed:

- [solarized](https://ethanschoonover.com/solarized/) colour scheme. 
- [homebrew](https://brew.sh/), a package manager for macOS.
- [zsh](https://formulae.brew.sh/formula/zsh#default), an alternative shell used instead of bash. 
- [oh-my-zsh](https://ohmyz.sh/), a framework for managing your zsh configuration. 

## Method

### Solarized

1. Clone the [Solarized](https://github.com/altercation/solarized) repository somewhere on your computer.
2. Follow the documentation to [Import and export Terminal profiles on Mac](https://support.apple.com/en-au/guide/terminal/trml4299c696/mac)

### Homebrew

Installing Homebrew is straightforward - run the following command to download and install Homebrew:

{% highlight bash %}
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
{% endhighlight %}

### zsh

Edit: I installed zsh on macOS Mojave using homebrew (outlined below), but [starting with macOS Catalina](https://support.apple.com/en-us/HT208050), it appears that zsh will be the default shell. 

Install via homebrew:

{% highlight bash %}
brew install zsh
{% endhighlight %}

### oh-my-zsh

Install via curl:

{% highlight bash %}
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
{% endhighlight %}

The installation script should set zsh to be your default shell, but if it doesn't you can do it manually:

{% highlight bash %}
chsh -s $(which zsh)
{% endhighlight %}
