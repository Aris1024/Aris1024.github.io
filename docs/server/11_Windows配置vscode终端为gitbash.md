# Windows vscode 配置终端为 gitbash

---

背景: Windows 中 vscode,在使用终端时是自带的 powershell,非常不习惯,这里切换成 gitbash

- 下载并安装 [Git for Windows](https://git-scm.com/downloads/win) 
- 在 vscode 的偏好设置中,搜索 `Integrated: Default Profile`
- 找到:Terminal › Integrated › Default Profile: Windows
- 下拉选择 Git Bash



这样就可以使用 tail 命令看日志了.

enjoy! 😄



在 **Git Bash**（基于 Linux 命令环境）和日常开发中，以下是一些 **最常用命令**，涵盖文件操作、Git 管理、系统查看等场景：

---

### **1. 文件与目录操作**
| 命令          | 作用                   | 示例                                       |
| ------------- | ---------------------- | ------------------------------------------ |
| `ls`          | 列出当前目录内容       | `ls -l`（详细列表）                        |
| `cd`          | 切换目录               | `cd ~/projects`（进入家目录下的 projects） |
| `pwd`         | 显示当前目录路径       | `pwd`                                      |
| `mkdir`       | 创建目录               | `mkdir new_folder`                         |
| `touch`       | 创建空文件             | `touch file.txt`                           |
| `cp`          | 复制文件/目录          | `cp file.txt backup/`                      |
| `mv`          | 移动或重命名文件       | `mv old.txt new.txt`                       |
| `rm`          | 删除文件               | `rm file.txt`                              |
| `rm -r`       | 递归删除目录（慎用！） | `rm -r old_dir`                            |
| `cat`         | 查看文件内容           | `cat log.txt`                              |
| `less`/`more` | 分页查看文件           | `less large_file.log`                      |
| `head`/`tail` | 查看文件开头/结尾      | `tail -f log.txt`（实时跟踪）              |
| `find`        | 查找文件               | `find . -name "*.js"`                      |
| `grep`        | 文本搜索               | `grep "error" log.txt`                     |

---

### **2. Git 版本控制**
| 命令           | 作用              | 示例                                         |
| -------------- | ----------------- | -------------------------------------------- |
| `git init`     | 初始化 Git 仓库   | `git init`                                   |
| `git clone`    | 克隆远程仓库      | `git clone https://github.com/user/repo.git` |
| `git status`   | 查看仓库状态      | `git status`                                 |
| `git add`      | 添加文件到暂存区  | `git add .`（添加所有文件）                  |
| `git commit`   | 提交更改          | `git commit -m "fix: bug"`                   |
| `git push`     | 推送到远程仓库    | `git push origin main`                       |
| `git pull`     | 拉取远程更新      | `git pull`                                   |
| `git branch`   | 管理分支          | `git branch -a`（查看所有分支）              |
| `git checkout` | 切换分支/恢复文件 | `git checkout dev`                           |
| `git merge`    | 合并分支          | `git merge feature`                          |
| `git log`      | 查看提交历史      | `git log --oneline`                          |
| `git diff`     | 查看差异          | `git diff HEAD~1`                            |

---

### **3. 系统与权限**
| 命令         | 作用             | 示例                         |
| ------------ | ---------------- | ---------------------------- |
| `chmod`      | 修改文件权限     | `chmod +x script.sh`         |
| `chown`      | 修改文件所有者   | `sudo chown user:group file` |
| `ps`         | 查看进程         | `ps aux \| grep node`        |
| `kill`       | 终止进程         | `kill -9 1234`（强制终止）   |
| `df`/`du`    | 查看磁盘使用情况 | `df -h`（人类可读格式）      |
| `top`/`htop` | 实时系统监控     | `top`                        |

---

### **4. 网络相关**
| 命令          | 作用               | 示例                                   |
| ------------- | ------------------ | -------------------------------------- |
| `ping`        | 测试网络连通性     | `ping google.com`                      |
| `curl`/`wget` | 下载文件或请求 API | `curl -O https://example.com/file.zip` |
| `ssh`         | 远程登录服务器     | `ssh user@192.168.1.1`                 |
| `scp`         | 安全传输文件       | `scp file.txt user@host:/path`         |

---

### **5. 其他实用命令**
| 命令        | 作用                        | 示例                           |
| ----------- | --------------------------- | ------------------------------ |
| `alias`     | 创建命令别名                | `alias ll='ls -al'`            |
| `tar`/`zip` | 压缩解压文件                | `tar -czvf archive.tar.gz dir` |
| `history`   | 查看命令历史                | `history \| grep git`          |
| `man`       | 查看命令手册                | `man grep`                     |
| `echo`      | 输出文本                    | `echo "Hello" > file.txt`      |
| `&&`/`\|\|` | 命令组合（成功/失败后执行） | `git add . && git commit`      |

---

### **6. 快捷键（Git Bash/VSCode 终端）**
- **清屏**：`Ctrl + L` 或 `clear`
- **终止命令**：`Ctrl + C`
- **补全路径/命令**：`Tab` 键
- **搜索历史命令**：`Ctrl + R`

---

### **附：常用符号**
| 符号 | 作用                   | 示例                           |
| ---- | ---------------------- | ------------------------------ |
| `>`  | 重定向输出到文件       | `echo "text" > file.txt`       |
| `>>` | 追加到文件末尾         | `echo "text" >> file.txt`      |
| `\|` | 管道（传递输出）       | `cat file.txt \| grep "error"` |
| `~`  | 用户家目录             | `cd ~/Downloads`               |
| `*`  | 通配符（匹配任意字符） | `rm *.tmp`                     |

---

### **学习建议**
1. **掌握基础命令**：`ls`、`cd`、`mkdir`、`rm`、`grep`、`git` 系列。
2. **善用 `--help` 和 `man`**：如 `git --help` 或 `man ls`。
3. **组合命令**：通过管道（`|`）和重定向（`>`）提高效率。

这些命令覆盖了 **80% 的日常开发场景**，建议先熟悉基础操作，再逐步深入！
