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
vim .git/info/sparse-checkout       # modify sparse checkout config
git stash save                      # optionally save cwd
git checkout HEAD                   # check out the latest commit, applying sparse checkout changes
```

File format, from [Git documentation](https://git-scm.com/docs/git-read-tree):

> While $GIT_DIR/info/sparse-checkout is usually used to specify what files are in, you can also specify what files are not in, using negate patterns. 
> For example, to remove the file unwanted:
> 
> ```
> /*
> !unwanted
> ```
> 
> Another tricky thing is fully repopulating the working directory when you no longer want sparse checkout. 
> You cannot just disable "sparse checkout" because skip-worktree bits are still in the index and your working directory is still sparsely populated. 
> You should re-populate the working directory with the $GIT_DIR/info/sparse-checkout file content as follows:
> 
> ```
> /*
> ```
>
> Then you can disable sparse checkout. Sparse checkout support in git read-tree and similar commands is disabled by default. You need to turn core.sparseCheckout on in order to have sparse checkout support.

# Submodules

Change URL: change in `.gitmodules`, then `git submodule sync`.

Sparse checkout for submodules:
```bash
git submodule add $url (<name>)                 # add submodule
git -C <name> config core.sparseCheckout true   # setup sparse checkout for submodule
git submodule absorbgitdirs                     # absorb submodule .git dir
vim .git/modules/<name>/info/sparse-checkout    # modify the sparse checkout file for submodule
git submodule update --force --checkout         # update, removing excluded dirs
cd <name> && git checkout master                # move back to submodule branch
cd .. && git add . && git commit && git push    # push commit with added submodule
```

# Rebase onto
```bash
git rebase --onto newbase from_commit to_commit
```
```bash
git checkout branch
git rebase --onto newbase from_commit     # will rebase up to latest commit on ‘branch'
```

# Bisect
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

# Blame
```bash
git blame <file>                # find out whodunnit
```

# Update gitignored files

If you previously committed a file/folder, then decided to added it to `.gitignore`, you have to run:

```bash
git rm -r --cached .
```

Do this in the root. But **make sure you saved/committed any changes to code**. Then push force with lease.

# Remove sensitive data from all commits

There's a way to do it through git, using git-filter-branch, but it's a bit tedious. Just get [bfg](https://rtyley.github.io/bfg-repo-cleaner/).
