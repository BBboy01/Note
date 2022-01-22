# 配置参数

## 常用配置项说明

`apps`： json结构，apps是一个数组，每一个数组成员就是对应一个pm2中运行的应用；

`name`：应用程序名称；

`cwd`：应用程序所在的目录；

`script`：应用程序的脚本路径；

`log_date_format`： 指定日志日期格式，如YYYY-MM-DD HH：mm：ss；

`error_file`：自定义应用程序的错误日志文件，代码错误可在此文件查找；

`out_file`：自定义应用程序日志文件，如应用打印大量的标准输出，会导致pm2日志过大；

`pid_file`：自定义应用程序的pid文件；

`interpreter`：指定的脚本解释器；

`interpreter_args`：传递给解释器的参数；

`instances`： 应用启动实例个数，仅在cluster模式有效，默认为fork；

`min_uptime`：最小运行时间，这里设置的是60s即如果应用程序在60s内退出，pm2会认为程序异常退出，此时触发重启max_restarts设置数量；

`max_restarts`：设置应用程序异常退出重启的次数，默认15次（从0开始计数）；

`autorestart` ：默认为true, 发生异常的情况下自动重启；

`cron_restart`：定时启动，解决重启能解决的问题；

`max_memory_restart`：最大内存限制数，超出自动重启；

`watch`：是否启用监控模式，默认是false。如果设置成true，当应用程序变动时，pm2会自动重载。这里也可以设置你要监控的文件。

`ignore_watch`：忽略监听的文件夹，支持正则表达式；

`merge_logs`： 设置追加日志而不是新建日志；

`exec_interpreter`：应用程序的脚本类型，默认是nodejs；

`exec_mode`：应用程序启动模式，支持fork和cluster模式，默认是fork；

`autorestart`：启用/禁用应用程序崩溃或退出时自动重启；

`vizion`：启用/禁用vizion特性(版本控制)；

`env`：环境变量，object类型；

`force`：默认false，如果true，可以重复启动一个脚本。pm2不建议这么做；

`restart_delay`：异常重启情况下，延时重启时间

## 实际配置

**ecosystem.config.js**

```json
{
    "apps": {
        "name": "wuwu",                             // 项目名          
        "script": "./bin/www",                      // 执行文件
        "cwd": "./",                                // 根目录
        "args": "",                                 // 传递给脚本的参数
        "interpreter": "",                          // 指定的脚本解释器
        "interpreter_args": "",                     // 传递给解释器的参数
        "watch": true,                              // 是否监听文件变动然后重启
        "ignore_watch": [                           // 不用监听的文件
            "node_modules",
            "logs"
        ],
        "exec_mode": "cluster_mode",                // 应用启动模式，支持fork和cluster模式
        "instances": 4,                             // 应用启动实例个数，仅在cluster模式有效 默认为fork；或者 max
        "max_memory_restart": 8,                    // 最大内存限制数，超出自动重启
        "error_file": "./logs/app-err.log",         // 错误日志文件
        "out_file": "./logs/app-out.log",           // 正常日志文件
        "merge_logs": true,                         // 设置追加日志而不是新建日志
        "log_date_format": "YYYY-MM-DD HH:mm:ss",   // 指定日志文件的时间格式
        "min_uptime": "60s",                        // 应用运行少于时间被认为是异常启动
        "max_restarts": 30,                         // 最大异常重启次数，即小于min_uptime运行时间重启次数；
        "autorestart": true,                        // 默认为true, 发生异常的情况下自动重启
        "cron_restart": "",                         // crontab时间格式重启应用，目前只支持cluster模式;
        "restart_delay": "60s"                      // 异常重启情况下，延时重启时间
        "env": {
           "NODE_ENV": "production",                // 环境参数，当前指定为生产环境 process.env.NODE_ENV
           "REMOTE_ADDR": "爱上大声地"               // process.env.REMOTE_ADDR
        },
        "env_dev": {
            "NODE_ENV": "development",              // 环境参数，当前指定为开发环境 pm2 start app.js --env_dev
            "REMOTE_ADDR": ""
        },
        "env_test": {                               // 环境参数，当前指定为测试环境 pm2 start app.js --env_test
            "NODE_ENV": "test",
            "REMOTE_ADDR": ""
        }
    }
}
```

## 模板文件

> pm2 ecosystem

