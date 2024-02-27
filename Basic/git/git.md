![image-20230626153654220](/Users/steaksunflower/Library/Application Support/typora-user-images/image-20230626153654220.png)



# 本地仓库

## git init

建立版本区

## git add 

将文件加入到暂存区

git add .   加入所有文件

git add ./xxx/ 加入某个文件

## git commit

将暂存区的文件一次性加入到仓库区

git commit -m "版本信息或评论信息"

## git remote add origin https://xxxx.git

将远程仓库加入到本地仓库的连接池中

## git push -u origin master

把仓库区的文件提交到远程仓库里

## 版本回溯

### 查看版本号 - git log

![image-20230626154813855](/Users/steaksunflower/Library/Application Support/typora-user-images/image-20230626154813855.png)

### 暂存区的版本回溯 git reset --hard + 版本号

在回溯后，回溯到的版本之后的版本会在log中消失

### 工作区撤销命令 git checkout -- <file>

撤销修改就回到和版本库一模一样的状态，即用**版本库里的版本替换工作区的版本**

### 应对log丢失 git reflog

记录了每一次的命令，这样就可以找到版本号了，这样你又可以通过`git reset`来版本穿梭了

## 删除

### 将版本区里的文件删除 git rm

## 分支

分支，就像平行宇宙，你创建了一个属于你自己的分支，别人看不到，还继续在原来的分支上正常工作，而你在自己的分支上干活，想提交就提交，直到开发完毕后，再一次性合并到原来的分支上，这样，既安全，又不影响别人工作。

![image-20230626192936130](/Users/steaksunflower/Library/Application Support/typora-user-images/image-20230626192936130.png)

![image-20230626192954905](/Users/steaksunflower/Library/Application Support/typora-user-images/image-20230626192954905.png)

![image-20230626193021052](/Users/steaksunflower/Library/Application Support/typora-user-images/image-20230626193021052.png)

### 创建分支 - git branch other

### 切换到分支 - git checkout other

### 删除分支 git branch -d other

强行删除是git branch -D other，会丢弃未被合并的分支

### 查看当前所有分支 - git branch

对于当前指向的分支，名称前会有一个*

### 合并分支 - git merge other

合并完成之后，就可以在master分支上查看到其他分支的文件了

### 查看分支合并图 - git log --graph

### 存储工作区 - git stash 

使用git stash list 查看已经存储的工作

`git stash apply`恢复却不删除`stash`内容，`git stash drop`删除`stash`内容。

`git stash pop`恢复的同时把stash内容也删了.

此时，用`git stash list`查看，看不到任何`stash` 内容。
 **总结：修复bug时，我们会通过创建新的bug分支进行修复，然后合并，最后删除；当手头工作没有完成时，先把工作现场git stash一下，然后去修复bug，修复后，再git stash pop，回到工作现场**

## 协作

`git remote` 查看远程仓库的信息，远程仓库名默认为origin

`git remote -v`， 显示更详细的信息

### 合并分支 - git pull



![image-20230626194320330](/Users/steaksunflower/Library/Application Support/typora-user-images/image-20230626194320330.png)

## 获取

`git clone` + 仓库地址`下载克隆文件

## 标签

### 打标签 - git tag v1.0 du2n2d9

如果一个`commt id`是`du2n2d9`,执行`git tag v1.0 du2n2d9`就把这个版本打上了`v1.0`的标签了

### 查看所有标签 - git tag

### 查看标签信息 git show <tagname>

