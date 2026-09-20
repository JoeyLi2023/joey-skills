---
name: product-wiki
description: 维护飞书产品知识库（Karpathy 风格）。检测新增原始资料，自动生成人物摘要，动态归纳概念文章，更新索引。当用户运行 /product-wiki 时触发。
---

# product-wiki

你是一个知识库编译器。你的工作是把飞书里的原始资料（podcast 访谈、文章等）自动"编译"成结构化的 wiki——人物摘要 + 动态概念文章 + 索引。

**核心原则**：概念文章不是预设固定的，而是从内容里动态归纳出来的。每次运行都必须先读取当前概念列表，再判断新建还是更新。

---

## 工作流

### Step 1 · 读取知识库当前状态

读取 `references/kb-config.md`，获取所有节点 token 和工具命令。

然后执行：
1. 列出原始资料节点下所有文档（含标题和 obj_token）
2. 列出人物摘要节点下已有摘要（含标题）
3. 列出概念节点下已有概念文章（含标题和 obj_token）
4. 对比，找出**还没有对应摘要的新文档**

如果没有新文档，报告"没有新文档需要处理"，流程结束。

---

### Step 2 · 读取新文档内容

对每篇新文档，用 `lark-cli docs +fetch` 读取全文 markdown。

一次处理一篇，不要批量跳过。

---

### Step 3 · 生成人物摘要

读取 `references/summary-template.md`，按模板为每篇新文档生成摘要。

生成后，在人物摘要节点下创建新的 wiki 节点并写入内容：
```bash
lark-cli wiki nodes create --params '{"space_id":"{{FEISHU_SPACE_ID}}"}' \
  --data '{"node_type":"origin","obj_type":"docx","parent_node_token":"{{KB_PERSON_NODE_TOKEN}}","title":"<人物名>"}'
lark-cli docs +update --doc <obj_token> --mode overwrite --markdown @<file>
```

**读取 summary-template.md 后即可放下，不需要持续参考。**

---

### Step 4 · 动态归纳概念

这是最核心的步骤，读取 `references/concept-criteria.md` 作为判断依据（全程持续参考）。

对每篇新文档的内容，执行：

**4a. 识别本文档涉及的核心概念**（2-5个，不要贪多）

**4b. 逐个概念判断**：
- 用 Step 1 拿到的现有概念列表比对
- 如果找到匹配的现有概念 → 在该文章末尾 append 补充段落
- 如果是新概念且达到"单独成文"标准 → 在概念节点下新建文章

**4c. 新建概念文章的格式**：
```markdown
# 概念名称

> 来源：[[人物A]]、[[人物B]]

## 核心定义

## 主要视角/框架

## 跨文档共识

## 相关人物
```

---

### Step 5 · 更新索引 & 质检

在索引文档末尾 append 新增条目（人物 + 概念）。

然后读取 `references/checklist.md`，逐条核对，确认本次编译完整。

---

## 文件加载顺序

| 步骤 | 读取文件 | 处理方式 |
|------|---------|---------|
| Step 1 | `references/kb-config.md` | 读完即放 |
| Step 3 | `references/summary-template.md` | 读完即放 |
| Step 4 | `references/concept-criteria.md` | 全程持续参考 |
| Step 5 | `references/checklist.md` | 最后一次性使用 |
