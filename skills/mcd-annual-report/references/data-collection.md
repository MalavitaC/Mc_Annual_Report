# 麦当劳 MCP 数据采集参考

本文记录实测过的工具行为，供生成年度报告时判断「什么能拿到、什么拿不到」。所有结论均来自真实调用，不是文档推测。

## 工具清单

| 工具 | 参数 | 返回 | 用途 |
|---|---|---|---|
| `order-list` | 无 | `data.list[]` 订单数组 | 全部订单主数据 |
| `query-order` | `orderId` | 单笔详情 | 优惠、券、取餐方式 |
| `list-nutrition-foods` | 无 | 160 条营养数据（非标准 JSON 文本） | 娱乐性热量估算 |
| `query-my-account` | 无 | 积分账户 | 收尾页的积分文案（可选） |

## `order-list` 的关键行为（已实测）

| 结论 | 证据 |
|---|---|
| **不接受任何参数** | 工具 schema 无入参，无翻页、无日期范围 |
| **不是「最近 N 条」截断** | 下第 6 单后返回 6 条，最老订单未被挤掉 |
| **按时间窗口返回** | 2025 及更早的订单不返回 |
| **窗口下界 ≥ 265 天** | 2026-01-31 的订单在 2026-10-23 查询时仍能取到 |
| **包含已取消订单** | `orderStatus` 为 `订单已取消` 的记录会返回 |
| **结果稳定** | 连续调用两次返回完全一致 |

**因此**：报告里的统计范围必须从 `min(createTime)` / `max(createTime)` 反推，不要硬编码「最近 365 天」或「本自然年」。

## 字段可用性

### `order-list` 每条订单提供

```
orderId, orderType, createTime, beType, beCode, storeCode,
storeName, orderStatus, orderProductList[], realTotalAmount
```

`orderProductList[]` 结构：

```
productCode, productName, quantity,
comboItemList[] { productCode, name, quantity }
```

### `query-order` 额外提供

```
totalAmount, realTotalAmount, totalDiscountAmount, couponList[],
deliveryPrice, realDeliveryPrice, realPackingFeeTotalPrice, tablewarePrice,
productPrice, takeWay, createTime, storeName, storeAddress,
pickupCode, lockerCode, lockerQrCode, remark, needTableware, payId
```

### 拿不到的数据

| 数据 | 说明 |
|---|---|
| 上一年订单 | 超出时间窗口，**跨年对比做不了** |
| 支付方式 | 无此字段 |
| 取消时间 | 只有 `expirePayTime`（下单后 15 分钟支付截止），**不是取消时刻** |
| 用户昵称 | `query-my-account` 只有 `accountId`，名称需用户提供或默认「麦友」 |
| 订单级营养 | 需按商品名自行映射 |
| 券的使用历史 | 只能逐单从 `couponList` 取，无汇总接口 |

## 三维度取数对照

挖掘洞察只看三个轴，各自需要的字段如下：

| 维度 | 字段 | 说明 |
|---|---|---|
| **时间** | `createTime` | 时段、星期、月份、首次/末次、下单间隔 |
| **时间 · 怎么吃的** | `orderType` + `takeWay` + `beType` | 合并为「就餐方式」：外送 / 店内 / 外带 / 得来速 |
| **地点** | `storeCode`（聚合）、`storeName`（展示） | 生活城市 = 订单数最多的城市（**推断**，只能说「你最常去的城市」） |
| **金额** | `realTotalAmount`、`totalDiscountAmount`、`couponList[]`、商品件数 | 基准用**中位数**，不用均值 |

### 就餐方式怎么判定

`takeWay` 实测取值为「堂食」「外带」；`orderType` 为 `1`（到店）或 `2`（外送）。

```
orderType = 2                    → 外送
takeWay 含「堂食」              → 店内
takeWay 含「得来速」或「车道」    → 得来速
takeWay 含「外带」              → 外带
都缺失                          → 未知（不要猜）
```

> `beType` 存在未文档化的值（实测出现过 `3`），**不要**用它做语义判断，只能作为城下提供兵。

### 场景门店关键词

从 `storeName` 里识别场景（不需要地址）：

```
机场 / 航站楼 / T1 / T2 / 车站 / 火车站 / 高铁 / 服务区 / 景区 /
医院 / 大学 / 商场 / 大悦城 / 万达 …
```

命中即可支撑「在出发和到达之间」这类场景叙说。地址字段 `storeAddress` **不**参与分析、**不**写入报告。

---

## 营养表解析

`list-nutrition-foods` 返回的是文本，不是 JSON：

```
[160]{productName,nutritionDescription,energyKj,energyKcal,protein,fat,carbohydrate,sodium,calcium}:
  猪柳蛋麦满分,null,1618,387,23,21,25,846,243
  中薯条,null,1210,289,4,12,38,165,18
  芝士安格斯厚牛堡,能量约鲈鱼1条,2914,696,32,44,43,897,150
```

解析：按行切分 → 逗号分隔 → 第 0 列品名，第 3 列 kcal。

**已知覆盖率问题（实测）**：季节性新品不在表内。实测 11 个商品中 8 个能匹配（73%），**漏掉的恰好是两份主汉堡**（美式辣卡真、麦芝魂），导致总热量被系统性低估。因此热量只能作为娱乐内容，且必须标注覆盖率。

## 已知陷阱

1. **门店名不一致**：`order-list` 返回 `麦当劳温州乐清银祥大厦餐厅`，`query-order` 返回 `温州乐清银祥大厦餐厅`。混用会把同一门店拆成两家，聚合时统一用列表版本。
2. **`beType` 有未文档化的值**：文档说明 1/2/5/6，实测出现 `3`。报告中不要对 `beType` 做硬编码判断，用 `takeWay` 更可靠。
3. **取消订单金额不为 0**：取消单仍带 `realTotalAmount`，直接求和会虚高。
4. **接口会瞬时 503**：实测 `query-my-account` 首次返回 `upstream connect error`，重试即成功。所有调用应重试一次。
5. **套餐与子项重复**：套餐的 `productName` 与 `comboItemList` 子项是同一份食物，聚合口味时必须二选一（用子项），否则重复计数。

## 合规约束

`CONTEST_DECLARATION.md` 已声明：不得包含真实 Token、密钥、账号凭证或他人个人信息。因此采集层**只向报告传递**：

- 店名（不含地址）
- 金额、时间、商品名
- 取消状态

**不传递**：门店地址、收货人姓名、手机号、骑手信息、取餐码、取餐柜密码、二维码、`accountId`、`payId`。
