# 基础

## `.git`目录

### `objects`

用于存放 git 暂存区和本地仓库的内容

`objects` 文件夹中所有文件的类型（使用 `git cat-file -t` 查看文件类型，`git cat-file -p` 查看文件内容）：

- tree 存放本次提交的所有 文件名称/文件夹名称 以及这些 文件/文件夹名称 对应的 hash
- commit 存放项目作者以及更改者的信息、更改时间戳和时区，同时有该提交的 commit message 和对应的 tree hash（如果是在一次提交之后提交，则还会有 parent commit hash)
- blob 存放文件内容
- tag 存放标签的描述信息，主要有打标签时对应的 commit hash、标签名称、标签描述信息、打标签的人的描述信息与时间戳

当存储暂存区中的内容时，会以 sha1 算法将 `blob 文件内容长度\0文件内容` 进行编码生成一个160位的二进制 hash，第一个字节作为文件夹名称，后面的 hash 值作为文件名称，其中存储二进制形式的文件内容

## `index`

`git ls-files -s`

存放本地库中所有文件的相对于 git 目录的文件 URL 和 hash

## `HEAD`

存放所处分支的信息

例如当 `git switch dev` 后，`cat .git/HEAD` 会输出 `refs/heads/dev`，此时 `cat .git/refs/heads/dev` 便会显示当前分支的最新一次 commit hash

当 `git switch master` 后，`cat .git/HEAD` 会输出 `refs/heads/master`

## HEAD 和 branch 指针

git 中有 HEAD 和 branch 两个指针，HEAD 用来指向当前所在的 commit hash，通常是指向分支指针而分支指针又指向它的最新一次 commit hash，因此当 HEAD 指向分支指针时相当于间接指向了一个 commit hash；branch 则是分支指针，指向当前分支最新的一次 commit hash

当 HEAD 指向的不是 branch 指针而是具体的 commit hash 时，也可以进行 add 和 commit 操作，并且此时执行 `git checkout -b xxx` 会直接将 HEAD 所指向的 commit hash 也由新创建的分支所指向

## 本地仓库添加远程仓库

`git remote add origin 仓库URL` origin 为自己起的远程仓库别名

当执行完该命令后，本地 .git/config 会多一条远程仓库的配置

```git
[remote "origin"]
				url = 远程仓库UIL
				fetch = +refs/heads/*:refs/remotes/origin/* 本地分支与远程分支文件夹
```

当本地修改完执行 `git push -u origin master` 后会创建一个新的 `master` 分支，同时会创建 `.git/refs/remotes` 目录、`.git/refs/remotes/origin` 目录、`.git/refs/remotes/origin/master` 文件、`.git/logs/refs/remotes` 目录和`.git/logs/refs/remotes/origin` 目录、`.git/logs/refs/remotes/master` 文件 

## git 的压缩

当使用 `git add` 或 `git commit` 后，git 会在 .git/objects 依照 git 的规则生成对应的 hash

每次对文件做更改后，再次使用 add 或 commit 后会再次按照 git 的规则生成文件 hash 将修改后的文件整个添加到 .git/objects

当使用 `git gc` 后，则会生成 .git/objects/pack 文件，其中有 .idx 和 .pack 两个文件，压缩方式主要为存储差异化

`git pack-verify -v .git/objects/pack/idx` 文件或pack文件可以查看压缩的文件的详细内容

当使用 `git clone` 后，git 会节约带宽提高传输效率只会将压缩后的文件下载下来

当删除分支后，可以使用 `git prune` 删除掉 unreachable 的 blob 对象文件（merge 一个 branch 后再删除这个 branch 所遗留的 blob 对象不会被 git 认为是 unreachable），使用 `git fsck` 可以查看所有 unreachable 的 blob 对象

## rebase 变基

