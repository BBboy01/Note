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

1. 首先添加一个 GitLab CI/CD 变量，将此变量命名为 `STAGING_PRIVATE_KEY` 并将其设置为服务器的**私有 SSH 密钥**

2. 确保 `ssh-client` 可以使用 `which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )`

3. 禁用主机检查（当第一次连接到服务器时，不要求用户接受，因为每个作业都等于第一次连接，所以我们需要这样做）

```bash
echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config'
```

如果连接出现问题，可以先检查 `~/.ssh/config` 的权限是否是 600

将本地 ssh 公钥添加到远程服务器中 `ssh-copy-id 用户名@远程服务器地址`，后需手动输入远程服务器密码

### 部署：

```yaml
script:
  - ssh-add <(echo "$STAGING_PRIVATE_KEY")
  - ssh -p22 server_user@server_host "mkdir www/_tmp"
  - scp -P22 -r dist/* server_user@server_host:www/_tmp
  - ssh -p22 server_user@server_host "mv www/live www/_old && mv www/_tmp www/live"
  - ssh -p22 server_user@server_host "rm -rf www/_old"
```

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
3. 当要想环境变量添加或修改值时，需要在当前 job 中添加 `artifacts`、`reports`、`dotenv` 这几个关键字指明环境变量文件
