---
share: true
title: ~git-subrepo~ vs ~git-subtree~
author: Erich L Foster
email: erichlf AT gmail DOT com
date: 2020-09-27 Sun
tags:
  - git
  - git-subrepo
  - git-submodule
  - git-tree
  - org-page
---

Simplifying ~git~ workflow using a ~git-subrepo~

* Some Background
  If you look into the menu above you will see that I have shared my recipes. I have had a ~git~ repo of my
  recipes for quite a while and when I decided to start a blog I thought this would be a good place to share
  those recipes. However, my blog is a separate repo from my recipes, so the question arose, "how should I
  sync these repos and get them to work together?" Well, my experience with doing such things has always been
  limited to submodules, except when I was working at Sandia National Labs, where subtrees were used. In that
  case someone else always took care of the details for that subtree, so I didn't really know how they worked.
  Anyway, I decided on using a submodule, since that is what I am use to.

  After getting all my recipes converted to org files (~org-page~ converts org files to html) and then adding
  my recipes as a submodule to my pages repo I noticed that while there was a link for recipes in the menu
  that link sent me to nowhere. After some reading I discovered that ~org-page~ determines what to publish by
  looking at ~git-diff~. Well, when you use a submodule the files associates with that submodule is not known
  to the main repo. All the main repo sees is a commit hash in the submodule.

* How do we solve this problem?
  We have two options to solve this problem:
  1. ~git-subtree~
  2. ~git-subrepo~

  Now I am not going to discuss all the short comings of any of these methods. I am just going to explain my
  reasoning for choosing the method I chose. To do this I will go through the process of setting things up in
  both tools and then I think it should become clear why one might want to choose a ~git-subrepo~ over a
  ~git-subtree~. For anyone with experience in ~git-submodules~ I believe the demonstration of ~git-subrepo~
  will also convince you why you might want to use a ~git-subrepo~ over a ~git-submodule~.

  This is not going to be an exhaustive explanation of how the ~git-subtree~ or ~git-subrepo~ works. I am
  going to setup things up as if I was setting up my blog and my recipes. I am going to assume that the blog
  repo and the recipe repo are already set up and we can just begin from there.

