---
title: NUnit
permalink: mydoc_tdd_start.html
keywords: jekyll on windows, pc, ruby, ruby dev kit
sidebar: mydoc_sidebar
folder: mydoc
---

{% include tip.html content="For a better terminal emulator on Windows, use [Git Bash](https://git-for-windows.github.io/). Git Bash gives you Linux-like control on Windows." %}

## Install Ruby and Ruby Development Kit

First you must install Ruby because Jekyll is a Ruby-based program and needs Ruby to run.

1. Go to [RubyInstaller for Windows](http://rubyinstaller.org/downloads/).
2. Under **RubyInstallers**, download and install one of the Ruby installers under the **WITH DEVKIT** list (usually the recommended/highlighted option).
3. Double-click the downloaded file and proceed through the wizard to install it. Run the `ridk install` step on the last stage of the installation wizard. 
4. Open a new command prompt window or Git Bash session.

<h2 id="bundler">Install the Jekyll gem</h2>

At this point you should have Ruby and Rubygem on your machine.

Now use `gem` to install Jekyll:

```
gem install jekyll
```

You can now use Jekyll to create new Jekyll sites following the quick-start instructions on [Jekyllrb.com](http://jekyllrb.com).


{% include links.html %}
