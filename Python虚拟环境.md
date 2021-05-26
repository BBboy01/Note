**创建虚拟环境：**

```shell
virtualenv envname01  |  mkvirtualenv envname02 
右边的创建后会自动进入虚拟环境
```

**基于某Python环境创建虚拟环境：**

```bash
mkvirtualenv -p python2.7 envname01
mkvirtualenv -p python3.6 envname02
```

**列出虚拟环境：**

```shell
lsvirtualenv -b  |  workon
# 或者
workon
```

**退出虚拟环境：**

```shell
deactivate
```

**切换虚拟环境：**

```shell
workon envname02  # 这里写你要去的虚拟环境的文件夹名
```

**为虚拟环境安装模块：**

```shell
pip install 模块名  |  pip3 install 模块名
```

**为虚拟环境卸载模块：**

```shell
pip uninstall 模块名  |  pip3 uninstall 模块名
```

**查看虚拟环境里安装了哪些包：**

```sh
lssitepackages  |  pip list  |  pip3 list
```

**进入|退出 该虚拟环境的Python环境**

```shell
python  |  exit()
```

**复制虚拟环境：**

```shell
cpvirtualenv env1 env2  # 前面的是原文件 后面的拷贝后的新文件
///   Copying env1 as env2...
```

**删除虚拟环境：**

```shell
rmvirtualenv env2
///   Removing env2...
```