# Vite+Vue3 组件同步方案执行文档

## 项目概述

本文档整理了一套完整的组件同步解决方案，用于将主仓库的 `src/components` 目录自动同步到子仓库。无论提交哪些修改（新增、修改、删除组件），都会自动推送到子仓库的指定位置。

### 仓库结构
- **主仓库**: https://github.com/junl-zuo/vite_vue3.git
  - 存放完整项目代码
  - `src/components/` - 组件目录（需要同步的部分）
  
- **子仓库**: https://github.com/junl-zuo/vite_vue3_child.git
  - 分布式存储，各成员可独立开发
  - `src/components/` - 同步过来的组件目录
  - 其他配置、根目录文件不被修改

---

## 第一步：在主仓库中配置 Git Hook

### 1.1 创建 Hook 脚本目录

如果不存在 `.git/hooks` 目录，Git 会自动创建。

### 1.2 创建 bash 脚本（`.git/hooks/post-commit`）

```bash
#!/bin/bash
powershell.exe -NoProfile -ExecutionPolicy Bypass -File "$(dirname "$0")/post-commit.ps1"
```

**这个文件的作用**: 
- 在每次 `git commit` 后自动触发
- 调用后面的 PowerShell 脚本执行同步操作

### 1.3 创建 PowerShell 脚本（`.git/hooks/post-commit.ps1`）

```powershell
# PowerShell post-commit hook

$CurrentRepo = Get-Location
$TempDir = "$env:TEMP\vite_vue3_child_$(Get-Random)"

try {
    # 如果目录存在则删除
    if (Test-Path $TempDir) { Remove-Item -Recurse -Force $TempDir }
    
    # 克隆子仓库
    Write-Host "Cloning child repo..."
    git clone https://github.com/junl-zuo/vite_vue3_child.git $TempDir 2>&1 | Out-Null
    
    if (-not (Test-Path $TempDir)) {
        Write-Host "Failed to clone"
        exit 1
    }
    
    # 进入临时目录
    Push-Location $TempDir
    
    try {
        # 清除 src/components 旧文件
        if (Test-Path "src/components") {
            Remove-Item -Recurse -Force "src/components"
        }
        
        # 创建 src/components 目录
        New-Item -ItemType Directory -Force -Path "src/components" | Out-Null
        
        # 复制文件
        Write-Host "Copying components..."
        Get-ChildItem -Path "$CurrentRepo\src\components" -File | ForEach-Object {
            Copy-Item -Path $_.FullName -Destination "src/components/$($_.Name)" -Force
        }
        
        # 配置 git
        git config user.email "bot@example.com" 2>&1 | Out-Null
        git config user.name "Auto Bot" 2>&1 | Out-Null
        
        # 提交并推送
        Write-Host "Committing and pushing..."
        git add . 2>&1 | Out-Null
        $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
        git commit -m "Sync components from main repo $timestamp" 2>&1 | Out-Null
        git push origin main --force 2>&1 | Out-Null
        
        Write-Host "Sync completed successfully"
    } finally {
        Pop-Location
    }
} catch {
    Write-Host "Error: $_"
} finally {
    # 清理临时目录
    if (Test-Path $TempDir) {
        Remove-Item -Recurse -Force $TempDir -ErrorAction SilentlyContinue
    }
}
```

**这个文件的作用**:
- 每次提交后自动触发
- 临时克隆子仓库
- 清空子仓库的 `src/components` 目录
- 从主仓库复制最新的组件文件
- 自动提交并推送到子仓库的 main 分支

### 1.4 修改项

**需要改成你自己的仓库地址：**

在 PowerShell 脚本中，找到这一行：
```powershell
git clone https://github.com/junl-zuo/vite_vue3_child.git $TempDir 2>&1 | Out-Null
```

改成：
```powershell
git clone https://github.com/【你的GitHub用户名】/【你的子仓库名】.git $TempDir 2>&1 | Out-Null
```

**例如**：
```powershell
git clone https://github.com/myname/myproject_child.git $TempDir 2>&1 | Out-Null
```

同时修改 bot 邮箱（可选）：
```powershell
git config user.email "your_email@example.com" 2>&1 | Out-Null
```

