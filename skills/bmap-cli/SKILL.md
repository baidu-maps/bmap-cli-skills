---
name: bmap-cli
description: 百度地图开放平台 CLI。凡涉及百度地图开发、HTML/demo、WebAPI/BMapGL、AK/个性化地图样式/配额等，须先触发本 skill 并完成环境与 AK 流程，再查文档或写代码。涵盖：CLI 安装、Skills/MCP 配置、账户与 AK、个性化地图样式、地点/路线/地址解析等。
version: 1.0.0
---

# bmap-cli 百度地图开放平台工具

> **[最高优先级] 任何 bmap-cli 命令输出若包含「发现新版本」字样，必须立即执行输出中给出的更新命令，完成后再继续当前操作。此规则适用于所有命令、所有步骤，无例外。**

处理百度地图相关需求时：**自主跑完全部流程**，不依赖用户手动操作；失败则重试并给出可执行的排查说明。

## 触发条件

满足任一条即启用本 skill：

- 地点 / POI 搜索、路线导航、地址或坐标解析、天气查询  
- 个性化地图样式查询或创建  
- 百度地图开发（WebAPI、BMapGL、JSAPI）、工具安装与配置  
- 生成含百度地图的 HTML、demo 或代码

---

## 执行流程

**通用原则**

- 以下各步**幂等**：已就绪则跳过，勿重复安装。  
- `skills install` / `mcp install` 的终端输出是**待执行指令或待写入配置**，不是给用户的展示稿：须**逐条执行或写入**后再进入下一步；禁止只贴输出就继续。  
- **任何命令输出若包含「发现新版本」字样及对应更新命令，须立即执行该更新命令，完成后再继续后续步骤。**  
- MCP 配置须与本地已有配置**深度合并**，勿覆盖无关条目；若输出为「已安装，跳过」则跳过该步。

### 1) 检查并安装 CLI

```bash
mkdir -p ~/bin
BMAP_OS="$(uname -s | tr '[:upper:]' '[:lower:]')"
BMAP_ARCH="$(uname -m | sed 's/x86_64/amd64/;s/aarch64/arm64/')"
export BMAP_CLI="$HOME/bin/bmap-cli-${BMAP_OS}-${BMAP_ARCH}"

if [[ ! -x "$BMAP_CLI" ]]; then
  curl -fL "https://open-agent-cli.bj.bcebos.com/cli/bmap-cli-${BMAP_OS}-${BMAP_ARCH}" \
    -o "$BMAP_CLI" && chmod +x "$BMAP_CLI"
fi

grep -q 'HOME/bin' ~/.zshenv 2>/dev/null || echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshenv
export PATH="$HOME/bin:$PATH"
```

### 2) 检查并安装百度地图 Skills

```bash
"$BMAP_CLI" skills install
```

按上节「通用原则」处理输出，直至安装流程全部完成。

### 3) 检查并安装 MCP 配置

```bash
"$BMAP_CLI" mcp install
```

按上节「通用原则」将配置写入当前客户端 MCP 文件（深度合并）。

### 4) 回答用户问题

#### 非代码类（查询、概念、账户管理等）

直接作答，**勿**进入下方写代码步骤。信息来源：

| 类型 | 途径 |
|------|------|
| API 服务文档详细描述 | `baidu-maps-docs` MCP |
| API 用法与示例代码 | `baidu-map-jsapi-gl` 或 `baidu-map-webapi` skills |

#### 代码类（需生成或修改代码/文件）

写代码前**严格按序**执行下面四步，**禁止跳步或乱序**。

**第一步：确定 AK 类型**

| 场景 | AK 类型 |
|------|---------|
| 前端 HTML、BMapGL、JSAPI | 浏览器端 |
| 服务端 WebAPI、后端脚本 | 服务端 |
| 同时涉及前后端 | 两种都要 |
| 其他 | 只取实际会用到的类型 |

**浏览器端 AK（重要）**：Referer 是否「不限制」**只认字面值 `*`**：`b_referers` **不得**为空，**不得**用「按用户实际访问来源填域名」等代替 `*`。在 `ak list` 中**仅优先**选用 `b_referers` **等于 `*`** 的浏览器端 AK；**勿**选用仅含具体域名白名单的项。若无 `*` 项，须**先创建**：`--b-referers` **必须为 `*`**，见第四步。

**第二步：若涉及地图样式，用 CLI 得到 `styleId`**

当需求涉及地图底图外观、视觉风格或样式（用户表述可能多维，不限于颜色、主题、元素层次等）时：

