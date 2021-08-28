

# 最终效果图

![image-20210828140429680](https://gitee.com/RealBBboy/mark-down-images-repo/raw/master/NoteImg/image-20210828140429680.png)

# 配置快捷方式与管理员权限

找到 `Git Bash` 所在位置，右键`属性` ，设置快捷键

![快捷方式配置](https://upload-images.jianshu.io/upload_images/5131627-96f9c44134daf322.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/813/format/webp)

然后点 `高级`，选择 `用管理员身份运行`，`应用` 后 `确认` 第一步就完成了

![管理员权限](https://upload-images.jianshu.io/upload_images/5131627-153faab98622724a.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/601/format/webp)

# 配置文件

如果你安装的有 VS Code，就可以直接 `code ~/.minttyrc` 打开配置文件，将下列代码复制粘贴进去，没有的话就用 `vim` 呗。`.minttyrc` 就是 `git bash` 的配置文件。

```swift
Font=Jetbrains Mono
FontHeight=14
Transparency=low
FontSmoothing=full
Locale=zh_CN
Charset=UTF-8
Columns=88
Rows=26
OpaqueWhenFocused=no
Scrollbar=none
Language=zh_CN

ForegroundColour=255,255,255
BackgroundColour=0,43,54
CursorColour=220,130,71

BoldBlack=128,128,128
Green=64,200,64
BoldGreen=64,255,64
Yellow=190,190,0
BoldYellow=255,255,64
Blue=135,144,255
BoldBlue=30,144,255
Magenta=211,54,130
BoldMagenta=255,128,255
Cyan=64,190,190
BoldCyan=128,255,255
White=250,240,230
BoldWhite=250,240,230

BellTaskbar=no
Term=xterm-256color
FontWeight=400
FontIsBold=no
BellType=0

CtrlShiftShortcuts=yes
ConfirmExit=no
AllowBlinking=yes
BoldAsFont=no
```



# 终端显示美化

修改`git安装目录`下的`etc/profile.d/git-prompt.sh`（有中文注释的表示自己过的修改）

```sh
if test -f /etc/profile.d/git-sdk.sh
then
	TITLEPREFIX=SDK-${MSYSTEM#MINGW}
else
	TITLEPREFIX=$MSYSTEM
fi

if test -f ~/.config/git/git-prompt.sh
then
	. ~/.config/git/git-prompt.sh
else
	PS1='\[\033]0;Bash In $PWD\007\]' # 窗口标题
	PS1="$PS1"''                 # 结果显示结束后不换行
	PS1="$PS1"'\[\033[32m\]'       # change to green
	PS1="$PS1"'λ'             # 用户名
	PS1="$PS1"'\[\033[35m\]'       # change to purple
	PS1="$PS1"''          # 不显示系统名
	PS1="$PS1"'\[\033[33m\]'       # change to brownish yellow
	PS1="$PS1"'\w'                 # current working directory
	if test -z "$WINELOADERNOEXEC"
	then
		GIT_EXEC_PATH="$(git --exec-path 2>/dev/null)"
		COMPLETION_PATH="${GIT_EXEC_PATH%/libexec/git-core}"
		COMPLETION_PATH="${COMPLETION_PATH%/lib/git-core}"
		COMPLETION_PATH="$COMPLETION_PATH/share/git/completion"
		if test -f "$COMPLETION_PATH/git-prompt.sh"
		then
			. "$COMPLETION_PATH/git-completion.bash"
			. "$COMPLETION_PATH/git-prompt.sh"
			PS1="$PS1"'\[\033[36m\]'  # change color to cyan
			PS1="$PS1"'`__git_ps1`'   # bash function
		fi
	fi
	PS1="$PS1"'\[\033[32m\]'        # 命令颜色为绿色
	PS1="$PS1"''                 # 提示符与命令间不换行
	PS1="$PS1"' '                 # 命令前无提示符
fi

MSYS2_PS1="$PS1"               # for detection by MSYS2 SDK's bash.basrc

# Evaluate all user-specific Bash completion scripts (if any)
if test -z "$WINELOADERNOEXEC"
then
	for c in "$HOME"/bash_completion.d/*.bash
	do
		# Handle absence of any scripts (or the folder) gracefully
		test ! -f "$c" ||
		. "$c"
	done
fi
```