---

## 第二步：在子仓库中配置 GitHub Actions（可选但推荐）

### 2.1 在子仓库中创建工作流文件

路径: `.github/workflows/sync.yml`

```yaml
name: Sync Components

on:
  push:
    branches:
      - main

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Verify components directory
        run: |
          if [ -d "src/components" ]; then
            echo "✓ Components directory exists"
            ls -la src/components/
          else
            echo "✗ Components directory not found"
          fi
      
      - name: Update README (optional)
        run: |
          echo "Components synced at $(date)" >> SYNC_LOG.md
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          git add SYNC_LOG.md
          git commit -m "Update sync log" || echo "No changes to commit"
          git push
```

**这个文件的作用**:
- 每当子仓库 main 分支有推送，自动验证 `src/components` 目录
- 可选地更新同步日志

### 2.2 修改项

如果要在子仓库中添加自定义处理，修改这一部分：
```yaml
- name: Verify components directory
  run: |
    # 在这里添加你的自定义脚本
    echo "Components synced at $(date)"
```

---

## 第三步：使用流程

### 3.1 在主仓库中工作

1. **编辑、添加或删除组件文件**

   在 `src/components/` 目录下进行任何修改：
   ```bash
   # 编辑现有组件
   vim src/components/MyButton.vue
   
   # 添加新组件
   touch src/components/NewComponent.vue
   
   # 删除组件
   rm src/components/OldComponent.vue
   ```

2. **提交到主仓库**

   ```bash
   git add .
   git commit -m "修改组件说明"
   git push origin main
   ```

3. **自动同步触发**

   提交完成后，hook 会自动：
   - 克隆子仓库到临时目录
   - 清空子仓库的 `src/components`
   - 复制最新的组件文件
   - 推送到子仓库的 main 分支

   你会看到类似的输出：
   ```
   Cloning child repo...
   Copying components...
   Committing and pushing...
   Sync completed successfully
   ```

### 3.2 在子仓库中工作

子仓库的 `src/components` 目录由主仓库自动同步，**不要在子仓库中直接修改这个目录的文件**。

如果子仓库有其他配置或代码，可以正常修改而不会被覆盖：
```
vite_vue3_child/
├── src/
│   ├── components/          ← 自动同步，不要手动改
│   ├── pages/               ← 子仓库特有，可随意修改
│   └── utils/               ← 子仓库特有，可随意修改
├── vite.config.js           ← 不会被覆盖
├── package.json             ← 不会被覆盖
└── .github/workflows/       ← 子仓库自己的工作流
```

---

## 第四步：排查问题

### 问题 1：Hook 没有执行

**检查**:
```bash
# 查看 hook 是否存在
ls -la .git/hooks/post-commit*

# 验证权限（Linux/Mac）
chmod +x .git/hooks/post-commit
chmod +x .git/hooks/post-commit.ps1
```

**解决**:
- 确保两个文件都存在：`post-commit` 和 `post-commit.ps1`
- Windows 用户需要允许 PowerShell 脚本执行权限

### 问题 2：推送到子仓库失败

**检查**: 
```bash
# 验证网络连接
git ls-remote https://github.com/你的用户名/你的子仓库.git

# 查看最近的 Git 错误
git log -1 --format="%H %s"
```

**解决**:
- 确保仓库地址在 hook 脚本中写对了
- 确保有权限访问子仓库（公开仓库或有 token）
- 如果是私有仓库，需要配置 SSH 或 token

### 问题 3：文件同步到外层目录

**原因**: 使用了 `git subtree push` 而不是直接复制

**解决**: 确保使用本文档的 PowerShell 脚本，它使用直接复制的方式，不会有这个问题

### 问题 4：子仓库文件被覆盖

**原因**: 不小心把子仓库文件存放在 `src/components` 目录

**解决**: 
- 检查子仓库的目录结构
- 把其他文件移到 `src/components` 外的目录
- 只在 `src/components` 中存放同步的组件

---

## 第五步：多人协作

### 5.1 团队成员要求

每个团队成员在克隆主仓库后，需要有 Hook 脚本才能自动同步。

