---
title: "Git merge two repositories (ZZ)"
date: 2018-11-23T18:56:00+08:00
draft: false
---

转自  <https://stackoverflow.com/questions/2428137/how-to-rebase-one-git-repository-onto-another-one>


 


If A and B are not the same repo (you created B by using the latest working copy you had), you have to use a [graft](https://git.wiki.kernel.org/index.php/GraftPoint) to pretend that they have common history.


Let’s assume you’ve added A as a remote for B as per [VonC’s answer](https://stackoverflow.com/a/2428224), and the repo looks like this1:




```
~/B$ git tnylog 
* 6506232 (HEAD, master) Latest work on B
* 799d6ae Imported backup from USB stick
~/B$ git tnylog A/master
* 33b5b16 (A/master) Head of A
* 6092517 Initial commit
```


Create a graft telling the root of B that its parent is the head of A:




```
echo '799d6aeb41095a8469d0a12167de8b45db02459c 33b5b16dde3af6f5592c2ca6a1a51d2e97357060' \
 >> .git/info/grafts
```


Now the two histories above will appear as one when you request the history for B. Making the graft permanent is a simple `git filter-branch` with no arguments. After the filter-branch, though, you aren’t on any branch, so you should `git branch -D master; git checkout -b master`.




---


1 `git tnylog` = `git log --oneline --graph --decorate`


 


补充句话：Git太厉害了！！！


 


