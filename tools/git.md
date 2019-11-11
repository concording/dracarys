
`git tag --delete 20180525.v1`

`git push --delete origin 20180525.v1`

`git stash show`

`git stash drop stash@{1}`

`git reset --hard <sha1-commit-id>`

`git push origin HEAD --force`

`git branch -D f_feature/f_repayNoticeQuery_191107`

`git push origin --delete f_feature/f_repayNoticeQuery_191107`

## undo a revert
`git cherry-pick <original commit sha>`
Will make a copy of the original commit, essentially re-applying the commit

Reverting the revert will do the same thing, with a messier commit message:
`git revert <commit sha of the revert>`

Either of these ways will allow you to git push without overwriting history, because it creates a new commit after the revert.
When typing the commit sha, you typically only need the first 5 or 6 characters:
`git cherry-pick 6bfabc`


### 回滚commit
假如要删除备注为add c.txt commit为0fb295fe0e0276f0c81df61c4fd853b7a000bb5c的这次提交

首先找到此次提交之前的一次提交的commit7753f40d892a8e0d14176a42f6e12ae0179a3210
执行如下命令
1. `git rebase -i  7753f40`
2. 弹出如下界面
3. 将0fb295f这一行前面的pick改为drop，然后按照提示保存退出
4. 至此已经删除了指定的commit，可以使用git log查看下

![The three trees of Git](https://wac-cdn.atlassian.com/dam/jcr:0c5257d5-ff01-4014-af12-faf2aec53cc3/01.svg?cdnVersion=329)

### reset revert checkout 
1. A checkout is an operation that moves the HEAD ref pointer to a specified commit. 
2. A revert is an operation that takes a specified commit and creates a new commit which inverses the specified commit. git revert can only be run at a commit level scope and has no file level functionality.
3. A reset is an operation that takes a specified commit and resets the "three trees" to match the state of the repository at that specified commit. A reset can be invoked in three different modes which correspond to the three trees.

Checkout and reset are generally used for making local or private 'undos'. They modify the history of a repository that can cause conflicts when pushing to remote shared repositories. Revert is considered a safe operation for 'public undos' as it creates new history which can be shared remotely and doesn't overwrite history remote team members may be dependent on.



| Command | Scope | Common use cases |
|---|---|---|
| `git reset` | Commit-level | Discard commits in a private branch or throw away uncommited changes |
| `git reset` | File-level | Unstage a file |
| `git checkout` | Commit-level | Switch between branches or inspect old snapshots |
| `git checkout` | File-level | Discard changes in the working directory |
| `git revert` | Commit-level | Undo commits in a public branch |
| `git revert` | File-level | (N/A) |

### git rebase 、git merge
Rebasing is the process of moving or combining a sequence of commits to a new base commit. Rebasing is most useful and easily visualized in the context of a feature branching workflow. The general process can be visualized as the following:

![Git tutorial: Git rebase](https://www.atlassian.com/dam/jcr:e4a40899-636b-4988-9774-eaa8a440575b/02.svg)

From a content perspective, rebasing is changing the base of your branch from one commit to another making it appear as if you'd created your branch from a different commit. Internally, Git accomplishes this by creating new commits and applying them to the specified base. It's very important to understand that even though the branch looks the same, it's composed of entirely new commits.

`git checkout HEAD~4`

`git branch -f master HEAD~3`

git reset 通过把分支记录回退几个提交记录来实现撤销改动。你可以将这想象成“改写历史”。git reset 向上移动分支，原来指向的提交记录就跟从来没有提交过一样。git reset HEAD~1
虽然在你的本地分支中使用 git reset 很方便，但是这种“改写历史”的方法对大家一起使用的远程分支是无效的哦！
为了撤销更改并分享给别人，我们需要使用 git revert。来看演示：


git describe 的​​语法是：
git describe <ref>
<ref> 可以是任何能被 Git 识别成提交记录的引用，如果你没有指定的话，Git 会以你目前所检出的位置（HEAD）。
它输出的结果是这样的：
<tag>_<numCommits>_g<hash>
tag 表示的是离 ref 最近的标签， numCommits 是表示这个 ref 与 tag 相差有多少个提交记录， hash 表示的是你所给定的 ref 所表示的提交记录哈希值的前几位。
当 ref 提交记录上有某个标签时，则只输出标签名称


