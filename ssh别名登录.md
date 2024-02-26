# ssh免密登录

- 生成公、私钥

> 一路`Enter`，最后会在 ~/.ssh 目录下生成两个文件 id_rsa.pub和id_rsa ，前者是公钥，后者是私钥

```shell
ssh-keygen -t ed25519 -C '注释'
```

- 之后将公钥添加到服务器上

```shell
ssh-copy-id -i ~/.ssh/id_rsa.pub 服务器主机用户名@服务器地址
```

# ssh别名登录

- 进入`~/.ssh`中，创建`config`文件，接着进行编辑`vim ~/.ssh/config`

```shell
#服务器别名
Host nickname
#服务器地址
HostName 服务器地址
#服务器用户名
User root
```

然后便可以通过`ssh nickname`别名登录服务器了

# 添加**心跳**避免长时间不操作导致断开

`vim ~/.ssh/config`

```bash
Host *
	ServerAliveInterval 30
  TCPKeepAlive yes
  ServerAliveCountMax 6
  Compression yes
```