```js

{
  /**
   * Application configuration section
   * PM2 - Application Declaration
   */
  apps : [
    // First application
    {
      name      : "API",
      script    : "app.js",
      env: {
        COMMON_VARIABLE: "true"
      },
      env_production : {
        NODE_ENV: "production"
      }
    },
    // Second application
    {
      name      : "WEB",
      script    : "web.js"
    }
  ],
  /**
   * Deployment section
   * PM2 - Deployment
   */
  deploy : {
    production : {
      user : "node",
      host : "47.98.154.75",
      ref  : "origin/master",
      repo : "git@github.com:awhlmycn/myWeb.git",
      path : "/home/temp",
      "post-deploy" : "npm install && pm2 startOrRestart ecosystem.json --env production"
    },
    dev : {
      user : "node",
      host : "47.98.154.75",
      ref  : "origin/master",
      repo : "git@github.com:repo.git",
      path : "/home/temp",
      "post-deploy" : "npm install && pm2 startOrRestart ecosystem.json --env dev",
      env  : {
        NODE_ENV: "dev"
      }
    }
  }
```

### deploy

服务器上所做的操作

```js
production : {
    user : "登录远程服务器的用户名，此处填写我们创建的yishi",
	host : "远程服务器的IP或hostname，此处可以是数组同步部署多个服务器，不过鉴于我们只有一个服务器，因此我们填写123.57.205.23",
	ref  : "远端名称及分支名，此处填写origin/master",
	repo : "git仓库地址，此处填写git@github.com:e10101/pm2app.git",
	path : "远程服务器部署目录，需要填写user具备写入权限的目录，此处填写/home/yishi/www/production",
	"post-deploy" : "部署后需要执行的命令，此处填写npm install && pm2 startOrRestart ecosystem.json --env production"
},
```

```js
production: {
      user: "root",
      host: process.env.SERVE_HOST,
      ref: "origin/master",
      repo: "git@github.com:BBboy01/bbboyBackend.git",
      path: "/ftp/node/bbboy_blog",
      "post-deploy":
        "git pull && yarn && pm2 reload ecosystem.config.js --env production",
    },
```

## 部署

```sh
# 第一次运行
$ pm2 deploy ecosystem.config.js production setup
$ pm2 deploy ecosystem.config.js production

# 以后更新代码后运行
$ pm2 deploy ecosystem.config.js production update
```

# 常用命令

- 查看运行中的列表 `pm2 ls`
- 以主机最大核心数启动服务`pm2 start app.js -i max`
- 生产配置文件模板`pm2 init`/`pm2 ecosystem`
- 启动服务 `pm2 start <id|name>`
- 停止服务 `pm2 stop 1`
- 重启服务 `pm2 restart 1`
- 删除服务 `pm2 delete 1`
- 重命名服务 `pm2 restart <id|name> --name NEW_NAME`
- 托管静态文件（index.html）`pm2 serve /usr/www/ --name YOUR_NAME`

```sh
$ pm2 start ecosystem.config.js # 根据配置文件启动
$ pm2 start app.js -i 4 #后台运行pm2，启动4个app.js 
                                # 也可以把'max' 参数传递给 start
                                # 正确的进程数目依赖于Cpu的核心数目
$ pm2 start app.js --name my-api # 命名进程
$ pm2 list               # 显示所有进程状态
$ pm2 monit              # 监视所有进程
$ pm2 logs               #  显示所有进程日志
$ pm2 stop all           # 停止所有进程
$ pm2 restart all        # 重启所有进程
$ pm2 reload all         # 0秒停机重载进程 (用于 NETWORKED 进程)
$ pm2 stop 0             # 停止指定的进程
$ pm2 restart 0          # 重启指定的进程
$ pm2 startup            # 产生 init 脚本 保持进程活着
$ pm2 web                # 运行健壮的 computer API endpoint (http://localhost:9615)
$ pm2 delete 0           # 杀死指定的进程
$ pm2 delete all         # 杀死全部进程

运行进程的不同方式：
$ pm2 start app.js -i max  # 根据有效CPU数目启动最大进程数目
$ pm2 start app.js -i 3      # 启动3个进程
$ pm2 start app.js -x        #用fork模式启动 app.js 而不是使用 cluster
$ pm2 start app.js -x -- -a 23   # 用fork模式启动 app.js 并且传递参数 (-a 23)
$ pm2 start app.js --name serverone  # 启动一个进程并把它命名为 serverone
$ pm2 stop serverone       # 停止 serverone 进程
$ pm2 start app.json        # 启动进程, 在 app.json里设置选项
$ pm2 start app.js -i max -- -a 23                   #在--之后给 app.js 传递参数
$ pm2 start app.js -i max -e err.log -o out.log  # 启动 并 生成一个配置文件

你也可以执行用其他语言编写的app  ( fork 模式):
$ pm2 start my-bash-script.sh    -x --interpreter bash
$ pm2 start my-python-script.py -x --interpreter python
```

