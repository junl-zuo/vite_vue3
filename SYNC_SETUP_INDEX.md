# 组件同步方案文档索引

本项目包含两份配置指南，请根据你使用的 Git 平台选择对应的文档。

## 📋 文档清单

### 🐙 GitHub 用户
**文档**: [SYNC_SETUP_GUIDE_GITHUB.md](SYNC_SETUP_GUIDE_GITHUB.md)

适用于使用 GitHub 作为代码托管平台的用户。包含：
- GitHub 仓库地址配置
- GitHub Actions CI/CD 配置（可选）
- 完整的 Git Hook 脚本（GitHub 版本）

**快速开始**:
```bash
# 根据文档在主仓库配置 .git/hooks/post-commit*
# 修改脚本中的 GitHub 仓库地址
# 提交代码后自动同步到子仓库
```

---

### 🦊 GitLab 用户
**文档**: [SYNC_SETUP_GUIDE_GITLAB.md](SYNC_SETUP_GUIDE_GITLAB.md)

适用于使用 GitLab 作为代码托管平台的用户。包含：
- GitLab 仓库地址配置
- GitLab CI 配置（可选）
- 私有仓库认证方式（Token / SSH）
- 完整的 Git Hook 脚本（GitLab 版本）

**快速开始**:
```bash
# 根据文档在主仓库配置 .git/hooks/post-commit*
# 修改脚本中的 GitLab 仓库地址
# 如果是私有仓库，配置 SSH 或 Token
# 提交代码后自动同步到子仓库
```

---

## 🔄 工作原理

```
主仓库 (vite_vue3)
    ↓ git commit
    ↓ Hook 触发
    ↓ 自动克隆子仓库
    ↓ 复制 src/components
    ↓ 提交并推送
子仓库 (vite_vue3_child)
    ↓ 自动接收更新
    ↓ src/components 目录更新
```

---

## ✅ 核心特性

- ✨ **自动同步**: 每次提交自动触发同步，无需手动操作
- 🎯 **精准同步**: 只同步 `src/components` 目录，其他文件不受影响
- 🔒 **安全可靠**: 使用 Git 标准 Hook，支持 SSH / Token 认证
- 👥 **团队友好**: 支持多人协作，配置一次全队使用
- 🚀 **零成本迁移**: 无需修改代码逻辑，纯配置级别

---

## 📝 两个平台的主要区别

| 方面 | GitHub | GitLab |
|------|--------|--------|
| **仓库地址** | github.com | gitlab.com |
| **仓库链接格式** | `https://github.com/user/repo.git` | `https://gitlab.com/user/repo.git` |
| **SSH 链接格式** | `git@github.com:user/repo.git` | `git@gitlab.com:user/repo.git` |
| **私有仓库认证** | Personal Access Token | Personal Access Token |
| **CI/CD 配置文件** | `.github/workflows/*.yml` | `.gitlab-ci.yml` |
| **CI/CD 配置放置位置** | 主仓库 `.github/workflows/` | 各仓库根目录 |
| **Git Hook 脚本** | 相同 | 相同（只改URL） |

---

## 🚀 快速检查清单

### 如果使用 GitHub：
- [ ] 打开 `SYNC_SETUP_GUIDE_GITHUB.md`
- [ ] 按照第一步配置 `.git/hooks/`
- [ ] 修改脚本中的 GitHub 仓库地址（`github.com/...`）
- [ ] （可选）按照第二步配置 GitHub Actions
- [ ] 提交测试

### 如果使用 GitLab：
- [ ] 打开 `SYNC_SETUP_GUIDE_GITLAB.md`
- [ ] 按照第一步配置 `.git/hooks/`
- [ ] 修改脚本中的 GitLab 仓库地址（`gitlab.com/...`）
- [ ] 如果是私有仓库，配置 SSH 或 Token
- [ ] （可选）按照第二步配置 `.gitlab-ci.yml`
- [ ] 提交测试

---

## 📚 文件说明

```
.
├── SYNC_SETUP_GUIDE_GITHUB.md   ← GitHub 用户文档
├── SYNC_SETUP_GUIDE_GITLAB.md   ← GitLab 用户文档
├── SYNC_SETUP_INDEX.md          ← 本文件（索引）
└── .git/hooks/
    ├── post-commit              ← Hook 入口脚本
    └── post-commit.ps1          ← PowerShell 实现脚本
```

---

## 🤔 常见问题

**Q: 我应该选择 GitHub 还是 GitLab 文档？**

A: 取决于你的仓库托管在哪个平台。
- 如果仓库地址是 `github.com`，使用 GitHub 文档
- 如果仓库地址是 `gitlab.com`，使用 GitLab 文档

**Q: Hook 脚本在两个平台是一样的吗？**

A: 基本逻辑相同，只需改仓库地址。Core 脚本（.git/hooks 部分）完全可以通用。

**Q: 我能同时支持 GitHub 和 GitLab 吗？**

A: 可以。但通常需要在子仓库地址或 Token 切换上做逻辑判断。详见各文档的 "脚本修改参考" 部分。

**Q: 两份文档内容重复，为什么不合并？**

A: 为了清晰性和易用性。用户可快速找到对应平台的完整指南，无需在文档中频繁切换。

---

## 🔗 相关链接

- [GitHub 版本完整指南](SYNC_SETUP_GUIDE_GITHUB.md)
- [GitLab 版本完整指南](SYNC_SETUP_GUIDE_GITLAB.md)

---

## 📞 技术支持

遇到问题？请按以下步骤排查：

1. 检查对应文档的「排查问题」部分
2. 验证 Hook 脚本是否正确安装和执行
3. 查看 Git 提交时的输出信息
4. 确认仓库地址和访问权限

记录时间：2026-03-27
