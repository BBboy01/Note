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