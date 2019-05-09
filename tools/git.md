
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
