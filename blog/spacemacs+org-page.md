---
share: true
title: spacemacs+org-page
author: Erich L Foster
email: erichlf AT gmail DOT com
date: 2020-08-19 Wed
tags:
  - org-page
  - spacemacs
  - org-mode
---

Using org-page and spacemacs
Note: I took a good portion of this from [[http://adlawren.github.io/blog/2018/05/20/using-org-page-to-publish-to-github-pages-in-spacemacs/][here]].
However, I felt some more details needed to be added.

* Installing [[https://github.com/erichlf/org-page][org-page]]:
First thing to do is to add org-page to the ~additional-packages~ portion of your ~.spacemacs~ file
#+BEGIN_SRC emacs-listp
    (org-page :location (recipe
                          :fetcher github
                          :repo "sillykelvin/org-page"
                          :files ("*.el" "doc" "themes")))
#+END_SRC
Next add the following to the ~user-config~ portion of your ~.spacemacs~ file
#+BEGIN_SRC emacs-lisp
  (require 'org-page)
  (setq op/repository-directory "/path/to/your/repo")
  (setq op/site-domain "https://githubuser.github.io")
  (setq op/personal-github-link "https://github.com/githubuser")
  (setq op/site-main-title "TITLE")
  (setq op/site-sub-title "SUBTITLE")
  (setq user-full-name "YOUR NAME")  ;; not org-page specific
  (setq user-mail-address "your@email.com")  ;; not org-page specific
#+END_SRC

* Create Your Repo:
  Create your github.io page using the following [[https://docs.github.com/en/github/working-with-github-pages/creating-a-github-pages-site][directions]].

  Then run ~SPC SPC op/new-repository~ and specify the location you want for your repo.
  This should probably be the same as what you put in your ~.spacemacs~ file.

  In this same location you will want to add the remote, which can be done using magit.
  For me the easiest way to do this is through magit-status (~SPC g s~) and while in
  magit-status type ~M a~. From there you can just follow the prompts to add your remote.

* Create Your First Blog Post:
  This is as simple as ~SPC SPC op/new-post~ and following the prompts.

* Publish Your Page:
  This should be as simple as ~SPC SPC op/do-publication~ and following the prompts,
  which is likely to result in you pushing ~y n y y~.
