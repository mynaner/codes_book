创建文件夹
> mkdir 

把这个目录变成Git可以管理的仓库
> git init

把文件添加到仓库
> git add ..

把文件提交到仓库
> git commit -m "提交说明"

可以让我们时刻掌握仓库当前的状态
> git status

查看具体修改的内容
> git diff

远程仓库查看提交地址
> git remote -v


**如果git status告诉你有文件被修改过，用git diff可以查看修改内容。**


### 版本退回

查看提交历史
> git log 或者 git log --pretty=oneline


退回上版本（上一个版本是HEAD^，上上一个版本是HEAD^^，上100个版本写成HEAD~100）
> git reset --hard HEAD^

也可以退到制定版本 （版本号可以写前面几位只要没重复就行）
> git reset --hard 1094a


## 分支

$ git checkout -b iss53 创建并切换分支

$ git branch iss53 创建分支
$ git checkout iss53 切换分支

### git切换分支

查看远程分支

> git branch -a

查看本地分支
> git branch

创建本地分支并拉取原创分支

> git checkout -b 本地分支名 origin/远程分支名

如果有本地分支，直接切换
> git checkout master

### 删除分支
删除远程分支

> git push origin --delete dev

删除本地分支
> git branch -D dev

### 分支操作 cherry-pick

拿取提交日志
> git log

单独合并一个提交
> git cherry-pick \< commit id> 

同上，不同点：保留原提交者信息。
>git cherry-pick -x \< commit id>




## 写配置文件

> vim ~/.gitconfig

```
[user]
        name = YLPS
        email = 994386508@qq.com
[alias]
        co=checkout
        ci=commit
        st=status
        pl=pull
        ps=push
        dt=difftool
        ca=commit -am
        b=branch
```

退出编辑

> esc  输入 :wq 

生成秘钥 全部回车跳过

> ssh-keygen -t rsa -C "994386508@qq.com"

> cd .ssh/

> cat id_rsa.pub

拉取项目

> git clone git@gitee.com:ract/react-admin.git



合并commit

> git rebase -i a2e9a696dd52ac444cb133d17dc369ac2cb07049

强制推送
> git push origin feature_add_export_excel_and_popup_list -f



#### git 切换远程仓库地址

- **查看远程仓库的地址**

> git remote -v

- **方式一：修改远程仓库地址**

> git remote set-url origin git@192.168.0.80:KK/KK_WebFront/kaka_admin.git

- **方式二：先删除远程仓库地址，然后再添加**

> git remote rm origin 
>
> git remote add origin git@192.168.0.80:KK/KK_WebFront/kaka_admin.git

```
git remote add origin git@gitee.com:SALONE/kaka_mp.git
```

http://192.168.0.80/KK/KK_WebFront/mp_demo.git