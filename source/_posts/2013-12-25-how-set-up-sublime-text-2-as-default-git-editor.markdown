---
layout: post
title: "How set up Sublime Text 2 as your default git editor"
date: 2013-12-25 12:42:58 +0400
comments: true
categories:
---

If you love Sublime Text 2 as I do then you probably would like to change default git editor (which is Vim nowadays). I suppose that you have installed Sublime text.

First of all, you should install `subl` shortcut. To do that open your shell and print

{% codeblock Terminal lang:bash %}
$ cd ~
$ mkdir bin
$ ln -s "/Applications/Sublime Text 2.app/Contents/SharedSupport/bin/subl" ~/bin/subl
{% endcodeblock %}

If you already have `~/bin` directory, then just use the last line of my code. Now restart your shell and try that:

{% codeblock Terminal lang:bash %}
$ subl -h
Sublime Text 2 Build 2221

Usage: subl [arguments] [files]         edit the given files
   or: subl [arguments] [directories]   open the given directories
   or: subl [arguments] -               edit stdin

Arguments:
  --project <project>: Load the given project
  --command <command>: Run the given command
  -n or --new-window:  Open a new window
  -a or --add:         Add folders to the current window
  -w or --wait:        Wait for the files to be closed before returning
  -b or --background:  Don't activate the application
  -s or --stay:        Keep the application activated after closing the file
  -h or --help:        Show help (this message) and exit
  -v or --version:     Show version and exit

--wait is implied if reading from stdin. Use --stay to not switch back
to the terminal when a file is closed (only relevant if waiting for a file).

Filenames may be given a :line or :line:column suffix to open at a specific
location.
{% endcodeblock %}

If you have the same output, then everything works great. To set up sublime as default editor just print in your shell that:

{% codeblock Terminal lang:bash %}
$ git config --global core.editor "subl -n -w"
{% endcodeblock %}

And that is all!
