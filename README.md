# 图表工具（SVG，多格式兼容，自动插入正文）

> **作者 / Author**：JiangNanGenius  
> **Github**：https://github.com/JiangNanGenius
> **版本 / Version**：1.0.0  
> **许可证 / License**：MIT  
> **适配 Open WebUI / Required Open WebUI Version**：0.6.3+

这是一个面向 **Open WebUI** 的图表渲染工具：把你给的图表配置（尽量兼容 ECharts / Highcharts / Apex / Chart.js 的常见写法）渲染为 **SVG**，上传到 Open WebUI 文件存储，并生成可直接在聊天中显示的 Markdown 引用。

**关键提醒（非常重要）**：

- 仅“生成/上传”图表文件 **不会** 自动显示在聊天正文。
- 必须在正文中引用 Markdown 图片语法：`![...](...)` 才会渲染成图片。
- 本工具默认会自动发送一条 `message` 事件，把引用片段插入正文，避免出现“能下载但不显示”的情况。

---

## 功能特性

- ✅ 多种图表输出为 **SVG**（清晰可缩放，便于报告/文档）
- ✅ 自动上传至 Open WebUI 文件存储，返回相对 URL
- ✅ 默认自动插入 Markdown 引用到聊天正文（强兜底）
- ✅ 多格式兼容：尽量接受 ECharts / Highcharts / Apex / Chart.js 的常见配置风格
- ✅ JSON 容错：支持 `spec_json` 为对象或字符串；可自动处理 “Extra data/尾随字符” 等问题（可关闭）
- ✅ 支持两种嵌入方式：URL（短） / Data URI（最稳但很长）

---

## 支持的图表类型（type）

- `line` / `area`
- `bar`
- `pie` / `donut`
- `candlestick`（也支持别名 `kline`）
- `scatter` / `bubble`
- `radar`
- `heatmap`
- `histogram`
- `boxplot`
- `gauge`
- `funnel`
- `waterfall`

---

## 兼容的配置风格

本工具不追求 100% 复刻某个图表库，而是尽可能“吃下”常见字段，让你少改配置、少重试。

### 1) ECharts 风格（常见字段）

- `xAxis.data`
- `series[].type`
- `series[].data`

### 2) Highcharts / Apex 风格（常见字段）

- `xAxis.categories`
- `yAxis.title.text`
- `series[].data`
- Apex 也常见：`chart.type`、`labels + series`

### 3) Chart.js 风格（常见字段）

- `data.labels`
- `data.datasets[]`
- scatter/bubble 支持点对象：`{x, y}`、`{x, y, r}`

---

## 工作流程（你调用一次工具，它做了什么）

1. 解析 `spec_json`（可传 **JSON object** 或 **JSON 字符串**，也支持 ```json 代码块）
2. 自动标准化配置（补全/推断 `type`、整合 `title`）
3. 根据 `type` 渲染 SVG
4. 上传 SVG 到 Open WebUI 文件存储，获得相对 URL
5. 生成 Markdown 引用片段：

   ```md
   ![chart title](/api/v1/files/xxx/content?t=...)

   [打开/下载SVG](/api/v1/files/xxx/content?t=...)
   ```

6. **默认**额外发送一条 `message` 事件，把上面的引用片段插入聊天正文（避免忘记引用）

---

## 安装与启用（Open WebUI）

> 由于不同部署方式（Docker / 本地 / 反代）目录结构可能不同，这里只给出通用做法。

常见方式通常包括：

- **方式 A：在 Open WebUI 的工具管理界面添加自定义工具**
  - 将本文件（含头部 metadata）粘贴/导入为一个 Tool
- **方式 B：自托管环境中，把 `.py` 放到你部署所使用的自定义工具目录**
  - 重启 Open WebUI，使工具加载生效

> 提示：该工具使用了 Open WebUI 内部的文件上传处理器（`open_webui.routers.files.upload_file_handler`）与用户模型（`open_webui.models.users.Users`），因此需要运行在 Open WebUI 进程环境中，而非纯独立脚本。

---

## 使用方法

工具入口：`render_svg_chart(spec_json=...)`

- `spec_json` 推荐直接传 **JSON object**
- 也兼容传 **JSON 字符串**（支持 ```json 代码块）
- 返回值中会包含 `[CHART_MARKDOWN_BEGIN] ... [CHART_MARKDOWN_END]` 包裹的 Markdown（便于你复制/定位）

### 最小示例：折线图（line）

```json
{
  "type": "line",
  "title": { "text": "月度销售额" },
  "x": ["Jan", "Feb", "Mar", "Apr"],
  "series": [
    { "name": "A", "data": [120, 132, 101, 134] },
    { "name": "B", "data": [220, 182, 191, 234] }
  ],
  "x_title": "Month",
  "y_title": "Sales"
}
```

