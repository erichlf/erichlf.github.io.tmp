---
share: true
title: How to Get a Continuous Status from dd
author: Erich L Foster
email: erichlf AT gmail DOT com
date: 2020-09-19 Sat
tags:
  - dd
  - linux
  - bash
---

Getting and updating the status of dd in bash

* note:
  Some versions of ~dd~ have the ability to provide the open ~-status=progress~ which makes the
  point of this post moot. But in the case your ~dd~ doesn't have this option, read on.

I found my self needing to return a hard drive that had sensitive information on it, so of course
I wanted to securely delete the contents before returning the drive.

* ~dd~ Comes to the Rescue
  To securely delete a drive one can run the following:
  #+BEGIN_SRC bash
  dd if=/dev/urandom of=/dev/sda bs=1M
  #+END_SRC
  This will write random data to the drive ~/dev/sda~ and overwrite any data that was already there.

  However, the disk was 8TB and so it was going to take quite some time and I wouldn't have a clue
  how far along things had progressed without some status update.

* ~kill~ to Get Update? What are Talking About?
  It turns out you can run ~dd~ in the background and then use ~kill~ to get an update of the ~dd~
  status. When I first came across this it sounded insane, because doesn't ~kill~ end processes?
  Well, that is true, but there are other signals that you can provide to ~kill~ which won't
  terminate a process. In this case we are going to use ~USR1~ (this signal may be different on
  your system).

  The first thing to do is run ~dd~ in the background and grab its ~PID~:
  #+BEGIN_SRC bash
  dd if=/dev/urandom of=/dev/sda bs=1M & PID=$!
  #+END_SRC
  Now that we have the ~PID~ and ~dd~ is running in the background we simply issue the ~kill~ signal:
  #+BEGIN_SRC bash
  kill -USR1 $PID
  #+END_SRC

* Nice! But Let's be Fancier
  Ok, but maybe I just want to look at the screen and not run a command every time I want an update.
  What we can do to solve this is to run the ~kill~ signal in a loop and use ~clear~ to print to the
  same lines by
  #+BEGIN_SRC bash
    while kill -0 $PID > /dev/null 2>&1
    do
      kill -USR1 $PID
      sleep 0.5
      clear
    done
  #+END_SRC
  The condition for the ~while~ loop is using ~kill~ with the flag ~-0~ to see if a signal can be sent
  to the process ~$PID~. A signal can only be sent if the process still exists, so this while loop
  should only run while ~$PID~ is still running. Next we pipe the output to ~/dev/null~ so that we
  aren't spammed by the result of this condition. We also don't want to see the output from ~stderr~,
  so we are piping that to ~stdout~, which in turn is piped to ~/dev/null~

  This is a bit janky, but it seems the best way to have this printed into the same area. If you don't
  like this options you can always go with the simpler, yet multiple output method using ~watch~
  #+BEGIN_SRC bash
  watch -n 1 kill -USR1 $PID
  #+END_SRC

  The ~-n 1~ says to run the ~kill~ signal every second. The solution here is much cleaner, but reults
  in many lines of output. So, it's up to you what you want here.
