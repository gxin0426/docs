### git常用命令

```shell
#初始化本地仓库
git init
#添加全部已经修改的文件，准备commit提交 该命令效果等同于 git add -A
git add .
#将修改后的文件提交本地仓库 
git commit -m "提交说明"
#链接到远程仓库，并将代码同步到远程仓库
git remote add origin 远程仓库地址

#创建一个 upStream （上传流），并将本地代码通过这个 upStream 推送到 别名为 origin 的仓库中的 master 分支
#-u ，就是创建 upStream 上传流，如果没有这个上传流就无法将代码推送到 github；同时，这个 upStream 只需要在初次推送代码的时候创建，以后就不用创建了
#另外，在初次 push 代码的时候，可能会因为网络等原因导致命令行终端上的内容一直没有变化，耐心等待一会就好。
git push -u origin master

#如果有多个远程仓库 或者 多个分支， 并且需要将代码推送到指定仓库的指定分支上，那么在 pull 或者 push 的时候，就需要 按照下面的格式书写：

git pull 仓库别名 仓库分支名 
git push 仓库别名 仓库分支名

#抛弃本地所有的修改，回到远程仓库的状态
git fetch --all && git reset --hard origin/mster

#重设第一个commit
#也就是把所有的改动都重新放回工作区，并清空所有的commit 这样就可以重新提交一次commit了
git update-ref -d HEAD

#还可以展示本地仓库中任意两个 commit 之间的文件变动
git diff <commit -id> <commit -id>

#输出暂存区和本地最近的版本（commit）的 different（不同）
git diff --cached

#输出工作区、暂存区 和本地最近的版本（commit）的 different（不同）
git diff HEAD

#快速切换到上一个分支
git checkout -

#删除已经合并到 master 的分支
git branch --merged master | grep -v '^\*\| master'|xargs -n git branch -d 

#展示本地分支关联远程仓库的情况
git branch -vv

#列出所有远程分支
git branch -r

#列出本地和远程分支
git branch -a

#创建并切换到本地分支
git checkout -b <branch-name>
#从远程分支中创建并切换到本地分支
git checkout -b <branch-name> origin/<branch-name>

#删除本地分支
git  branch -d <local-branchname>

#删除远程分支
git push origin --delete <remote-branchname>
git push origin :<remote-branchname>

#重命名本地分支
git branch -m <new-branch-name>

#查看标签
git tag

#展示当前分支的最近的tag
git describe --tags --abbrev=0

#放弃工作区的修改
git checkout <file-name>

#放弃所有修改
git checkout .

#恢复删除的文件
git rev-list -n 1 HEAD --<file_path>#得到deleting_commit
git checkout <delete_commit>^ -- <file_path> #回到删除文件 deleting_commit 之前的状态


#本地与远程的差集
git log local_branch..origin/remote_branch

#统计文件的改动
git diff --stat local_branch origin/remote_branch

#git 回滚到之前的某一个commit

1. git log 查看要回滚的一个commit
2.git reset --hard <commit hash id>

#展示提交了很多次 push之前查看修改了哪些文件
git diff origin/分知名...HEAD
git diff origin/分知名...HEAD --name-status


# 修改最近提交的 commit 信息
$ git commit --amend --message="modify message by daodaotest" --author="jiangliheng <jiang_liheng@163.com>"

# 仅修改 message 信息
$ git commit --amend --message="modify message by daodaotest"

# 仅修改 author 信息
$ git commit --amend --author="jiangliheng <jiang_liheng@163.com>"
```

分支管理：`https://www.cnblogs.com/jiaoshou/p/11808361.html`

### 在dev分支获取master上最新的代码

```bash
#简单方法
git pull origin master

#第二种方法
git checkout master #切换到主分支
git pull
git checkout dev
git pull
git merge master #合并主分支最新代码到dev分支
解决冲突
git push # 推送本地dev分支到远程dev分之
```

### 检查本地代码修改情况

```bash
#可以查看当前没有add的内容修改（不在缓冲区的文件变化）
git diff
#查看已经add但没有commit的改动（在缓冲区的文件变化）
git diff --cached
#上面两个命令的合并
git diff HEAD
```

### 修改commit 信息

```bash
#修改上一条提交的commit message
git commit --amend
#修改之前提交的commit message 比如说 修改距离此版本之前的第三条message信息
git rebase -i HEAD～3
#显示：
pick 56b2308 feat(pages): home DONE
pick 82f65eb fix(pages movie): slides bug fixed
pick 08b2087 feat(pages home & movie): add FABs animation 
#将要修改的comimit信息 如 pick 56b2308 feat(pages): home DONE。 pick改成edit，完后保存
#然后运行
git commit --amend
#修改完后 运行
git rebase --continue
```

### 追加和合并提交

```bash
#demo
首先新建一个text1和text2名，将text1添加到暂存区，并提交到版本库（git commit），然后将text2添加到暂存区（git add），然后提交的时候使用--amend。这样可以将第二次提交追加到第一次。
```



### git_learn

#### git_basic

```shell
git log命令

git log --oneline # 显示一行
git log --graph #显示层级关系
git log -n # 只显示几行
git log --all # 显示所有分支commit信息
```

```
git ls-files -s
git cat-file -p
git cherry-pick
```



