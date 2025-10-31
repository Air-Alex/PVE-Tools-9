# GitHub Actions Workflows 使用指南

本项目使用 GitHub Actions 实现自动化的 CI/CD 流程，包括代码检查、版本发布和更新日志生成。

## 📦 Workflow 列表

### 1. Syntax Check on Push（语法检查）
**文件**: `.github/workflows/syntax-check.yml`

**触发条件**:
- Push 到 `main` 或 `beta` 分支
- 修改了 `.sh` 文件

**功能**:
- ✅ Bash 语法检查 (`bash -n`)
- ✅ ShellCheck 静态分析
- ✅ 危险命令检测
- ✅ 自动检查所有脚本文件

**用途**: 确保每次代码推送都通过语法检查，防止将错误代码合并到主分支。

---

### 2. PR Validation（PR 验证）
**文件**: `.github/workflows/pr-validation.yml`

**触发条件**:
- 创建或更新 Pull Request 到 `main` 或 `beta` 分支

**功能**:
- ✅ ShellCheck 代码质量检查
- ✅ Bash 语法验证
- ✅ 版本号一致性检查
- ✅ 安全扫描（检测 eval/source 等危险命令）

**用途**: 在合并 PR 前自动验证代码质量和安全性。

---

### 3. Release to Main（正式版本发布）
**文件**: `.github/workflows/release.yml`

**触发条件**:
- 推送符合以下格式的 Tag:
  - `v[0-9]+.[0-9]+.[0-9]+` (例如: `v4.2.0`)
  - `v[0-9]+.[0-9]+.[0-9]+-stable` (例如: `v4.2.0-stable`)

**功能**:
- 🔍 语法检查（Bash + ShellCheck）
- 📝 自动更新 VERSION 文件和脚本版本号
- 📋 生成格式化的更新日志（Changelog）
  - ✨ 新增功能
  - 🐛 错误修复
  - ⚡ 性能优化
  - 📚 文档更新
  - 📊 版本统计
  - 👥 贡献者列表
- 🚀 创建 GitHub Release
- 📦 上传脚本文件到 Release

**使用方法**:
```bash
# 1. 确保本地代码已提交
git add .
git commit -m "feat: 添加新功能"

# 2. 创建并推送 Tag
git tag v4.3.0
git push origin v4.3.0

# 3. GitHub Actions 自动执行发布流程
# 4. 检查 Release 页面查看发布结果
```

**Changelog 格式示例**:
```markdown
## 📋 更新日志

### ✨ 新增功能
- 添加多块 NVMe 硬盘识别功能 (marecyra)
- 优化温度监控显示 (contributor1)

### 🐛 错误修复
- 修复 SATA 硬盘温度显示问题 (marecyra)
- 修复版本检查逻辑 (contributor2)

### ⚡ 性能优化
- 优化脚本启动速度 (marecyra)

### 📊 版本统计
- 提交数量: 15
- 贡献者数: 3

### 👥 贡献者
- @marecyra
- @contributor1
- @contributor2
```

---

### 4. Release to Beta（测试版本发布）
**文件**: `.github/workflows/beta-release.yml`

**触发条件**:
- 推送符合以下格式的 Tag:
  - `v[0-9]+.[0-9]+.[0-9]+-beta*` (例如: `v4.3.0-beta1`)
  - `v[0-9]+.[0-9]+.[0-9]+-alpha*` (例如: `v4.3.0-alpha1`)

**功能**:
- 🔍 语法检查（更宽松的规则）
- 📝 自动更新 VERSION 文件
- 📋 生成 Beta 版本更新日志
  - 包含实验性功能标注
- 🚧 创建 Pre-release
- 📦 上传脚本文件

**使用方法**:
```bash
# 发布 Beta 版本
git tag v4.3.0-beta1
git push origin v4.3.0-beta1

# 发布 Alpha 版本
git tag v4.3.0-alpha1
git push origin v4.3.0-alpha1
```

---

## 🎯 最佳实践

### Commit Message 规范

为了生成美观的 Changelog，请遵循 Conventional Commits 规范：

```bash
# 新增功能
git commit -m "feat: 添加温度监控功能"
git commit -m "feature(nvme): 支持多块 NVMe 硬盘"

# 错误修复
git commit -m "fix: 修复语法错误"
git commit -m "bugfix(kernel): 修复内核版本检测"

# 性能优化
git commit -m "perf: 优化脚本启动速度"

# 文档更新
git commit -m "docs: 更新 README"

# 代码重构
git commit -m "refactor: 重构日志系统"

# 测试
git commit -m "test: 添加单元测试"

# 构建/CI
git commit -m "build: 更新 workflow 配置"
git commit -m "ci: 添加语法检查"

# 其他维护
git commit -m "chore: 更新版本号"
git commit -m "style: 格式化代码"
```

### 版本发布流程

#### 正式版本发布
```bash
# 1. 切换到 main 分支
git checkout main
git pull origin main

# 2. 确保所有改动已提交
git status

# 3. 创建版本 Tag（遵循 SemVer 规范）
# 主版本.次版本.修订版本
git tag v4.3.0

# 4. 推送 Tag 触发发布
git push origin v4.3.0

# 5. 等待 GitHub Actions 完成（约 2-3 分钟）
# 6. 检查 Release 页面
```

#### Beta 版本发布
```bash
# 1. 切换到 beta 分支
git checkout beta

# 2. 创建 Beta Tag
git tag v4.3.0-beta1

# 3. 推送触发发布
git push origin v4.3.0-beta1
```

---

## 🔧 Workflow 配置说明

### ShellCheck 忽略规则

以下规则在项目中被忽略（根据实际需求调整）:
- `SC2086`: 双引号引用防止分词
- `SC2181`: 直接检查命令而非 $?
- `SC2162`: read 命令使用 -r 参数

**修改位置**:
- `syntax-check.yml`: 第 49 行
- `release.yml`: 第 36 行
- `beta-release.yml`: 第 36 行

### 自定义 Changelog 格式

如需修改 Changelog 格式，编辑以下文件：
- `release.yml`: 第 76-171 行
- `beta-release.yml`: 第 75-150 行

---

## 📊 监控和调试

### 查看 Workflow 运行状态
1. 访问仓库的 "Actions" 标签页
2. 选择对应的 Workflow
3. 查看运行日志和结果

### 常见问题

**Q: 为什么 Release 没有自动创建？**
A: 检查以下几点：
- Tag 格式是否正确（必须以 `v` 开头）
- GitHub Actions 是否启用
- GITHUB_TOKEN 权限是否足够

**Q: Changelog 为什么是空的？**
A: 可能原因：
- 这是第一个 Tag（没有前一个版本比较）
- Commit message 不符合 Conventional Commits 规范

**Q: 语法检查失败如何处理？**
A:
1. 本地运行 `bash -n PVE-Tools.sh` 检查语法
2. 安装 ShellCheck: `brew install shellcheck`
3. 运行 `shellcheck PVE-Tools.sh` 查看问题

---

## 🚀 未来改进

- [ ] 添加自动化测试
- [ ] 集成代码覆盖率检查
- [ ] 支持多语言 Changelog
- [ ] 自动生成 Release Notes 草稿
- [ ] 集成 Dependabot 依赖更新

---

## 📞 联系方式

如有问题或建议，请通过以下方式联系：
- [提交 Issue](https://github.com/Mapleawaa/PVE-Tools-9/issues/new)
- [发起 Discussion](https://github.com/Mapleawaa/PVE-Tools-9/discussions)
