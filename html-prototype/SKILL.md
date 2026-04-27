---
name: html-prototype
description: >
  生成单文件交互式 HTML 原型图。当用户提到「原型」「原型图」「prototype」「做个页面看看」「交互稿」「demo 页面」，
  或者要求把 PRD / 设计方案 / 线框图转成可交互的页面时，使用此 Skill。
  也适用于用户发来截图或设计图并说「照这个做一个」「还原一下」的场景。
  即使用户没有明确说"HTML"，只要意图是生成可点击、可交互的页面原型，都应触发。
---

# HTML 交互原型生成

## 你要做什么

生成**单个 HTML 文件**，包含内联 CSS 和 JS，用户可以直接在浏览器中打开并交互。
这不是真正的前端开发——不用框架、不用打包工具、不连后端。目标是**快速产出可点击的高保真原型**，帮助用户验证交互逻辑和页面结构。

## 技术栈

- 纯 HTML + 内联 CSS + 内联 vanilla JS
- 不使用任何框架（React/Vue/Angular 等）
- 不使用外部 CDN 依赖（所有样式和逻辑自包含）
- 唯一例外：如果用户明确要求使用某个库（如 Chart.js），可以引入 CDN

---

## 设计原则

### 1. 信息层级决定视觉权重

每个页面的内容必须有明确的优先级。视觉权重匹配信息重要程度：

| 层级 | 视觉表现 | 例子 |
|------|---------|------|
| 主要 | 最大面积、最少装饰、最显眼位置 | 核心输入框、主内容区 |
| 次要 | 始终可见但紧凑，不抢主角 | 图片附件横排、辅助信息 |
| 可选 | 默认隐藏，按需展开 | 高级设置、补充说明 |

**反模式**：所有元素线性堆叠、视觉重量相同、没有主次之分。

### 2. 渐进展示（Progressive Disclosure）

不要一次性把所有选项摊开：
- 高级选项收在**工具栏按钮**后面
- 配置项放在 **Popover** 中
- 详情用**折叠面板**展开
- 在入口处用**状态摘要**暗示内容存在（如 chip 显示「8 个问题 · 含原型」）

### 3. Composer 模式优于表单模式

当页面核心是「文字输入 + 若干辅助项」时，用 Composer 模式：

```
┌─────────────────────────────────┐
│  大文本输入区（无边框，融入卡片）    │
│                                 │
├─────────────────────────────────┤
│  次要输入（紧凑横排）              │
├─────────────────────────────────┤
│ [附加操作] | [设置]   [摘要] [提交]│
└─────────────────────────────────┘
```

而不是：
```
标题
[输入框]
标题
[输入框]
[提交按钮]
```

### 4. 紧凑优于铺张

次要元素用紧凑表达：
- 图片上传：56×56px 缩略图横排 + 小型添加按钮，不用大拖拽区域
- 配置项：Popover 里的 toggle/数字输入，不占满整行
- 状态提示：小 chip，不用独立段落

---

## 视觉风格

### 浅色主题（默认）

使用 CSS 变量统一管理颜色，方便全局切换：

```css
:root {
    --bg-primary: #f0f2f7;      /* 页面背景 */
    --bg-secondary: #ffffff;     /* 卡片/面板背景 */
    --bg-tertiary: #f8fafc;      /* 嵌套区域/工具栏背景 */
    --text-primary: #111827;     /* 主文字 */
    --text-secondary: #6b7280;   /* 次要文字 */
    --text-tertiary: #9ca3af;    /* 占位符/禁用文字 */
    --border: #e5e7eb;           /* 边框 */
    --border-light: #f3f4f6;     /* 轻边框/分割线 */
    --primary: #7c3aed;          /* 主色调/品牌色 */
    --primary-light: #f5f3ff;    /* 主色调浅底 */
    --primary-dark: #6d28d9;     /* 主色调深色（hover） */
    --success: #10b981;
    --warning: #f59e0b;
    --danger: #ef4444;
    --danger-light: #fef2f2;
}
```

