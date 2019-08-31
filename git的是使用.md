
# 安装
centos 7 安装

+ 查看是否安装了
rpm -qa|grep git
+ 若已安装需要先卸载
rpm -e --nodeps git  或者  rpm -e git
+ 安装git
yum install git



## 注意
执行一下 这个命令：

git config --global credential.helper store

然后，下次再输入一次 账号密码 就可以了。


一 基本知识

文档地址：

https://www.git-scm.com/book/zh/v2

git config	
	用来控制git 的外观和行为的配置变量 变量存在三个位置

1	/etc/gitconfig 文件: 包含系统上每一个用户及他们仓库的通用配置。 如果使用
	--system 	  控制的是这个文件的变量
2	~/.gitconfig 或 ~/.config/git/config 文件：只针对当前用户。 
	 --global  控制这个文件
3	当前使用仓库的 Git 目录中的 config 文件（就是 .git/config）：针对该仓库。

例如：
	git config --global user.name "zhangyong"
	git config --global user.email 7900499@qq.com

	emacs 是一种编辑器   下面命令可以用来修改编辑器
 	git config --global core.editor emacs

git config --list
 	查看当前所有git 能找到的命令	

git config <key>
	来检查 Git 的某一项配置

git config user.name
	检查用户名


获取仓库的两种方法
	git init	
		初始一个本地库
	git clone [url]
		克隆一个现有仓库 url 为地址
		远程 Git 仓库中的每一个文件的每一个版本都将被拉取下来

	git clone [url] 本地仓库的名字
		获取下来  可以修改本地仓库的名字
	注意：url  可以使用多种协议
获取 帮组文档
	git help <verb>
	git <verb> --help
	man git-<verb>
例如：
	git help config



	git add  文件名
		通过这个对文件进行跟踪
二 常用命令
1.常用
git remote add origin git@github.com:yeszao/dofiler.git         # 配置远程git版本库
git pull origin master                                          # 下载代码及快速合并 
git push origin master                                          # 上传代码及快速合并
git fetch origin                                                # 从远程库获取代码

git branch                                                      # 显示所有分支
git checkout master                                             # 切换到master分支
git checkout -b dev                                             # 创建并切换到dev分支
git commit -m "first version"                                   # 提交

git status                                                      # 查看状态
git log                                                         # 查看提交历史


git config --global core.editor vim                             # 设置默认编辑器为vim（git默认用nano）
git config core.ignorecase false                                # 设置大小写敏感
git config --global user.name "YOUR NAME"                       # 设置用户名
git config --global user.email "YOUR EMAIL ADDRESS"             # 设置邮箱

2.别名
git config --global alias.br="branch"                 # 创建/查看本地分支
git config --global alias.co="checkout"               # 切换分支
git config --global alias.cb="checkout -b"            # 创建并切换到新分支
git config --global alias.cm="commit -m"              # 提交
git config --global alias.st="status"                 # 查看状态
git config --global alias.pullm="pull origin master"  # 拉取分支
git config --global alias.pushm="push origin master"  # 提交分支
git config --global alias.log="git log --oneline --graph --decorate --color=always" # 单行、分颜色显示记录
git config --global alias.logg="git log --graph --all --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(bold white)— %an%C(reset)%C(bold yellow)%d%C(reset)' --abbrev-commit --date=relative" # 复杂显示

3.创建版本库
git clone <url>                 # 克隆远程版本库
git init                        # 初始化本地版本库

4.修改和提交
git status                      # 查看状态
git diff                        # 查看变更内容
git add .                       # 跟踪所有改动过的文件
git add <file>                  # 跟踪指定的文件
git mv <old> <new>              # 文件改名
git rm <file>                   # 删除文件
git rm --cached <file>          # 停止跟踪文件但不删除
git commit -m “commit message”  # 提交所有更新过的文件
git commit --amend              # 修改最后一次提交

5.查看提交历史
git log                         # 查看提交历史
git log -p <file>               # 查看指定文件的提交历史
git blame <file>                # 以列表方式查看指定文件的提交历史

6.撤销
git reset --hard HEAD           # 撤消工作目录中所有未提交文件的修改内容
git reset --hard <version>      # 撤销到某个特定版本
git checkout HEAD <file>        # 撤消指定的未提交文件的修改内容
git checkout -- <file>          # 同上一个命令
git revert <commit>             # 撤消指定的提交


7.分支与标签

git branch                      # 显示所有本地分支
git checkout <branch/tag>       # 切换到指定分支或标签
git branch <new-branch>         # 创建新分支
git branch -d <branch>          # 删除本地分支
git tag                         # 列出所有本地标签
git tag <tagname>               # 基于最新提交创建标签
git tag -a "v1.0" -m "一些说明"  # -a指定标签名称，-m指定标签说明
git tag -d <tagname>            # 删除标签

git checkout dev                # 合并特定的commit到dev分支上
git cherry-pick 62ecb3

8.合并与衍合
git merge <branch>              # 合并指定分支到当前分支
git merge --abort               # 取消当前合并，重建合并前状态
git merge dev -Xtheirs          # 以合并dev分支到当前分支，有冲突则以dev分支为准
git rebase <branch>             # 衍合指定分支到当前分支


9.远程操作
git remote -v                   # 查看远程版本库信息
git remote show <remote>        # 查看指定远程版本库信息
git remote add <remote> <url>   # 添加远程版本库
git remote remove <remote>      # 删除指定的远程版本库
git fetch <remote>              # 从远程库获取代码
git pull <remote> <branch>      # 下载代码及快速合并
git push <remote> <branch>      # 上传代码及快速合并
git push <remote> :<branch/tag-name> # 删除远程分支或标签
git push --tags                 # 上传所有标签

10.打包
git archive --format=zip --output ../file.zip master    # 将master分支打包成file.zip文件，保存在上一级目录
git archive --format=zip --output ../v1.2.zip v1.2      # 打包v1.2标签的文件，保存在上一级目录v1.2.zip文件中
git archive --format=zip v1.2 > ../v1.2.zip             # 作用同上一条命令


11.远程与本地合并
如果在远程创建了代码仓，而且已经初始化，本地是具体的源代码，那么工作流程应该是：
git init                              # 初始化本地代码仓
git add .                             # 添加本地代码
git commit -m "add local source"      # 提交本地代码
git pull origin master                # 下载远程代码
git merge master                      # 合并master分支
git push -u origin master             # 上传代码

12.全局和局部变量配置
全局配置保存在：$Home/.gitconfig
本地仓库配置保存在：.git/config