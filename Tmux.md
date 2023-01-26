# Terminal command

- create a new session `tmux new -s SESSION_NAME`
- detach the current session `tmux detach`
- attach a session with session name `tmux attach -t SESSION_NAME`
- list all sessions on terminal `tmux ls`


# Tmux command

> My prefix key is <C-a>


## Session

- list all of sessions of current window `<C-a>s`


## Window

- create a new window `<C-a>c [window name]`
- rename current window `<C-a>,`
- move to next window `<C-a>n`
- move to previous window `<C-a>p`
- move to specific index of window `<C-a>WINDOW_INDEX_NUMBER`
- list all of windows with sessions under `<C-a>w`


## Copy mode

> On copy mode, you can move/select terminal content like vim(`v` to select, `y` to copy)

```conf
set-window-option -g mode-keys vi

bind-key -T copy-mode-vi 'v' send -X begin-selection
bind-key -T copy-mode-vi 'y' send -X copy-selection

# make mouse selectable
unbind -T copy-mode-vi MouseDragEnd1Pane
```

- open copy mode `<C-a>[`
- exit copy mode `<C-c>`