如果用户要求深色主题，替换变量值即可，组件代码不变。
如果用户指定了品牌色，替换 `--primary` 系列变量。

### 布局规范

- 圆角：16px（大卡片）、12px（Popover/弹窗）、8px（按钮/输入框）、6px（小标签/chip）
- 阴影：`0 1px 3px rgba(0,0,0,0.06)`（卡片）、`0 4px 20px rgba(0,0,0,0.12)`（弹窗/Popover）
- 间距：8px 基数系统（4/8/10/12/16/20/24/32）
- 字体：`-apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif`
- 字号层级：24px（页面标题）、15px（正文/输入）、13px（按钮/标签）、11px（chip/提示）、9px（微型标注）

---

## 交互模式

### 工具栏（Toolbar）

底部工具栏是 Composer 模式的核心交互区：

```html
<div class="toolbar">
    <!-- 左侧：操作按钮 -->
    <div class="toolbar-left">
        <button class="toolbar-btn" id="toggle-extra">
            <svg>...</svg> 补充说明
        </button>
        <div class="toolbar-divider"></div>
        <div class="relative">
            <button class="toolbar-btn" id="toggle-settings">
                <svg>...</svg> 设置
            </button>
            <!-- Popover 从按钮上方弹出 -->
            <div class="popover" id="settings-popover">...</div>
        </div>
    </div>
    <!-- 右侧：状态 + 提交 -->
    <div class="toolbar-right">
        <span class="chip">● 8 个问题 · 含原型</span>
        <button class="submit-btn">✦ 提交</button>
    </div>
</div>
```

样式要点：
- 背景用 `--bg-tertiary`，上方加 1px `--border-light` 分隔线
- 按钮默认 `--text-secondary`，hover 变 `--text-primary` + 浅灰底
- 激活态 `--primary` 文字 + `--primary-light` 底色
- Popover 从按钮上方弹出（`bottom: calc(100% + 8px)`），点击外部关闭

### Toggle 开关

```css
.toggle {
    position: relative;
    width: 36px; height: 20px;
    background: var(--border);
    border-radius: 10px;
    cursor: pointer;
    transition: background 0.2s;
}
.toggle.on { background: var(--primary); }
.toggle::after {
    content: '';
    position: absolute;
    top: 2px; left: 2px;
    width: 16px; height: 16px;
    background: white;
    border-radius: 50%;
    transition: transform 0.2s;
    box-shadow: 0 1px 2px rgba(0,0,0,0.15);
}
.toggle.on::after { transform: translateX(16px); }
```

### 紧凑图片上传区

```html
<div class="image-row">
    <!-- 已上传的缩略图 -->
    <div class="image-thumb">
        <img src="..." />
        <button class="remove-btn">×</button>  <!-- 右上角，hover 显示 -->
    </div>
    <!-- 添加按钮 -->
    <div class="image-add-btn">
        <svg>...</svg>
        <span>附图</span>
    </div>
    <span class="hint">可选，最多 3 张</span>
</div>
```

缩略图 56×56px，`border-radius: 12px`。删除按钮绝对定位在右上角 `-4px`，父容器**不能用 `overflow: hidden`**（否则按钮会被裁剪）。

### 可展开区域

用 `max-height` + `transition` 实现平滑展开/收起：

```css
.expandable {
    max-height: 0;
    overflow: hidden;
    transition: max-height 0.25s ease, padding 0.25s ease;
}
.expandable.visible {
    max-height: 200px;  /* 足够大的值 */
    padding-bottom: 16px;
}
```

### Tab 切换
```javascript
document.querySelectorAll('[data-tab]').forEach(tab => {
    tab.addEventListener('click', () => {
        // 切换 active 状态
        // 显示对应内容面板
    });
});
```

### 弹窗 / Modal
```javascript
function openModal(modalId) {
    document.getElementById(modalId).style.display = 'flex';
}
function closeModal(modalId) {
    document.getElementById(modalId).style.display = 'none';
}
modal.addEventListener('click', (e) => {
    if (e.target === modal) closeModal(modalId);
});
```

