---
name: zotero-polish
description: 当用户要求按其 zotero 写作笔记／note／写作 note 润色或改写英文段落时使用。触发短语包括"按我的 note 改"、"用 note 润色"、"参考 zotero 写作笔记"、"polish with my notes"等。本技能仅以 Zotero 写作笔记 collection 下的 note 为依据，不引入其他文献或通用润色经验。
---

# zotero-polish

按用户的 Zotero 写作笔记 collection 中的笔记规范，润色英文学术段落。**仅以该 collection 下的 note 内容为唯一依据**，不得参考其他文献或通用学术英语经验。

> 配置说明：下面的 `<YOUR_ZOTERO_USER_ID>`、`<YOUR_COLLECTION_KEY>` 和各 `<NOTE_KEY_*>` 为占位符，使用前请替换为你自己的 Zotero User ID、collection key 与 note key（见 README 的 Setup 部分）。

## 适用触发

用户消息中同时出现：
1. 要求润色／改写英文（"润色"、"polish"、"改一下英文"、"refine"）；
2. 明确点名 zotero／note／写作笔记。

若用户未点名 note，**不要触发本技能**。

## 资源与凭证

- Zotero User ID：`<YOUR_ZOTERO_USER_ID>`
- API Key：从环境变量 `zotero_api_key` 读取（PowerShell 用 `$env:zotero_api_key`，Bash 用 `$zotero_api_key`）
- Base URL：`https://api.zotero.org/users/<YOUR_ZOTERO_USER_ID>`
- 写作笔记 Collection key：`<YOUR_COLLECTION_KEY>`

### note 索引

按论文章节组织的 note（key 替换为你自己的）：

| Note Key | 标题 | 适用段落类型 |
|---|---|---|
| `<NOTE_KEY_ABSTRACT>` | 00 摘要与标题 | abstract、title |
| `<NOTE_KEY_INTRO>` | 01 引言和综述 | introduction、related work、literature review |
| `<NOTE_KEY_METHODS>` | 02 方法 | methods、methodology、experimental setup |
| `<NOTE_KEY_RESULTS>` | 03 描述结果 | results、findings、experiments 描述部分 |
| `<NOTE_KEY_DISCUSSION>` | 04 讨论 | discussion、implications、limitations |
| `<NOTE_KEY_GENERAL>` | 05 特定表达和其他 | 通用句式、过渡、连接词、术语对照 |

## 工作流程

### 第 1 步：识别段落归属

根据用户给的英文段落内容，从上表 note 中选 **1 条主 note**：
- abstract（高度概括、含数字、无引用）→ `<NOTE_KEY_ABSTRACT>`
- introduction / related work（含 "Recently"、文献引用密集）→ `<NOTE_KEY_INTRO>`
- methods（描述模型、数据、算法、参数）→ `<NOTE_KEY_METHODS>`
- results（实验数字、表格描述、对比）→ `<NOTE_KEY_RESULTS>`
- discussion（"suggest that"、"imply"、limitations）→ `<NOTE_KEY_DISCUSSION>`
- 难以归类、或主要问题是术语/连接句 → `<NOTE_KEY_GENERAL>`

**始终额外读取 `<NOTE_KEY_GENERAL>`**（"05 特定表达和其他"），因为它含通用术语与排版规范，对所有章节适用。

### 第 2 步：通过 Web API 拉取 note 正文

用 Bash 工具调 curl（Windows 下 Git Bash 可直接读环境变量 `$zotero_api_key`）：

```bash
curl -s -H "Zotero-API-Key: $zotero_api_key" \
  "https://api.zotero.org/users/<YOUR_ZOTERO_USER_ID>/items/<NOTE_KEY>" \
  | python -c "import sys,json,html,re; d=json.load(sys.stdin)['data']['note']; t=re.sub(r'<[^>]+>','',html.unescape(d)); print(t)"
```

把 `<NOTE_KEY>` 换成上一步选定的 key。该命令会：
1. 拉取 JSON
2. 取 `data.note` 字段（含 HTML 的 note 正文）
3. 解 HTML 实体并剥除标签，输出为纯文本

如 Python 不可用，可改用：

```bash
curl -s -H "Zotero-API-Key: $zotero_api_key" \
  "https://api.zotero.org/users/<YOUR_ZOTERO_USER_ID>/items/<NOTE_KEY>" \
  | jq -r '.data.note' \
  | sed -e 's/<[^>]*>//g' -e 's/&nbsp;/ /g' -e 's/&amp;/\&/g' -e 's/&lt;/</g' -e 's/&gt;/>/g'
```

如需保留 HTML 结构（如列表、加粗）供模型更好地解析，可省略剥标签那一步，直接将 `data.note` 的 HTML 内容传入对话。

### 第 3 步：按 note 规则润色

- 允许结构性改写：句序调整、合并/拆分句子、按 note 中给出的范式重组段落，前提是保持原意。
- 用词、句式、过渡完全依照 note 中给出的样例与建议；note 未涉及的部分保留原文。
- 如 note 中出现"避免 X，改用 Y"型明确指引，必须遵守。
- `<NOTE_KEY_GENERAL>` 中的术语中英对照、排版规范（如引用只保留姓、避免缩写）对所有段落均生效。

### 第 4 步：输出格式（三段式，中英双语）

````
**润色后段落（English）**

<polished paragraph>

**修改说明（中文）**

- 句 1：将「...」改为「...」——依据 note《<标题>》中关于「...」的规则。
- 句 2：合并原 X、Y 句——note 指出 ...。
- ...（每条修改都必须援引 note 的具体段落或规则）

**参考的 note**

- 《<标题>》（key: `<KEY>`）
- 《05 特定表达和其他》（key: `<NOTE_KEY_GENERAL>`）
````

**硬性要求**：每条修改说明必须指向 note 中的具体规则或示例。**若某处修改在所引 note 中找不到依据，禁止做出该修改**（保留原文）。

## 注意事项

1. note 内容代表用户个人风格偏好，可能与通用学术英语建议冲突，**一律以 note 为准**。
2. 不要混入 `humanizer`／`research-paper-writing` 等其他技能的建议。
3. note 多用中文记录英文写作经验。读取后无论语言，按其规则改英文段落即可。
4. 若 note 中规则与段落场景明显不符，如实说明「note 中未覆盖此场景」，仅做不冲突的最小润色。
5. 若 `$zotero_api_key` 未设置或 API 返回 403，请提示用户检查环境变量或重新生成 key。

## 故障排查

| 现象 | 处理 |
|---|---|
| curl 返回空 | 检查 `echo $zotero_api_key` 是否非空；Windows 重启 shell 才能读到新环境变量 |
| HTTP 403 | API key 失效或权限不足，登录 https://www.zotero.org/settings/security 重新生成 |
| HTTP 404 | note key 错误；核对你 collection 下的 note key |
| 返回 HTML 含乱码 | 终端编码问题，加 `--output -` 或在 PowerShell 中用 `Invoke-RestMethod` 替代 |
