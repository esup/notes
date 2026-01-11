# Git 使用手册

## 安装 Git

如果您还未安装 Git，请参阅 **[Git 安装手册](INSTALL.md)**，其中包含：
- Windows、macOS、Linux 系统的详细安装步骤
- 多种安装方法（官方安装包、包管理器等）
- 初始配置指南
- 常见问题解决方案

---

## 目录
- [基础操作](#基础操作)
- [分支管理](#分支管理)
- [远程操作](#远程操作)
- [进阶用法](#进阶用法)

---

## 基础操作

### 仓库初始化

**创建新仓库**
```bash
git init
```
说明：在当前目录创建一个新的 Git 仓库。

示例：
```bash
mkdir my-project
cd my-project
git init
# 输出：Initialized empty Git repository in /path/to/my-project/.git/
```

**克隆现有仓库**
```bash
git clone <url>
```
说明：从远程服务器克隆一个仓库到本地。

示例：
```bash
git clone https://github.com/username/repository.git
git clone https://github.com/username/repository.git my-folder  # 克隆到指定文件夹
```

### 文件操作

**查看状态**
```bash
git status
```
说明：查看工作区和暂存区的状态。

示例：
```bash
git status
# 输出显示哪些文件被修改、哪些文件在暂存区等
git status -s  # 简洁模式
```

**添加文件到暂存区**
```bash
git add <file>
```
说明：将文件的修改添加到暂存区。

示例：
```bash
git add index.html           # 添加单个文件
git add *.css                # 添加所有 CSS 文件
git add .                    # 添加所有修改的文件
git add -A                   # 添加所有修改（包括删除）
```

**提交修改**
```bash
git commit -m "提交信息"
```
说明：将暂存区的修改提交到本地仓库。

示例：
```bash
git commit -m "添加登录功能"
git commit -am "修复bug"     # -a 参数自动添加已跟踪文件的修改
git commit --amend           # 修改最后一次提交
```

**查看提交历史**
```bash
git log
```
说明：查看提交历史记录。

示例：
```bash
git log                      # 详细历史
git log --oneline            # 单行简洁模式
git log --graph              # 图形化显示
git log --graph --oneline --all  # 所有分支的图形化历史
git log -n 5                 # 显示最近5条记录
git log --author="张三"      # 查看某人的提交
```

**查看差异**
```bash
git diff
```
说明：查看文件的修改差异。

示例：
```bash
git diff                     # 工作区与暂存区的差异
git diff --staged            # 暂存区与最后一次提交的差异
git diff HEAD                # 工作区与最后一次提交的差异
git diff branch1 branch2     # 两个分支的差异
```

**撤销操作**
```bash
git restore <file>
git reset
```
说明：撤销修改或重置文件状态。

示例：
```bash
git restore index.html       # 撤销工作区的修改（Git 2.23+）
git restore --staged index.html  # 取消暂存
git checkout -- index.html   # 撤销工作区修改（旧版本）
git reset HEAD index.html    # 取消暂存（旧版本）
git reset --hard HEAD        # 重置到上次提交（危险操作，会丢失修改）
```

**删除文件**
```bash
git rm <file>
```
说明：从 Git 仓库中删除文件。

示例：
```bash
git rm file.txt              # 删除文件并暂存删除操作
git rm --cached file.txt     # 只从仓库删除，保留本地文件
```

**移动/重命名文件**
```bash
git mv <old> <new>
```
说明：移动或重命名文件。

示例：
```bash
git mv old_name.txt new_name.txt
git mv file.txt folder/      # 移动文件到文件夹
```

---

## 分支管理

### 分支基础

**查看分支**
```bash
git branch
```
说明：列出所有本地分支。

示例：
```bash
git branch                   # 列出本地分支
git branch -r                # 列出远程分支
git branch -a                # 列出所有分支（本地+远程）
git branch -v                # 显示分支及最后一次提交
```

**创建分支**
```bash
git branch <branch-name>
```
说明：创建新分支，但不切换。

示例：
```bash
git branch feature-login     # 创建名为 feature-login 的分支
git branch develop           # 创建 develop 分支
```

**切换分支**
```bash
git checkout <branch-name>
git switch <branch-name>
```
说明：切换到指定分支。

示例：
```bash
git checkout develop         # 切换到 develop 分支
git switch develop           # 切换到 develop 分支（Git 2.23+）
git checkout -b feature-new  # 创建并切换到新分支
git switch -c feature-new    # 创建并切换到新分支（Git 2.23+）
```

**合并分支**
```bash
git merge <branch-name>
```
说明：将指定分支合并到当前分支。

示例：
```bash
git checkout main
git merge feature-login      # 将 feature-login 合并到 main
git merge --no-ff develop    # 非快进模式合并，保留分支历史
git merge --squash feature   # 压缩合并，将多个提交合并为一个
```

**删除分支**
```bash
git branch -d <branch-name>
```
说明：删除已合并的分支。

示例：
```bash
git branch -d feature-login  # 删除已合并的分支
git branch -D feature-test   # 强制删除未合并的分支
```

**解决冲突**

当合并出现冲突时：
```bash
# 1. 查看冲突文件
git status

# 2. 手动编辑冲突文件，解决冲突标记：
#    <<<<<<< HEAD
#    当前分支的内容
#    =======
#    要合并分支的内容
#    >>>>>>> branch-name

# 3. 标记冲突已解决
git add <冲突文件>

# 4. 完成合并
git commit -m "解决合并冲突"
```

### 分支策略

**变基（Rebase）**
```bash
git rebase <branch-name>
```
说明：将当前分支的提交变基到指定分支。

示例：
```bash
git checkout feature
git rebase main              # 将 feature 分支变基到 main
git rebase --continue        # 解决冲突后继续变基
git rebase --abort           # 放弃变基操作
git rebase -i HEAD~3         # 交互式变基最近3次提交
```

**Cherry-pick**
```bash
git cherry-pick <commit-hash>
```
说明：将指定的提交应用到当前分支。

示例：
```bash
git cherry-pick abc1234      # 应用某个提交
git cherry-pick abc1234 def5678  # 应用多个提交
git cherry-pick --continue   # 解决冲突后继续
git cherry-pick --abort      # 放弃操作
```

---

## 远程操作

### 远程仓库管理

**查看远程仓库**
```bash
git remote
```
说明：查看远程仓库信息。

示例：
```bash
git remote                   # 列出远程仓库名称
git remote -v                # 显示远程仓库详细信息（URL）
git remote show origin       # 显示远程仓库详细信息
```

**添加远程仓库**
```bash
git remote add <name> <url>
```
说明：添加一个新的远程仓库。

示例：
```bash
git remote add origin https://github.com/username/repo.git
git remote add upstream https://github.com/original/repo.git
```

**删除远程仓库**
```bash
git remote rm <name>
```
说明：删除远程仓库引用。

示例：
```bash
git remote rm origin
git remote remove upstream
```

**重命名远程仓库**
```bash
git remote rename <old> <new>
```
说明：重命名远程仓库引用。

示例：
```bash
git remote rename origin github
```

### 同步操作

**拉取更新**
```bash
git fetch
git pull
```
说明：从远程仓库获取更新。

示例：
```bash
git fetch origin             # 获取 origin 的所有更新
git fetch origin main        # 获取 origin 的 main 分支
git pull origin main         # 拉取并合并 origin 的 main 分支
git pull --rebase origin main  # 拉取并变基
```

**推送更新**
```bash
git push
```
说明：将本地提交推送到远程仓库。

示例：
```bash
git push origin main         # 推送 main 分支到 origin
git push origin feature      # 推送 feature 分支
git push -u origin main      # 推送并设置上游分支
git push --all origin        # 推送所有分支
git push origin --delete feature  # 删除远程分支
git push --force             # 强制推送（危险操作）
git push --force-with-lease  # 更安全的强制推送
```

**设置上游分支**
```bash
git branch --set-upstream-to=<remote>/<branch>
```
说明：设置本地分支的上游分支。

示例：
```bash
git branch --set-upstream-to=origin/main main
git push -u origin feature   # 推送时设置上游
```

---

## 进阶用法

### 标签管理

**创建标签**
```bash
git tag <tag-name>
```
说明：给当前提交打标签。

示例：
```bash
git tag v1.0.0               # 创建轻量标签
git tag -a v1.0.0 -m "版本 1.0.0"  # 创建附注标签
git tag -a v0.9.0 abc1234    # 给指定提交打标签
```

**查看标签**
```bash
git tag
```
说明：列出所有标签。

示例：
```bash
git tag                      # 列出所有标签
git tag -l "v1.*"            # 列出 v1.x 版本的标签
git show v1.0.0              # 查看标签详细信息
```

**推送标签**
```bash
git push origin <tag-name>
```
说明：推送标签到远程仓库。

示例：
```bash
git push origin v1.0.0       # 推送单个标签
git push origin --tags       # 推送所有标签
```

**删除标签**
```bash
git tag -d <tag-name>
```
说明：删除本地标签。

示例：
```bash
git tag -d v1.0.0            # 删除本地标签
git push origin --delete v1.0.0  # 删除远程标签
```

### 储藏（Stash）

**储藏修改**
```bash
git stash
```
说明：临时保存当前工作区和暂存区的修改。

示例：
```bash
git stash                    # 储藏当前修改
git stash save "修复bug的临时保存"  # 带消息的储藏
git stash -u                 # 包括未跟踪的文件
git stash --include-untracked  # 同上
```

**查看储藏**
```bash
git stash list
```
说明：列出所有储藏。

示例：
```bash
git stash list
# 输出示例：
# stash@{0}: WIP on main: abc1234 提交消息
# stash@{1}: On feature: 修复bug的临时保存
```

**应用储藏**
```bash
git stash apply
git stash pop
```
说明：恢复储藏的修改。

示例：
```bash
git stash apply              # 应用最新储藏，保留储藏记录
git stash apply stash@{1}    # 应用指定储藏
git stash pop                # 应用最新储藏并删除记录
git stash pop stash@{1}      # 应用并删除指定储藏
```

**删除储藏**
```bash
git stash drop
```
说明：删除储藏记录。

示例：
```bash
git stash drop stash@{0}     # 删除指定储藏
git stash clear              # 删除所有储藏
```

### 子模块（Submodule）

**添加子模块**
```bash
git submodule add <url> <path>
```
说明：添加一个子模块到当前仓库。

示例：
```bash
git submodule add https://github.com/user/lib.git libs/lib
```

**初始化和更新子模块**
```bash
git submodule init
git submodule update
```
说明：初始化和更新子模块。

示例：
```bash
git submodule init           # 初始化子模块配置
git submodule update         # 更新子模块代码
git submodule update --init  # 初始化并更新
git submodule update --init --recursive  # 递归初始化和更新
```

**克隆含子模块的项目**
```bash
git clone --recursive <url>
```
说明：克隆项目并同时克隆所有子模块。

示例：
```bash
git clone --recursive https://github.com/user/project.git
```

### 高级技巧

**查找提交**
```bash
git log --grep="关键词"
git log -S"代码片段"
```
说明：搜索提交历史。

示例：
```bash
git log --grep="bug"         # 搜索提交信息包含 bug 的提交
git log -S"function_name"    # 搜索添加/删除了某代码的提交
git log --author="张三" --since="2024-01-01"  # 搜索某人某时间后的提交
```

**清理未跟踪文件**
```bash
git clean
```
说明：删除未跟踪的文件。

示例：
```bash
git clean -n                 # 预览将要删除的文件
git clean -f                 # 删除未跟踪的文件
git clean -fd                # 删除未跟踪的文件和目录
git clean -fX                # 只删除被忽略的文件
```

**重写历史**
```bash
git filter-branch
git rebase -i
```
说明：修改提交历史（谨慎使用）。

示例：
```bash
git rebase -i HEAD~5         # 交互式修改最近5次提交
# 在编辑器中可以：
# pick：保留提交
# reword：修改提交信息
# edit：修改提交内容
# squash：合并到上一个提交
# drop：删除提交
```

**二分查找问题提交**
```bash
git bisect
```
说明：使用二分法查找引入问题的提交。

示例：
```bash
git bisect start             # 开始二分查找
git bisect bad               # 标记当前版本有问题
git bisect good v1.0.0       # 标记某个版本正常
# Git 会自动切换到中间的提交，测试后继续标记
git bisect good              # 当前版本正常
git bisect bad               # 当前版本有问题
git bisect reset             # 结束二分查找
```

**别名配置**
```bash
git config --global alias.<别名> <命令>
```
说明：为常用命令创建别名。

示例：
```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.lg "log --graph --oneline --all"
# 使用：git st, git co, git br 等
```

**忽略文件**

创建 `.gitignore` 文件来忽略不需要跟踪的文件：

示例：
```bash
# .gitignore 文件内容示例
*.log                        # 忽略所有 .log 文件
node_modules/                # 忽略 node_modules 目录
.env                         # 忽略环境变量文件
dist/                        # 忽略构建输出目录
*.swp                        # 忽略 vim 临时文件
.DS_Store                    # 忽略 macOS 系统文件
```

**查看文件历史**
```bash
git blame <file>
git log -p <file>
```
说明：查看文件的修改历史和责任人。

示例：
```bash
git blame index.html         # 查看每行代码的最后修改者
git log -p index.html        # 查看文件的详细修改历史
git log --follow index.html  # 追踪文件重命名前的历史
```

---

## 常用场景速查

### 撤销操作汇总
```bash
# 撤销工作区修改
git restore <file>           # Git 2.23+
git checkout -- <file>       # 旧版本

# 取消暂存
git restore --staged <file>  # Git 2.23+
git reset HEAD <file>        # 旧版本

# 修改最后一次提交
git commit --amend

# 回退到某个提交
git reset --soft HEAD~1      # 保留修改，取消提交
git reset --mixed HEAD~1     # 保留工作区修改，取消暂存和提交（默认）
git reset --hard HEAD~1      # 完全回退（危险）

# 撤销某次提交（创建新提交）
git revert <commit-hash>
```

### 协作流程
```bash
# 1. 克隆项目
git clone <url>

# 2. 创建功能分支
git checkout -b feature-xxx

# 3. 开发并提交
git add .
git commit -m "实现 xxx 功能"

# 4. 拉取最新代码
git checkout main
git pull origin main

# 5. 合并主分支到功能分支
git checkout feature-xxx
git merge main

# 6. 推送功能分支
git push origin feature-xxx

# 7. 在 GitHub/GitLab 上创建 Pull Request/Merge Request

# 8. 代码审查通过后，合并到主分支
git checkout main
git merge feature-xxx
git push origin main

# 9. 删除功能分支
git branch -d feature-xxx
git push origin --delete feature-xxx
```

---

## 帮助命令

```bash
git help                     # 显示帮助信息
git help <command>           # 显示某个命令的帮助
git <command> --help         # 同上
git <command> -h             # 显示简短帮助
```

---

**提示：**
- 使用 `git status` 经常检查仓库状态
- 提交前使用 `git diff` 检查修改内容
- 使用有意义的提交信息
- 定期推送到远程仓库备份代码
- 重要操作前先备份或使用 `git stash`
- 使用分支进行功能开发，不直接在主分支修改
