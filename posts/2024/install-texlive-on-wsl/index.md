# WSL2安装TeXLive踩坑记录


众所周知，Windows 上 Latex 编译速度被 Linux 吊着打不知道几个来回。故打算以后都用 Linux 写 Latex。

但是毕竟 Linux 还算不上是主力系统，要写 Latex 重启切换 Linux 十分麻烦，那么我们可以在 wsl 上装 TeXLive。

又众所周知的是，wsl 默认装在 C 盘，而 TeXLive 近 10GB，所以有了这篇记录。

## 安装 WSL2 虚拟机到非系统盘

安装并启用 WSL2 教程已经满天飞了，故不再赘述。直接从安装 Linux 发行版开始。

1. 到 [下载发行版](https://learn.microsoft.com/zh-cn/windows/wsl/install-manual#downloading-distributions) 选择你喜欢的系统下载，这里我选择了 Ubuntu 24.04。

2. 把下载的文件 `Ubuntu2404-240425.AppxBundle` 后缀改为 `zip` 并解压。

3. 在解压出的文件夹中选择符合你机器的系统包，AMD64 下所需文件为 `Ubuntu_2404.0.5.0_x64.appx`，改后缀为 `zip` 并将其<font color=red>**解压到你要安装的路径下**</font>。比如我把它放在了 `D:\WSL\Ubuntu2404-240425` 下。
4. 双击路径中的 `ubuntu2404.exe` 等待并设置用户名和密码就大功告成了。
5. 更换清华源等更多个性化配置自行搜索教程。

## 下载 TeXLive 并安装至 WSL

到国内镜像站下载最新 TeXLive 镜像，比如 [清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/CTAN/systems/texlive/Images/)。

WSL 是可以直接访问 Windows 的文件的，比如 C 盘的内容在 `/mnt/c/` 下。假设我们下载的 TeXLive 镜像路径是 `C:\texlive2024.iso`。

```bash
# 新建要挂载的目录
sudo mkdir /mnt/texlive 
# 把TeXLive镜像挂载过来
sudo mount /mnt/c/texlive2024.iso /mnt/texlive
# 执行安装脚本
sudo /mnt/texlive/install-tl
```

之后的页面你可以修改一些安装可选项，如果你懒得折腾就直接输入 `I` 默认安装即可。（个性化选项若有所需请自行了解）

安装过程可能需要几分钟，等待即可。之后弹出镜像并删除文件夹：

```bash
sudo umount /mnt/texlive
sudo rm -r /mnt/texlive
```

之后更新环境变量，修改 `~/.bashrc` 文件在最后新增：

```bash
# Add TeX Live to the PATH, MANPATH, INFOPATH
export PATH=/usr/local/texlive/2024/bin/x86_64-linux:$PATH
export MANPATH=/usr/share/man:/usr/local/texlive/2024/texmf-dist/doc/man:$MANPATH
export INFOPATH=/usr/local/texlive/2024/texmf-dist/doc/info:$INFOPATH
```

重启终端或执行 `source ~/.bashrc` 即可。

之后可以执行 `tex -v` 查看安装是否成功。

我看的教程还有一个操作，不知道什么作用，反正执行就对了（

```bash
sudo cp /usr/local/texlive/2024/texmf-var/fonts/conf/texlive-fontconfig.conf /etc/fonts/conf.d/09-texlive.conf
sudo fc-cache -fsv
```

## VSCode 的 LaTex 配置

### 基础配置

安装以下几个插件：

- LaTeX Workshop（必选）
- English word hint
- Path Autocomplete

新建一个文件夹用于存放 `tex` 文件，假设名为 `LaTexSpace`。

新建 `LaTexSpace/.vscode/settings.json` 。

配置编译工具链，添加如下内容：

```json
{
    "latex-workshop.latex.tools": [
        {
            "name": "pdflatex",
            "command": "pdflatex",
            "args": [
                "-synctex=1",
                "-interaction=nonstopmode",
                "-file-line-error",
                "%DOCFILE%"
            ]
        },
        {
            "name": "xelatex",
            "command": "xelatex",
            "args": [
                "-synctex=1",
                "-interaction=nonstopmode",
                "-file-line-error",
                "%DOCFILE%"
            ]
        },
        {
            "name": "bibtex",
            "command": "bibtex",
            "args": [
                "%DOCFILE%"
            ]
        },
        // 下面这个是使用外部 PDF 查看器用的，仅配置编译工具链无需添加
        {
            "name": "correct_path",
            "command": "/home/spirit/LaTexSpace/.vscode/correct_path"
        },
        ],
        // 下面 tools 中的 correct_path 也是要在 VSCode+wsl 环境下使用 Windows 的 PDF 查看器才需要添加的。
        "latex-workshop.latex.recipes": [
        {
            "name": "xelatex",
            "tools": [
                "xelatex",
                "correct_path"
            ],
        },
        {
            "name": "pdflatex",
            "tools": [
                "pdflatex",
                "correct_path"
            ]
        },
        {
            "name": "xe->bib->xe->xe",
            "tools": [
                "xelatex",
                "bibtex",
                "xelatex",
                "xelatex",
                "correct_path"
            ]
        },
        {
            "name": "pdf->bib->pdf->pdf",
            "tools": [
                "pdflatex",
                "bibtex",
                "pdflatex",
                "pdflatex",
                "correct_path"
            ]
        },
    ]
}
```

### 配置 SumatraPDF 作外部查看器

首先贴上 [SumatraPDF官网](https://www.sumatrapdfreader.org/)。

在设置文件中添加以下内容：

```json
{
    "latex-workshop.view.pdf.viewer": "external",
    "latex-workshop.view.pdf.external.viewer.command": "/mnt/d/Program/SumatraPDF/SumatraPDF.exe", // 改为自己的路径
    "latex-workshop.view.pdf.external.viewer.args": [
        "%PDF%"
    ],
    "latex-workshop.view.pdf.external.synctex.command": "/mnt/d/Program/SumatraPDF/SumatraPDF.exe", // 改为自己的路径
    "latex-workshop.view.pdf.external.synctex.args": [
        "-forward-search",
        "\\\\wsl$\\Ubuntu-24.04\\%TEX%", // 中间的 Ubuntu-24.04 改为自己的发行版名称
        "%LINE%",
        "-reuse-instance",
        "\\\\wsl$\\Ubuntu-24.04\\%PDF%" // 改为自己的发行版名称
    ],
}
```

不知道自己发行版准确名称的话可以使用 `wsl -l -v` 命令查看。

此外还需要把 WSL 的路径转为 Windows 的，如上一小节添加 correct_path 工具，并新建 `LaTexSpace/.vscode/correct_path` 文件。其内容如下，注意其中的路径和发行版名称都要修改为自己的。

```bash
gunzip *.synctex.gz
sed -i "s/\/home\/spirit\/LaTexSpace\//\\\\\\\\wsl$\\\\Ubuntu-24.04\\\\home\\\\spirit\\\\LaTexSpace\\\\/g" *.synctex
gzip *.synctex
```

在 Windows 上新建一个 `vbs` 文件，路径自定义即可。

注意修改为自己的 wsl 发行版名称和 VSCode 路径。

```vbscript
Set args = WScript.Arguments
arg = ""
For Each s In args
arg = arg & " " & s
Next
WScript.CreateObject("WScript.Shell").Run "wsl -d Ubuntu-24.04 '/mnt/d/Program Files/Microsoft VS Code/bin/code' " & arg, 0
```

在 SumatraPDF 的设置-选项中填写反向查找命令，注意微调。

```shell
wscript "D:\Code\LaTex\Inverse Search.vbs" -r -g $(echo $(wslpath '%f:%l') "|" sed 's/\/mnt\/d\/wsl$\/Ubuntu-24.04//g')
```

### latexindent 格式化 tex 文档

<font color=red>**真没必要搞这个，费劲还没用**</font>

在 `settings.json` 中添加如下内容：

```json
{
    "latex-workshop.formatting.latex": "latexindent",
    // 修改为自己的 TeXLive 路径
    "latex-workshop.formatting.latexindent.path": "/usr/local/texlive/2024/bin/x86_64-linux/latexindent"
}
```

latexindent 是一个 perl 脚本。要使用它需要安装依赖：

```bash
sudo cpan Log::Log4perl
sudo cpan Log::Dispatch
sudo cpan YAML::Tiny
sudo cpan File::HomeDir
sudo cpan Unicode::GCString
```

### 原生Linux中配置Okular作为外部查看器

**安装Okular**

```bash
sudo apt install okular
```

**Okular设置**

设置 -> 配置Okular -> 常规 -> 程序功能 -> 勾选“在标签页中打开新的文档”

设置 -> 配置Okular -> 编辑器 -> 编辑器选择自定义文本编辑器，命令为 `code --goto %f:%l` 

**settings.json 配置**

```json
{
    "latex-workshop.view.pdf.viewer": "external",
    "latex-workshop.view.pdf.external.viewer.command": "okular",
    "latex-workshop.view.pdf.external.viewer.args": [
        "--unique",
        "%PDF%"
    ],
    "latex-workshop.view.pdf.external.synctex.command": "okular",
    "latex-workshop.view.pdf.external.synctex.args": [
        "--unique",
        "%PDF%#src:%LINE%%TEX%"
    ],
}
```

<font color=red>**Okular的反向查找需要按住shift再单击鼠标左键**</font>

## 踩坑

### 'xxx.sty' not found

缺啥装啥就行。

```bash
sudo tlmgr install xxx
```

### sudo 执行 tlmgr，command not found

因为 sudo 并不使用用户的环境变量，也不调用shell执行命令。故你折腾 `~/.bashrc` `/etc/profile` 什么的都是没用的。

可以在命令中指定环境变量：

```bash
sudo env PATH="$PATH" tlmgr install package
```

但是这样做好麻烦，在 `~/.bashrc` 中用 `alias` 简化一下吧：

```bash
alias psudo='sudo env PATH="$PATH"'
```

这样用 `psudo` 代替 `sudo` 就是使用当前终端的环境变量了。

还有一种强制 sudo 在 shell 中执行命令的办法，对应内容如下：

```bash
sudo sh -l -c tlmgr install package
alias shsudo='sudo sh -l -c'
```

### 编译一个 Beamer 模板莫名失败

缺包导致的，TeXLive 镜像默认不带这些东西吗？

```bash
sudo apt install latex-cjk-all texlive-lang-chinese texlive-lang-english
```

如果最开始用 `sudo apt install texlive-full` 安装环境似乎没有这个问题，但我没试。感兴趣的可以在新的系统上试试，坏处就是仓库里的是什么版本你就只能用什么版本。

### 复制 Windows 字体到 WSL

```bash
sudo mkdir /usr/share/fonts/windows_fonts
cd /usr/share/fonts/windows_fonts
sudo cp /mnt/c/Windows/Fonts/*.ttf ./
sudo cp /mnt/c/Windows/Fonts/*.TTF ./
sudo cp /mnt/c/Windows/Fonts/*.otf ./
sudo cp /mnt/c/Windows/Fonts/simsun.ttc ./
sudo mkfontscale
sudo mkfontdir
fc-cache -fv
```

<font color=red>**好消息**</font>：如果你能使用 deb 包，就不需要上面这个用处不大的东西了。

可以直接安装[Linux字体包（百度网盘，提取码yfsb）](https://pan.baidu.com/s/1J7Q4Fyhnjgg58c_rfjHYdA?pwd=yfsb)。非常全面的同时操作还简单。

[LinuxFont（夸克网盘，提取码ks4L）](https://pan.quark.cn/s/a2d2868c8bd5) 。

### perl 依赖装不上

需要先安装一些工具：

```bash
# 事到如今我也不知道哪个有用了，都tm装上得了
sudo apt install gcc g++ gcc-multilib g++-multilib make cmake build-essential libperl-dev build-essential libperl-dev 
```

这样还会有一个依赖 Log::Dispatch 装不上。顺着报错的依赖一路追到上游然后手动编译安装就好了：

```bash
wget https://www.cpan.org/modules/by-module/Devel/Module-Pluggable-6.2.tar.gz
gzip -dc Module-Pluggable-6.2.tar.gz | tar xf -
cd Module-Pluggable-6.2
perl Makefile.PL
make
make test 
sudo make install
```

然后顺着这一串报错的依赖再装回来：

```bash
 sudo cpan Test2::Plugin::NoWarnings
 sudo cpan Params::ValidationCompiler
 # 说不定直接跑这个就行了？试试？
 sudo cpan Log::Dispatch
```

然后这 byd latexindent 就终于能用了。

说实话我现在感觉手写的 tex 挺好看的，没事别瞎折腾（能跑就行

## ref

[WSL2 虚拟机 其他盘安装(非C盘)及调试(含填坑)](https://blog.mstg.top/archives/997)

[TeX Live 2024 安装教程（Windows/WSL/Linux）](https://www.cnblogs.com/eslzzyl/p/17358405.html)

[Permission problem when installing package from tlmgr](https://tex.stackexchange.com/questions/187073/permission-problem-when-installing-package-from-tlmgr)

[Setting TeX Live path for root](https://askubuntu.com/questions/66498/setting-tex-live-path-for-root)

[CPAN general build error (or fail during "install Module::ScanDeps")](https://groups.google.com/g/comp.lang.perl.modules/c/8ww36CFWRw0)

[hilinxinhui/VSCode_LaTeX_configuration](https://github.com/hilinxinhui/VSCode_LaTeX_configuration?tab=readme-ov-file)

[ubuntu下vscode LaTeX文档格式化](https://blog.csdn.net/lyh458/article/details/125930154)

[shinyypig/latex-vscode-config](https://github.com/shinyypig/latex-vscode-config)

[基于WSL-LaTeX的VS Code+SumatraPDF协作](https://blog.csdn.net/weixin_42694350/article/details/123320383)

[WSL下LaTeX的VS Code+SumatraPDF协作](https://zhuanlan.zhihu.com/p/337459645)

[VS Code的LaTeX配置](https://zhuanlan.zhihu.com/p/112108701)

[LaTeX配置安装大对比：TeXLive/MiKTeX、Windows/WSL、xelatex/lualatex/pdflatex编译器的速度性能详细对比](https://zhuanlan.zhihu.com/p/374491983)

[Ubuntu 20.04 系统环境下配置LaTeX环境（正反向搜索）](https://zhuanlan.zhihu.com/p/492965557)


---

> 作者: [Spirit](https://github.com/Ssspirit)  
> URL: https://rxzcloud.top/posts/2024/install-texlive-on-wsl/  