* Comparing ~git-subtree~ and ~git-subrepo~
  The first order of business is setting up ~git-subrepo~, since it is not an official part of ~git~ there
  is some work to be done here. The first part is simply cloning the repo and then the next part is adding
  a ~source~ line to your ~.bashrc~.
  #+begin_src bash
    git clone https://github.com/ingydotnet/git-subrepo
    echo "source $PWD/git-subrepo/.rc" >> ~/.bashrc
  #+end_src
  Now that ~git-subrepo~ is installed we can continue on with the demonstration.

  The location of my recipes will be in the root of my website and so we will want to add the recipes
  subtree or subrepo there.

  To add the subtree we would issue the following command
  #+begin_src bash
    git subtree add --prefix recipes git@github.com:erichlf/Recipes.git master --squash
  #+end_src
  The ~--squash~ is to make it where the history of ~recipes~ doesn't appear in the logs of the current repo.
  Instead, this will simply show up as a simple merge.

  To add the subrep we would issue the following
  #+begin_src bash
    git subrepo clone git@github.com:erichlf/Recipes.git recipes
  #+end_src
  So far things are too bad in either ~git-subtree~ or ~git-subrepo~, however the ~git-subtree~ command is
  already a little more verbose than I would like. To me the ~git-subrepo~ command is quite intuitive and is
  quite similar to commands we already know, e.g. ~git clone~.

  Now all my files are visible to the current repo. If something changes in the upstream to the subtree we can
  run the following command to pull in those changes:
  #+begin_src bash
    git fetch git@github.com:erichlf/Recipes.git master
    git subtree pull --prefix recipes git@github.com:erichlf/Recipes.git master --squash
  #+end_src
  Wow! that is verbose and quite ugly! Now, we are starting to see the problem with ~git-subtrees~. Who wants
  to type that garbage?

  On the other hand the same task can be accomplished using ~git-subrepo~ in the following way:
  #+begin_src bash
  git subrepo pull recipes
  #+end_src
  This is so much better than the ~git-subtree~ version. Again, this looks vary familiar and is way less
  verbose.

  Now the thing we should also consider here is what the logs look like... The above two actions will look
  as follows using ~git-subrepo~
  #+begin_quote
  50a8fcd git subrepo pull recipes
  002b0ba git subrepo clone git@github.com:erichlf/Recipes.git recipes
  #+end_quote

  On the other hand ~git-subtree~ logs will look like a merge
  #+begin_quote
  b80b296 (HEAD -> source) Merge commit 'a4077f7009c770e844fb765bdec9c567628b91e5' into source
  a4077f7 Squashed 'recipes/' changes from 5021cb2..8d40ae7
  2fcc780 Merge commit 'd7831056c1f029473bac95522d18b280fb722be7' as 'recipes'
  d783105 Squashed 'recipes/' content from commit 5021cb2
  #+end_quote

  Now say I make a change to a recipes while in my website repo... In fact, I did this right before writing
  this blog post. I had a problem with how I was adding tags to ~org-page~ files. The template in ~org-page~
  states that tags should be separated by ", ", however, tags should actually be surrounded by colons like so
  ":tag1:tag2:...:tagN:". Well, I had already converted all my recipes and I had a few blog posts which I had
  already written. To fix the issue I decided to do everything within the repo for my website. No, I didn't
  do this by hand. I simply ran a ~sed~ one-liner to change all org files. But now all these changes are in
  the website repo and not the recipes repo. So how do we get those changes to be seen by the recipes repo?

  First we just add and commit all changes like you would normally do. For me I want to commit all changes so
  #+begin_src bash
    git commit -am "Fix Tags"
  #+end_src
  Now that the changes have been committed to the main repo we need to push those changes to the recipes repo.
  To do this in ~git-subtree~ we run
  #+begin_src bash
    git subtree push --prefix recipes git@github.com:erichlf/Recipes.git master
  #+end_src
  while using ~git-subrep~ is
  #+begin_src bash
    git subrepo push recipes
  #+end_src
  Once again the ~git-subtree~ is much more verbose and can be quite annoying to type out.

  As for the logs the ~git-subtree~ logs are quite simple and don't really tell the whole story. In the main
  the log simply has the commit we made in our main repo and there is no indication that those changes were
  pushed to the recipes repo.
  #+begin_quote
  6ea8c9f (HEAD -> source) Fix Tags
  #+end_quote

  On the other hand ~git-subrepo~ is a bit more verbose in this instance. We see the commit to the main
  branch and one extra commit telling us that commit was pushed to the recipes repo.
  #+begin_quote
  8b5a5c4 (HEAD -> source) git subrepo push recipes
  580f0f4 Fix Tags
  #+end_quote
  and in the recipes repo there is no difference between the two methods:
  #+begin_quote
  5021cb2 (master) Fix Tags
  #+end_quote

  But what the heck is in that commit for the push to the recipes repo?
  #+begin_quote
diff --git a/recipes/.gitrepo b/recipes/.gitrepo
index 5844cb2..0cccb6a 100644
--- a/recipes/.gitrepo
+++ b/recipes/.gitrepo
@@ -6,7 +6,7 @@
 [subrepo]
        remote = git@github.com:erichlf/Recipes.git
        branch = master
-       commit = 64c15f2076fe8b866b9a61a2f40d801946428857
-       parent = 002b0ba072a92067ecec940563584c6a8f310fbb
+       commit = 5021cb204abd271718021aed8b942d397a3364b5
+       parent = 580f0f4d498198278385b22a24cec84ae856a7b2
        method = merge
        cmdver = 0.4.1
  #+end_quote
Apparently, there is a ~.gitrepo~ file that tracks commits within your subrepo and this file is what
gets changed when you push to the subrepo.

* Conclusions
  For me this was an easy decision... ~git-subrepo~ was more intuitive and simpler. I like less verbosity
  when typing out commands and more verbosity in the push commits. It is nice to know when something has
  been pushed upstream. I am sure there are people that will disagree and well isn't it nice to have
  choices.
