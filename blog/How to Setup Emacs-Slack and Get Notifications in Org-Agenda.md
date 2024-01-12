---
share: true
title: How to Setup Emacs-Slack and Get Notifications in Org-Agenda
author: Erich L Foster
email: erichlf AT gmail DOT com
date: 2020-09-09 Wed
tags:
  - slack
  - org-agenda
  - emacs
---

How I got emacs-slack to work with org-agenda and stop worrying
* note:
  Thanks to [[https://ag91.github.io/blog/2020/08/14/slack-messages-in-your-org-agenda/][Andrea (ag91)]] for the initial work he provided.

* [[https://github.com/yuya373/emacs-slack][emacs-slack]] Oh My!
There is a nice little bit of code that allows you to run slack inside emacs. Seriously, it is pretty nice.
However, one thing that is lacking when using slack in emacs is being notified of messages. For me this is
a problem, and I am sure there is someone out there thinking, "wait, this is a problem?" Anyway, the
following will, hopefully, give all the details you need to get things up and running.

* Setting up emacs-slack
** Install emacs-slack
   As I have stated in previously posts, I am using [[https://www.spacemacs.org][spacemacs]] and so installation of emacs-slack consists of
   adding ~slack~ to your ~dotspacemacs-configuration-layers~. For what ever distribution you are using you
   will need to look into how to install packages.

** Slack settings
   First thing to do is get your token, which you can do by
   1. Using Chrome, open and sign into the slack customization page, e.g. https://my.slack.com/customize
   2. Right click anywhere on the page and choose "inspect" from the context menu. This will open the
      Chrome developer tools.
   3. Find the console (it's one of the tabs in the developer tools window)
   4. At the prompt ("> ") type the following: window.prompt("your api token is: ", TS.boot_data.api_token)
   5. Copy the displayed token elsewhere, and close the window.

   Once you have your token the next thing that needs to be done is setup a login for slack. I don't like
   to make UIDs and such public, so I am using [[https://www.passwordstore.org/][pass]] to save my information, including passwords, as encrypted
   files on my system and then using ~password-store-get~ to access that information. However, you can just
   enter any information you like as plane text.
   #+BEGIN_SRC emacs-lisp
      ;; setup slack
      (slack-register-team
        :name <your-slack-team>
        :default t
        :client-id (password-store-get <path-to-uid>)
        :client-secret (password-store-get <path-to-password>)
        :token (password-store-get <path-to-slack-token>)
        :full-and-display-names t
        :subscribed-channels '(<a list of channels you want to be notified about>))
   #+END_SRC

   Now that you have your slack login information setup you might want to have some settings to make things
   easier. One thing emacs-slacks loves to do is ask what team it should use (all the time), so that might
   be something you want to avoid.
   #+BEGIN_SRC emacs-lisp
      (setq slack-prefer-current-team t)  ;; stop asking me which team to use
   #+END_SRC

   A nice timestamp was also something that I really wanted, so let's add that.
   #+BEGIN_SRC emacs-lisp
      ;; display a nice timestamp in slack
      (setq lui-time-stamp-format "[%Y-%m-%d %H:%M]")
      (setq lui-time-stamp-only-when-changed-p t)
      (setq lui-time-stamp-position 'right)
   #+END_SRC

   Another thing that annoyed me was that in spacemacs the ~:~ key was set
   as a shortcut to insert an emoji, well I am a C++ programmer so that was getting quite annoying.
   #+BEGIN_SRC emacs-lisp
      (evil-define-key 'insert slack-mode-map (kbd ":") nil)  ;; don't insert emoji
      (evil-define-key 'insert slack-message-buffer-mode-map (kbd ":") nil)  ;; don't insert emoji
      (evil-define-key 'insert slack-thread-message-buffer-mode-map (kbd ":") nil)  ;; don't insert emoji
   #+END_SRC

   And for some reason spacemacs didn't provide a shortcut for deleting a slack message, so let's add that too.
   #+BEGIN_SRC emacs-lisp
      (define-key slack-mode-map (kbd "C-c C-d") #'slack-message-delete)
   #+END_SRC

   Finally, let's make sure that slack starts
   #+BEGIN_SRC emacs-lisp
      (slack-start)  ;; start slack when opening emacs
   #+END_SRC

   For me this is everything I have to setup emacs-slack in a reasonable way.

* What About Notifications?
  While emacs-slack is pretty sweet, it does lack the ability to notify you when a new message shows up.
  Of course, you can use ~slack-all-unread~ (~SPC a c s u~ in spacemacs), but for some reason this wasn't
  enough bells and whistles for me. However, we have org-agenda, so why not use that to display direct
  messages?

  First thing to do is create ~slack.org~ file. I put this in my org directory, which is located in
  ~$HOME/org/~. So change the following to fit your setup. In this ~slack.org~ file we will create the
  following list of TODO states
  #+BEGIN_SRC emacs-lisp
  todo: UNREAD(u) | READ(r)
  #+END_SRC
  This tells org-mode that the UNREAD state is the only TODO state, while the READ state is the only
  closed state. It also says not to log state changes.

  Now add that org file to your ~org-agenda-files~
  #+BEGIN_SRC emacs-lisp
  (setq org-agenda-files (list "~/org/slack.org" "your other org files you want in the agenda"))
  #+END_SRC

  The rest will require us to setup [[https://github.com/jwiegley/alert][alert]] (which is included in spacemacs by default). To do this we
  define a style for alert to use when dealing with slack messages. This style will create TODO items
  in a ~slack.org~ file of your choosing (I put it in ~$HOME/org/~). Alert will get the message from
  emacs-slack and create a TODO item that I mark as UNREAD. It also limits the message length to 127
  characters.
  #+BEGIN_SRC emacs-lisp
    ;; setup org-agenda to keep track of unread messages in slack
    (alert-define-style
      'my/alert-style :title
      "Make Org headings for messages I receive - Style"
      :notifier
      (lambda (info)
        (when (get-buffer "slack.org") (with-current-buffer "slack.org" (save-buffer)))
        (write-region
          (s-concat
            "* UNREAD "
            (plist-get info :title)
            " : "
            (format "%s %s" (plist-get info :title)
                            (s-truncate 127 (plist-get info :message)))
            "\n"
            (format "<%s>" (format-time-string "%Y-%m-%d %H:%M"))
            "\n")
          nil
          "~/org/slack.org"
          t)))
    (setq alert-default-style 'message)
    (add-to-list 'alert-user-configuration
      '(((:category . "slack")) my/alert-style nil))
  #+END_SRC

  With the above setup will be able to see your slack messages show up in org-agenda. Hooray!

* Okay, but How do I Mute These Things?
  Now that we can display or direct messages in org-agenda we probably want a way to mark them as READ
  and stop displaying it in our agenda. To do this we are going to add a hook to ~org-agenda-mode-hook~
  that will provide the keybinding ~T~. This keybinding will mark our message as READ and subsequently
  archive it. One annoyance with doing this is that every time you want to do anything emacs will ask
  if you want to save the ~slack.org~ and ~slack.org_archive~ buffers. To prevent this we can just
  save those buffers any time we hit ~T~. Be aware this will save those buffers even if the item you
  are using ~T~ on in org-agenda isn't a slack item.
  #+BEGIN_SRC emacs-lisp
    (add-hook 'org-agenda-mode-hook (lambda () (local-set-key (kbd "T") 'my/org-agenda-todo-archive)))

    ;; my functions follow
    (defun my/save-slack ()
      "Save slack buffers"
      (interactive)
      (save-excursion
        (dolist (buf '("slack.org_archive" "slack.org"))
          (set-buffer buf)
          (if (and (buffer-file-name) (buffer-modified-p))
            (basic-save-buffer)
            )
          )
        )
      )

    (defun my/org-agenda-todo-archive ()
      "Mark an agenda item as done, archive it, and save the slack buffers"
      (interactive)
      (org-agenda-todo 'done)
      (org-agenda-archive)
      (my/save-slack)
      )
  #+END_SRC
