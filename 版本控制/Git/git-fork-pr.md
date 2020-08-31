# 在Git-Fork-请求PR

1. 在github页面上, 点击fork按钮, 将B的项目拷贝一份到A自己的代码仓库中.
2. 克隆A自己的代码仓库到本地.
    $ git clone [https://github.com/A/A.git](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2FA%2FA.git)
3. 将B的项目作为最新代码的参考标准（upstream 是上游仓库的别名，别名随意命名）
    $ git remote add upstream [https://github.com/B/B.git](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2FB%2FB.git)
4. 在本地更改代码（增删查等操作）
5. 暂存已经编辑的目录和文件.
    $ git add .
    $ git stash
6. 拉取B仓库的新代码
    $ git fetch upstream
7. 将B新的部分合并到A的代码仓库中, 使A的代码仓库变成最新的代码.
    $ git rebase upstream/master origin/master
8. 查看分支，确保在自己的本地分支上
    $ git branch
9. 如果不在本地分支，则执行  git checkout master ，如果在则忽略此步骤
10. 将刚刚暂存的代码合并到现在最新的代码中.
     $ git stash apply stash@{0}
11. 本地提交代码.
     $ git add .
     $ git commit -m 提交代码的注释信息"
12. 将代码推送到A的github仓库.
     $ git push -u origin/master
13. 在A的github仓库页面上，点击pull request向B发起PR请求.

如上是第一次进行fork、克隆代码、创建关联、修改代码提交等操作
 当再次本地修改代码提交时，执行步骤6-11即可

