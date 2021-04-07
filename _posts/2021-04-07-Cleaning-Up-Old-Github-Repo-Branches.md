---
layout: post
title: "Cleaning Up Old Github Repo Branches"
date: 2021-04-07
mathjax: false
---
----------------

A while back I was searching for a specific branch in one of my Github repositories.
While scrolling through the branch list, it occurred to me that there were just too many branches to go through now (around 30 or so).
Many of these branches were from the initial stages of the project, debugging branches, branches made for backups and branches which just
had straight up wrong code. It was time for a clean-up.

Now, one could simply delete all those stale branches. But I would caution against it. When you have spent some time in the
research business you realise that the universe is just waiting for you to commit such an action. Like deleting your old code
without any backups. It will watch with glee as your professors, co-workers, paper reviewers and random people on the internet 
suddenly come out of the woodwork and start requesting you for the exact same code that you just deleted. Deleted because no one was using it!  

So we will tag these branches before deleting them. [Git](https://git-scm.com/book/en/v2/Git-Basics-Tagging) provides this feature and we will create a lightweight tag per branch. These lightweight tags are simply pointers to a specific commit, hence the term "lightweight".
Tagging will keep the code in these stale branches in your repository (...for posterity or close enough) but at least away from sight 
and from your branch list.

Full confession, most of the commands given below this sentence you're reading is from [this Stackoverflow query](https://stackoverflow.com/questions/1307114/how-can-i-archive-git-branches), so make sure to check it out.  

First get a fresh clean clone of the desired repository to be pruned.
```bash
git clone YOUR_REPOSITORY.git
```

Now a simple clone command will just clone the master (or as the young kids call it these days, the main branch). We want all the branches.
Hence inside the newly cloned directory, we track all the branches

```bash
for remote in `git branch -r`; do git branch --track ${remote#origin/} $remote; done
```

Tag the required branch directed for future deletion.
```bash
git tag archive/<branchname> <branchname>
```

Delete the old, decrypt branch
```bash
 git branch -D <branchname>
```

Delete the old, decrypt branch...from your remote (i.e the Github repository)
```bash
git branch -d -r origin/<branchname>
```

Push the tags to the remote
```bash
git push --tags
```

Push changes to remote
```bash
git push origin :<branchname>
```

And you are done. Now make a shell/bash script to automate the above steps if you have too many branches. May your code be safe, secured and look up-to-date.


PS - To get a branch back from a tag
```bash
git checkout -b <branchname> archive/<branchname>
```
