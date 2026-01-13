# GitHub Pages 静态网站部署完整流程文档

## 概述

本文档详细描述了如何将静态HTML项目部署到GitHub Pages，使其可以通过公网URL直接访问。适用于AI工具自动化执行或开发者手动操作。

## 前置条件

- 拥有GitHub账号
- 项目已存在于本地或需要创建新项目
- 具备Git基本操作权限

## 流程步骤

### 第一阶段：项目准备和Git初始化

#### 1.1 检查项目状态
```bash
# 检查当前目录是否为Git仓库
git status
```

#### 1.2 初始化Git仓库（如果需要）
```bash
# 如果不是Git仓库，初始化
git init

# 配置用户信息
git config --global user.name "用户名"
git config --global user.email "邮箱地址"
```

#### 1.3 添加项目文件
```bash
# 添加所有文件到暂存区
git add .

# 提交初始版本
git commit -m "feat: 初始化项目

- 添加静态HTML文件
- 配置项目基础结构"
```

### 第二阶段：GitHub仓库配置

#### 2.1 创建GitHub仓库
**方式A：通过GitHub网页界面**
1. 访问 https://github.com/new
2. 填写仓库名称（如：`static-html`）
3. 选择 Public（公开仓库，GitHub Pages免费）
4. 不要初始化README、.gitignore或LICENSE
5. 点击 "Create repository"

**方式B：通过GitHub CLI（如果已安装）**
```bash
gh repo create static-html --public --source=. --remote=origin --push
```

#### 2.2 添加远程仓库
```bash
# 添加GitHub远程仓库
git remote add origin https://github.com/用户名/仓库名.git

# 验证远程仓库配置
git remote -v
```

### 第三阶段：身份认证配置

#### 3.1 生成个人访问令牌（Personal Access Token）

**步骤**：
1. 访问 https://github.com/settings/tokens
2. 点击 "Generate new token (classic)"
3. 填写令牌描述：`Static Website Deployment`
4. 选择过期时间（建议30-90天）
5. 权限配置：
   - ✅ `repo` - 完整仓库访问权限
   - ✅ `workflow` - GitHub Actions权限（可选）
6. 点击 "Generate token"
7. **重要**：立即复制令牌（格式：`github_pat_xxxxxxxxxx`）

#### 3.2 使用令牌推送代码
```bash
# 使用令牌推送到GitHub
git push https://用户名:令牌@github.com/用户名/仓库名.git master
```

**示例**：
```bash
git push https://VitaCocoo:github_pat_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx@github.com/VitaCocoo/static-html.git master
```

### 第四阶段：GitHub Pages部署配置

#### 4.1 方法一：手动配置（推荐新手）

**步骤**：
1. 访问仓库设置页面：`https://github.com/用户名/仓库名/settings/pages`
2. 在 "Source" 部分：
   - 选择 "Deploy from a branch"
   - Branch: 选择 `master` 或 `main`
   - Folder: 选择 `/ (root)`
3. 点击 "Save"
4. 等待3-5分钟，页面会显示部署URL

#### 4.2 方法二：GitHub Actions自动化部署（推荐）

**创建工作流文件**：
```bash
# 创建GitHub Actions目录
mkdir -p .github/workflows
```

**创建部署配置文件** `.github/workflows/deploy.yml`：
```yaml
name: Deploy Static HTML to GitHub Pages

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Setup Pages
        uses: actions/configure-pages@v4
      
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: '.'
      
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

**提交并推送配置**：
```bash
# 添加工作流文件
git add .github/workflows/deploy.yml

# 提交配置
git commit -m "feat: 添加GitHub Pages自动部署配置

- 添加GitHub Actions工作流
- 支持自动部署静态HTML到GitHub Pages
- 每次推送master分支时自动触发部署"

# 推送到GitHub
git push https://用户名:令牌@github.com/用户名/仓库名.git master
```

**配置Pages源为GitHub Actions**：
1. 访问 `https://github.com/用户名/仓库名/settings/pages`
2. Source 选择 "GitHub Actions"
3. 保存设置

### 第五阶段：验证部署

#### 5.1 检查部署状态
```bash
# 访问Actions页面查看部署状态
# https://github.com/用户名/仓库名/actions
```

#### 5.2 访问部署的网站
**预期URL格式**：
- 主域名：`https://用户名.github.io/仓库名/`
- 具体文件：`https://用户名.github.io/仓库名/文件名.html`

**示例**：
- `https://vitacocoo.github.io/static-html/`
- `https://vitacocoo.github.io/static-html/店员端完整版.html`
- `https://vitacocoo.github.io/static-html/app/index.html`

## AI工具自动化脚本模板