git 的分支会指向当前最新一次提交的 commit hash，而当有新的分支，例如 dev 从 master 分支创建，而 master 和 dev 分支后续又都进行了新的提交操作，此时在 master 上想要合并 dev 的内容，因为两个分支所指向的最新的 commit hash 对方分支都没有，则会产生新的合并 commit hash

而 rebase 则是将当前分支的基本（从某一个分支切过来时所指向的 commit hash）指向指定的分支的 HEAD commit hash 形成 Fast-forword 合并

branch dev: `git rebase master`

## tag

`git tag` 查看所有的标签

`git tag 标签名称`，当执行完此命令后，`.git/refs/tags` 目录下会多出一个 `标签名称` 的文件，里面存储了打标签时的 commit hash

`git tag -d 标签名称` 会删除对应的 tag，同时会删除 `.git/refs/tags/标签名称` 文件，但不会删除 objects 中对应的 tag 对象

`git tag -a 标签名称 [特定的 commit hash] -m 描述信息`，如果有描述信息则会在 .git/objects 里产生一个 tag 类型的文件存放标签信息

## clone

本地 .git 中会向 .git/packed-refs 的文件添加当前远程分支所指向的最新的 commit hash 的记录

本地也会创建一个和远程分支指向相同的分支

## fetch

更新 .git/refs/remotes/origin 文件夹下所有的远程分支的最新的 commit hash 并在 objects 添加对应的 commit 文件，本地没有而远程有则创建，但 .git/refs/heads 下所对应的本地分支的最新 commit hash 并没有更新，若要更新需再执行 merge(`git checkout master && git merge origin/master`) 操作，或者直接 pull（会生成保存合并之前的 commit hash 信息的 .git/ORIG_HEAD 文件便于回滚 `git reset --hard ORIG_HEAD）

默认若本地有而远程没有不会删除，但可以通过 `git fetch --prune`/`git remote prune` 删除本地有而远程没有的分支

创建 .git/FETCH_HEAD 文件，存储远程分支所指向的最新的 commit hash，顺序则是将当前所处的分支对应的远程分支的信息放在第一个，而其他的全为 not-for-merge

`git fetch -v` 查看本地远程信息是否是最新的

## remote

`git remote -v` 查看当前本地仓库所对应的远程 fetch/push 地址

`git remote` 查看本地仓库所设置的远程仓库的别名

`git remote show 远程仓库别名` 通过网络查看本地仓库所配置的远程仓库的最新信息以及与本地仓库信息的差异

## branch

`git branch` 查看本地分支

`git branch -r` 查看远程分支

`git branch -a` 查看本地与远程分支

`git branch -vv` 显示本地分支并对本地与远程分支有关联的本地分支额外显示其与远程分支的位置信息

## hooks

![git hook flow](https://delicious-insights.com/assets/images/articles/git-hooks.png)

## 第一个阶段

想要让git对一个目录进行版本管理需要以下步骤：

- 进入要管理的文件夹 执行初始化命令

    ```git
    git init
    ```

- 管理目录下的文件状态

    ```git
    git status
    
    注：新增的文件和修改过后的文件都是红色
    ```

- 管理指定文件（红变绿）

    ```git
    git add 文件名
    git add .
    ```

- 个人信息配置：用户名、邮箱『一次即可』

    ```git
    git config --global user.email "you@example.com"
    git config --global user.name "Your Name"
    ```

- 生成版本

    ```git
    git commit -m '描述信息'
    ```

- 查看版本记录

    ```git
    git log
    ```

## 第二个阶段：拓展新功能

- 回滚至之前的版本

    ```git
    git log
    git reset --hard 版本号
    ```

- 回滚之后的版本

    ```git
    git reflog
    git reset --hard 版本号
    ```


# 分支

## 命令总结

- 查看分支

    ```git
    git branch
    ```

- 创建分支

    ```git
    git branch 分支名称
    ```

- 切换分支

    ```git
    git checkout 分支名称
    ```

- 分支合并（可能产生冲突）

    ```git
    git merge 要合并的分支
    注意：切换分支再合并
    ```

- 删除分支

    ```git
    git branch -d 分支名
    ```

# 提交代码到仓库

在家里上传代码

```git
1.给远程仓库起别名
	git remote add origin 远程仓库地址
