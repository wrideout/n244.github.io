---
layout: post
title:  "The Search for Reliable SSH"
date:   2016-04-11 20:12:41 -0700
categories: tools vim osx
---

# The Search for Reliable SSH

Working as s software developer can be complicated at times, usually due to some
over-the-top build system that is necessary to build _The Product_.  If you're
fortunate, it's all Makefiles underneath.  If not, well, that's another post
altogether.  Usually, however, these build systems are run on remote-access
machines.  I personally have worked with several such systems, and up until now
I have relied on SSH to access these machines.  Lately I've been taking this
whole process a little further, and have been editing source code remotely as
well.  This is partly due to the size of the code base I'm working on and the
version control system we're using to manage it.  Frankly, editing code remotely
is easier option, if only because I can't fit multiple instances of the codebase
on my Macbook!  So like I said before, SSH works fine.  The thing is, though,
that SSH has this habit of timing out due to inactivity.  I'm also lazy (most
programmers are!), and it's a pain to constantly reconnect and change directory
to where I was before my connection was hosed.  

## Enter, Mosh

Mosh is a neat program from the clever folks over at MIT.  The basic idea is
that Mosh is a replacement for SSH that is persistant.  Connections formed using
Mosh are able to survive network drops, and even __roams__.  That's right, you
can even switch networks without losing your remote connection.  Cool stuff,
right?

Initially, I was thrilled with Mosh; it provided the exact feature I was looking
for.  It wasn't long, however, until I found a shortcoming.  The build system I
use at work prints a _ton_ of output to the console.  Much of it is
uninteresting, but sometimes failure messages aren't printed in the last
screen's-worth of text.  As it turns out, Mosh behaves much like telnet used (?)
to: output that is pushed beyond the upper bounds of the terminal window is
abandoned.  Simply put, I didn't have enough scrollback space in order to see
the messages I needed.  I now needed a way to extend this scrollback.  Now,
there is a bug filed for Mosh that requests this feature, and in fact extended
scrollback is slated as a feature for the 1.3 release of Mosh.  I was, as you
can imagine, unwilling to wait for a new release.  I needed a workaround.

## Tmux to the rescue

Googling around showed that I was not the only one out there looking for a
solution to the Mosh scrollback problem.  I played around with several options,
ultimately settling on using Tmux to manage the scrollback, among other things.
If you're not familiar with Tmux, it's essentially the BSD version of screen.
There is a raging debate out in the Internet as to which is the better
application.  Feel free to draw your own conclusions; I'll provide links to all
my resources at the bottom.  I've found Tmux to be adequate for my needs, albeit
with a little configuration.

All configuration happens in a file in your home directory, named ".tmux.conf".  
First of all, I wanted to start with a new session, and make sure I had a good
long scrollback to work with.  I'm also a Vim user, and wanted to use Vim-style
keybindings inside Tmux.  Do do all that, I put the following in my configuration
file:

```
new-session
set -g history-limit 30000
setw -g mode-keys vi
```

I tested my configuration, and I had scrollback!  I could execute a lengthy
command, such as `ps ax` on my remote server and scroll back over the output to
my heart's content.  There was one hangup, though: I could only scroll using the
keybindings in Tmux.  I wanted to be able to scroll using my Magic Trackpad (a
wonderful piece of gadgetry).  So, I did some more googling.  I found a project
on Github that provides exactly this functionality, right
[here](https://github.com/NHDaly/tmux-scroll-copy-mode).  I followed the
directions in the project's README by cloning the repository, and adding the
floowing to my Tmux config:

```
setw -g mouse on
run-shell PATH/TO/scroll_copy_mode.tmux
```

Tested the updated config... and Success!  I could now scroll using my trackpad.
Happy, I started up a new session, and began editing code, only to find that my
Vim syntax colors were a little off.  As it turns out, one more line was
required to fix the colors:

```
set -g default-terminal "screen-256color"
```

## One last note

Of course, over the course of this adventure I had to reload my Tmux
configuration several times.  Thankfully, Tmux includes the ability to
dynamically reload config file.  Instructions to do so may be found
[here](http://blogs.sanctum.geek.nz/reloading-tmux-config)


### Resources

* [Mosh](https://mosh.mit.edu/)
* [Tmux](https://tmux.github.io/)
* [Reloading Tmux Config](https://blog.sanctum.geek.nz/reloading-tmux-config/)
* [Getting 256-Colors to Work in Tmux](https://unix.stackexchange.com/questions/1045/getting-256-colors-to-work-in-tmux)
* [Better Mouse Scrolling in Tmux](https://www.davidverhasselt.com/better-mouse-scrolling-in-tmux/)


