# Understanding Git 

by <a href="https://twitter.com/tednaleid">@tednaleid</a>

!SLIDE shout

# Git 이해하기 

## by <a href="https://twitter.com/tednaleid">@tednaleid</a>

!SLIDE quietest shout
# Commits은 불변이다.

## commits은 수정할 수 없다.<br/><br/>새로운 commits을 추가할 수만 있다.

!SLIDE quietest shout 
# commits은 사라지지 않는다.

## 의도치않게 commits을 제거하는 것은  _불가능_ 하다.<br/><br/>하지만 <code>rm -rf .git</code> 을 하면 push하지 않은 commits은 사라진다.

!SLIDE quietest shout 
# 빨리 &amp; 자주

## 커밋하지 않은 작업은 쉽게 사라지니, 빨리 &amp; 자주 commit하자.

!SLIDE quietest shout
# garbage collection?

## garbage collection은 유일한 파괴적인 git action이다.

!SLIDE quietest shout
# garbage collection?

## garbage collection은 point가 없는 commits만 제거한다.

!SLIDE 
# points? 

# other commits

# tags 

# branches 

# the reflog

!SLIDE 
<br/>
# commit

point at 0..N parent commits 

```
                          E---F---G
                         /         \
                    A---B---C---D---H---I
```

대부분 하나나 두개의 parent commits 을 가지고 있다.

!SLIDE 

# tag 
고정된 commit pointers

```
                      A---B---C 
                              ↑
                         release_1.0
```

```
% git commit -m "adding stuff to C"
```

```
                      A---B---C---D 
                              ↑
                         release_1.0
```

!SLIDE 
# branch 

유동적인 commit pointer

```
                      A---B---C
                              ↑
                            master
```

```
% git commit -m "adding stuff to B"
```

```
                      A---B---C---D
                                  ↑
                                master
```

!SLIDE 
# remote branch 
 &#8220;remote&#8221; branch 는 local branch에 있는 하나의 포인터일 뿐인다. 

```
                                 master
                                    ↓
                    A---B---C---D---E
                            ↑       
                      origin/master    
```

`fetch` 나 `pull` 을 할때 업데이트된다. 그 외에는 아무 변화도 없다. 

!SLIDE 
# branch

`.git` 하위 디렉토리에 텍스트 파일로 되어있다.

```
% ls -1 .git/refs/heads/**/*
.git/refs/heads/master
.git/refs/heads/my_feature_branch
```

```
% ls -1 .git/refs/remotes/**/*  
.git/refs/remotes/origin/HEAD
.git/refs/remotes/origin/master
.git/refs/remotes/origin/my_feature_branch
```

!SLIDE 
# branch 

pointing하고 있는 커밋의 SHA 텍스트가 들어있다.

```
% cat .git/refs/heads/master 
0981e8c8ffbd3a1277dda1173fb6f5cbf4750d51

# .git/objects/09/81e8c8ffbd3a1277dda1173fb6f5cbf4750d51
```

!SLIDE 
# branches point at commits 

`tree` (filesystem), `parent` commits, commit metadata를 포함한다.
```
% git cat-file -p 0981e8c8ffbd3a1277dda1173fb6f5cbf4750d51
tree 4fd7894316b4659ef3f53426166697858d51a291
parent e324971ecf1e0f626d4ba8b0adfc22465091c100
parent d33700dde6d38b051ba240ee97d685afdaf07515
author Ted Naleid <contact@naleid.com> 1328567163 -0800
committer Ted Naleid <contact@naleid.com> 1328567163 -0800

merge commit of two branches
```

The ID is the SHA of the commit's contents

!SLIDE 
<br/>
# branches 
commits은 branches에 &#8220;종속&#8221; 되지 않는다. commit meta 데이터에는 branches에 대한 내용은 없다.

!SLIDE 
# branches 
branch의 commits은 branch point의 족보다.

```
                                 feature
                                    ↓
                            E---F---G 
                           /
                      A---B---C---D 
                                  ↑ 
                                master
```

<code>master</code> 는 <code>A-B-C-D</code> 고 <code>feature</code> 는 <code>A-B-E-F-G</code>