2.向远程推送代码
	git push -u origin 分支名称
```

到公司新电脑上第一次获取代码

```git
1.克隆远程仓库代码
	git clone 远程仓库地址
2.切换分支
	git checkout 分支名称
```

在公司进行开发

```git
1.切换到dev分支进行开发
	git checkout dev
2.把master分支合并到dev[仅一次]
	git merge master
3.修改代码
4.提交代码
	git add .
	git commit -m 'xx'
	git push origin dev
```

回到家中继续写代码

```gti
1.切换到dev分支进行开发
	git checkout dev
2.拉代码
	git pull origin dev
3.继续开发
4.提交代码
	git add .
	git commit -m 'xx'
	git push origin dev
```

回到公司继续开发

```git
1.切换到dev分支进行开发
	git checkout dev
2.拉代码
	git pull origin dev
3.继续开发
4.提交代码
	git add .
	git commit -m 'xx'
	git push origin dev
```

## 快速解决冲突

1.安装beyond compare

2.在git中配置

```git
git --config --global merge.tool bc3
git --config --global mergetool.path '/usr/local/bin/bcom'
git --config --global mergetool.keepBackup false
```

3.应用beyond compare解决冲突

```git
gti mergetool
```

## 总结

- 添加远程连接（别名）

    ```git
    git remote add origin 地址
    ```

- 推送代码

    ```git
    git push origin dev
    ```

- 下载代码

    ```git
    git clone 地址
    ```

- 拉取代码（更新）

    ```git
    git pull origin dev
    等价于
    git fetch origin dev
    git merge origin dev
    ```

- 保持代码提交整洁（变基）

    ```git
    git rebase 分支
    ```

- 记录图形展示

    ```git
    git log --graph --pretty==format:"%h %s"
    ```

# 给开源软件贡献代码

1.fork源代码

​	将别人源码代码拷贝到自己的远程仓库

2.在自己仓库进行修改代码

3.给源代码作者提交修改的申请(pull request)

# 其他

## 配置

- 项目配置文件：项目/.git/config

    ```git
    git config --local user.name BBboy
    git config --local user.name BBboy@xx.com
    ```

- 全局配置文件：~/.gitconfig

    ```git
    git config --global user.name BBboy
    git config --global user.name BBboy@xx.com
    ```

- 系统配置文件：/etc/.gitconfig

    ```git
    git config --system user.name BBboy
    git config --system user.name BBboy@xx.com
    
    注意：需要有root权限
    ```

## 免密登录

- URL中体现

    ```git
    原来的地址：https://github.com/BBboy01/ChangeStype.git
    修改的地址：https://用户名:密码@github.com/BBboy01/ChangeStype.git
    
    git remote add origin https://用户名:密码@github.com/BBboy01/ChangeStype.git
    git push origin master
    ```

- SSH实现

    ```git
    1.生成公钥和私钥(默认放在 ~/.ssh)
    	ssh-keygen
    2.拷贝公钥的内容，并设置到github中
    3.在git本地中配置ssh地址
    	git remote add origin git@github.com:BBboy01/ChangeStype.git
    4.以后使用
    	git push origin master
    ```

- git自动管理凭证

## .gitignore规则不生效

.gitignore只能忽略那些原来没有被track的文件，如果某些文件已经被纳入了版本管理中，则修改.gitignore是无效的。

解决方法就是先把本地缓存删除（改变成未track状态），然后再提交:

```bash
git rm -r --cached .
git add .
git commit -m 'gitignore updated'
git push
```

## 配置 `commitizen`

1. 下载 `npm install -g commitizen cz-conventional-changelog`
2. 配置 `echo '{ "path": "cz-conventional-changelog" }' > ~/.czrc`
