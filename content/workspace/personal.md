+++
title  = "personal"
toc    = true
weight = 0
tags = [ "terminal", "gnome" ]
+++

## Intro

개인 Desktop Environment 구축하기 위해 필요한 Step 정리.
현재 사용 중인 System 기준

- H/W : Chrombook Pixel2 LS
- OS : ApricityOS
- Desktop : Gnome3
- Terminal : zsh + tmux + nvim + powerline + fzf

Terminal 관련 상세 설정은 [여기](https://github.com/keyolk/.dotfiles)에서 확인할 수 있다.

## Chromebook
* enable developer mode
* enable SeaBIOS

## ApricityOS
### Install
https://apricityos.com/download

### Kernel
https://github.com/raphael/linux-samus

### Package
```
#!/bin/bash
git submodule update --init --recursive

sudo pacman -Syy zsh tmux neovim git xclip meld python-pip
sudo pacman -Syy python-neovim python2-neovim
sudo pacman -Syy fzf the_silver_searcher
sudo pacman -Syy docker vagrant

# install python virtualenv
sudo pacman -Syy pythob-virtualenv python2-virtualenv

# install gvm to manage golang version and workspace
zsh < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)

# install docker-machine
sudo sh -c "curl -L https://github.com/docker/machine/releases/download/v0.7.0/docker-machine-`uname -s`-`uname -m` > /usr/local/bin/docker-machine && \
chmod +x /usr/local/bin/docker-machine"

# install docker-compose
sudo sh -c "curl -L https://github.com/docker/compose/releases/download/1.7.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose; chmod +x /usr/local/bin/docker-compose"

# install nautilus-compare to intergrate meld with nautilus
git clone https://aur.archlinux.org/nautilus-compare.git ~/workspace/build/aur/nautilus-compare
cd ~/workspace/build/aur/nautilus-compare
makepkg -si --noconfirm
```

## Gnome
Gnome 환경 설정 관련 정리.

### Workspace
Dual Monitor를 쓸때 Secondary Monitor가 Workspace와 동기화되게 하려면 아래와 gnome setting이 수정되어야 한다.
```bash
$ gsettings set org.gnome.shell.overrides workspaces-only-on-primary false
```

Alt tab이 현재 workspace 내에 존재하는 application 사이에서만 switching되게 하려면,
```bash
$ gsettings set org.gnome.shell.app-switcher current-workspace-only true
```

### InputMethod
한글 입력기론 ibus-hangul을 쓴다.
입력기 관련 설정은 아래에서 찾을 수 있다.

```
$ cat /etc/environment
```
```
#
# This file is parsed by pam_env module
#
# Syntax: simple "KEY=VAL" pairs on separate lines
#
BROWSER=/usr/bin/google-chrome-stable
EDITOR=nvim
GTK_IM_MODULE=ibus
XMODIFIERS=@im=ibus
QT_IM_MODULE=ibus
```

Universal Access에서 Screen Keyboard를 Disable해도
계속 나타나는 버그가 있다.

아래 dbus service에서 관련 항목에 Exec들을 comment해버리면 일단 해결된다.
`$ vi /usr/share/dbus-1/services/org.gnome.Caribou.Antler.service`
`$ vi /usr/share/dbus-1/services/org.gnome.Caribou.Daemon.service`

### Window Manager
Gnome 3.22가 되면서 wayland가 deafult로 설정된다. +
아직 wayland랑 안붙는 utility가 많으므로 아래와 같이 다시 xorg를 쓰도록 한다.

`$ vi /etc/gdm/custom.conf`

daemon 설정에 아래 line을 넣자.

`WaylandEnable=false`


### Desktop
FreeDesktopGroup 과 관련된 표준이 존재한다.
기본 Directory가 거슬릴때가 있는데 아래와 같이 수정할 수 있다.

```bash
$ xdg-user-dirs-update --set DOWNLOAD $HOME/desktop/download
$ xdg-user-dirs-update --set PUBLICSHARE $HOME/desktop/public
$ xdg-user-dirs-update --set VIDEOS $HOME/desktop/videos
$ xdg-user-dirs-update --set TEMPLATES $HOME/desktop/templates
$ xdg-user-dirs-update --set PICTURES $HOME/desktop/pictures
$ xdg-user-dirs-update --set DOCUMENTS $HOME/desktop/documents
$ xdg-user-dirs-update --set MUSIC $HOME/desktop/music
```

## ZSH
ZSH 관련 설정 정리.

### Plugin Manager
antigeen을 plugin managemnt tool로 사용한다.
아래 bundler들을 사용한다.
```bash
# set dotfiles path
export DOTFILES=$HOME/.dotfiles

# Use antigen as plugin manager
source $DOTFILES/zsh/antigen.zsh

antigen use oh-my-zsh
antigen bundle zsh-users/zsh-syntax-highlighting
antigen bundle zsh-users/zsh-history-substring-search
antigen bundle command-not-found
antigen bundle colorize
antigen bundle colored-man-pages
antigen bundle vagrant
antigen bundle docker
antigen bundle python
antigen bundle git
antigen bundle aws
antigen bundle golang
antigen bundle python
antigen bundle pip

antigen apply

# Source all *.zsh under .dotfiles
for config_file ($DOTFILES/**/*.zsh) source $config_file

# Put here private configuration
if [[ -a ~/.localrc ]]
then
  source ~/.localrc
fi

# Prevet nested tmux session
if [[ -z "$TMUX" ]]; then
  exec tmux
fi

# Set GVM for Golang
[[ -s "/home/keyolk/.gvm/scripts/gvm" ]] && source "/home/keyolk/.gvm/scripts/gvm"

# Set default golang
gvm use go1.7.3 &> /dev/null

# Set default golang project
#gvm pkgset use container &> /dev/null

# Use fzf for fuzzy search
source /usr/share/fzf/completion.zsh
source /usr/share/fzf/key-bindings.zsh

export PATH=$PATH:/home/keyolk/.gem/ruby/2.3.0/bin
export PATH=$PATH:/home/keyolk/tool

source <(kubectl completion zsh)

source ~/workspace/cnct/k2.sh
```

```
# history file
HISTFILE=~/.histfile
HISTSIZE=10000
SAVEHIST=10000

# for better directory navigation
setopt AUTO_PUSHD
setopt PUSHD_MINUS
setopt CDABLE_VARS
zstyle ':completion:*:directory-stack' list-colors '=(#b) #([0-9]#)*( *)==95=38;5;12'

# Use vim mode
bindkey -v
export KEYTIMEOUT=1

# Use powerline
source /usr/share/zsh/site-contrib/powerline.zsh

## binding for home/end key
# for system
bindkey '\e[1~' beginning-of-line
bindkey '\e[4~' end-of-line
bindkey '\e[7~' beginning-of-line
bindkey '\e[8~' end-of-line
bindkey '\eOH' beginning-of-line
bindkey '\eOF' end-of-line
bindkey '\e[H' beginning-of-line
bindkey '\e[F' end-of-line

# for vi-mode
bindkey -M vicmd '\e[1~' beginning-of-line
bindkey -M vicmd '\e[4~' end-of-line

## binding for delete key
# for system
bindkey '\e[3~' delete-char

# for vi-mode
bindkey -M vicmd '\e[3~' delete-char

## binding for history-substirng-search
# for system
zmodload zsh/terminfo
bindkey "$terminfo[kcuu1]" history-substring-search-up
bindkey "$terminfo[kcud1]" history-substring-search-down

# for vi-mode
bindkey -M vicmd 'k' history-substring-search-up
bindkey -M vicmd 'j' history-substring-search-down
```

```
alias ls='ls --color -h --group-directories-first'
```

## TMUX
```
# To prevent ESC holding on vi
set -s escape-time 0

# Change prefix key
set -g prefix C-a
unbind C-b
bind C-a send-prefix

# Set mouse
set -g mouse on

# act like vim
setw -g mode-keys vi

# Set the terminal type so colors get rendered correctly
set -g default-terminal "screen-256color"

set -g default-shell "/usr/bin/zsh"
set -g default-command "zsh"
set -g renumber-windows on

# Powerline
source /usr/share/tmux/powerline.conf

#### Key bindings
# ctrl-r: Reload tmux config
bind r source-file ~/.tmux.conf \; display 'Config reloaded'

# sync pane
bind y set-window-option synchronize-panes

# window naviation
bind-key space next-window
bind-key bspace previous-window

# layout
bind-key enter next-layout

# Ctrl-[hjkl]:
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R

bind -n C-h run "(tmux display-message -p '#{pane_current_command}' | grep -iq vim && tmux send-keys C-h) || tmux select-pane -L"
bind -n C-j run "(tmux display-message -p '#{pane_current_command}' | grep -iq vim && tmux send-keys C-j) || tmux select-pane -D"
bind -n C-k run "(tmux display-message -p '#{pane_current_command}' | grep -iq vim && tmux send-keys C-k) || tmux select-pane -U"
bind -n C-l run "(tmux display-message -p '#{pane_current_command}' | grep -iq vim && tmux send-keys C-l) || tmux select-pane -R"

# alt-[hjkl]
bind M-h resize-pane -L 5
bind M-j resize-pane -D 5
bind M-k resize-pane -U 5
bind M-l resize-pane -R 5

bind -n M-h run "(tmux display-message -p '#{pane_current_command}' | grep -iq vim && tmux send-keys M-h) || tmux resize-pane -L 5"
bind -n M-j run "(tmux display-message -p '#{pane_current_command}' | grep -iq vim && tmux send-keys M-j) || tmux resize-pane -D 5"
bind -n M-k run "(tmux display-message -p '#{pane_current_command}' | grep -iq vim && tmux send-keys M-k) || tmux resize-pane -U 5"
bind -n M-l run "(tmux display-message -p '#{pane_current_command}' | grep -iq vim && tmux send-keys M-l) || tmux resize-pane -R 5"

# Window Split
bind v split-window -h -c "#{pane_current_path}"
bind s split-window -v -c "#{pane_current_path}"
bind q confirm killp

# Make Home and End keys work in copy mode
unbind-key -t vi-copy Home
bind-key -t vi-copy Home start-of-line
unbind-key -t vi-copy End
bind-key -t vi-copy End end-of-line

# Plugins
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'
set -g @plugin 'christoomey/vim-tmux-navigator'
run '~/.tmux/plugins/tpm/tpm'
```

## NVIM
NVim 설정 정리.

### Key Binding
nvim 에서 Ctrl+[hH] 동작은 terminal 예약키와 겹쳐서 key binding이 안될 수 있다.

아래와 같이 terminal 정보 수정이 필요하다.
```
$ infocmp $TERM | sed 's/kbs=^[hH]/kbs=\\177/' > ~/$TERM.ti
$ tic ~/$TERM.ti
$ rm ~/$TERM.ti
```

## Troubleshooting
### Powerline
* gnome 3.22 update 이후 powerlien character가 밀리는 [현상](https://github.com/powerline/powerline/issues/1652)이 있다.

아래와 같이 문제가되는 문자를 바꿔준다.

_powerline.json_
```diff
"time": {
-   "before": "◴ "
+   "before": " "
},
```
