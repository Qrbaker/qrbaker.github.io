---
title: "Considering the Philosophy of Squashing"
date: 2022-01-02T10:24:27-04:00
tags: [git,development,philosophy]
summary: 'Linear history at what cost? If we should clean up our local histories before merges/patches, how best to do it?'
draft: false 
---

I have a sort of "git philosophy" discussion to pose. Recently, I had a massive headache doing an interactive 
rebase because I have a habit of periodically merging upstream master/dev/main/whatever branch commits into my 
working branch to keep the work of resolving merge conflicts to a minimum.  
Unfortunately, when you have the situation of:

```text
a----b----c----d----e----f
                \         \
        g----h----i----j----k----l
```

doing something like `git rebase -i g` will try and rewrite these pulls (a/b/c and d/e/f), even if you leave 
them as "picked" commits, and you only mark your commits (g/h/i/j/k/l) as "squash". I really, really hate the 
idea of effectively changing/rewriting other developer's commits; it can screw up stuff like git logs and git 
blames.

My workaround, of sorts, as been to:

1. checkout a new branch from the upstream (`git checkout -b merge-temp develop`)

1. do a "squash merge" to that branch from my working branch (`git merge --squash feature/my-branch merge-temp`)

1. reset my working branch to the resulting merge commit (`git checkout feature/my-branch && git reset --hard merge-temp`)

1. Delete the temp branch (`git branch -D merge-temp`)

This basically does the following:

```text
a----b----c----d----e----f
                \         \
       g----h----i----j----k----l
```

becomes

```text
a----b----c----d----e----f
                          \
                           g'
```

commits g through l get "rolled up" into a new commit, g', which can be merged. As a nice side-effect, a 
`git merge --squash` produces a very nice "squash summary" of each commit with its hash, author, and commit 
message in the squash commit message.  

---

Now, on doing some more reading about git best practices, this LWN article came to my attention: 
https://lwn.net/Articles/328436/. In it, it talks about how Linus Torvalds handles the Linux kernel merges:

>I want clean history, but that really means (a) clean and (b) history.
>People can (and probably should) rebase their _private_ trees (their own
>work). That's a _cleanup_. But never other peoples code. That's a "destroy
>history"
>So the history part is fairly easy. There's only one major rule, and one
>minor clarification:
>
> - You must never EVER destroy other peoples history. You must not rebase
>   commits other people did. Basically, if it doesn't have your sign-off
>   on it, it's off limits: you can't rebase it, because it's not yours.
>
>   Notice that this really is about other peoples _history_, not about
>   other peoples _code_. If they sent stuff to you as an emailed patch,
>   and you applied it with "git am -s", then it's their code, but it's
>   _your_ history.

   
It seems he feels that (a) people _should_ squash their local histories before committing upstream, 
and (b) you **should not** periodically merge from upstream.
That makes sense when you're dealing with hundreds of commits from dozens of contributors, but what 
about us mere mortal devs, who work with maybe 7 developers 
on a project at a time?  

I'd love to hear other, more experienced devs' thoughts.