### 条形图（bar）：分组 / 堆叠 & 竖向 / 横向

```json
{
  "type": "bar",
  "mode": "group",
  "orientation": "v",
  "title": "渠道对比",
  "xAxis": { "data": ["小红书", "抖音", "B站"] },
  "series": [
    { "name": "曝光", "data": [1200, 2200, 900] },
    { "name": "转化", "data": [120, 180, 70] }
  ],
  "yAxis": { "name": "数量" }
}
```

堆叠横向：

```json
{
  "type": "bar",
  "mode": "stack",
  "orientation": "h",
  "title": { "text": "部门预算（堆叠）" },
  "x": ["研发", "市场", "销售"],
  "series": {
    "人力": [50, 20, 30],
    "设备": [20, 10, 15],
    "其他": [10, 5, 8]
  },
  "x_title": "金额（万）"
}
```

### 饼图 / 圆环图（pie / donut）

数据可以是 dict 或 list：

```json
{
  "type": "donut",
  "title": "市场份额",
  "data": { "A": 35, "B": 25, "C": 40 },
  "inner_ratio": 0.62,
  "show_total": true
}
```

### K线（candlestick / kline）

方式 1：`ohlc`（推荐更直观）

```json
{
  "type": "candlestick",
  "title": "示例K线",
  "ohlc": [
    { "t": "2025-01-01", "open": 100, "high": 115, "low": 95, "close": 110 },
    { "t": "2025-01-02", "open": 110, "high": 120, "low": 105, "close": 108 }
  ],
  "y_title": "Price"
}
```

方式 2：ECharts 常见 `series[0].data`：`[open, close, low, high]`

```json
{
  "type": "kline",
  "x": ["Day1", "Day2"],
  "series": [
    { "data": [[100, 110, 95, 115], [110, 108, 105, 120]] }
  ]
}
```

### 散点 / 气泡图（scatter / bubble）

Chart.js 风格点对象：

```json
{
  "type": "bubble",
  "title": { "text": "用户分群（气泡）" },
  "data": {
    "datasets": [
      {
        "label": "Group A",
        "data": [
          { "x": 10, "y": 20, "r": 8 },
          { "x": 25, "y": 12, "r": 14 }
        ]
      }
    ]
  },
  "x_title": "X",
  "y_title": "Y"
}
```

ECharts/Highcharts 风格二维点：

```json
{
  "type": "scatter",
  "title": "散点",
  "series": [
    { "name": "S1", "data": [[1, 2], [2, 3], [3, 2]] }
  ]
}
```

### 雷达图（radar）

支持显式 `indicators`，也可自动从 `x / labels / xAxis.categories` 推导：

```json
{
  "type": "radar",
  "title": "能力雷达",
  "x": ["速度", "质量", "成本", "风险", "满意度"],
  "series": {
    "方案A": [80, 70, 60, 50, 90],
    "方案B": [60, 85, 75, 65, 70]
  },
  "yAxis": { "max": 100 }
}
```

### 热力图（heatmap）

需要 `x` + `y` + `data`（矩阵或三元组）：

```json
{
  "type": "heatmap",
  "title": "热力图",
  "x": ["Mon", "Tue", "Wed"],
  "y": ["A", "B", "C"],
  "data": [
    [1, 3, 2],
    [4, 2, 1],
    [2, 1, 5]
  ],
  "show_values": true
}
```

### 直方图（histogram）

```json
{
  "type": "histogram",
  "title": "分布",
  "values": [1, 2, 2, 3, 5, 8, 13, 21],
  "bins": 6
}
```

### 箱线图（boxplot）

```json
{
  "type": "boxplot",
  "title": "箱线图",
  "data": {
    "A": [1, 2, 2, 3, 10],
    "B": [2, 3, 3, 4, 5, 6],
    "C": [1, 1, 2, 2, 2, 3]
  }
}
```

### 仪表盘（gauge）

```json
{
  "type": "gauge",
  "title": "CPU 使用率",
  "value": 63.5,
  "min": 0,
  "max": 100,
  "unit": "%"
}
```

### 漏斗图（funnel）

```json
{
  "type": "funnel",
  "title": "转化漏斗",
  "data": [
    { "name": "曝光", "value": 12000 },
    { "name": "点击", "value": 3200 },
    { "name": "下单", "value": 600 },
    { "name": "支付", "value": 420 }
  ]
}
```

### 瀑布图（waterfall）

