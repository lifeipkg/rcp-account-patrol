# rcp-account-patrol · 账号巡查线

平台四条巡查线之一，只被 `api-gateway` 调用，**不对外暴露**（红线 RL-002）。

| 项 | 值 |
|---|---|
| 服务名 | `account-patrol` |
| 端口 | `8083` |
| 契约 | `rcp-contracts/contracts/openapi/account-patrol.yaml` |
| 上游 | `api-gateway`（`POST /api/v1/patrol/dispatch` 转发） |
| 依赖 | `rcp-core` 钉版 `v1.0.0` |

## 判定口径

- 规则引擎按事实组合打分：虚拟号段、同 IP 24h 注册数、同设备账号数、注册时段、24h 发帖量；
- 分值映射到 `Decision`，命中多条累加，不单条即封禁；
- 阈值集中在 `AccountPatrolEngine`，改动须带标定依据。

## 接口

```
POST /api/v1/patrol/check    # 巡查入口，返回 PatrolVerdict

```

请求/响应结构见契约仓；**本仓不得自行改动字段**，改字段先改契约并抬钉。

## 构建与验证

```bash
mvn -B install -f ../rcp-core/pom.xml -DskipTests   # 首次需先装基座
mvn -B verify                                        # 编译 + 测试
python ../rcp-contracts/tools/contract_check.py \
  --repo . --contract ../rcp-contracts/contracts/openapi/account-patrol.yaml
```

## 边界

- 不直连数据库（红线 RL-005）：需要持久化时经 `dataway`；
- 不调用其他巡查线（红线 RL-001）：跨线协同由网关在 dispatch 层编排；
- 不自定义决策等级（红线 RL-003）：只用 `rcp-core` 的 `Decision`。
