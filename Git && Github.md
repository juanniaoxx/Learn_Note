`git`是一种工具，`github`则是一个代码托管平台

# Part I Git
`git`是一个难得被CSDIY标记为困难的`工具`，当然无论多困难的工具，只要经常使用总能掌握，但仅仅会用一个工具或许不难但弄懂GIT的背后逻辑还是挺困难的。
按照CSDIY的解释，可以参考如下资料
[Version Control (Git) · Missing Semester](https://missing.csail.mit.edu/2020/version-control/)
[Git - Book](https://git-scm.com/book/en/v2) [可以只阅读前五章]
[Learn Git Branching](https://learngitbranching.js.org/?locale=zh_CN) [一个可以学习git命令的游戏平台]
[How to Write a Git Commit Message](https://cbea.ms/git-commit/)
[Write yourself a Git!](https://wyag.thb.lt/)[造轮子]

### Git的理论基础
### Git的使用
[不要忘了GIT的底层是一个DAG，有向无环图]
`git commit` 向仓库提交本次修改
>--amend : Replace the tip of the current branch by creating a new commit.


`git branch` 创建一个新的分支
`git checkout` Switch branches or restore working tree files.
`git checkout -b <brach of yourname>`可以直接创建并跳转到该分支
`git merge <brach of yourname>`  Join two or more development histories together
`git rebase <brach of branch'name>  <another branch'name>` : Reapply commits on top of another base tip ,后面参数是可选的，没说就默认移动到HEAD的下一个结点。
`git rebase main `如下 把输入的结点之前的`所有线性`结点复制到HEAD对应的分支下面 
![[Pasted image 20240430214058.png|rebase命令]]
![[Pasted image 20240430235632.png|注意看下一步]]
run `git rebase bugFix side` that is prefect!!
![[Pasted image 20240430235737.png]]
`git rebase bugFix main` `把bugFix合并到 main下，并把HEAD指向main`
![[Pasted image 20240430230850.png]]

>可选参数 `-i` \  `--interactive` 会打开一个可交互的UI界面来调整\\删除\\合并

>`线性`

`在提交树中移动`
`HEAD`  
[What is HEAD in Git? - Stack Overflow](https://stackoverflow.com/questions/2304087/what-is-head-in-git)(国内CS真的...拾人牙慧罢了)
`相对移动` 
1. `branch_name^`把HEAD移动到branch_name对应结点的parent结点
```git
git cheackout bugFix^ //移动到bugFix对应结点的parent结点
```
2. `~<numbers>`向上移动numbers对应的步数[想上移动其实是往更早的提交结点移动]
```git
git branch -f main HEAD~3
```
![[Pasted image 20240430220153.png|相对移动]]

`-f`允许将分支强制移动到我们想要的位置。

`对于多分支结构，即有多个parent结点`可以通`^[number]`来选择，默认值为1代表正上方parent结点。
`git branch [branch_name] HEAD\[current_name]~[numbers]^[numbers]...`支持链式运算。
`git reset` 
```git
git reset HEAD~1 //建议只在本地仓库使用，这种是直接改写历史。
git revert HEAD //可视化如下
```
![[Pasted image 20240430222630.png|revert可视化]]

------到此为止，已经完成了git百分之90的操作啦------------
`git cheery-pick <branch name>` : 把名字为[brach name]的结点合并到HEAD所对应的分支
>注意该指令支持一次输入多个结点，并且支持相对索引`^`或者`~[numbers]` 
>注意这个结点不能位于HEAD的上游，不能是HEAD这条路径上的早期修改。

`git tag [tag_name] [node_name]` 标签名 结点Hash名
>标签在代码库中起着“锚点”的作用
>故在tag上不能直接操作比如commit....

>寻址距离[ref]最近的那个标签。
`git describe <ref>`  -> return `<tag>_<numCounts>_g<hash>` 标签名_中间的提交数_g哈希值的前几位[注意是你的输入ref的哈希值前几位]。
![[Pasted image 20240430233500.png]]



# Part II GitHub [远程仓库]
`git clone`
`git fetch` 从远处仓库获取数据 [并不需要参数]
![[Pasted image 20240501011921.png]]
>`git fetch`并不会改变本地仓库的状态。不会更新`main`分支，也不会修改磁盘文件。

`git pull` 先抓取更新再合并到本地分支 
>实际上是`git fetch` & `git merge` 的组合 

`git pull --rebase` 将`fecth`和`rebase` [GitHub拒绝基于旧代码做的提交，必须先将当前分支与最新的远程仓库分支合并才允许push]


`git push` 把本地仓库上传到原创仓库

`push被拒绝`有的时候并不允许直接`push`到` main`分支，必须先新建一个分支`feature`在推送到远程分支，然后与`main`合并。

`有的时候并不需要总是移动到main上在push`
可以让别的结点也能够push到远程仓库的main结点上
1.`git branch -b <branch_name> <orgin/main>` 
```git
git branch -b foo orgin/main
git pull --rebase //如果需要先和远程仓库同步的话
...
git push
```
2.``