```json
{
  "type": "waterfall",
  "title": "利润拆解",
  "start": 1000,
  "values": [200, -150, 300, -80],
  "labels": ["起点", "收入", "成本", "其他收入", "税费", "总计"],
  "show_total": true
}
```

---

## Valves 配置项（工具开关）

工具内置 `Valves`（Pydantic）配置，常用参数如下：

- `width` / `height`：SVG 尺寸（默认 980×520）
- `background`：背景色（默认 `transparent`；可设 `#fff`）
- `font_family`：字体族
- `emit_files_event`：是否发送 `chat:message:files`（附件预览）
- `emit_markdown_message`：是否自动发送 `message` 插入 Markdown 引用（默认 **true**，推荐开启）
- `embed_image`：
  - `url`：`![...](相对URL)`（最短）
  - `data_uri`：`![...](data:image/svg+xml;base64,...)`（最稳，但消息会很长）
- `auto_fix_json`：解析失败时是否尝试 `raw_decode` 截断修复（默认 **true**）

---

## 常见问题（FAQ）

### 1) “我能下载 SVG，但聊天里不显示图片”
这是 Markdown 语法的问题：必须在正文里出现 `![...](...)` 才能渲染。  
本工具默认会自动插入该引用；如果你把 `emit_markdown_message` 关了，就需要你手动复制返回的片段到正文。

### 2) `spec_json` 报错 `Extra data` 或者 JSON 解析失败
工具默认开启 `auto_fix_json`：会尝试解析第一个合法 JSON 并忽略尾随字符（例如模型输出了多段 JSON）。  
如果你希望严格模式，把 `auto_fix_json=false`。

### 3) 为什么某些图表报 “series 长度与 x 不一致”
折线/柱状/面积图要求每条 series 的数据点数量与 x 轴标签数量一致。请检查 `x`/`xAxis.data` 和 `series[].data` 长度。

### 4) 我希望“消息里引用短一点”
把 `embed_image` 设置为 `url`（默认）。  
`data_uri` 会把完整 SVG base64 写进消息，稳定但很长。

---

## 扩展与二次开发

- 新增图表类型的典型路径：
  1. 增加一个 `render_xxx_svg(spec, cfg)` 函数
  2. 在 `render_svg_dispatch()` 中加入分发映射
  3. 按需扩展解析函数（例如 `get_series_dict_numeric` / `get_point_series`）

> 工具只暴露 `Tools.render_svg_chart()` 作为入口函数，其余为内部辅助函数。

---

## License

MIT License.

---

# English README

## Overview

This tool is designed for **Open WebUI**. It renders various chart specifications into **SVG**, uploads the SVG file into Open WebUI’s file storage, and returns a Markdown snippet that can be embedded directly in the chat.

**Important note**:

- Uploading a file alone will **NOT** display it in the chat body.
- You must reference it in Markdown as an image: `![...](...)` to render it.
- By default, this tool emits an extra `message` event to **auto-insert** the Markdown snippet into the chat, so you won’t forget to embed it.

## Features

- ✅ SVG output (crisp & scalable)
- ✅ Auto upload to Open WebUI file storage and return a relative URL
- ✅ Auto-insert Markdown snippet into the chat body (enabled by default)
- ✅ Compatible with common ECharts / Highcharts / Apex / Chart.js spec styles
- ✅ JSON parsing tolerance (`spec_json` can be object or string; optional auto-fix for trailing garbage)
- ✅ Two embedding modes: URL (short) / Data URI (most robust but long)

## Supported chart types (`type`)

`line`, `area`, `bar`, `pie`, `donut`, `candlestick` (`kline` alias), `scatter`, `bubble`, `radar`, `heatmap`, `histogram`, `boxplot`, `gauge`, `funnel`, `waterfall`.

## How it works

1. Parse `spec_json` (object or JSON string; supports fenced code blocks)
2. Normalize spec (infer `type`, normalize `title`)
3. Render SVG
4. Upload SVG and get a relative URL
5. Build a Markdown snippet like:

```md
![alt](/api/v1/files/.../content?t=...)

[Open/Download SVG](/api/v1/files/.../content?t=...)
```

6. By default, emit an extra `message` event to insert the snippet into the chat.

## Configuration (Valves)

- `width`, `height`: SVG size (default 980×520)
- `background`: `transparent` or `#fff`, etc.
- `font_family`: font-family for text
- `emit_files_event`: emit `chat:message:files` for attachment preview
- `emit_markdown_message`: auto-insert Markdown snippet (default **true**, recommended)
- `embed_image`: `url` or `data_uri`
- `auto_fix_json`: attempt to auto-fix JSON parsing failures (default **true**)

## License

MIT.
