# Wander → Transcribe Service 技术对齐清单

以下表格整理了我们与 Wander 系统对接时，已经确认的内容和待确认的内容，按优先级从高到低排序。便于第一次会议快速对齐。

| 优先级 | 内容 | 类型 | 说明 / 备注 |
|--------|------|------|-------------|
| 高 | Session ID 唯一 | 已确认 | 由 Genesis 系统生成，全局唯一，保证每通话对应一个唯一 session |
| 高 | Transcript 消息带 sequence number | 已确认 | Wander 发送的每条消息都有递增序号，用于严格保证顺序 |
| 高 | Transcript 为 final | 已确认 | Wander 只发送最终稿，不会发送 partial transcript，顺序不会被修改 |
| 高 | EOL 消息存在 | 已确认 | Wander 会发送 EOL（End Of Line / End Of Call），明确 session 结束 |
| 高 | Kafka 顺序保证 | 已确认 | session_id 作为 key，保证 partition 内消息顺序一致 |
| 高 | Redis 用作分布式锁 | 已确认 | 用于 session ownership，防止同一 session 被多个 ECS instance 同时处理 |
| 高 | ECS 支撑 600–700 concurrent calls | 已确认 | 估算 3–4 个 task，200–250 session / task 即可满足需求 |
| 中 | Vendor reconnect 行为 | 待确认 | 确认断线重连时：1) 是否使用同一个 session_id 2) 是否可能重复发送 transcript |
| 中 | Call end 判定 | 待确认 | 确认 EOL 是否绝对最后一条消息，以及 vendor 是否可能在 EOL 后发消息 |
| 中 | sequence 连续性 | 待确认 | 确认 sequence 是否严格连续，是否可能跳号或重复（用于异常检测） |
| 中 | 最大并发 session | 待确认 | 以便调整 ECS scaling 和 Kafka partition 数量 |
| 低 | Redis TTL 策略 | 待确认 | 可选，主要用于 crash recovery，如果 EOL 明确，TTL 可短一些 |
| 低 | Kafka producer 配置 | 待确认 | 幂等 producer、ack 配置、batch size，可优化重试和吞吐 |
| 低 | WebSocket close 行为 | 待确认 | 确认 vendor 是否主动关闭 WebSocket，或者需服务端主动关闭 |