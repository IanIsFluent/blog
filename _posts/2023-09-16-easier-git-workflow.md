---
tags:
  - git
---

# Easier and cleaner git workflow

I have recently fallen in love with `git rebase` ❤️. It's a great way to keep your git history clean and tidy by making it look like work was done in a simple linear fashion, even if it wasn't. Now if I have work locally, I want to always rebase it onto the latest changes from the remote branch, rather than merge. I wondered if I could make this happen automatically when I pull in VSCode?

## Auto rebase

[StackOverflow to the rescue!](https://stackoverflow.com/a/38911284) (As usual!).

You can [set git config to rebase whenever pulling](https://git-scm.com/docs/git-config#Documentation/git-config.txt-pullrebase) using `git config pull.rebase true`.

## Auto stash

But then you can't pull without first committing your changes. This is a pain if you don't want to or can't keep your working directory clean.

To deal with this you can [set another git config setting to stash whenever rebasing](https://git-scm.com/docs/git-config#Documentation/git-config.txt-rebaseautoStash) `git config rebase.autoStash true`. This will stash your changes before pulling, and then reapply them after the pull is complete.

## Set per repo

Now I can pull and rebase easily without having to commit my changes, and my git history is clean and simple by setting these two config options on each repo:

```
git config pull.rebase true
git config rebase.autoStash true
```

Yay! 🙌
