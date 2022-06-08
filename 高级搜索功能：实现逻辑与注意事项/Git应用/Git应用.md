# Git应用



#### 基本操作

工作区：正在编辑的文件夹下的文件，没有通过git add添加的文件
暂存区：通过git add添加，但是没有git commit的文件所在的区域

```
// 若拉取失败，一般为 提交文件 与 服务器文件 存在冲突，需要手动解决冲突

# 查看文件状态
git status

# 取消添加到暂存区 ——> 不会被提交
git rm --cached <fileName>

# 还原指定文件
git checkout -- <fileName>

# 还原所有文件
git checkout .

# 添加工作区指定修改文件到暂存区
git add <file>

# 添加所有修改文件到暂存区(忽略文件除外)
git add . 

# 提交所有暂存区文件
git commit -m "commit message"

# push到远程仓库
git push

# 更改编码
git config --global core.quotepath false
git config --global gui.encoding utf-8
git config --global i18n.commitencoding utf-8
git config --global i18n.logoutputencoding utf-8
export LESSCHARSET=utf-8
```



#### 从某次提交中新建分支

方式一：在命令窗口直接 git checkout -b <新分支名>，创建同时checkout新分支

方式二：

![20200929221732426](C:\Users\Asus\Desktop\key\Git应用\images\20200929221732426.png)





#### 重置RESET分支至某提交处

- Reset Current Branch to Here —— 在其后的提交记录（即添加打印3、4）将在log上消失
- 注意：reset后添加打印3、4的变化依旧在工作区，除非后继选择hard
- 后继选项包括：
  - soft：文件保持当前工作区状态，所有差异都在待提交的add区里
  - mixed：文件保持当前工作区状态，所有差异都不保存（但完全可以手动add）
  - hard：硬核恢复文件，所有的差异都消失且无法恢复

![无标题](C:\Users\Asus\Desktop\key\Git应用\images\无标题.png)







#### 重置RESET与撤销REVERT的区别

revert将保留C2的提交记录，若远程有C2提交，则C2'可以与之合并，但RESET则不行——它适用于本地折腾

![无标题2](C:\Users\Asus\Desktop\key\Git应用\images\无标题2.png)





#### merge合并方向性

合并前：

![20200929224220154](C:\Users\Asus\Desktop\key\Git应用\images\20200929224220154.png)



##### 将bugfix合并入master

操作：checkout master分支，选择本地bugfix，merge into current

![2020092922504427](C:\Users\Asus\Desktop\key\Git应用\images\2020092922504427.png)

结果：新建出一个新的提交，head和master都在新提交处

![20200929230314273](C:\Users\Asus\Desktop\key\Git应用\images\20200929230314273.png)



##### 将master合并入bugfix

操作：checkout bugfix分支（C2处），选择本地master，merge into current

结果：新建出一个新的提交，head、master和bugfix都在新提交处，bugFix分支仅仅只是移动到了C4处，因为C4里包含了其所有内容 —— 在上一步已实现

![2020092923093219](C:\Users\Asus\Desktop\key\Git应用\images\2020092923093219.png)



#### rebase命令

操作：

\# checkout bugfix分支（C2处）

\# git rebase C3

![Snipaste_2022-06-04_17-23-31](C:\Users\Asus\Desktop\key\Git应用\images\Snipaste_2022-06-04_17-23-31.png)



#### cherrypick命令

**情景：**只需要某几次提交，debug和println调试的提交都不需要，比如希望main分支获取C3-4-7的内容，则

\# git checkout main;

\# git cherry-pick C3 C4 C7;

![Snipaste_2022-06-04_17-40-23](C:\Users\Asus\Desktop\key\Git应用\images\Snipaste_2022-06-04_17-40-23.png)

**结果：**

![Snipaste_2022-06-04_17-40-11](C:\Users\Asus\Desktop\key\Git应用\images\Snipaste_2022-06-04_17-40-11.png)



##### 暂缓提交

\# git cherry-pick **-n** C3 C4 C7;

\# git status;		—— 将看到三个待提交的更改文件

\# git commit -m "commit message";

##### 提交区间选择

\# git cherry-pick **C2..C3** C4 C7;   	—— ..为左开右闭( ] 结构，所以相当于只选定了C3



#### stash命令储存修改

```
$ git status
列出缓存区modified的文件

$ git stash save "stashing name"

$ git status
无任何待提交文件 —— 均已保存到stash中

$ git stash list
存储列表查看

$ git stash pop [stash_id]
恢复某次存储版本，若不指定id默认为最新存储进度，恢复后记录消失

$ git stash apply
恢复某次存储版本，若不指定id默认为最新存储进度，
恢复后记录不消失，适用于多分枝恢复操作

$ git stash drop [stash_id]
删除一个存储的进度，若不指定id默认为最新存储进度

$ git stash clear
清除所有存储进度

$ git stash show
查看最新保存的stash与当前目录的差异

$ git stash show stash@{1} —— 查看指定的stash和当前目录差异
$ git stash show stash@{1} -p	—— 查看详细差异
```

