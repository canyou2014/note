### 安装基础开发环境

#### brew
```
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

#### git

```
$ brew install git
```

#### golang

不要直接使用`bew install go`, 从官网直接下载安装包进行安装

在`$HOME`目录新建`go`目录，并在该目录中新建三个子目录`src`，`bin`，`pkg`
 
然后在新增配置文件`~/.bash_profile`，并填入如下配置：
```
export GOROOT=/usr/local/go  
export PATH=$PATH:$GOROOT/bin  
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
```
#### vim
Vundle安装插件： `PluginInstall`

其中vim-go需要执行`:GoInstallBinaries`将二进制安装，在这之前先安装`golang.org/x/tool`， `golang.org/x/lint`等go的package


https://github.com/caixw/VimIDE.git