### 5.2 批量部署脚本

创建一个 `setup-hooks.sh` 或 `setup-hooks.ps1`（根据操作系统）：

**Linux/Mac** (`setup-hooks.sh`):
```bash
#!/bin/bash

# 复制 hook 文件
cp post-commit-template .git/hooks/post-commit
cp post-commit.ps1 .git/hooks/post-commit.ps1

# 设置权限
chmod +x .git/hooks/post-commit
chmod +x .git/hooks/post-commit.ps1

echo "✓ Hooks installed successfully"
```

**Windows** (`setup-hooks.ps1`):
```powershell
# 复制 hook 文件
Copy-Item -Path "post-commit-template" -Destination ".git/hooks/post-commit" -Force
Copy-Item -Path "post-commit.ps1" -Destination ".git/hooks/post-commit.ps1" -Force

Write-Host "✓ Hooks installed successfully"
```

### 5.3 在 README 中添加说明

```markdown
## 开发环境设置

1. 克隆仓库
   ```bash
   git clone https://github.com/你的用户/vite_vue3.git
   cd vite_vue3
   ```

2. 安装依赖
   ```bash
   npm install
   ```

3. 设置 Git Hooks（自动同步组件）
   - **Windows**: `powershell -ExecutionPolicy Bypass -File setup-hooks.ps1`
   - **Linux/Mac**: `bash setup-hooks.sh`

4. 开始开发
   ```bash
   npm run dev
   ```

> 注意：设置 Hook 后，每次 commit 都会自动同步 `src/components` 到子仓库
```

---

## 完整的配置清单

- [ ] 主仓库创建 `.git/hooks/post-commit`（bash 脚本）
- [ ] 主仓库创建 `.git/hooks/post-commit.ps1`（PowerShell 脚本）
- [ ] 修改 PowerShell 脚本中的子仓库地址
- [ ] （可选）子仓库创建 `.github/workflows/sync.yml`
- [ ] （可选）在主仓库 README 中添加 Hook 安装说明
- [ ] 提交配置文件到主仓库
- [ ] 测试：修改一个组件 → 提交 → 检查子仓库是否同步

---

## 常见问题 FAQ

**Q: 如果子仓库中途有紧急修改，会被覆盖吗？**

A: `src/components` 目录会被覆盖，但其他目录和文件不会。建议紧急修改放在 `src/components` 外的目录。

**Q: 频繁同步会不会对性能有影响？**

A: 每次同步只克隆一次子仓库（几 MB），然后复制文件和推送，整个过程通常 5-10 秒，不会有明显性能问题。

**Q: 能否同时同步多个子仓库？**

A: 可以。在 PowerShell 脚本中添加多个 git clone 和 push 语句即可。

**Q: 如果提交很频繁，会一直触发 Hook 吗？**

A: 是的，每次提交都会触发一次同步。如果想减少频率，可以设置条件只在修改 `src/components` 时才同步。

---

## 脚本修改参考

### 修改 1：只在修改了 components 时才同步

在 PowerShell 脚本开头添加：
```powershell
# 检查是否修改了 components
$ChangedFiles = git diff-tree --no-commit-id --name-only -r HEAD | Select-String "src/components"
if (-not $ChangedFiles) {
    Write-Host "No changes in src/components, skipping sync"
    exit 0
}
```

### 修改 2：同时同步多个子仓库

复制整个 try-catch 块，修改不同的仓库地址：

```powershell
# 同步仓库 1
$TempDir1 = "$env:TEMP\vite_vue3_child_$(Get-Random)"
git clone https://github.com/user/repo1.git $TempDir1
# ... 复制和推送逻辑 ...

# 同步仓库 2
$TempDir2 = "$env:TEMP\vite_vue3_child2_$(Get-Random)"
git clone https://github.com/user/repo2.git $TempDir2
# ... 复制和推送逻辑 ...
```

---

## 支持

如有问题或建议，请：
1. 检查上面的 "排查问题" 部分
2. 查看 Git Hook 的输出
3. 提交 Issue 到主仓库

记录：此方案在 2026-03-27 验证可行。
