# Phase 1 笔记：Git 与 GitHub 基础

> 新手第一个项目 · 2026-06-26

## 1. SSH 密钥连接 GitHub

### 密钥位置
Windows 下 SSH 密钥存放在 `C:\Users\<用户名>\.ssh\`：
- `id_ed25519` — 私钥（不能泄露）
- `id_ed25519.pub` — 公钥（上传到 GitHub）

### 上传到 GitHub
1. 打开 https://github.com/settings/keys
2. New SSH Key → 粘贴 `.pub` 文件内容
3. 如果中文名导致乱码，去掉注释部分，只留 `ssh-ed25519 AAA...` 即可

### 验证连接
```bash
# 首次连接需添加信任
ssh-keyscan github.com >> ~/.ssh/known_hosts
ssh -T git@github.com
# 输出 "successfully authenticated" 即成功
```

---

## 2. Git 核心概念

### 三种状态
```
工作目录（你写代码）──git add──▶ 暂存区（购物车）──git commit──▶ 本地仓库（存档）
                                                                    │
                                                              git push
                                                                    │
                                                              GitHub（远程）
```

### git status 解读
| 状态 | 含义 |
|------|------|
| `Untracked files`（红） | 新文件，Git 还没管 |
| `Changes to be committed`（绿） | 已暂存，下次 commit 会提交 |
| `nothing to commit` | 暂存区是空的 |

---

## 3. 常用命令速查

| 命令 | 作用 |
|------|------|
| `git init` | 初始化本地仓库（创建 `.git` 文件夹） |
| `git remote add origin <地址>` | 关联远程仓库（只是记地址，不会自动同步） |
| `git remote -v` | 查看已关联的远程仓库 |
| `git add .` | 所有文件加入暂存区 |
| `git commit -m "消息"` | 提交到本地仓库 |
| `git push -u origin main` | 首次推送 + 建立本地/远程绑定关系 |
| `git push` | 后续推送（已绑定的分支） |
| `git pull origin main --allow-unrelated-histories` | 强行合并两个无关仓库 |
| `git branch -M main` | 重命名当前分支 |

---

## 4. 遇到的问题与解决

### 问题1：Windows 没有 `touch` 命令
- **原因**：`touch` 是 Linux/Mac 命令
- **解决**：PowerShell 用 `New-Item 文件名`，CMD 用 `type nul > 文件名`

### 问题2：push 被拒绝 `rejected...fetch first`
- **原因**：GitHub 仓库有初始文件（README），本地也有 commit，两边历史不一致
- **解决**：先 `git pull origin main --allow-unrelated-histories` 合并，再 `git push`

### 问题3：LS/CRLF 警告
- **原因**：Windows 用 `\r\n`，Linux 用 `\n`，Git 自动转换
- **处理**：不用管，Git 自动处理换行符

### 问题4：Vim 编辑器
- 出现场景：`git pull` 合并时需要输入合并信息
- **退出方法**：按 `:wq` 回车

### 问题5：`-u` 参数
- `git push -u origin main` 有效（建立绑定关系）
- `git pull -u origin main` 无效（pull 没有 `-u` 参数）
