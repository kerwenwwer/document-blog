# 美化終端機 zsh+powerlevel9k


![](https://i.imgur.com/6uGLTn3.png)



## 前言
如果你是個類Unix系統使用者肯定經常使用終端機(terminal)，但是預設的shell何文字樣式卻經常讓你感到頭痛，那麼今天就來教大家使用zsh+powerlevel9k 美化你的終端機吧！！！

#### 最後的結果大概長這樣
![](https://i.imgur.com/0g3VQRE.png)


## zsh
首先我們要先來安裝zsh。
### 安裝
在系統中是沒有預設安裝zsh的，所以我們先用個發行板的安裝管理員安裝：

Mac
```bash
brew install zsh zsh-completions
```
Arch Linux
```bash
sudo paman -S zsh 
```
Ubuntu
```bash
sudo apt-get install zsh 
```
### 變更預設 shell
因為在linux中預設的shell是bash所以我們須要手動改變它：
Mac 
```bash
sudo sh -c "echo $(which zsh) >> /etc/shells"

 chsh -s $(which zsh)
```
linux
```bash
sudo chsh
```
然後輸入```/bin/zsh```按enter即可

## 安裝 oh-my-zsh

oh-my-zsh是用來配置zsh的強大工具，使用下面的指令可以直接啟動安裝包中的```install.sh```檔：
```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```
結束後你會發現你的加目錄下多出了```~/.oh-my-zsh```這個資料夾，裡頭有

![](https://i.imgur.com/nrd5ie1.png)

我們目前要關心的只有themes這個資料夾，裡頭擺放著所有所有主題的檔案

那麼整個zsh設定檔是由```~/.zshrc```來控制，我們來很快的看一下我們有什麼可以改得東西：

首先用你的編輯器打開```~/.zshrc```然後編輯```ZSH_THEME=”robbyrussell”``` 改成你想要的：

```
#編輯~/.zshrc
ZSH_THEME=”robbyrussell”
```
然後除存

記住對於shell的設定檔做了任何改動後，除了除存以外還需要手動輸入
```
exec $SHELL
```
重新啟動shell變更才會作用

## 安裝powerlevel9k主題
![](https://i.imgur.com/lRX72GR.png)

我個人非常喜歡powerlevel9k這個主題，它類似於大眾熟知的[powerline](https://powerline.readthedocs.io/en/master/)
但是又更加的模塊化，因此會更好配置
它也導入了一些比較有趣的功能，包括顯示電池電量、cpu使用率、wifi訊號等等，許多配置可以玩
以下是他的github內容：
https://github.com/bhilburn/powerlevel9k
![](https://camo.githubusercontent.com/b5d7eb49a30bfe6bdb5706fa3c9be95fe8e5956e/687474703a2f2f67696679752e636f6d2f696d616765732f70396b6e65772e676966)
> 圖片來源 https://github.com/bhilburn/powerlevel9k

那麼我們就直接從github上面把源碼拉下來吧

```bash
git clone https://github.com/bhilburn/powerlevel9k.git ~/.oh-my-zsh/custom/themes/powerlevel9k
```
這個指令在把源碼拉下來後直接送入oh-my-zsh的theme資料夾
接下來一個很重要的就是我們需要這類擴充主題所需要的字體
### 字體
這裡建議使用nerd-fonts

Mac
```bash
brew tap caskroom/fonts

brew cask install font-sourcecodepro-nerd-font
```
Arch linux 
```bash
sudo pacman -S nerd-fonts-complete nerd-fonts-git
```
其他發行板未提供在安裝管理員內
清參考這篇文章
https://shuhm-gh.github.io/2017/03/23/%E5%AE%89%E8%A3%85nerd-fonts%E5%AD%97%E4%BD%93/
>那麼對於Mac用戶裝完後，要修改 iTerm2 字型設定否則不會生效。
將其改成 SauceCodePro Nerd Font 

## 配置powerlevel9k主題

 首先我們要更改`~/.zshrc`中的`ZSH_THEME`，將其改成：
```
ZSH_THEME="powerlevel9k/powerlevel9k"
```
然後重起zsh後就可以看到它預設的主題樣式
![](https://i.imgur.com/tL8wE4N.png)

接下來介紹在幾個powerlevel9k可以玩的設定

首先在zshrc中powerlevel9k的配置命令分成幾種，分別為：


---

控制視窗左方的
#### POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(參數)

控制視窗右方的

#### POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=(參數)

額外icon的引入
#### POWERLEVEL9K_MODE='nerdfont-complete'


---


那麼參數便是放在括號中，那麼如果想再同一邊加入超過一個參數，兩個參數之間要使用"空格"隔開。


接下來介紹一些好玩的參數


---

### 通常放在左側的參數
#### dir 
預設參數可以顯示所在的資料夾：

![](https://i.imgur.com/pCwFdDl.png)

#### vcs
可以顯示git版本控制的參數：

![](https://i.imgur.com/ODPuapV.png)
#### dir_writable
當你進入了一個沒有寫入權限的資料夾時還可以給你提醒：

![](https://i.imgur.com/nSDx636.png)
#### vi_mode
![](https://i.imgur.com/ODAQ2Jl.png)


---

### 通常放在右側的參數
#### time
右側預設函數：

![](https://i.imgur.com/MIyYxgo.png)

#### free memory
還可以顯示目前電腦的 free memory：

![](https://i.imgur.com/4TP6nRL.png)


#### status
linux return code 為 0 時會有個綠色小勾勾

![](https://i.imgur.com/T9TeCmX.png)

若是指令錯誤

![](https://i.imgur.com/nBooGZw.png)

#### CPU load average
顯示系統cpu的覆載狀況：

![](https://i.imgur.com/D1kGAsj.png)

#### 電池
顯示電池狀態：

![](https://i.imgur.com/lJxT6Ao.png)


---

最後有個通用的設定給大家參考下
```bash
POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(context dir dir_writable vcs vi_mode)
POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=(status background_jobs history ram load time)
POWERLEVEL9K_MODE='nerdfont-complete'

```
如果你還想知道更多可以參考powerlevel9k的官方Wiki
https://github.com/bhilburn/powerlevel9k/wiki


參考的zshrc配置

```bash=
# If you come from bash you might have to change your $PATH.
#export PATH=$HOME/bin:/usr/local/bin:$PATH
export PATH=/bin:/usr/bin:/usr/local/bin:$HOME/.local/bin:${PATH}
# Path to your oh-my-zsh installation.
export ZSH="/home/kerwin/.oh-my-zsh"


#for lengacy GTK adn QT app
export GDK_USE_XFT=1
export QT_XFT=true
# Path fot Python Django
export WORKON_HOME=$HOME/.virtualenvs
export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
export VIRTUALENVWRAPPER_VIRTUALENV_ARGS=' -p /usr/bin/python3 '
export PROJECT_HOME=$HOME/Devel
source /usr/bin/virtualenvwrapper.sh
# Set name of the theme to load --- if set to "random", it will
# load a random theme each time oh-my-zsh is loaded, in which case,
# to know which specific one was loaded, run: echo $RANDOM_THEME
# See https://github.com/robbyrussell/oh-my-zsh/wiki/Themes
#ZSH_THEME="robbyrussell"
ZSH_THEME="powerlevel9k/powerlevel9k"
# Customise the Powerlevel9k prompts
POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(
  os_icon
  dir
  vcs
  user
  status
)
POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=(
	#public_ip
	virtualenv
	time
)

#CUSTOM ARCH 
POWERLEVEL9K_CUSTOM_ARCH="echo -n '\uf303'ArchLinux"
POWERLEVEL9K_CUSTOM_ARCH_FOREGROUND="black"
POWERLEVEL9K_CUSTOM_ARCH_BACKGROUND="yellow"

#CUSTOM PYTHON
#POWERLEVEL9K_CUSTOM_PYTHON="echo -n '\uf81f'&& python --version"
#POWERLEVEL9K_CUSTOM_PYTHON_FOREGROUND="black"
#POWERLEVEL9K_CUSTOM_PYTHON_BACKGROUND="red"

# 
POWERLEVEL9K_USER_ICON="\uF415" 
POWERLEVEL9K_ROOT_ICON="#"
POWERLEVEL9K_SUDO_ICON=$'\uF09C'

# Load Nerd Fonts with Powerlevel9k theme for Zsh
POWERLEVEL9K_MODE='nerdfont-complete'
POWERLEVEL9K_PROMPT_ON_NEWLINE=true
POWERLEVEL9K_SHORTEN_DIR_LENGTH=2
# Reversed time format
#POWERLEVEL9K_TIME_FORMAT='%D{%S:%M:%H}'


#source ~/powerlevel9k/powerlevel9k.zsh-theme

# Set list of themes to pick from when loading at random
# Setting this variable when ZSH_THEME=random will cause zsh to load
# a theme from this variable instead of looking in ~/.oh-my-zsh/themes/
# If set to an empty array, this variable will have no effect.
# ZSH_THEME_RANDOM_CANDIDATES=( "robbyrussell" "agnoster" )

# Uncomment the following line to use case-sensitive completion.
# CASE_SENSITIVE="true"

# Uncomment the following line to use hyphen-insensitive completion.
# Case-sensitive completion must be off. _ and - will be interchangeable.
# HYPHEN_INSENSITIVE="true"

# Uncomment the following line to disable bi-weekly auto-update checks.
# DISABLE_AUTO_UPDATE="true"

# Uncomment the following line to change how often to auto-update (in days).
# export UPDATE_ZSH_DAYS=13

# Uncomment the following line to disable colors in ls.
# DISABLE_LS_COLORS="true"

# Uncomment the following line to disable auto-setting terminal title.
# DISABLE_AUTO_TITLE="true"

# Uncomment the following line to enable command auto-correction.
# ENABLE_CORRECTION="true"

# Uncomment the following line to display red dots whilst waiting for completion.
# COMPLETION_WAITING_DOTS="true"

# Uncomment the following line if you want to disable marking untracked files
# under VCS as dirty. This makes repository status check for large repositories
# much, much faster.
# DISABLE_UNTRACKED_FILES_DIRTY="true"

# Uncomment the following line if you want to change the command execution time
# stamp shown in the history command output.
# You can set one of the optional three formats:
# "mm/dd/yyyy"|"dd.mm.yyyy"|"yyyy-mm-dd"
# or set a custom format using the strftime function format specifications,
# see 'man strftime' for details.
# HIST_STAMPS="mm/dd/yyyy"

# Would you like to use another custom folder than $ZSH/custom?
# ZSH_CUSTOM=/path/to/new-custom-folder

# Which plugins would you like to load?
# Standard plugins can be found in ~/.oh-my-zsh/plugins/*
# Custom plugins may be added to ~/.oh-my-zsh/custom/plugins/
# Example format: plugins=(rails git textmate ruby lighthouse)
# Add wisely, as too many plugins slow down shell startup.
plugins=(
  git
  sudo
  pipenv
  archlinux
)

source $ZSH/oh-my-zsh.sh

# User configuration

# export MANPATH="/usr/local/man:$MANPATH"

# You may need to manually set your language environment
# export LANG=en_US.UTF-8

# Preferred editor for local and remote sessions
# if [[ -n $SSH_CONNECTION ]]; then
#   export EDITOR='vim'
# else
#   export EDITOR='mvim'
# fi

# Compilation flags
# export ARCHFLAGS="-arch x86_64"

# ssh
# export SSH_KEY_PATH="~/.ssh/rsa_id"

# Set personal aliases, overriding those provided by oh-my-zsh libs,
# plugins, and themes. Aliases can be placed here, though oh-my-zsh
# users are encouraged to define aliases within the ZSH_CUSTOM folder.
# For a full list of active aliases, run `alias`.
#
# Example aliases
# alias zshconfig="mate ~/.zshrc"
# alias ohmyzsh="mate ~/.oh-my-zsh"

```

## 總結
美化自己的終端機是很有趣的一見事情，只要你願意嘗試，你一定可以找到一套自己最喜歡的終端機配置。