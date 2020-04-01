# Setup
```git
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com

jikangs-MBP:basics jikangwang$ git config --list
credential.helper=osxkeychain
user.name=jikangwang
user.email=wjk32111@gmail.com
core.repositoryformatversion=0
core.filemode=true
core.bare=false
core.logallrefupdates=true
core.ignorecase=true
core.precomposeunicode=true
remote.origin.url=https://github.com/zzzyyyxxxmmm/basics.git
remote.origin.fetch=+refs/heads/*:refs/remotes/origin/*
branch.master.remote=origin
branch.master.merge=refs/heads/master
```

# Basic
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/git_state.png" width="500" height="300">
</div>

## gitignore
```git
# no .a files
*.a
# but do track lib.a, even though you're ignoring .a files above
!lib.a
# only ignore the TODO file in the current directory, not subdir/TODO
/TODO
# ignore all files in the build/ directory
build/
# ignore doc/notes.txt, but not doc/server/arch.txt
doc/*.txt
# ignore all .pdf files in the doc/ directory
doc/**/*.pdf
```

## git commit
```
Adding the -a option to the git commit command makes Git automatically stage every file that is **already tracked** before doing the commit, letting you skip the git add part:

git commit -a -m 'added new benchmarks'
```

### undoing things
```
git commit --amend
```

## git rm 
Another useful thing you may want to do is to keep the file in your working tree but remove it from your staging area. In other words, you may want to keep the file on your hard drive but not have Git track it anymore. This is partic- ularly useful if you forgot to add something to your .gitignore file and acci- dentally staged it, like a large log file or a bunch of .a compiled files. To do this, use the --cached option:
```
git rm --cached README
```

## git log
One of the more helpful options is -p, which shows the difference intro- duced in each commit. You can also use -2, which limits the output to only the last two entries:
```
jikangs-MBP:basics jikangwang$ git log -p -2
commit 802c8291acf73779c55424e630cff5bcb4c8d359 (HEAD -> master, origin/master, origin/HEAD)
Author: jikangwang <wjk32111@gmail.com>
Date:   Sun Jan 26 11:05:13 2020 -0500

    add git

diff --git a/README.md b/README.md
index 2e0c6df..ff74bf0 100644
--- a/README.md
+++ b/README.md
@@ -35,11 +35,14 @@
 2. The Linux Programming Interface 基于这本书开始学习linux
 3. Intellij Plugin Development √
 4. 鸟哥 √
-5. Kafka definitive guide
+5. Kafka definitive guide √
 6. 基于反射实现一个go sort library. TDD开发
 
 ### FEB, 2020
 1. 6.824 Lab3: Fault-tolerant Key/Value Service
 2. HTTP - The Definitive Guide
-3. MongoDB Definitive Guide, 顺便温习数据库
-4. Pro Git
```

```
jikangs-MBP:basics jikangwang$ git log --stat
commit 802c8291acf73779c55424e630cff5bcb4c8d359 (HEAD -> master, origin/master, origin/HEAD)
Author: jikangwang <wjk32111@gmail.com>
Date:   Sun Jan 26 11:05:13 2020 -0500

    add git

 README.md                              |  9 ++++++---
 git/README.md                          |  9 +++++++++
 microservice/docker/advanced_docker.md | 12 ++++++++++++
 3 files changed, 27 insertions(+), 3 deletions(-)

commit bda04690f602c88c908819e2dd54f090580bb3e8
Author: jikangwang <wjk32111@gmail.com>
Date:   Fri Jan 24 17:42:03 2020 -0500

    kafka last

 microservice/kafka/README.md | 5 +++++
 1 file changed, 5 insertions(+)

commit 9cb9c77d32adad1ac401d8ad203919e8a3ebe398
Author: jikangwang <wjk32111@gmail.com>
```

```
jikangs-MBP:basics jikangwang$ git log --pretty=oneline
802c8291acf73779c55424e630cff5bcb4c8d359 (HEAD -> master, origin/master, origin/HEAD) add git
bda04690f602c88c908819e2dd54f090580bb3e8 kafka last
9cb9c77d32adad1ac401d8ad203919e8a3ebe398 :)
c19ac503f8a17f2889c7ca33bdc936e19107b534 :)
bb976fd925923c7db72c1456d615bdc3977727b1 :)
19c2a446549d1373b33e4e749028f5031bb22c42 :)
c5e47f82d013acf3c9c2b2a3414093d245ba2218 :)
ac4a16b4090a50499e616d606a93dab083b0ca69 :)'
11b5bcc3d8b3cf4954f8e7ca1092d6271b2406f3 :)
```