!SLIDE
# HEAD 

<code>HEAD</code> 는 현재 branch의 commit이다.

HEAD는 다음 커밋의 부모가 된다.

```
% cat .git/HEAD
ref: refs/heads/master
```

대부분 branch를 point하지만 detach됐을 경우 SHA를 point한다.


!SLIDE 
# the reflog 
<code>HEAD</code>이동 로그이다

```
% git reflog                                       
d72efc4 HEAD@{0}: commit: adding bar.txt
6435f38 HEAD@{1}: commit (initial): adding foo.txt
```

```
% git commit -m "adding baz.txt"
```

```
% git reflog                                       
b5416cb HEAD@{0}: commit: adding baz.txt
d72efc4 HEAD@{1}: commit: adding bar.txt
6435f38 HEAD@{2}: commit (initial): adding foo.txt
```

기본적으로 30일간 history가 보관된다.


!SLIDE 
<br/>
# the reflog 
unique to a repository instance

!SLIDE 
# the reflog 
개별 branch scope를 가진다.

```
% git reflog my_branch
347f5fe my_branch@{0}: merge master: Merge made by the recurs… 
4e6007e my_branch@{1}: merge origin/my_branch: Fast-forward
32834d8 my_branch@{2}: commit (amend): upgrade redis version
2720e40 my_branch@{3}: commit: upgrade redis version 
```

!SLIDE 
<br/>
# dangling commit 
commit을 point하는 것이 reflog뿐이라면 이 커밋은 dangling이다.

!SLIDE 
# dangling commit 
```
                    A---B---C---D---E---F
                                        ↑
                                      master
```

```
% git reset --hard SHA_OF_B
```

```
                    A---B---C---D---E---F
                        ↑
                      master
```

<code>C..F</code> 는 dangling

!SLIDE 
# dangling commit 

reflog덕분에 30일동안은 남아있다.

```
                                     HEAD@{1}
                                        ↓
                    A---B---C---D---E---F
                        ↑
                     master (also HEAD@{0})

```

reflog가 추가되면 <code>HEAD@{1}</code> 는 <code>HEAD@{2}</code>..<code>HEAD@{N}</code> 가 된다.

!SLIDE 
<br/>
# garbage collection 
once a dangling commit leaves the reflog, it is &#8220;loose&#8221; and is at risk of garbage collection

!SLIDE 
<br/>
# garbage collection 
loose objects가 1000개 이상이 되면 git은 gc를 한다.

!SLIDE 
<br/>
# garbage collection 
gc되는 것을 막고 싶으면 point를 만들면 된다.

```
% git tag mytag SHA_OF_DANGLING_COMMIT
```

!SLIDE

# the index 

commit전의 staging area

<code>add -A :/</code> 는 모든 변경사항을 index에 넣고 commit을 준비한다.

<code>git commit -a -m "msg"</code>은 index를통하지 않고 commit한다.

!SLIDE 
<br/>

# 실험정신

뭐가 잘못되어도 이전 커밋으로 되돌릴 수 있다.

!SLIDE shout

# where you are?

## 이동하기전에 현재 어디에 있는지(어느 branch, commit을 point하고 있는지) 알아야 한다.


!SLIDE quietest shout

# Visualization 툴이 하나쯤은 있어야 한다.


!SLIDE small-code

# 나의 경우:

```
~/.gitconfig:
[alias]
l = log --graph --pretty='%Cred%h%Creset -%C(yellow)%d%Creset %s %Cblue[%an]%Creset %Cgreen(%cr)%Creset' --abbrev-commit --date=relative
la = !git l --all
```
```
git la
```

<img src="images/terminal.png" alt="" height="400px">

!SLIDE

# Git Tower

<img src="images/tower.png" alt="" height="500px">

!SLIDE

# SourceTree

<img src="images/sourcetree.png" alt="" height="500px">

!SLIDE shout
# 좋은 부분을 배워서 자기 것으로 만들어라.

!SLIDE
# checkout -

