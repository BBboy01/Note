## 令远程服务器可以使用本地代理

- `ssh -NR localhost:local_proxy_port:localhost:remote_will_proxy_port username:host`
- 登录远程服务器 `ssh username:host`
- 设置服务器网络请求代理
  - `export https_proxy=https://127.0.0.1:remote_will_proxy_port`
  - `export http_proxy=http://127.0.0.1:remote_will_proxy_port`

## 创建的子用户可以使用免密登录

创建用户

- `useradd username` 创建用户
- `passwd username` 设置密码
- `usermod -aG username` 添加组

添加免密

- `ssh-keygen -t rsa` 生成密钥对
- `chmod 700 .ssh` 更正文件夹权限
- `chmod 600 .ssh/authorized_keys` 更正文件权限

## 文件查找

- 搜索当前文件中指定内容 `grep 'content' filename`
- 搜索当前目录下所有包含制定内容的文件 `grep -rl 'content' *`
- 搜索当前目录下所有包含制定内容的文件并且指定文件类型

  ```bash
  for x in `find . -name '*.ext' -o -name '*.html'`
  do
    grep -il 'content' "$x"
  done

  for x in `find . \( ! -iname '*.ext' \) -a \( -type f \)`
  do
    grep -il 'content' "$x"
  done
  ```
- 提取文件后缀 `${filename%.*}`
- 提取去除后缀后的文件名 `${filename##*.}`

## 终端操作

- `<C-u>` 从光标开始清除至行首
- `<C-k>` 从光标开始清除至行尾
- `<C-w>` 从光标开始向前清除一个以空格分隔的部分
- `<C-a>` 光标移动至行首
- `<C-e>` 光标移动至行尾部
- `<C-xx>` 光标跳转于当前位置和尾部
