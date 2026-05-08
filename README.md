# bmap-cli Skills

bmap-cli 百度地图开放平台 AI 助手 Skills，供 Claude、Cursor 等在使用百度地图相关 API 时加载，以提供准确的 API 说明、AK 管理流程与代码示例。

## 包含的 Skill

| Skill | 说明 |
|-------|------|
| bmap-cli | 百度地图开放平台 CLI 工具入口。凡涉及百度地图开发、生成地图相关 HTML/代码/demo、调用地图 API，必须第一时间触发此 Skill，先完成 CLI 安装、Skills 注册、MCP 配置与 AK 获取流程。支持：自动安装配置、用户账户管理、AK 管理、个性化地图样式管理、API 消费量/配额查询、地点搜索、路线导航、地址解析等。 |

## 安装方式

任选以下一种方式安装即可。

### 方式一：npx skills add（推荐）

若你的环境支持 skills CLI，可用一条命令安装本仓库的 skills：

```bash
npx skills add baidu-maps/bmap-cli-skills
```

会将本仓库中的全部 skills 安装到当前环境的 skills 目录。

### 方式二：手动安装

**第一步：获取仓库**

克隆本仓库：

```bash
git clone https://github.com/baidu-maps/bmap-cli-skills.git
cd bmap-cli-skills
```

或从 [Releases](https://github.com/baidu-maps/bmap-cli-skills/releases) 下载附件 `bmap-cli-skills.zip` 后解压：

```bash
unzip bmap-cli-skills.zip
```

**第二步：注册 Skill**

将 `skills/` 目录下的 `bmap-cli` 链接或复制到当前环境对应的 skills 目录。

**Claude Desktop（本地）**

Skills 目录一般为：`~/.claude/skills/`

注册（软链，推荐）：

```bash
ln -sfn "$(pwd)/skills/bmap-cli" ~/.claude/skills/bmap-cli
```

或直接把 `skills/bmap-cli` 文件夹复制到 `~/.claude/skills/` 下。

**Cursor**

Skills 目录一般为：`~/.cursor/skills/`

注册（软链，推荐）：

```bash
ln -sfn "$(pwd)/skills/bmap-cli" ~/.cursor/skills/bmap-cli
```

或直接把 `skills/bmap-cli` 文件夹复制到 `~/.cursor/skills/` 下。

## 如何使用

在支持 Skills 的客户端里，当你的问题涉及「百度地图」「BMapGL」「JSAPI」「WebAPI」「POI 搜索」「路线规划」「地址解析」「天气」「地图样式」等时，助手会优先参考本仓库中对应 skill 的文档来回答，从而给出更贴合百度地图开放平台的代码与用法。

## 仓库结构

```text
.
├── skills/
│   └── bmap-cli/                # 百度地图 CLI 入口 Skill
│       ├── SKILL.md             # Skill 入口与索引
└── README.md
```

`SKILL.md` 中会列出该 Skill 下所有参考文档，便于 AI 按需读取。

## 许可

MIT