`cd -` 와 유사하게 이전 branch로 이동할 수 있다.
```
                        E---F  ← feature & HEAD
                       /
                  A---B---C---D 
                              ↑ 
                           master
```
```
% git checkout -
```
```
                        E---F  ← feature 
                       /
                  A---B---C---D 
                              ↑ 
                       master & HEAD
```


!SLIDE
# commit --amend

마지막 커밋을 다시 한다.
```
                        A---B---C
                                ↑    
                          master & HEAD
```

```
<... change some files ... > 
% git commit -a --amend --no-edit
```
```
                              C' ← master & HEAD
                             /
                        A---B---C
                                ↑    
                  (dangling but still in reflog)
```


!SLIDE 
<br/>
# rebasing 

여러 순차적인 커밋들의 부모 커밋을 다시 적용하고, 

current branch의 pointer를 이동시킨다.

!SLIDE 
# rebasing 

```
                        E---F  ← feature & HEAD
                       /
                  A---B---C---D 
                              ↑ 
                           master
```

```
% git rebase master
```

```
                  (dangling but still in reflog)
                            ↓
                        E---F
                       /
                  A---B---C---D---E'--F'
                              ↑       ↑ 
                           master  feature & HEAD
```

!SLIDE 
# rebasing 

```
% git rebase --abort
```

문제가 있으면 `--abort` 를 추가해서 rebase하고 그래도 문제가 있으면

`reset --hard`를 해서 마지막 커밋으로 돌아가라.

!SLIDE 
<br/>
# rebasing - a private activity
push한 commits은 절대 rebase하지 마라.

!SLIDE
<br/>
# rebasing - a private activity
public rebasing is bad as others could have the same commits with different SHAs


!SLIDE 
# cherry picking 

다른 branch의 commit을 가져 올 수 있다.

```
                            E---F---G 
                           /
                      A---B---C---D 
                                  ↑ 
                             master & HEAD
```

```
% git cherry-pick SHA_OF_F
```

```
                            E---F---G 
                           /
                      A---B---C---D---F' 
                                      ↑ 
                                 master & HEAD
```


!SLIDE quietest shout

# `reset` 은 branch의 pointer를 옮긴다.


!SLIDE 
# reset --soft 

```
                    A---B---C---D---E
                                    ↑
                                  master
```

```
% git reset --soft SHA_OF_C
```

```
                    working dir & index still look like
                                    ↓
                    A---B---C---D---E
                            ↑
                          master
```
1. <code>HEAD</code> & current branch 를 특정 <code>&lt;SHA&gt;</code>로 옮긴다.
2. index - unchanged 
3. working directory - unchanged 


!SLIDE 
# reset --soft 

몇개의 commits를 하나의 commit로 바꿀 때 유용하다.
```
                    working dir & index still look like
                                    ↓
                    A---B---C---D---E
                            ↑
                          master
```

```
% git commit -m "perfect code on the 'first' try"
```

```
                    A---B---C---E'
                                ↑
                              master
```

!SLIDE 
# reset --soft 

좀 더 복잡한 상황을 보자.

```
                              master
                                ↓
                A---B---C---D---E 
                 \       \
                  F---G---H---I ← feature & HEAD
```

위의 방법으로는 할 수 없다.

!SLIDE 
# reset --soft 

merge 를 한번 하자.
```
% git merge master
```

```
                              master
                                ↓
                A---B---C---D---E 
                 \       \       \
                  F---G---H---I---J ← feature & HEAD
```

!SLIDE 
# reset --soft 

이전 커밋으로 `reset`하자

```
% git reset --soft master
```

```
                A---B---C---D---E ← feature & HEAD & master
                                 \
                                  J ← working dir & index 
```

```
% git commit -m "pristine J"
```

```
                              master
                                ↓
                A---B---C---D---E---J' ← feature & HEAD 
```

!SLIDE 
# reset --hard
```
% git reset --hard <SHA>
```

<br/>
1. moves <code>HEAD</code> & the current branch to the specified <code>&lt;SHA&gt;</code> 
2. clean the index, make it look like <code>&lt;SHA&gt;</code> 
3. clean the working copy, make it look like <code>&lt;SHA&gt;</code> 

