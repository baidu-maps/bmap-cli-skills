---
name: bmap-cli
description: 百度地图开放平台 CLI 工具。凡涉及百度地图开发、生成地图相关 HTML/代码/demo、调用地图 API，必须第一时间触发此 skill，在做任何其他操作（查文档、写代码）之前先完成 AK 获取流程。支持：自动安装配置、用户账户管理、AK 管理、个性化地图样式管理、API 消费量/配额查询、地点搜索、路线导航、地址解析等。
version: 1.0.0
---

# bmap-cli 百度地图开放平台工具

当用户提出百度地图相关需求时，**全程自主执行以下流程，不等待用户手动操作，失败时自行重试并给出可执行的排查建议**。

## 触发条件

出现以下任一意图即触发：地点/POI 搜索、路线导航、地址或坐标解析、天气查询、个性化地图样式查询或创建、百度地图开发（WebAPI/BMapGL）、地图工具安装配置、生成含百度地图的 HTML/demo/代码。

---

## 执行流程

> 各步骤均为幂等操作：已就绪则直接跳过，不重复执行。

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

**命令输出的内容是安装指令，不是展示内容。必须立即逐条执行输出中的每一个操作步骤，直到全部完成。不得跳过、不得仅展示输出后继续下一步。**

### 3) 检查并安装 MCP 配置

```bash
"$BMAP_CLI" mcp install
```

**命令输出中包含需要写入当前客户端 MCP 配置文件的配置内容。必须立即将配置写入（与已有配置深度合并，不覆盖其他条目），写入完成后再继续下一步。若输出显示"已安装，跳过"则直接跳过。**

### 4) 回答用户原始问题

**若不需要生成代码**（查询信息、问概念、管理账户等），跳过以下步骤直接回答。信息获取途径：
- 文档/概念类 → 通过 `baidu-maps-docs` MCP 查询官方文档
- API 用法/代码类 → 通过 `baidu-map-jsapi-gl` 或 `baidu-map-webapi` skills 获取准确用法
- 知识库/SOP/QA 类 → 通过 `knowledge-rag` MCP 检索知识文档

**若需要生成代码/文件，写代码前按以下顺序逐步执行，不得跳过、不得乱序：**

**第一步：确定所需 AK 类型**

- 前端 HTML / BMapGL / JSAPI → **浏览器端** AK
- 服务端 WebAPI / 后端脚本 → **服务端** AK
- 两者都有 → 两种 AK 均需获取
- 只获取实际用到的类型，不多取

**第二步：若需求涉及地图样式，查询并确定 styleId**

需求中出现"深色地图"、"道路高亮"、"个性化风格"等样式描述时，执行：

```bash
"$BMAP_CLI" style list 2>&1
```

- `user_style_list` 中有合适样式 → 直接取其 `style_id`
- 无合适样式 → 从 `template_list` 选最匹配的（优先 `need_business_accredit: false`），执行创建：

```bash
"$BMAP_CLI" style create --tpl-id <tpl_id> 2>&1
```

取返回的 `style_id`，代码中**必须**使用 `styleId` 方式，禁止直接手写 `styleJson` 绕过此步骤：

```javascript
map.setMapStyleV2({ styleId: '从 CLI 获取的 style_id' });
```

> 例外：需求明确要求完全自定义样式规则时，才允许使用 `styleJson`。

**第三步：获取 AK 列表**

```bash
"$BMAP_CLI" ak list 2>&1
```

**第四步：选取 AK 并写入代码**

阅读 `ak list` 完整输出，结合需求选出最匹配的 AK（类型、服务范围、白名单均需匹配）。

- **必须使用 `ak list` 输出的原始字符串**，禁止凭记忆填写或推断
- 遮掩格式（`前4位****后4位`）**仅用于聊天展示**，绝不写入代码
- 写入代码前，先在聊天中输出声明行（强制格式）：

```
> 已获取 AK：{前4位}****{后4位}（{app_type}，{app_name}，白名单：{b_referers}）
```

- 代码中**不得出现任何占位符、输入框或"请替换"类提示**

---

## 注意事项

- **AK 安全**：AK 仅写入代码文件，不得 echo/print 到终端输出；向用户展示时遮掩为前 4 位 + `****` + 后 4 位（示例：`ByM8****KMxw`）。
- **AK 列表展示**：禁止区间汇总、分组合并或省略；按原始顺序逐条展示，条目多时每 20 条一段连续展示直到全部输出完毕。
- **输出完整性**：登录授权链接、报错堆栈、命令执行结果禁止以"... +N lines"方式省略，必须完整原样展示。
- **配置复用**：环境配置成功一次后，后续对话直接复用，无需重复安装。
- **切换账户后必须重新获取 AK**：执行 `bmap-cli login` 切换账户后，旧账户的 AK 全部失效，必须按以下步骤处理：
  1. 重新执行 `ak list` 获取新账户 AK，替换当前会话生成的所有代码文件中的旧 AK
  2. 重新执行 `bmap-cli mcp install` 更新 MCP 配置中的 AK
  3. 重新执行 `bmap-cli skills install` 更新 skills 配置中的 AK
  4. 提示用户：历史会话生成的代码文件需手动排查替换，无法感知跨会话写入的文件