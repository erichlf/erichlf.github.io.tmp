---
share: true
title: Speed Up Spacemacs using Emacs as a Daemon
author: Erich L Foster
email: erichlf AT gmail DOT com
date: 2020-12-09 Wed
tags:
  - spacemacs
  - emacs
  - daemon
  - ubuntu
---

How to speed up Spacemacs using emacs in daemon mode on Ubuntu

* The Problem:
  Spacemacs takes quite a while to load when you have added all sorts of nice modes. I have had other issues
  with multiple emacs open causing issues with my slack setup.

* The Fix:
  Run spacemacs with emacs in daemon mode.

* How to Run Spacemacs in Daemon Mode on Ubuntu:

  The cleanest and easiest way to do this is to use ~systemd~. So we will set up them emacs daemon to start
  when a user logs in. To do this we first need to create a user specific systemd folder and then create a unit file for the emacs daemon.
  1. ~mkdir $HOME/.config/systemd/user~
  2. Then add the following to the unit file at ~$HOME/.config/systemd/user/emacs.service~
     #+begin_src
       [Unit]
       Description=Emacs text editor
       Documentation=info:emacs man:emacs(1) https://gnu.org/software/emacs/

       [Service]
       Type=simple
       ExecStart=/usr/bin/emacs --fg-daemon
       ExecStop=/usr/bin/emacsclient --eval "(kill-emacs)"
       Environment=SSH_AUTH_SOCK=%t/keyring/ssh
       Restart=on-failure

       [Install]
       WantedBy=default.target
     #+end_src
  3. Now you can go ahead and run the daemon:
     ~systemd --user enable --now emacs~

  If you wanted you could leave it at that and then just run emacs using ~emacsclient -c -a emacs~. This will
  create a new frame and if the emacs daemon isn't running it will use emacs. If you don't want to run emacs
  in a new frame just use ~-t~ instead of ~-c~. However, no one really wants to run that every time you want
  to edit a file.

  To address this issue the first thing to do would be to add the following to your ~.bashrc~
  #+begin_src bash
    alias emacs='emacsclient -t -a emacs'
    export EDITOR='emacsclient -t -a emacs'
    export VISUAL='emacsclient -c -a emacs'
  #+end_src
  Next we will add a desktop file for emacsclient at ~$HOME/.local/share/application/emacsclient.desktop~ containing
  #+begin_src
    [Desktop Entry]
    Name=Emacs (Client)
    GenericName=Text Editor
    Comment=Edit text
    MimeType=text/english;text/plain;text/x-makefile;text/x-c++hdr;text/x-c++src;text/x-chdr;text/x-csrc;text/x-java;text/x-moc;text/x-pascal;text/x-tcl;text/x-tex;application/x-shellscript;text/x-c;text/x-c++;
    Exec=emacsclient -c -a "emacs" %F
    Icon=emacs
    Type=Application
    Terminal=false
    Categories=Development;TextEditor;Utility;
    StartupWMClass=Emacs
    Keywords=Text;Editor;
  #+end_src

  One thing to be sure of after all this is that any special fonts are install system wide. To do this place
  the otf files in ~/usr/local/share/fonts~ and then run ~fc-cache -f~.
