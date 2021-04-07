# 基础

## 第一个阶段

想要让git对一个目录进行版本管理需要以下步骤：

- 进入要管理的文件夹

- 执行初始化命令

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
    git config --local user.name = 'BBboy'
    git config --local user.name = 'BBboy@xx.com'
    ```

- 全局配置文件：~/.gitconfig

    ```git
    git config --global user.name = 'BBboy'
    git config --global user.name = 'BBboy@xx.com'
    ```

- 系统配置文件：/etc/.gitconfig

    ```git
    git config --system user.name = 'BBboy'
    git config --system user.name = 'BBboy@xx.com'
    
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

