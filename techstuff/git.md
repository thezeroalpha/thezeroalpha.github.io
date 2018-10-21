---
title: Bash/Shell stuff
---

# Sparse checkout
initialise:

```bash
git init                                      # new empty repository
git remote add “$url”                         # add remote to repository
git config core.sparseCheckout true           # enable sparse checkout
echo “!thing” >> .git/info/sparse-checkout    # exclude anything matching “dir”
echo “/thing” >> .git/info/sparse-checkout    # include anything matching “dir” at root
echo “dir/“ >> .git/info/sparse-checkout      # include any directory matching “dir”
git pull                                      # pull from repository
```

modify:

```bash
vim .git/info/sparse-checkout    # modify sparse checkout config
[git stash save]                 # optionally save cwd
git checkout HEAD                # check out the latest commit, applying sparse checkout changes
```

# Rebase onto:
```bash
git rebase --onto newbase from_commit to_commit
git checkout branch
git rebase --onto newbase from_commit     # will rebase up to latest commit on ‘branch'
```

# Bisect:
```bash
git bisect start
git checkout <working commit>    # check out a good commit
git bisect good                  # mark as good
git checkout <broken commit>     # check out a bad commit
git bisect bad                   # mark as bad

# repeat...
git bisect next                  # this
git bisect good|bad              # and this until the causing commit is identified
```

# Blame:
git blame <file>                # find out whodunnit