commit하지 않은 것은<span class="danger">위험</span> 잘못 커밋한 것을 수정할 때 좋다.

!SLIDE 
# reset --hard HEAD 
```
% git reset --hard HEAD
```

working directory, index를 깨끗이 한다. branch pointer는 변화없다.

<code>reset</code>의 더 많은 정보는: <a href="http://progit.org/2011/07/11/reset.html">http://progit.org/2011/07/11/reset.html</a>


!SLIDE 
<br/>
# fetch 

download new commits and update the remote branch pointer

local branches는 변화가 없다.

!SLIDE 
# fetch 

```
                   origin/master
(local)                  ↓
                     A---B---C---D ← master & HEAD 
```
```
                     A---B---E---F   
(origin)                         ↑ 
                              master (in remote repo)  
```


```
% git fetch
```


```
                         origin/master
                               ↓
                           E---F
(local)                   /
                     A---B---C---D ← master & HEAD
```


!SLIDE 
<br/>
# pull 

<code>pull</code> 은<code>fetch</code> + <code>merge</code>

!SLIDE 
# pull 

```

                   origin/master
(local)                  ↓
                     A---B---C---D ← master & HEAD
```
```
                     A---B---E---F   
(origin)                         ↑ 
                              master (local ref in remote repo)  
```

```
% git pull
```


```
                         origin/master
                               ↓
                           E---F----
                          /         \
(local)              A---B---C---D---G ← master & HEAD
```

!SLIDE 
# the &#8220;right&#8221; way to pull down changes from the server

1. <code>stash</code> any uncommitted changes (if any)
2. <code>fetch</code> the latest refs and commits from origin
3. <code>rebase -p</code> your changes (if any) onto origin's head<br/> else, just fast-forward your head to match origin's
4. un-<code>stash</code> any previously stashed changes

<code>fetch</code> + <code>rebase</code> avoids unnecessary commits

!SLIDE

# rebasing pull
<br/>
As of git 1.8.5, git has finally added a rebase switch to `pull`:

```
% git pull --rebase
```

This will do the `fetch` + `rebase` for you (you still stash on your own).

!SLIDE shout myth
# git 은 위험하다?
## myth #1

!SLIDE shout
# git  _가잔 안전한_ version control이다 
## reality

!SLIDE shout myth
# git lets you<br/>rewrite history
## myth #2

!SLIDE shout
# rewriting <br/>history is a _lie_
## reality

!SLIDE shout myth
# git syntax is terrible
## myth #3

!SLIDE shout 
#  git syntax is<br/>_really terrible_
## reality

!SLIDE shout
# git mislabels things

## ex: git branches 는 당신이 생각하고 있는 그런게 아니다.

!SLIDE shout
# 이전 version control을 사용하며 생긴 예상을 버려라.

!SLIDE shout
# Questions? 


!SLIDE shout
# Bonus Section!

!SLIDE 
# reset (default)

```
% git reset [--mixed] <SHA>
```

<br/>

1. moves <code>HEAD</code> & the current branch to the specified <code>&lt;SHA&gt;</code> 
2. clean the index, make it look like <code>&lt;SHA&gt;</code> 
3. working directory - unchanged

<code>git reset HEAD</code> 는 index에 있는 것을 unstage한다.


!SLIDE 
<br/>
# squashing 

여러개의 branch commits를 하나의 commit으로 merge한다.

!SLIDE 
# squashing 

```
                              E---F---G ← feature
                             /
                A---B---C---D 
                            ↑ 
                     master & HEAD
```

```
% git merge --squash feature
```

```
                              E---F---G ← feature
                             /
                A---B---C---D---G' 
                                ↑ 
                         master & HEAD
```

여러개의 commits가 중요하지 않을 때 history를 깔끔히 할 수 있다.

!SLIDE

# recovering commits

<code>C</code>를 살리고 싶어!

```
                              C' ← master & HEAD
                             /
                        A---B---C ← (dangling)
```
```
% git reflog master  # find SHA_OF_C 
% git reset --hard SHA_OF_C
```
```
                              C' ← (dangling)
                             /
                        A---B---C
                                ↑    
                            master & HEAD
```