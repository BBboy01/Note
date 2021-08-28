最终效果（基于`Windows Terminal`）

![image-20210828153145111](https://gitee.com/RealBBboy/mark-down-images-repo/raw/master/NoteImg/image-20210828153145111.png)

打开 powershell 检查是否以及拥有配置文件

```shell
Test-Path $profile
```

如果返回结果为`false`则说明没有，`true`则表示以及有相应的配置文件，直接输入`$profile`即可查看配置文件路径

- 生成配置文件

```shell
New-Item -path $profile -type file -force
```

然后找到配置文件，复制如下代码

```shell
cls  # 清除微软广告

$path = $pwd.path
if ( $path.split("\")[-1] -eq "System32" ) {
    # change default path to desktop
    $desktop = "C:\Users\" + $env:UserName + "\Desktop\"
    cd $desktop
}

Set-PSReadLineOption -Colors @{
    Command             = "#e5c07b"
    Number              = "#cdd4d4"
    Member              = "#e06c75"
    Operator            = "#e06c75"
    Type                = "#78b6e9"
    Variable            = "#78b6e9"
    Parameter           = "#e06c75"  #命令行参数颜色
    ContinuationPrompt  = "#e06c75"
    Default             = "#cdd4d4"
    Emphasis            = "#e06c75"
    #Error
    Selection           = "#cdd4d4"
    Comment             = "#cdd4d4"
    Keyword             = "#e06c75"
    String              = "#78b6e9"
}

function prompt
{
    $path = $pwd.path
    if ( -not $path.EndsWith("\") ) {
        "" + $path.split("\")[-1] + " λ "
    }
    else {
        "" + $path.split("\")[0] + " λ "
    }
}
```

