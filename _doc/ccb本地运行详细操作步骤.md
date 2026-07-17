# ccb 本地运行详细操作步骤

## 背景

`claude-code-best` 已发布到 npm（`npm i -g claude-code-best`），全局安装后通过 `ccb` 命令使用。如果你需要在本地修改代码并直接使用本地版本（而非 npm 上的发布版本），可以通过 `npm link` 将全局命令指向本地构建产物。

## 前置条件

- Node.js >= 18（推荐使用 nvm 管理）
- Bun >= 1.3.0（构建运行时），注意bun的版本，必须大于1.3.11版本
- 已 clone 项目到本地

## 第一步：安装依赖

```bash
cd /path/to/claude-code
bun install
```

## 第二步：构建项目

```bash
bun run build:vite
```

该命令分两步执行：
1. `vite build` — 代码分割构建，产物输出到 `dist/` 和 `dist/chunks/`
2. `scripts/post-build.ts` — 对 `dist/` 下所有 `.js` 文件做 `globalThis.Bun` 解构 patch，并复制 vendor 文件到 `dist/vendor/`

关键产物：
- `dist/cli-node.js` — `ccb` 命令的实际入口（Node.js 兼容）
- `dist/cli-bun.js` — `ccb-bun` 命令的入口（Bun 直接运行）

另有一个备选构建方式，通常不需要使用：

```bash
bun run build        # 使用 Bun.build() 构建，产物为 dist/cli.js
```

## 第三步：将全局命令链接到本地构建

```bash
# 在项目根目录执行
npm link
```

这一步会：
1. 将全局 `node_modules/claude-code-best` 替换为指向本地项目目录的符号链接
2. 更新 `ccb`、`ccb-bun`、`claude-code-best` 三个 bin 命令，使其指向本地 `dist/` 下的文件

## 第四步：验证

```bash
# 确认 ccb 指向本地构建
which ccb                    # 应输出 npm 全局 bin 目录下的 ccb
readlink $(which ccb)        # 应输出 /path/to/claude-code/dist/cli-node.js

# 确认能正常启动
ccb --version                # 输出版本号即为成功
```

## 日常开发流程

每次修改代码后：

```bash
bun run build:vite           # 重新构建
ccb                          # 直接使用最新构建产物
```

如果只是想跑类型检查 + lint + 测试：

```bash
bun run precheck
```

## 恢复为 npm 版本

如果后续想切换回 npm 发布的正式版本：

```bash
# 1. 先清除本地链接
npm unlink -g claude-code-best

# 2. 重新从 npm 安装
npm install -g claude-code-best

# 3. 验证
which ccb && readlink $(which ccb)
# 应输出 -> ../lib/node_modules/claude-code-best/dist/cli-node.js
```

## 注意事项

- `npm link` 创建的是符号链接，不会复制文件；删除项目目录会导致 `ccb` 不可用
- 如果使用 nvm 切换 Node.js 版本，`npm link` 需要在新版本下重新执行
- 开发模式下也可直接运行：`bun run dev`（不经过构建，直接执行 TSX 源码），但这种方式不走 `ccb` 命令
