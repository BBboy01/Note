# 如何访问全局命令
以 `create-react-app` 为例，当全局安装之后，获取环境变量中所使用的 `create-react-app`：

```bash
which create-react-app
# C:\Users\11566\scoop\apps\nvm\current\nodejs\nodejs\create-react-app.ps1
```

而此处的 `nodejs` 则是 `nvm` 对当前使用版本的 `node` 的软连接：

```bash
cd C:\Users\11566\scoop\apps\nvm\current\nodejs && ll
# nodejs -> C:\Users\11566\scoop\persist\nvm\nodejs\v16.13.2
```

当进入到该 `node` 的目录下后