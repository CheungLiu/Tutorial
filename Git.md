#### Git教程

1. ##### 基础

   + git status命令看看仓库的当前状态
   + git diff readme.txt 
   + git log --pretty=oneline
   + git log --graph --pretty=oneline --abbrev-commit
   + git relog

2. ##### 分支

   + git branch
   + git branch -d dev
   + git switch -c dev
   + git chechout -b dev
   + git checkout master

3. ##### 撤回

   + 场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令git checkout -- file。
     * 命令git checkout -- readme.txt意思就是，把readme.txt文件在工作区的修改全部撤销
   + 场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用      命令git reset HEAD <file>，就回到了场景1，第二步按场景1操作。
     + git reset HEAD <file>可以把暂存区的修改撤销掉（unstage），重新放回工作区
   + 场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，参考***版本回退***一节，不过前提是没有推送到远程库。
     + git reset --hard HEAD^退回到上一个版本

4. ##### 误删

   + git checkout -- test.txt
     git checkout其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”。

5. ##### 远程库

   + git remote add origin git@github.com:CheungLiu/repo.git
     远程库的名字就是origin，这是Git默认的叫法，也可以改成别的，但是origin这个名字一看就知道是远程库。
   + 当你从远程仓库克隆时，实际上Git自动把本地的master分支和远程的master分支对应起来了，并且，远程仓库的默认名称是origin。

6. ##### 合并

   + git merge dev
     + 可能会有冲突
   + 保留分支记录的合并：git merge --no-ff -m "merge with no-ff" dev
     + 可能会有冲突

7. ##### 保存现场（在当前分支dev上需要立即修改BUG，但又不希望commit）

   + git stash可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作：
     Saved working directory and index state WIP on dev: f52c633 add merge
   + 现在，用git status查看工作区，就是干净的（除非有没有被Git管理的文件），因此可以放心地创建分支来修复bug。
   + git stash list
     stash@{0}: WIP on dev: f52c633 add merge
     + 恢复
       1. git stash apply，但是恢复后，stash内容并不删除，你需要用git stash drop来删除
       2. git stash pop，恢复的同时把stash内容也删了
   + 可以多次stash，恢复的时候，先用git stash list查看，然后恢复指定的stash，用命令：
     git stash apply stash@{0}
   + 在master分支上修复了bug后，我们要想一想，dev分支是早期从master分支分出来的，所以，这个bug其实在当前dev分支上也存在。同样的bug，要在dev上修复，我们只需要把4c805e2 fix bug 101***（就是在stash保存当前现场切换到master，在分支bug修改后的commit_ID,不是merge的ID）***这个提交所做的修改“复制”到dev分支。
     ***注意***：我们只想复制4c805e2 fix bug 101这个提交所做的修改，并不是把整个master分支merge过来。
   + cherry-pick命令，让我们能复制一个特定的（提交）到当前分支：
     在当前dev分支git cherry-pick 4c805e2
     Git自动给dev分支做了一次提交，注意这次提交的commit是1d4b803，它并不同于master的4c805e2，因为这两个commit只是改动相同，但确实是两个不同的commit。
   + 既然可以在master分支上修复bug后，在dev分支上可以“重放”这个修复过程，那么直接在dev分支上修复bug，然后在master分支上“重放”行不行？当然可以，不过你仍然需要git stash命令保存现场，才能从dev分支切换到master分支。（因为要合并）

8. ##### 准备合并前删除

   在分支feature开发星际飞船的功能完成后切换到dev分支，准备合并，要求立即删除。
   如果要丢弃一个没有被合并过的分支，可以通过git branch -D <name>强行删除。
   Git友情提醒，feature-vulcan分支还没有被合并，如果删除，将丢失掉修改，如果要强行删除，需要使用大写的-D参数。

9. ##### 推送分支，多人协作

   - git remote/git remote -v

   - 推送分支，就是把***该分支***上的所有本地提交推送到远程库。推送时，要指定本地分支，这样，Git就会把该分支推送到远程库***对应***的远程分支上
     git push origin master
     git push origin dev

     + master分支是主分支，因此要时刻与远程同步
     + dev分支是开发分支，团队所有成员都需要在上面工作，所以也需要与远程同步
     + bug分支只用于在本地修复bug，就没必要推到远程了，除非老板要看看你每周到底修复了几个bug
     + feature分支是否推到远程，取决于你是否和你的小伙伴合作在上面开发

   - 当你的小伙伴从远程库clone时，默认情况下，你的小伙伴只能看到本地的master分支，要在dev分支上开发，就必须创建远程origin的dev分支到本地
     git checkout -b dev origin/dev

   - 你的小伙伴已经向origin/dev分支***推送了他***的提交，而碰巧你也对同样的文件作了修改，并试图推送

     $ git push origin dev
     To github.com:CheungLiu/Tutorial.git
      ! [rejected]        dev -> dev (non-fast-forward)
     error: failed to push some refs to 'git@github.com:CheungLiu/Tutorial.git'
     hint: Updates were rejected because the tip of your current branch is behind
     hint: its remote counterpart. Integrate the remote changes (e.g.
     hint: 'git pull ...') before pushing again.
     hint: See the 'Note about fast-forwards' in 'git push --help' for details.

   - 推送失败，因为你的小伙伴的最新提交和你试图推送的提交有冲突，解决办法也很简单，Git已经提示我们，先用git pull把最新的提交从origin/dev抓下来，然后，在本地合并，解决冲突，再推送

     $ git pull
     There is no tracking information for the current branch.
     Please specify which branch you want to merge with.
     See git-pull(1) for details.

     ​	git pull <remote> <branch>

     If you wish to set tracking information for this branch you can do so with:

     ​	git branch --set-upstream-to=origin/<branch> dev

   - git pull也失败了，原因是没有指定***本地dev分支***与***远程origin/dev分支***的~~链接~~，根据提示，设置dev和origin/dev的链接：

     $ git branch --set-upstream-to=origin/dev dev
     Branch 'dev' set up to track remote branch 'dev' from 'origin'.

   - 再pull：

     $ git pull
     Auto-merging env.txt
     CONFLICT (add/add): Merge conflict in env.txt
     Automatic merge failed; fix conflicts and then commit the result.

   - 这回git pull成功，但是合并有冲突，需要手动解决，解决的方法和***分支管理中的解决冲突***完全一样。解决后，提交，再push：

     + 就是在本地手动保留需要的部分--add--commit

10. #### Rebase

    + 因为远程有别人新的提交，pull导致本地提交产生分支，用rebase
    + 最后，通过push操作把本地分支推送到远程
    + rebase操作的特点：把分叉的提交历史“整理”成一条直线，看上去更直观。缺点是本地的分叉提交已经被修改过了