
`git tag --delete 20180525.v1`

`git push --delete origin 20180525.v1`

`git stash show`

`git stash drop stash@{1}`

`git reset --hard <sha1-commit-id>`

`git push origin HEAD --force`

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
