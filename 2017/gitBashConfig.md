# Git bash alias config

SourceTree是一个功能很强大的Git GUI工具，但是就是太慢，且在Windows下会出现莫名占用一个核心CPU。于是开始折腾Git bash，目前基本够用。其中alias部分的配置文件如下，供参考，放到 /username/.gitconfig里面即可。

<code>
[alias]
   l = log --pretty=format:"%C(yellow)%h\\ %ad%Cred%d\\ %Creset%s%Cblue\\ [%cn]" --decorate --date=short
    a = add
    ap = add -p
    c = commit --verbose
    ca = commit -a --verbose
    cm = commit -m
    cam = commit -a -m
    m = commit --amend --verbose
    d = diff
    ds = diff --stat
    dc = diff --cached
    s = status
    ss = status -s
    co = checkout
    cob = checkout -b
    # list branches sorted by last modified
    b = "!git for-each-ref --sort='-authordate' --format='%(authordate)%09%(objectname:short)%09%(refname)' refs/heads | sed -e 's-refs/heads/--'"
    lg1 = log --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset)%C(bold yellow)%d%C(reset)' --all
    lg2 = log --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset) - %C(bold cyan)%aD%C(reset) %C(bold green)(%ar)%C(reset)%C(bold yellow)%d%C(reset)%n''          %C(white)%s%C(reset) %C(dim white)- %an%C(reset)' --all
    lg = !"git lg1"
    # list aliases
    la = "!git config -l | grep alias | cut -c 7-"
</code>

其中我最喜欢的是git lg，附图2张，右键查看大图。
<img src="https://www.darkerror.com/wp-content/uploads/2017/10/Untitled.png" alt="git lg" />
<img src="https://www.darkerror.com/wp-content/uploads/2017/10/snipaste_20171013_172741.png" alt="git lg2" />