### Bash脚本版本
```bash
#!/bin/bash

# 配置变量
GITHUB_USERNAME="用户名"
REPO_NAME="仓库名"
GITHUB_TOKEN="令牌"
COMMIT_MESSAGE="feat: 部署静态网站"

# 检查Git状态
if ! git rev-parse --git-dir > /dev/null 2>&1; then
    echo "初始化Git仓库..."
    git init
fi

# 添加文件并提交
echo "添加文件到Git..."
git add .
git commit -m "$COMMIT_MESSAGE"

# 添加远程仓库（如果不存在）
if ! git remote get-url origin > /dev/null 2>&1; then
    git remote add origin https://github.com/$GITHUB_USERNAME/$REPO_NAME.git
fi

# 创建GitHub Actions配置
mkdir -p .github/workflows
cat > .github/workflows/deploy.yml << 'EOF'
name: Deploy Static HTML to GitHub Pages
on:
  push:
    branches: [ master ]
permissions:
  contents: read
  pages: write
  id-token: write
jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/configure-pages@v4
      - uses: actions/upload-pages-artifact@v3
        with:
          path: '.'
      - id: deployment
        uses: actions/deploy-pages@v4
EOF

# 提交Actions配置
git add .github/workflows/deploy.yml
git commit -m "feat: 添加GitHub Pages自动部署配置"

# 推送到GitHub
echo "推送到GitHub..."
git push https://$GITHUB_USERNAME:$GITHUB_TOKEN@github.com/$GITHUB_USERNAME/$REPO_NAME.git master

echo "部署完成！"
echo "网站将在几分钟后可访问：https://$GITHUB_USERNAME.github.io/$REPO_NAME/"
```

### Python脚本版本
```python
import os
import subprocess
import sys

class GitHubPagesDeployer:
    def __init__(self, username, repo_name, token):
        self.username = username
        self.repo_name = repo_name
        self.token = token
        self.repo_url = f"https://github.com/{username}/{repo_name}.git"
        self.push_url = f"https://{username}:{token}@github.com/{username}/{repo_name}.git"
    
    def run_command(self, command, check=True):
        """执行shell命令"""
        try:
            result = subprocess.run(command, shell=True, check=check, 
                                  capture_output=True, text=True)
            return result.stdout.strip()
        except subprocess.CalledProcessError as e:
            print(f"命令执行失败: {command}")
            print(f"错误信息: {e.stderr}")
            if check:
                sys.exit(1)
            return None
    
    def init_git_repo(self):
        """初始化Git仓库"""
        if not os.path.exists('.git'):
            print("初始化Git仓库...")
            self.run_command("git init")
    
    def add_and_commit(self, message="feat: 部署静态网站"):
        """添加文件并提交"""
        print("添加文件到Git...")
        self.run_command("git add .")
        self.run_command(f'git commit -m "{message}"')
    
    def setup_remote(self):
        """设置远程仓库"""
        try:
            self.run_command("git remote get-url origin")
        except:
            print("添加远程仓库...")
            self.run_command(f"git remote add origin {self.repo_url}")
    
    def create_github_actions(self):
        """创建GitHub Actions配置"""
        os.makedirs('.github/workflows', exist_ok=True)
        
        workflow_content = '''name: Deploy Static HTML to GitHub Pages

on:
  push:
    branches: [ master ]

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Setup Pages
        uses: actions/configure-pages@v4
      
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: '.'
      
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
'''
        
        with open('.github/workflows/deploy.yml', 'w') as f:
            f.write(workflow_content)
        
        print("创建GitHub Actions配置...")
        self.run_command("git add .github/workflows/deploy.yml")
        self.run_command('git commit -m "feat: 添加GitHub Pages自动部署配置"')
    
    def push_to_github(self):
        """推送到GitHub"""
        print("推送到GitHub...")
        self.run_command(f"git push {self.push_url} master")
    
    def deploy(self):
        """执行完整部署流程"""
        print(f"开始部署 {self.repo_name} 到GitHub Pages...")
        
        self.init_git_repo()
        self.add_and_commit()
        self.setup_remote()
        self.create_github_actions()
        self.push_to_github()
        
        print("部署完成！")
        print(f"网站将在几分钟后可访问：https://{self.username}.github.io/{self.repo_name}/")
        print(f"请访问 https://github.com/{self.username}/{self.repo_name}/settings/pages 配置Pages源为 'GitHub Actions'")

# 使用示例
if __name__ == "__main__":
    deployer = GitHubPagesDeployer(
        username="VitaCocoo",
        repo_name="static-html", 
        token="github_pat_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
    )
    deployer.deploy()
```

## 常见问题和解决方案

### 问题1：推送权限被拒绝
**错误信息**：`Permission denied` 或 `403 Forbidden`

**解决方案**：
1. 检查个人访问令牌是否正确
2. 确认令牌具有 `repo` 权限
3. 检查仓库是否为公开仓库
4. 验证用户名和仓库名是否正确

### 问题2：GitHub Pages未生效
**可能原因**：
1. 仓库为私有仓库（需要GitHub Pro）
2. Pages源配置错误
3. 部署仍在进行中

**解决方案**：
1. 确保仓库为公开状态
2. 检查 Settings → Pages 配置
3. 查看 Actions 页面的部署状态
4. 等待3-10分钟让部署完成

### 问题3：网站访问404
**可能原因**：
1. 文件路径不正确
2. 主页文件名不是 `index.html`
3. 部署未完成

**解决方案**：
1. 检查文件是否正确推送到仓库
2. 确认访问的URL路径正确
3. 在根目录添加 `index.html` 作为主页

## 安全注意事项

1. **令牌安全**：
   - 不要在代码中硬编码令牌
   - 定期更换令牌
   - 使用环境变量存储敏感信息

2. **仓库权限**：
   - 仅授予必要的权限
   - 定期审查协作者权限

3. **内容安全**：
   - 不要在静态文件中包含敏感信息
   - 注意公开仓库的内容都是可见的

## 总结

本流程文档提供了完整的GitHub Pages部署方案，包括：
- 详细的手动操作步骤
- 自动化脚本模板
- 常见问题解决方案
- 安全最佳实践

AI工具可以根据此文档实现自动化部署功能，开发者也可以参考进行手动部署。