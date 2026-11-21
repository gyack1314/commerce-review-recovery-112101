# 资料约定

service_id是一次实际服务，多个order_id可引用它；cancelled_before_service不进入履约表现，已履约后退款不删除机会。invitation_probability已知时可用于邀请抽样校正，response_probability未知，不可假设等于邀请概率。reviews有service_id及author_id，同一主体同服务只能有一个当前评价。disputes的pending保留两种可能责任，accepted责任按evidence字段，provider和merchant责任可以共同存在。policies按category规定minimum_observation、risk_limit和period_capacity；capacity_pools由多个商家共享，每个曝光机会带accepted_load上界，已确认订单committed_load优先占用。流量用整数opportunities分配，总量与各上限均为硬约束，最低观察若与硬风险线冲突返回infeasible及具体冲突范围，不放宽风险线。allocations记录已确认计划，actual_exposures才是实际执行机会。

## 文件与时间

所有数据都是本地合成记录，不代表新闻中的企业或交易。history.json为首批事实，updates.json为后续补充资料；后者可引用前者的对象。id标识一条事实；revises引用被更正的事实身份，不表示删除历史。known_at、recorded_at、published_at等字段表示资料可获知时间，occurred_at或effective_at表示实际发生或适用时间，二者不可互换。时间使用带时区的ISO 8601字符串，区间无特别说明均含起点、不含终点。历史分析只能采用指定截点已可得资料；空列表、null、数值零分别保留其含义。合成金额不触及真实账户。

## 数据表

### merchants

字段：id, category, capacity_pool, capacity, active_from。

### services

字段：id, merchant_id, customer_id, category, state, promised_at, fulfilled_at, invited, invitation_probability, response_probability, known_at。

### orders

字段：id, service_id, quantity, paid_minor, known_at。

### reviews

字段：id, service_id, author_id, stars, received_at, known_at。

### disputes

字段：id, review_id, status, evidence, known_at。

### policies

字段：id, category, minimum_observation, risk_limit, period_capacity。

### capacity_pools

字段：id, period_capacity。

### committed_load

字段：id, capacity_pool, load, period_start, period_end。

### allocations

字段：id, merchant_id, opportunities, accepted_load, executed, period_start, period_end, confirmed_at。

### actual_exposures

字段：id, allocation_id, opportunities, occurred_at。

input-schema.json列出资料中的字段类型概况和必要外键；同一字段可能存在null值，类型概况不是严格JSON Schema。后续资料可更正事实，但原记录可按其身份回查。

## 补充领域含义

服务不良结果指实际完成晚于promised_at一小时以上，或已有有效裁决确认商家责任；未决责任单列可能范围，评价星级是证据而非单独确定责任。风险线是同类有效机会中商家不良结果率的上界要求，缺少识别支持时不能声称已经达标。每个机会的最大承接负载为1，actual_exposures保留真实发生量，minimum_observation按商家及category给定。已确认订单负载优先；既有计划未执行部分也占相应容量，只有明确撤销后才释放。
