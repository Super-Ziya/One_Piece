####一、添加到库步骤

- `cd d:`定位到git仓库所在文件夹

- 要添加文件移动到git仓库所在文件夹

- `git add 文件名`添加到暂存区（stage/index）

- `git add .` 添加所有

- `git commit -m"对本次提交的说明"`提交到当前分支

  > add只能添加单个文件 ，commit可以一次提交多个文件
  >
  > master分支是主分支	

---

- `git status`可以查看仓库的状态
- `git diff 文件名`可以查看修改内容
- `git log`查看所有提交日志

#### 二、回退版本

- `git reset --hard HEAD^`回退上一个版本

  > HEAD^^是上上个版本，HEAD~100是回退100个版本

- `git reset --hard + id`回退到指定版本（版本号不用写全）
- `git reflog`记录每一条指令（可以查看版本id）

#### 三、撤销修改

- `git checkout -- 文件名`（文件在工作区）

  > 文件为放进暂存区，文件回到和版本库一样的状态
  >
  > 文件已添加到暂存区，文件回到添加到暂存区后的状态
  >
  > 总之，就是回到最近一次`git commit`或`git add`时的状态

- `git reset HEAD <文件名>`（文件在暂存区）可以把暂存区的修改撤销，放回工作区
- 文件在版本库，使用版本回退

#### 四、删除文件

- `git rm 文件名`从版本库中删除文件，并且`git commit`
- 错删了，`git checkout -- 文件名`

#### 五、远程仓库

- `git remote add origin 远程仓库网址` 把本地库内容推送到远程

- `git push -u origin master` 把本地库内容推送到远程

  > origin默认为远程库
  >
  > 第一次提交要加 `-u`

- `git clone git@github.com:账户名/本地git文件夹名.git`从远程库克隆本地库

- `git remote -v` 查看项目远程地址

- `git remote rm origin` 删除远程仓库

#### 六、分支

- `git checkout -b 分支名`或`git switch -c 分支名`创建分支并切换到该分支

  > 相当于
  >
  > `git branch 分支名`创建分支
  >
  > ``git checkout 分支名`或`git switch 分支名`转到某分支

- `git branch`查看当前分支（当前分支前带有`*`）

- `git branch -d 分支名`删除该分支

- `git branch -D 分支名`强行删除未合并分支

- `git merge 分支名`合并指定分支到当前分支

  > `Fast-forward`“快进模式”，直接把`master`指向`dev`的当前提交
  >
  > `--no-ff`参数表示禁用`Fast forward`

- `git log --graph`查看分支合并图

---

- `git stash`存储工作区

- `git stash list`查看存储工作区

- `git stash pop`恢复存储区同时删除存储区内容

  > 相当于
  >
  > `git stash apply`回复存储区
  >
  > `git stash drop`删除存储区

- `git cherry-pick 版本id`复制一个特定的修改提交到当前分支

---

- `git remote`查看远程库的信息
- `git remote -v`显示更详细的信息
- `git pull`把最新的提交从`origin/dev`抓下来
- `git rebase`吧分支梳理成一条直线（不过本地的分支提交会被修改）

####七、多人协作一般模式

1. 首先，可以试图用`git push origin 分支名`推送自己的修改；
2. 如果推送失败，则因为远程分支比你的本地更新，需要先用`git pull`试图合并；
3. 如果合并有冲突，则解决冲突，并在本地提交；
4. 没有冲突或者解决掉冲突后，再用`git push origin 分支名`推送就能成功！

如果`git pull`提示`no tracking information`，则说明本地分支和远程分支的链接关系没有创建，用命令`git branch --set-upstream-to 分支名 origin/分支名`。

#### 八、标签

- `git tag 标签名`或`git tag 标签名 版本id`打一个新标签
- `git tag -a 标签名 -m "说明文字" 版本id`创建带有说明的标签
- `git tag`查看所有标签
- `git show 标签名`查看说明文字
- `git tag -d 标签名`删除标签
- `git push origin :refs/tags/标签名`删除远程标签
- `git push origin 标签名`推送标签到远程
- `git push origin --tags`一次性全部推送

#### 九、其他

- `git init` 创建本地仓库
- `git count-objects -v` 统计大小