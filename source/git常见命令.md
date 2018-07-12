# git 命令

## 初始化本地仓库



```
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/rjli/booktobook.git
git push -u origin master
```

> 在github上面创建远程仓库的时候，不要创建readme.md,即可用上述命令创建成功

#### 查看远程仓库的信息

```
git remote -v  
```

##### 查看所有分支，远程分支是红色的

```
git branch -a
```



## 分支

把分支推送到远端

https://blog.csdn.net/kakadiablo/article/details/79517985



###  删除分支

https://blog.csdn.net/qq_32452623/article/details/54340749

### 查看远程分支

- 使用如下git命令查看所有远程分支：

`git branch -r`

- 查看远程和本地所有分支： 

`git branch -a`

- 查看本地分支： 

`git branch` 

> 在输出结果中，前面带`*` 的是当前分支。

### 拉取远程分支并创建本地分支

#### 方法一

使用如下命令：

`git checkout -b 本地分支名x origin/远程分支名x`

使用该方式会在本地新建分支x，并自动切换到该本地分支x。

采用此种方法建立的本地分支会和远程分支建立映射关系。

#### 方式二

使用如下命令：

`git fetch origin 远程分支名x:本地分支名x`

使用该方式会在本地新建分支x，但是不会自动切换到该本地分支x，需要手动checkout。

采用此种方法建立的本地分支不会和远程分支建立映射关系。

### 本地分支和远程分支

https://blog.csdn.net/carfge/article/details/79691360



#### 建立映射关系的作用

#### 建立映射关系（或者为跟踪关系track）。

这样使用git pull或者git push时就不必每次都要指定从远程的哪个分支拉取合并和推送到远程的哪个分支了。 
`git branch -vv` 
输出： 
![这里写图片描述](https://img-blog.csdn.net/20180208101910427?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvemhhbmd4aWFveWFuZzA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

上面的本地分支和远程分支都有映射关系，如果没有，就需要手动建立： 
`git branch -u origin/分支名`， 
或者 
`git branch --set-upstream-to origin/分支名` 
`origin` 为git地址的标志，可以建立当前分支与远程分支的映射关系。

#### 撤销本映射关系

`git branch --unset-upstream` 
之后可以再次用`git branch -vv` 查看本地分支和远程分支映射关系

#### 其他

- **本地分支只能跟踪远程的同名分支吗？**

  答案是否定的，本地分支可以与远程不同名的分支建立映射关系

​       操作和之前的一样，只是可以指定和本地分支名不同的远程分支名，然后使用`git branch -vv` 查看映射关系，可以发现建立映射成功。

 



## 常见问题

###fatal: 'origin' does not appear to be a git repository

 

​    fatal: Could not read from remote repository.

​    Please make sure you have the correct access rights and the repository exists.

​    翻译： 致命：“origin”不是一个 git 存储库

​                 致命：无法读取远程存储库

​                 请确定你有正确的访问权力，并且 存储库存在 

   解决方案：  1. 首先,检查你的起源是设定的运行
                              git remote -v
                           显示
                         orgin    git@github.com:chaorwin/chaorwin.git (fetch)
                         orgin    git@github.com:chaorwin/chaorwin.git (push)
                          "origin" 不存在

​                       2.重命名它,或改变URL,删除它,然后添加正确的一个。

​                            git remote remove orgin

​                      3. 增加一个

​                            git remot add origin git@github.com:chaorwin/chaorwin.git

​                      4. git push origin master

