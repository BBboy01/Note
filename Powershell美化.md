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

## 个人 `oh-my-posh` 配置
```json
{
  "$schema": "https://raw.githubusercontent.com/JanDeDobbeleer/oh-my-posh/main/themes/schema.json",
  "blocks": [
    {
      "type": "prompt",
      "alignment": "left",
      "segments": [
        {
          "type": "root",
          "style": "powerline",
          "powerline_symbol": "\uE0C4",
          "foreground": "#242424",
          "background": "#f1184c",
          "properties": {
            "prefix": "",
            "postfix": ""
          }
        },
        {
          "type": "time",
          "style": "powerline",
          "powerline_symbol": "\uE0C4",
          "foreground": "#FFBB00",
          "background": "#242424",
          "properties": {
            "time_format": "15:04",
            "prefix": ""
          }
        },
        {
          "type": "path",
          "style": "powerline",
          "powerline_symbol": "\uE0B0",
          "foreground": "#33DD2D",
          "background": "#242424",
          "properties": {
            "prefix": "\uE5FF ",
            "style": "folder",
            "folder_separator_icon": "/"
          }
        },
        {
          "type": "git",
          "style": "powerline",
          "powerline_symbol": "\uE0B0",
          "foreground": "#3A86FF",
          "background": "#242424",
          "properties": {
            "fetch_status": true,
            "fetch_stash_count": true,
            "fetch_upstream_icon": true,
            "template": "{{ .UpstreamIcon }}{{ .HEAD }}{{ if .Staging.Changed }} \uF046 {{ .Staging.String }}{{ end }}{{ if and (.Working.Changed) (.Staging.Changed) }} |{{ end }}{{ if .Working.Changed }} \uF044 {{ .Working.String }}{{ end }}{{ if gt .StashCount 0 }} \uF692 {{ .StashCount }}{{ end }}",
            "prefix": ""
          }
        },
        {
          "type": "dotnet",
          "style": "powerline",
          "powerline_symbol": "\uE0C4",
          "foreground": "#ffffff",
          "background": "#0184bc",
          "properties": {
            "prefix": ""
          }
        },
        {
          "type": "exit",
          "style": "powerline",
          "powerline_symbol": "\uE0B4",
          "foreground": "#242424",
          "background": "#33DD2D",
          "background_templates": ["{{ if gt .Code 0 }}#f1184c{{ end }}"],
          "properties": {
            "template": "\uF7d4"
          }
        },
        {
          "type": "python",
          "style": "powerline",
          "powerline_symbol": "\uE0B0",
          "foreground": "#100e23",
          "background": "#906cff",
          "properties": {
            "prefix": "\uE235"
          }
        },
        {
          "type": "node",
          "style": "powerline",
          "powerline_symbol": "\uE0B0",
          "foreground": "#845EC2",
          "background": "#242424",
          "properties": {
            "prefix": "\uE718"
          }
        }
      ]
    }
  ]
}
```