```
git log --pretty="%h - %s" --author=gitster --since="2008-10-01" \ --before="2008-11-01" --no-merges -- t/
5610e3b - Fix testcase failure when extended attributes are in use
acd3b9e - Enhance hold_lock_file_for_{update,append}() API
f563754 - demonstrate breakage of detached checkout with symbolic link HEAD
```

## git remote
This command shows which branch is automatically pushed to when you run git push while on certain branches. It also shows you which remote branches on the server you don’t yet have, which remote branches you have that have been removed from the server, and multiple local branches that are able to merge automatically with their remote-tracking branch when you run git pull.
```
jikangs-MBP:basics jikangwang$ git remote show origin
* remote origin
Fetch URL: https://github.com/zzzyyyxxxmmm/basics.git
Push  URL: https://github.com/zzzyyyxxxmmm/basics.git
HEAD branch: master
Remote branch:
master tracked
Local branch configured for 'git pull':
master merges with remote master
Local ref configured for 'git push':
master pushes to master (up to date)

$ git remote
origin
$ git remote add pb https://github.com/paulboone/ticgit $ git remote -v
origin https://github.com/schacon/ticgit (fetch) origin https://github.com/schacon/ticgit (push)
pb https://github.com/paulboone/ticgit (fetch)
pb https://github.com/paulboone/ticgit (push)
$ git fetch pb
remote: Counting objects: 43, done.
remote: Compressing objects: 100% (36/36), done. remote: Total 43 (delta 10), reused 31 (delta 5)
```

## git branch

```
jikangs-MBP:gitTest jikangwang$ git branch -d branch1
Deleted branch branch1 (was 75765bf).

jikangs-MBP:gitTest jikangwang$ git branch -v
  branch3 d1cc24e branch3
* master  b20bb00 [ahead 4] Merge branch 'branch1'


git checkout -b iss53 =  $ git branch iss53 $ git checkout iss53
```

## git fetch

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/git_1.png" width="700" height="500">
</div>

Let’s say you have a Git server on your network at git.ourcompany.com. If you clone from this, Git’s clone command automatically names it origin for you, pulls down all its data, creates a pointer to where its master branch is, and names it origin/ master locally. Git also gives you your own local master branch starting at the same place as origin’s master branch, so you have something to work from.

```
git fetch origin #updates your remote references
git fetch -- all #所有的remote, 一般不用, 除非你指向了很多remote
git checkout -b serverfix origin/serverfix # 然后用checkout创建local分支
```

It’s important to note that when you do a fetch that brings down new remote-tracking branches, you don’t automatically have local, editable copies of them.

### git fetch 和 git pull的区别
While the git fetch command will fetch down all the changes on the server that you don’t have yet, it will not modify your working directory at all. It will simply get the data for you and let you merge it yourself. However, there is a command called git pull which is essentially a git fetch immediately fol- lowed by a git merge in most cases. If you have a tracking branch set up as demonstrated in the last section, either by explicitly setting it or by having it created for you by the clone or checkout commands, git pull will look up what server and branch your current branch is tracking, fetch from that server and then try to merge in that remote branch.
Generally it’s better to simply use the fetch and merge commands explicitly as the magic of git pull can often be confusing.

## git push
```
git push origin server-fix:awesomebranch #push your local serverfix branch to the awesome- branch branch on the remote project.
git push origin --delete serverfix #删除远端分支
```

## git rebase 
```
1 <- 2 <- 3(master)
       <- 4(hotfix)

$ git checkout experiment
$ git rebase master
1 <- 2 <- 3(master) <- 4(hotfix)


$ git checkout master 
$ git merge experiment   
1 <- 2 <- 3 <- 4(hotfix, master)

```

### Rebase vs. Merge
git pull可能会遇到冲突
You can also simplify this by running a git pull --rebase instead of a normal git pull. Or you could do it manually with a git fetch followed by a git rebase teamone/master in this case.
If you are using git pull and want to make --rebase the default, you can set the pull.rebase config value with something like git config --global pull.rebase true.

# .gitignore
```
# no .a files
*.a
# but do track lib.a, even though you're ignoring .a files above
!lib.a
# only ignore the TODO file in the current directory, not subdir/TODO
/TODO
# ignore all files in the build/ directory
build/
# ignore doc/notes.txt, but not doc/server/arch.txt
doc/*.txt
# ignore all .pdf files in the doc/ directory
doc/**/*.pdf
```

# tips
HEAD 指向的是branch, branch指向某个commit