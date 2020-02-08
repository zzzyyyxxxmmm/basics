# Setup
```git
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com

git config --list
```

# Basic
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/git_state.png" width="500" height="300">
</div>

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
```

## git branch

```
jikangs-MBP:gitTest jikangwang$ git branch -d branch1
Deleted branch branch1 (was 75765bf).

jikangs-MBP:gitTest jikangwang$ git branch -v
  branch3 d1cc24e branch3
* master  b20bb00 [ahead 4] Merge branch 'branch1'


```


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