### Toast 提示
```javascript
function showToast(message, type = 'success') {
    // 固定在顶部居中，2 秒后自动消失，带滑出动画
}
```

### 表单验证
- 必填字段为空 → 红色边框 + 提示文字
- 验证通过 → Toast 提示"操作成功"

### 数据展示
- 表格数据硬编码在 HTML 中
- 使用真实感的中文数据（人名、日期、金额），不用 "测试数据1"
- 统计数字千分位格式化

---

## 多方案对比原型

当用户需要对比多个设计方案时：

```html
<nav class="scheme-nav">
    <button class="scheme-tab active" data-scheme="a">方案 A <span>聚焦输入</span></button>
    <button class="scheme-tab" data-scheme="b">方案 B <span>双栏布局</span></button>
</nav>
<div class="scheme-container active" id="scheme-a">...</div>
<div class="scheme-container" id="scheme-b">...</div>
```

- 顶部 Tab 切换不同方案
- 每个方案完全独立，可交互
- 每个方案一句话定位标注在 Tab 旁

---

## 常见布局模式

### Composer 卡片

适合：AI 对话、需求描述、文章编辑——以文本输入为核心的页面。

```
Card (overflow: hidden, border-radius: 16px)
├── 主文本区 (padding: 24px, textarea 无边框无背景)
├── 附件横排 (padding: 8px 24px, 缩略图 + 添加按钮)
├── 可展开区域 (max-height 动画，默认隐藏)
└── 工具栏 (border-top, bg-tertiary, 左操作右提交)
```

### 侧边栏 + 主内容区

适合：管理后台类原型。

```
Page
├── Sidebar (width: 240px, fixed, bg-secondary)
│   ├── Logo
│   ├── Nav items
│   └── User info
└── Main (margin-left: 240px, padding: 32px)
    ├── Page header
    ├── Stats cards (横排)
    └── Content (表格/列表/表单)
```

### 双栏布局

适合：配置丰富、需同时可见的场景。

```
Grid (1fr 320px, gap: 20px)
├── Left: 主内容输入区
└── Right (sticky): 配置面板 + 提交按钮
```

---

## 常见组件

### 状态 Chip
```html
<span class="chip">
    <span class="chip-dot"></span>
    8 个问题 · 含原型
</span>
```
小圆点 + 文字，`border-radius: 20px`，`font-size: 11px`。

### 数字步进器
```html
<div class="stepper">
    <button>−</button>
    <span>8</span>
    <button>+</button>
</div>
```

### 统计卡片
横向排列，每个包含：标签 + 大数字 + 可选趋势/副标题。

### 搜索 + 筛选栏
左侧搜索框 + 右侧筛选下拉/按钮组。

### 数据表格
表头固定、行 hover 高亮、操作列右对齐、支持全选 checkbox。

---

## 语言

- UI 文案全部使用**中文**
- 代码注释中英文均可
- 按钮、标签、标题、占位符、提示信息一律中文

## 工作流程

1. **理解需求**：读取用户提供的 PRD / 截图 / 描述，理清页面结构和交互逻辑
2. **明确层级**：确定哪些是主要内容、次要内容、可选内容，决定它们的视觉表达方式
3. **一次性输出**：生成完整的单个 HTML 文件，不要分步输出
4. **命名规范**：文件名使用中文或有意义的名称，如 `积分管理原型.html`
5. **迭代修改**：用户反馈后，用 Edit 工具局部修改，不要每次重写整个文件

## 注意事项

- 单文件体积控制在合理范围，超过 2000 行时考虑精简
- 所有样式内联在 `<style>` 标签中，所有脚本内联在 `<script>` 标签中
- 不要留 TODO 或占位符——用户打开就是完整可用的原型
- 如果原型有多个页面，用 Tab / 导航切换模拟，不拆成多个文件
- 交互重于视觉——确保点击、切换、弹窗等交互都能正常工作
- 绝对定位元素（如删除按钮）不要被父容器 `overflow: hidden` 裁剪
- 无边框输入框要加 `background: transparent` 避免默认背景色露出
