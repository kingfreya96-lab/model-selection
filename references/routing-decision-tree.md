# 路由与降级策略（Routing & Fallback）

选型交付物必须包含路由策略，否则只完成一半。目标：简单请求走低成本层，复杂请求升级到前沿层，在不牺牲质量前提下降本。

## 模式一：规则路由（任务类型已知）

```
if task_type in {分类, 抽取, 格式转换, 简单问答}:
    → cheap_model
elif task_type in {多步 Agent, 代码修复, 复杂推理}:
    → frontier_model
else:
    → classifier_route()
```

优点：可预测、零额外延迟；缺点：任务边界需维护。

## 模式二：分类器路由（任务类型未知）

```
score = lightweight_classifier(request)   # 小模型或 embedding 分类器
if score < threshold_low:
    → cheap_model
elif score < threshold_high:
    → mid_model
else:
    → frontier_model
```

阈值用业务评测集校准：保证 cheap 层通过率 ≥ 目标下限。

## 模式三：级联升级（质量优先）

```
resp = cheap_model(request)
if confidence(resp) < tau or validator_fail(resp):
    resp = frontier_model(request)
return resp
```

适合：错误代价高但多数请求简单的场景。需实现 confidence 估计或规则校验器。

## Fallback 链（可用性保障）

```
primary_model
  ├─ 超时 / 429 / 5xx → fallback_model_1（质量可接受）
  ├─ 仍失败 → fallback_model_2
  └─ 全部失败 → 规则兜底 / 转人工 / 明确报错
```

要求：fallback 模型也需通过硬排除和最低质量门槛。

## 再评估触发器（Re-evaluation triggers）

满足任一条件即重新跑选型：

- 供应商调价 ≥ 20%
- 主模型版本升级或弃用公告
- 业务评测集通过率下降 ≥ 5 个百分点
- 延迟 P95 恶化 ≥ 30%
- 出现新的强相关公开基准且候选差距显著

## 成本估算示例

| 层 | 流量占比 | 单价（相对） | 加权成本 |
|---|---|---|---|
| cheap | 70% | 1× | 0.7 |
| mid | 20% | 5× | 1.0 |
| frontier | 10% | 20× | 2.0 |
| 合计 | 100% | — | 3.7× |

对比"全量 frontier"= 20×，路由后约降本 80%。数字为示意，需按实际单价替换。