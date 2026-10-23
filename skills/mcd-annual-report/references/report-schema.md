# 报告数据结构

模板只负责排版。你产出的 `window.REPORT` 决定了报告长什么样。

```js
window.REPORT = {
  "meta": { ... },
  "pages": [ { ... }, { ... } ]
};
```

---

## meta

| 字段 | 必填 | 说明 |
|---|---|---|
| `year` | ✅ | 报告年份，用于刊头与跑马灯 |
| `nickname` | | 称呼，默认「麦友」 |
| `generatedAt` | | 生成日期，如 `2026-10-23` |
| `coverage` | ✅ | 统计口径，显示在收尾页脚。例：`统计范围：2026-01-31 至 2026-07-02，共 3 笔有效订单` |

---

## page 公共字段

| 字段 | 说明 |
|---|---|
| `layout` | 必填，见下方版式表 |
| `theme` | `red` / `cream` / `dark` / `yellow`，默认 `cream` |
| `eyebrow` | 标题上方的小标签，2–10 字 |
| `title` | 主标题，≤14 字，`\n` 表示换行 |
| `lead` | 一句话正文，≤50 字 |
| `note` | 补充说明，小字 |
| `strip` | `true` 时底部出现跑马灯，建议只用在首尾 |

---

## 版式

### `cover` 封面

```json
{
  "layout": "cover", "theme": "red",
  "eyebrow": "2026 · 麦当劳年度报告",
  "title": "麦友，\n这一年麦麦都记得",
  "lead": "从 1月31日，到 7月2日。\n有些事你可能忘了，麦麦还留着。",
  "hint": "向下滑动，或按 ↓ 键",
  "strip": true
}
```

### `hero` 大数字

整页只讲一个数字。适合「1 次深夜」「3 座城市」这种单点冲击。

```json
{
  "layout": "hero", "theme": "dark",
  "eyebrow": "有些夜晚",
  "value": "1", "unit": "次",
  "title": "麦麦陪你到了很晚",
  "lead": "22:06，浦东机场 T2 到达层。",
  "note": "大概是刚落地，或者还要赶下一程。"
}
```

### `stats` 数据宫格

左文右卡（4 个数字）。适合开场总览。

```json
{
  "layout": "stats", "theme": "cream",
  "eyebrow": "这一年，我们",
  "title": "见了 3 次面",
  "lead": "在三个不同的地方。",
  "items": [
    { "value": "3", "unit": "次", "label": "一起吃饭" },
    { "value": "80.5", "unit": "元", "label": "你花掉的" },
    { "value": "1", "unit": "座", "label": "走过的城市" },
    { "value": "3", "unit": "家", "label": "认出你的店" }
  ]
}
```

### `bars` 分布条

左文右条形图。适合时段、月份、品类等分布。`value` 决定长度，`display` 决定右侧显示的文字（省略则显示 `value`）。

```json
{
  "layout": "bars", "theme": "yellow",
  "eyebrow": "一天的麦麦时刻",
  "title": "三次，落在三个角落",
  "lead": "没有哪一顿是重复的。",
  "items": [
    { "label": "早餐", "value": 1 },
    { "label": "晚餐", "value": 1 },
    { "label": "夜宵", "value": 1 }
  ]
}
```

### `list` 小票清单

居中一张「小票」，虚线分隔。适合复购榜、商品清单、门店清单。

```json
{
  "layout": "list", "theme": "cream",
  "eyebrow": "每次都点的那一个",
  "title": "脆薯饼",
  "lead": "这一年你点了 2 次。",
  "items": [
    { "name": "脆薯饼", "meta": "2 次" },
    { "name": "大杯鲜萃咖啡", "meta": "2 次" }
  ],
  "numbered": true
}
```

### `cards` 卡片墙

2–6 张卡。适合门店足迹、城市列表。

```json
{
  "layout": "cards", "theme": "cream",
  "eyebrow": "麦麦见过你的地方",
  "title": "三家店，三条路线",
  "items": [
    { "kicker": "上海", "title": "上海浦东机场T2到达层快捷小站", "foot": "见过你 1 次" }
  ]
}
```

### `duo` 左右对比

正好 2 个。适合最贵/最便宜、上半年/下半年、工作日/周末。

```json
{
  "layout": "duo", "theme": "cream",
  "eyebrow": "这一年，最特别的两顿",
  "items": [
    { "tag": "最丰盛的一顿", "value": "41.5", "unit": "元", "meta": "1月31日 · 07:28\n虹桥站 · 2 样" },
    { "tag": "最简单的一顿", "value": "12", "unit": "元", "meta": "6月13日 · 18:24\n大悦城 · 1 样" }
  ],
  "lead": "丰盛和简单，都是好好吃饭的一天。"
}
```

### `quote` 引言页

用来放一个不长数字的洞察。`value` 可选。

```json
{
  "layout": "quote", "theme": "dark",
  "eyebrow": "在出发和到达之间",
  "quote": "三家店里，有两家在车站和机场。",
  "note": "07:28 虹桥站，22:06 浦东机场。"
}
```

### `closing` 收尾

页脚自动附加免责声明，不要自己写。

```json
{
  "layout": "closing", "theme": "dark",
  "eyebrow": "最后",
  "title": "麦麦永远\n欢迎你",
  "lead": "你还有 80.5 积分在麦麦这里。",
  "note": "不管你多久来一次，来了就是麦友。",
  "strip": true
}
```

---

## 洞察 → 版式对照

| 洞察形状 | 推荐版式 |
|---|---|
| 开场总览（3–4 个数字） | `stats` |
| 单个极端数字（1 次、1 座城、1 家店） | `hero` |
| 分布 / 占比 | `bars` |
| 排名 / 清单 / 复购 | `list` |
| 多点位（门店、城市） | `cards` |
| 两件事对比 | `duo` |
| 没有数字、但有画面感的结论 | `quote` |

---

## 完整示例（3 页）

```js
window.REPORT = {
  "meta": {
    "year": 2026,
    "nickname": "麦友",
    "generatedAt": "2026-10-23",
    "coverage": "统计范围：2026-01-31 至 2026-07-02，共 3 笔有效订单"
  },
  "pages": [
    {
      "layout": "cover", "theme": "red", "strip": true,
      "eyebrow": "2026 · 麦当劳年度报告",
      "title": "麦友，\n这一年麦麦都记得",
      "lead": "从 1月31日，到 7月2日。"
    },
    {
      "layout": "quote", "theme": "dark",
      "eyebrow": "在出发和到达之间",
      "quote": "三家店里，有两家在车站和机场。"
    },
    {
      "layout": "closing", "theme": "dark", "strip": true,
      "eyebrow": "最后",
      "title": "麦麦永远\n欢迎你",
      "lead": "不管你多久来一次，来了就是麦友。"
    }
  ]
};
```

---

## 校验

- `pages` 至少 3 页（cover + 至少 1 个中段 + closing）
- 每页必须有 `layout`，否则会退化成 `quote`
- `duo` 恰好 2 个 item；`stats` 建议 4 个；`cards` 2–6 个
- `title` 里的 `\n` 会被转成换行，其余字符会被转义
