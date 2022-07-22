---
title: 命令行配置备忘
categories: [Tech]
tags: [linux, shell, bash, alias]
date: 2020-04-04
---

换到新的机器，命令行配置少不了。

<!-- more -->

## zsh & oh-my-zsh

大概是需要代理加速的，下面是简要步骤，适用于 MacOS 或者 Linux 平台。

```sh
# 安装zsh，各平台命令不一样，但差不多
yum install zsh -y

# 检查已经安装好的shell
cat /etc/shells

# 交互式更换当前用户的shell，输入上面看到的 /bin/zsh
chsh

# 免交互直接更改root的shell
chsh -s /bin/zsh root

# 安装oh-my-zsh
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# 修改配置文件
vi ~/.zshrc

# 以下是我必修改的配置，主题用ys，在配置文件的开头部分
ZSH_THEME="ys"

# 启用的插件，在配置文件的中间位置，autojump, zsh-autosuggestions 非常好用
plugins=(git pip python autojump zsh-autosuggestions)

# 在文件末尾补充几部分内容

# 兼容bash的配置文件，忽略导入失败的错误
source ~/.bashrc &>/dev/null
source ~/.bash_profile  &>/dev/null

# 添加路径到PATH
export PATH="/usr/local/sbin:$PATH"

# 常用的别名
alias s=systemctl
alias k=kubectl
alias n=nginx
alias vi=vim
alias cls=clear
alias ll='ls -l'
alias la='ls -a'
alias grep="grep --color=auto"
```

## autojump

autojump 是一个很方便的让你跳转目录的命令行工具。需要额外安装，在 MacOSX 可以用 brew 安装。

```bash
brew install autojump
```

在其他 Linux 平台需要从源码安装。

```bash
git clone git://github.com/wting/autojump.git
cd autojump
./install.py
# or ./uninstall.py
```

装完之后注意看安装成功后提示，你需要把这段内容加到 `.bash_profile`里。

```
[[ -s /root/.autojump/etc/profile.d/autojump.sh ]] && source /root/.autojump/etc/profile.d/autojump.sh
```

然后重启命令行就可以开心的 autojump 了。

```
j toby  # cd 到带有toby的最近目录
j music # cd 到有可能是music的目录
j doc   # cd 到有可能是doc的目录
```

autojump 非常智能，你只要给少量关键字就可以 cd 到你想要到的目录。

## autosuggestions

有了 oh-my-zsh 大部分命令是可以通过 TAB 补全的，autosuggestion 可以锦上添花。但这玩意还是需要额外安装。

```
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

然后在 zsh 的配置文件里启用即可，参考上文。

```
plugins=(zsh-autosuggestions)
```

成功启用后的效果如下。

![image-20200404202408289](https://raw.githubusercontent.com/tobyqin/img/master/image-20200404202408289.png)

灰色部分是自动提示的，主要是根据输入历史和自动完成的可能性，按右方向键就可以直接使用提示的完整命令，爽歪歪。

## alias

命令行的别名可以极大提高效率。

```bash
ls --color=auto &>/dev/null && alias ls='ls --color=auto' && eval "$(dircolors)"

alias g='git'
alias k='kubectl'
alias n='nginx'
alias h='history'
alias s='systemctl'
alias vi='vim'
alias svi='sudo vim'
alias c='clear'
alias cls='clear'
alias l='ls -lah'
alias ll='ls -l'
alias la='ll -la'
alias grep="grep --color=auto"
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'

# bind file with default actions
alias -s html='vim'
alias -s rb='vim'
alias -s py='vim'
alias -s js='vim'
alias -s c='vim'
alias -s java='vim'
alias -s txt='vim'
alias -s gz='tar -xzvf'
alias -s tgz='tar -xzvf'
alias -s zip='unzip'
alias -s bz2='tar -xjvf'

alias cdback='cd -'
alias '..'='cd ..'
alias '...'='../..'
alias '....'='../../..'
alias '.....'='../../../..'
alias '......'='../../../../..'

alias df='df -h'
alias du='du -h'
alias cp='cp -v'
alias mv='mv -v'
alias mkdir='mkdir -pv'
alias which='which -a'
alias path='echo -e ${PATH//:/\\n}'
alias ping='ping -c 5'
alias ports='netstat -tulanp'
alias rm='rm -I --preserve-root'
alias chown='chown -v --preserve-root'
alias chmod='chmod -v --preserve-root'
alias chgrp='chgrp --preserve-root'

alias virc='vi ~/.vimrc'
alias barc='vi ~/.bashrc && source ~/.bashrc'
alias baprofile='vi ~/.bash_profile && source ~/.bash_profile'
alias bareload='source ~/.bash_profile &>/dev/null || source ~/.bashrc  &>/dev/null'
alias zshrc='vi ~/.zshrc && source ~/.zshrc'

alias untar='tar -zxvf'
alias www='python2 -m SimpleHTTPServer 8000'

alias ngx='nginx'
alias ngxreload='sudo ng -s reload'
alias ngxtest='sudo ng -t'
alias ngxconf='sudo vi /etc/nginx/conf/nginx.conf && ngxtest'

alias help='tldr'
```
