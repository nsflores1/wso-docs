# Using Git
## Commands you will probably learn out of habit
* `git pull origin <branch>` to get the latest version of a branch.
* `git fetch` if you don't have the latest branches.
* `git checkout <branch>` to switch to a branch.
* `git checkout -t origin/haml` -- checks out the `haml` branch from the remote repo known as `origin`. The `-t` flag is short for ``--track``, which allows you to simply use `git push` and `git pull` without specifying the remote and remote-branch. 
* `git add <file>` if you've created a new file or if you want to explicitly choose which files to commit.
* `git commit -am "<message>"` to commit your changes. Leave out the `a` if you chose specific files with `git add`.
* `git push origin <branch>` so the rest of us can see your changes.

## Creating a branch
When working on something, you should be on your own branch. When beginning a project, simply do `git checkout -b <branch_name>`. Push all of your changes there.

[Read more about branching.](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging)

## Viewing logs
```bash
git log --abbrev-commit --pretty=oneline
git log --abbrev-commit --pretty=oneline --graph
```

Alternatively, just alias `git log --graph --abbrev-commit --decorate --oneline --date=relative --all` to an abbreviation you'll remember.

## The Three States
[Read about the various states a file can be in here](http://stackoverflow.com/questions/7564841/concept-of-git-tracking-and-git-staging).

Generally, you create a "untracked" new files (`touch file.txt`). To "stage" a file, you do a `git add file.txt`. When you're ready to commit everything you've changed, do `git commit -am "message"`. The `-a` switch stands for `--all` and tells the command to automatically stage files that have been modified and deleted (but new files you haven't told Git about are not affected).

## Switching Branches without Committing
Say you're on a **feature-A**, did some work, don't want to commit that work yet, and also you want to switch to **feature-B**. You can stash your changes and recover them at a later time.
```bash
git stash
git checkout feature-B
git checkout feature-A
git stash list # list files stashed
git stash show # inspect files stashed
git stash apply # recover stashed changes
```
`apply` will keep the stash changes on top of the *stash stack*. Overall, you can [pop, drop, or apply](http://stackoverflow.com/a/15286090).

## Fixing that last commit message I screwed up
To amend the last commit message, `git commit --amend`.
If you want to amend the commit message that occurred 10 commits ago, use `git rebase -i HEAD~10`. This will open up interactive git-rebase, which will allow you to (among other things) amend the last 10 log messages.

## Viewing Diffs
To see the diff between the last commit and the current working directory, use `git diff`.

To see only *staged* changes, use `git diff --cached`.
# Rebasing is a life-saver
## Scenario
Bob is working on his feature branch. Alice pulls her feature branch into `master`. Bob's feature is now several commits behind `master`.
```
                     A---B---C topic
                    /
               D---E---F---G master
```

## Solution
What does Bob do?
```
git checkout master
git pull
git checkout bob-feature
git rebase master
```
And voila...
```
                             A'--B'--C' topic
                            /
               D---E---F---G master
```