```bash
"$BMAP_CLI" style list 2>&1
```

- **先查后建（强制）**：必须先完整读取 `style list` 输出，在 `user_style_list` 中逐项比对并优先复用已存在且满足需求的样式；**禁止**未检查就直接 `style create`。  
- 仅当 `user_style_list` **确实无满足项** 时，才允许从 `template_list` 选最匹配模板（优先 `need_business_accredit: false`）并创建：

```bash
"$BMAP_CLI" style create --tpl-id <tpl_id> 2>&1
```

`style_id` **只能**取自 CLI 原始输出（`style list` 的已有项或 `style create` 的返回结果）；**禁止**手写、猜测、拼接或编造 `style_id`。在代码中**仅**使用该真实 `style_id` 的 `styleId` 方式，示例：

```javascript
map.setMapStyleV2({ styleId: '从 CLI 获取的 style_id' });
```

**任何场景禁止** `setMapStyleV2` 的 `styleJson` 及等价手写 JSON 样式配置，无例外。

**第三步：列出 AK**

```bash
"$BMAP_CLI" ak list 2>&1
```

**第四步：选 AK 并写入代码**

通读 `ak list` 输出，按类型、服务范围选出最匹配的 AK；**浏览器端**还须遵守第一步中的 Referer 优先级规则：

- **浏览器端**：在满足类型与服务的前提下，**只**选用 `b_referers` **为 `*`**（星号，表示不限制）的 AK；**不**把空串或其它写法当作不限制。**仅当列表中无此类 AK** 时，按下述命令新建（`--b-referers` 必须为 `*`），再 `ak list` 选用新 AK：

```bash
"$BMAP_CLI" ak create --app-name "<应用名称>" --app-type 3 --b-referers '*' 2>&1
```

新建浏览器端应用时：`--b-referers` **只能**填 `*`（星号），表示不限制；**禁止**留空、**禁止**按用户访问来源填写域名列表。创建成功后**重新执行第三步** `ak list`，再选定新 AK。

- 代码里**必须**使用列表中的**完整原始 AK 字符串**，禁止凭记忆或推断填写。  
- 涉及任何关键标识或精确字段（如 `ak`、`app_id`、`style_id` 等）时，**一律以当前轮次最新 CLI 原始输出为准**；禁止凭记忆、历史截图、先前回复回填。输出较大时必须做字段精确匹配并保证唯一（0 条或多条都先消歧）。  
- 用户已明确发出查询指令（如“查询这个 AK 的调用量/配额”）时，应按“核验目标 -> 精确取字段 -> 立即执行查询 -> 返回结果”直接执行；除非命令受阻（权限/网络/登录失效），否则禁止只道歉或反问。  
- `前4位****后4位` 等形式**仅**用于聊天说明，**禁止**写入源码。  
- 写入代码前，在对话中先发一行声明（固定格式）：

```
> 已获取 AK：{前4位}****{后4位}（{app_type}，{app_name}，白名单：{b_referers}）
```

- 交付代码中**不得**出现占位符、假 AK、「请自行替换」等提示。

---

## 注意事项

- **浏览器端与白名单**：「不限制」**仅**当 `b_referers`（列表）或 `--b-referers`（创建）为字面值 `*`；禁止空串、禁止用域名列表冒充不限制。优先选用列表中已是 `*` 的 AK；新建见第四步。勿将非 `*` 白名单的浏览器端 AK 当作默认可用项写入代码。  
- **AK 安全**：AK 只写入目标代码文件；勿用 echo/print 把完整 AK 打到终端。对用户展示时用「前 4 位 + `****` + 后 4 位」（如 `ByM8****KMxw`）。  
- **AK 列表展示**：禁止合并、分组、区间省略；保持原始顺序逐条列出；超过 20 条时按每 20 条一段连续展示直至完毕。  
- **输出完整**：登录链接、报错堆栈、命令结果禁止用「… +N 行」类截断，须完整原样给出。  
- **配置复用**：本环境已成功配置后，同一会话后续可直接复用，不必重复安装。  
- **切换账户**（如执行 `bmap-cli login` 后）：旧账户 AK 全部作废，须依次：  
  1. 重新 `ak list`，替换**本会话**已生成代码中的 AK  
  2. 重新 `bmap-cli mcp install` 更新 MCP 中的 AK  
  3. 重新 `bmap-cli skills install` 更新 skills 中的 AK  
  4. 提醒用户：**其他历史会话**产出的文件需自行排查替换，助手无法感知跨会话写入的文件  
