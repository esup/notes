# Git 安装手册

本文档提供了在不同操作系统上安装和配置 Git 的详细指南。

## 目录
- [Windows 安装](#windows-安装)
- [macOS 安装](#macos-安装)
- [Linux 安装](#linux-安装)
- [初始配置](#初始配置)
- [验证安装](#验证安装)
- [常见问题](#常见问题)

---

## Windows 安装

### 方法一：使用官方安装包（推荐）

#### 1. 下载安装包
1. 访问 Git 官网：https://git-scm.com/download/win
2. 选择适合您系统的安装包：
   - 64-bit Git for Windows Setup（适用于大多数现代 Windows 系统）
   - 32-bit Git for Windows Setup（适用于旧系统）
3. 下载完成后，运行安装程序

#### 2. 安装配置选项

运行安装程序后，建议按以下方式配置：

**选择组件（Select Components）：**
- ✅ Windows Explorer integration（Windows 资源管理器集成）
- ✅ Git Bash Here
- ✅ Git GUI Here
- ✅ Associate .git* configuration files with the default text editor
- ✅ Associate .sh files to be run with Bash

**选择默认编辑器（Choosing the default editor）：**
- 可选择 Vim、Nano、Notepad++ 或 Visual Studio Code 等
- 推荐：Visual Studio Code（如果已安装）

**调整 PATH 环境变量（Adjusting your PATH environment）：**
- ✅ 推荐选择："Git from the command line and also from 3rd-party software"
- 这样可以在 cmd、PowerShell 和其他工具中使用 Git

**选择 HTTPS 传输后端（Choosing HTTPS transport backend）：**
- ✅ 推荐选择："Use the OpenSSL library"

**配置行尾转换（Configuring the line ending conversions）：**
- ✅ 推荐选择："Checkout Windows-style, commit Unix-style line endings"
- 这样可以避免跨平台协作时的换行符问题

**配置终端模拟器（Configuring the terminal emulator）：**
- ✅ 推荐选择："Use MinTTY (the default terminal of MSYS2)"
- MinTTY 提供更好的终端体验

**配置默认 git pull 行为（Choose the default behavior of git pull）：**
- ✅ 推荐选择："Default (fast-forward or merge)"

**选择凭据管理器（Choose a credential helper）：**
- ✅ 推荐选择："Git Credential Manager"
- 可以安全地存储你的 Git 凭据

**额外选项（Configuring extra options）：**
- ✅ Enable file system caching（启用文件系统缓存，提高性能）
- ✅ Enable symbolic links（如果需要符号链接支持）

#### 3. 完成安装
- 点击 "Install" 开始安装
- 安装完成后，可以选择启动 Git Bash 来验证安装

### 方法二：使用包管理器

#### 使用 Chocolatey
```bash
choco install git
```

#### 使用 Scoop
```bash
scoop install git
```

#### 使用 winget（Windows 包管理器）
```bash
winget install --id Git.Git -e --source winget
```

---

## macOS 安装

### 方法一：使用 Homebrew（推荐）

Homebrew 是 macOS 最流行的包管理器。

#### 1. 安装 Homebrew（如果未安装）
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

#### 2. 安装 Git
```bash
brew install git
```

#### 3. 更新 Git（已安装的情况下）
```bash
brew upgrade git
```

### 方法二：使用 Xcode Command Line Tools

macOS 自带了 Git，但版本可能较老。可以通过 Xcode Command Line Tools 安装：

```bash
xcode-select --install
```

### 方法三：使用官方安装包

1. 访问：https://git-scm.com/download/mac
2. 下载 macOS 安装包
3. 运行 .dmg 文件并按照提示安装

### 方法四：使用 MacPorts

```bash
sudo port install git
```

---

## Linux 安装

### Debian/Ubuntu

```bash
# 更新软件包列表
sudo apt update

# 安装 Git
sudo apt install git

# 验证安装
git --version
```

### Fedora

```bash
# Fedora 22 及更高版本
sudo dnf install git

# 旧版本 Fedora
sudo yum install git
```

### CentOS/RHEL

```bash
# CentOS 8 及更高版本
sudo dnf install git

# CentOS 7 及更低版本
sudo yum install git
```

### Arch Linux

```bash
sudo pacman -S git
```

### openSUSE

```bash
sudo zypper install git
```

### Alpine Linux

```bash
apk add git
```

### 从源代码编译（适用于所有 Linux 发行版）

如果需要最新版本或自定义编译选项：

```bash
# 1. 安装依赖
# Debian/Ubuntu:
sudo apt install dh-autoreconf libcurl4-gnutls-dev libexpat1-dev \
  gettext libz-dev libssl-dev

# Fedora/CentOS/RHEL:
sudo dnf install dh-autoreconf curl-devel expat-devel gettext-devel \
  openssl-devel perl-devel zlib-devel

# 2. 下载源代码
cd /tmp
wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.43.0.tar.gz
tar -zxf git-2.43.0.tar.gz
cd git-2.43.0

# 3. 编译安装
make configure
./configure --prefix=/usr
make all
sudo make install

# 4. 验证安装
git --version
```

---

## 初始配置

安装完成后，需要进行基本配置。

### 配置用户信息

Git 需要知道你的身份信息，用于标识提交者：

```bash
# 设置用户名
git config --global user.name "你的名字"

# 设置邮箱
git config --global user.email "your.email@example.com"
```

**说明：**
- `--global` 参数表示这些设置对当前用户的所有仓库生效
- 如果想为特定仓库设置不同的用户信息，在该仓库目录下执行命令时去掉 `--global` 参数

### 配置默认分支名称

从 Git 2.28 开始，可以配置新仓库的默认分支名称：

```bash
git config --global init.defaultBranch main
```

### 配置默认编辑器

```bash
# 使用 Vim
git config --global core.editor "vim"

# 使用 Nano
git config --global core.editor "nano"

# 使用 VS Code
git config --global core.editor "code --wait"

# 使用 Sublime Text（Windows）
git config --global core.editor "'C:/Program Files/Sublime Text/sublime_text.exe' -w"

# 使用 Notepad++（Windows）
git config --global core.editor "'C:/Program Files/Notepad++/notepad++.exe' -multiInst -notabbar -nosession -noPlugin"
```

### 配置换行符处理

跨平台协作时，不同操作系统的换行符可能不同：

```bash
# Windows 系统
git config --global core.autocrlf true

# macOS/Linux 系统
git config --global core.autocrlf input

# 禁用自动转换
git config --global core.autocrlf false
```

### 配置颜色显示

```bash
git config --global color.ui auto
```

### 配置别名（可选）

为常用命令创建快捷方式：

```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.lg "log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

### 查看配置

```bash
# 查看所有配置
git config --list

# 查看特定配置
git config user.name
git config user.email

# 查看配置来源
git config --list --show-origin
```

### 配置 SSH 密钥（推荐）

使用 SSH 可以避免每次推送时输入密码。

#### 1. 生成 SSH 密钥

```bash
ssh-keygen -t ed25519 -C "your.email@example.com"
# 如果系统不支持 Ed25519，使用 RSA：
# ssh-keygen -t rsa -b 4096 -C "your.email@example.com"
```

按回车接受默认文件位置，然后设置密码（可选）。

#### 2. 启动 SSH 代理并添加密钥

```bash
# Linux/macOS
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Windows (Git Bash)
eval $(ssh-agent -s)
ssh-add ~/.ssh/id_ed25519
```

#### 3. 复制公钥

```bash
# Linux/macOS
cat ~/.ssh/id_ed25519.pub

# Windows (Git Bash)
cat ~/.ssh/id_ed25519.pub

# Windows (PowerShell)
Get-Content ~/.ssh/id_ed25519.pub | Set-Clipboard
```

#### 4. 添加到 GitHub/GitLab

- **GitHub**: Settings → SSH and GPG keys → New SSH key
- **GitLab**: Preferences → SSH Keys → Add new key

粘贴公钥内容并保存。

---

## 验证安装

### 检查版本

```bash
git --version
```

应该看到类似输出：
```
git version 2.43.0
```

### 检查配置

```bash
git config --list
```

应该看到你配置的用户名、邮箱等信息。

### 测试基本功能

```bash
# 创建测试目录
mkdir git-test
cd git-test

# 初始化仓库
git init

# 创建文件
echo "# Test" > README.md

# 添加并提交
git add README.md
git commit -m "Initial commit"

# 查看历史
git log
```

如果以上命令都能正常执行，说明 Git 安装成功！

### 测试 SSH 连接（如果配置了 SSH）

```bash
# 测试 GitHub 连接
ssh -T git@github.com

# 测试 GitLab 连接
ssh -T git@gitlab.com
```

成功的话会看到欢迎信息。

---

## 常见问题

### 问题 1：命令不被识别（Windows）

**症状：**
```
'git' is not recognized as an internal or external command
```

**解决方案：**
1. 检查 Git 是否已安装：查看 `C:\Program Files\Git`
2. 将 Git 添加到 PATH 环境变量：
   - 右键"此电脑" → "属性" → "高级系统设置" → "环境变量"
   - 在"系统变量"中找到 `Path`，编辑
   - 添加：`C:\Program Files\Git\cmd`
   - 重启命令提示符或 PowerShell

### 问题 2：中文乱码

**症状：**
- Git Bash 中中文显示为乱码
- git status 显示中文文件名为数字

**解决方案：**
```bash
# 解决 git status 中文显示问题
git config --global core.quotepath false

# 解决 Git Bash 中文乱码（Windows）
# 在 Git Bash 的选项中设置：
# Options → Text → Locale: zh_CN, Character set: UTF-8

# 或在命令行设置：
git config --global i18n.commitencoding utf-8
git config --global i18n.logoutputencoding utf-8
```

### 问题 3：SSL 证书错误

**症状：**
```
SSL certificate problem: unable to get local issuer certificate
```

**解决方案：**
```bash
# 临时解决（不推荐用于生产环境）
git config --global http.sslVerify false

# 推荐方案：更新 CA 证书
# Windows: 重新安装 Git 并选择使用 OpenSSL
# Linux: sudo apt install ca-certificates
# macOS: brew update && brew upgrade
```

### 问题 4：代理设置

**症状：**
无法连接到 GitHub 或其他远程仓库。

**解决方案：**
```bash
# 设置 HTTP 代理
git config --global http.proxy http://proxy.server.com:port
git config --global https.proxy https://proxy.server.com:port

# 设置 SOCKS5 代理
git config --global http.proxy socks5://127.0.0.1:1080
git config --global https.proxy socks5://127.0.0.1:1080

# 取消代理
git config --global --unset http.proxy
git config --global --unset https.proxy

# 仅为特定域名设置代理
git config --global http.https://github.com.proxy socks5://127.0.0.1:1080
```

### 问题 5：权限问题（Linux/macOS）

**症状：**
```
Permission denied (publickey)
```

**解决方案：**
```bash
# 确保 SSH 密钥权限正确
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub

# 检查 SSH 密钥是否已添加
ssh-add -l

# 如果没有，添加密钥
ssh-add ~/.ssh/id_ed25519
```

### 问题 6：Git 版本过低

**症状：**
某些命令不可用，如 `git switch`、`git restore` 等。

**解决方案：**
```bash
# Ubuntu/Debian: 使用官方 PPA
sudo add-apt-repository ppa:git-core/ppa
sudo apt update
sudo apt upgrade git

# macOS: 使用 Homebrew
brew upgrade git

# Windows: 重新下载最新版安装包
```

### 问题 7：凭据管理

**症状：**
每次 push/pull 都需要输入用户名和密码。

**解决方案：**
```bash
# 使用凭据存储（Windows）
git config --global credential.helper wincred

# 使用凭据存储（macOS）
git config --global credential.helper osxkeychain

# 使用凭据存储（Linux）
git config --global credential.helper store
# 或使用更安全的 libsecret
git config --global credential.helper /usr/share/doc/git/contrib/credential/libsecret/git-credential-libsecret

# 推荐：使用 SSH 密钥代替 HTTPS（参见"配置 SSH 密钥"部分）
```

### 问题 8：文件名大小写问题

**症状：**
修改文件名大小写后，Git 不识别更改。

**解决方案：**
```bash
# 配置 Git 识别大小写
git config core.ignorecase false

# 或使用 git mv 重命名
git mv oldname.txt Oldname.txt
```

---

## 下一步

安装和配置完成后，你可以：

1. 阅读 [Git 使用手册](README.md) 学习 Git 的基本操作
2. 访问 [Git 官方文档](https://git-scm.com/doc) 深入学习
3. 尝试在 [GitHub](https://github.com) 或 [GitLab](https://gitlab.com) 上创建仓库进行实践

---

## 相关资源

- [Git 官方网站](https://git-scm.com/)
- [Git 官方文档](https://git-scm.com/doc)
- [GitHub 学习实验室](https://lab.github.com/)
- [Pro Git 电子书（中文版）](https://git-scm.com/book/zh/v2)
- [Git 可视化学习](https://learngitbranching.js.org/?locale=zh_CN)

---

**最后更新：** 2024-01-11
