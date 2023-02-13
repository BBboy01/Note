# CI

## GPG Key

- 安装 `brew install gpg2`
- 生成 rsa-rsa 密钥对 `gpg --full-gen-key`，基本可以一路回车，输入邮箱环节需要输入对应的远程 git 仓库注册邮箱
- 查看所有以生成的 gpg `gpg -k`
- 输出对应邮箱的 gpg rsa 公钥 `gpg --armor --export 邮箱`
- 将输出的公钥添加到远程 git 仓库中个人信息的 GPG 配置
- 于对应的本地仓库中添加 git 开启签名配置：
  - `git config user.name 用户名`
  - `git config user.email 签名邮箱`
  - `git config user.signingkey 签名邮箱`
  - `git config commit.gpgsign true`

## Runner

### 权限问题：

当第一次安装完 gitlab-runner 后，gitlab-runner 是基于 `gitlab-runner` 用户组运行的，
如果是注册为 docker 类型的 runner，则需要运行 `usermod -aG docker` 将其添加到 docker 组中以获得对应的执行权限

### 连接远程服务器：

gitlab-runner 执行命令需要避免交互，因此在连接远程服务器时如果想避免出现首次连接未知地址服务器确认的交互，则需要添加以下配置：

```bash
cat > ~/.ssh/config << 'EOF'
Host *
  ServerAliveInterval=30
  StrictHostKeyChecking no
  UserKnownHostsFile=/dev/null
EOF
```

- 切换到 gitlab-runner 用户生成 ssh-key `su - gitlab-runner`
- 配置 ssh 配置文件权限 `chmod 600 -R ~/.ssh/config`
- 生成 ssh 公私钥 `ssh-keygen`
- 将本地 ssh 公钥添加到远程服务器中 `ssh-copy-id 用户名@远程服务器地址`，后需手动输入远程服务器密码

## 环境变量

前置：

- 于 CI 设置中添加 `INSTANCE_VARIABLE=instance_variable_dev` 环境为 dev
- 于 CI 设置中添加 `INSTANCE_VARIABLE=instance_variable_pro` 环境为 pro

使用：

```yaml
stages:
  - deps
  - build
  - deploy

variables:
  LOCAL_VARIABLE: "local_variable"

install_deps:
  stage: deps
  script:
    - echo $LOCAL_VARIABLE
    - echo "DEPS_VARIABLE=deps_variable" >> env.txt
    - echo $INSTANCE_VARIABLE
  environment:
    name: dev
  artifacts:
    reports:
      dotenv: env.txt

build:
  stage: build
  script:
    - echo $DEPS_VARIABLE
    - echo "BUILD_VARIABLE=build_variable" >> env.txt
    - echo $INSTANCE_VARIABLE
  environment:
    name: pro
  artifacts:
  reports:
    dotenv: env.txt

deploy:
  stage: deploy
  script:
    - echo $DEPS_VARIABLE
    - echo $BUILD_VARIABLE
```

结论：

1. instance 级别的变量可以通过不同的环境来区分使用
2. 不同 job 间可以随时读取环境变量中的值，不必声明额外配置
3. 当要想环境变量添加或修改值时，需要在当前 job 中添加 `artifacts`、`reports`、dotenv` 这几个关键字指明环境变量文件
