# GitHub 上传指南

## 第一步：安装 Git

### Windows 用户

1. 下载 Git for Windows: https://git-scm.com/download/win
2. 运行安装程序，一路点击"Next"使用默认设置
3. 安装完成后，重新打开终端（PowerShell）

### 验证安装

```bash
git --version
```

如果显示版本号（如 `git version 2.45.2`），说明安装成功。

---

## 第二步：生成 SSH 密钥

1. 打开终端，运行：
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

2. 按三次 Enter 使用默认设置（不设置密码）
3. 找到公钥文件：
```bash
type $env:USERPROFILE\.ssh\id_ed25519.pub
```

4. 复制输出的内容（类似 `ssh-ed25519 AAAA... your_email@example.com`）

---

## 第三步：在 GitHub 添加 SSH 密钥

1. 登录 GitHub: https://github.com/
2. 点击右上角头像 → Settings
3. 左侧菜单选择 "SSH and GPG keys"
4. 点击 "New SSH key"
5. 标题填写：`LBS Lab`（或任意名称）
6. Key 粘贴刚才复制的公钥内容
7. 点击 "Add SSH key"

---

## 第四步：测试 SSH 连接

```bash
ssh -T git@github.com
```

如果看到类似提示，说明连接成功：
```
Hi tyttyt-ops! You've successfully authenticated...
```

---

## 第五步：初始化本地 Git 仓库

进入项目目录：
```bash
cd C:\Users\Administrator\Desktop\lbs
```

初始化仓库：
```bash
git init
```

---

## 第六步：创建 .gitignore 文件

创建 `.gitignore` 文件，排除不需要上传的文件：

```bash
# 创建 .gitignore 文件
@"
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
venv/
ENV/
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/
.installed.cfg
*.egg

# PyTorch
*.pth

# 数据和模型（根据需要调整）
models/smpl/*.pkl

# 输出文件（根据需要调整）
outputs/*.png
outputs/*.txt

# IDE
.vscode/
.idea/
*.swp
*.swo
*~

# 操作系统
.DS_Store
Thumbs.db

# Jupyter Notebook
.ipynb_checkpoints

# 分发 / 打包
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
share/python-wheels/
*.egg-info/
.installed.cfg
*.egg
MANIFEST
"@ | Out-File -FilePath .gitignore -Encoding utf8
```

---

## 第七步：添加文件到暂存区

```bash
# 添加所有文件
git add .

# 查看状态
git status
```

---

## 第八步：创建初始提交

```bash
git commit -m "Initial commit: SMPL LBS visualization experiment

- 完整实现了 SMPL 线性蒙皮（LBS）的四个阶段
- 提供了详细的可视化对比
- 手写 LBS 实现与官方结果完全一致
- 包含完整的 README 和实验说明"
```

---

## 第九步：关联远程仓库

```bash
# 关联你的 GitHub 仓库
git remote add origin git@github.com:tyttyt-ops/LBS_lab.git

# 验证远程仓库
git remote -v
```

---

## 第十步：推送到 GitHub

```bash
# 首次推送（设置主分支）
git branch -M main
git push -u origin main
```

如果提示输入密码，说明 SSH 密钥未配置成功，请回到第二步重新配置。

---

## 第十一步：验证上传

在浏览器访问：https://github.com/tyttyt-ops/LBS_lab

你应该能看到：
- ✅ `run_lbs_lab.py` - 主脚本
- ✅ `README.md` - 项目说明
- ✅ `GITHUB_UPLOAD_GUIDE.md` - 上传指南

---

## 常见问题

### Q1: 推送时提示 "Permission denied"
**A:** 检查 SSH 密钥是否正确添加到 GitHub，使用 `ssh -T git@github.com` 测试

### Q2: 推送时提示 "remote repository not found"
**A:** 确认你的 GitHub 仓库已创建，URL 是否正确

### Q3: 文件太大无法上传
**A:** Git 限制单个文件不超过 100MB。SMPL 模型文件（235MB）已通过 .gitignore 排除

### Q4: 如何更新代码
```bash
# 修改代码后
git add .
git commit -m "Update: 描述你的更改"
git push
```

### Q5: 如何回退提交
```bash
# 回退到上一个提交（保留更改）
git reset --soft HEAD~1

# 回退到上一个提交（删除更改）
git reset --hard HEAD~1

# 强制推送到远程（慎用）
git push --force
```

---

## 一键上传脚本

如果你已配置好 Git 和 SSH，可以运行以下脚本一键上传：

```bash
cd C:\Users\Administrator\Desktop\lbs

git init
git branch -M main
git add .
git commit -m "Initial commit: SMPL LBS visualization experiment"
git remote add origin git@github.com:tyttyt-ops/LBS_lab.git
git push -u origin main
```

---

## 需要上传的文件清单

✅ **必须上传：**
- `run_lbs_lab.py` - 主脚本
- `README.md` - 项目说明
- `GITHUB_UPLOAD_GUIDE.md` - 上传指南

❌ **不上传：**
- `models/smpl/SMPL_NEUTRAL.pkl` - 模型文件太大（通过 .gitignore 排除）
- `outputs/*.png` - 输出图片（通过 .gitignore 排除）
- `outputs/summary.txt` - 输出摘要（通过 .gitignore 排除）

---

## 后续建议

1. **上传模型文件**：如果你想分享模型文件，可以使用 Git LFS 或其他大文件存储服务

2. **上传输出图片**：如果想让用户看到效果，可以修改 `.gitignore`，允许上传 `outputs/` 目录

3. **创建 GitHub Releases**：如果项目版本稳定，可以创建 Release 标签

4. **添加 License**：选择合适的开源许可证，如 MIT、Apache 2.0 等

---

**上传完成后，你的项目链接将是：** https://github.com/tyttyt-ops/LBS_lab

祝你上传顺利！🎉