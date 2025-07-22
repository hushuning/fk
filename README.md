# fk
一套用于中心化交易所（CEX）的风控中间件
一套用于中心化交易所（CEX）的风控中间件，需要考虑安全性、实时性、可扩展性和可插拔性等多个维度。以下是一个完整的风控中间件系统设计方案，适用于典型的CEX场景，包括风控策略引擎、数据管道、告警系统和接口层等组件。

🧠 一、系统目标与功能概述
1.1 核心目标
实时识别异常交易行为和安全风险

提供灵活、可配置的风控策略支持

支持多种数据接入（订单、充值、提现、登录、设备指纹等）

快速响应（毫秒级），支持阻断、标记、告警等多种操作

1.2 功能模块
数据接入模块（Ingestion）

风控策略引擎（Rule Engine）

特征服务（Feature Service）

模型服务（Model Service，可选）

执行引擎（Action Engine）

日志与监控模块（Audit & Monitoring）

配置与策略管理平台（Console）

🔧 二、系统架构图（简略）<img width="991" height="793" alt="{43D56B88-BC5B-4D27-8C1A-BF72CC942891}" src="https://github.com/user-attachments/assets/fc7c78d3-ff9e-4e56-a6fc-d0d6552aadd9" />


🧩 三、模块设计细节
3.1 数据接入模块（Ingestion）
功能：接收交易行为数据，例如登录、下单、充值、提现等

实现：Kafka 消息队列、HTTP/gRPC、Webhook等方式

数据格式统一标准（如使用 protobuf）

3.2 特征服务（Feature Service）
功能：构造风控所需的实时/历史特征，如账户行为、IP黑名单、设备指纹等

存储：Redis + ClickHouse，实时与离线特征结合

支持：

窗口聚合（5分钟下单量）

用户画像（过去7天提现行为）

设备关联关系（设备→用户，用户→IP等图结构）

3.3 风控策略引擎（Rule Engine）
规则形式：

json
复制
编辑
{
  "rule_id": "withdraw_abnormal",
  "event": "withdraw",
  "conditions": [
    { "feature": "withdraw_amount", "op": ">", "value": 50000 },
    { "feature": "account_age", "op": "<", "value": 1 }
  ],
  "action": "BLOCK"
}
表达能力：支持 DSL / JSON 规则语言，内置运算符、时间窗口、正则表达式等

性能要求：ms 级处理，支持批量规则并行评估

3.4 模型服务（可选）
功能：引入机器学习模型评估复杂行为（如羊毛党识别）

模型类型：

分类模型（XGBoost、LightGBM）

图神经网络（多账户设备共用）

部署方式：

ONNX / TensorFlow Serving / PyTorch Serving

实时在线推理 + 周期性离线训练

3.5 执行引擎（Action Engine）
操作类型：

阻断（BLOCK）

风控标记（TAG）

延迟/二次验证（MFA）

触发人工审核

与交易系统接口：支持同步/异步模式

📊 四、管理与可视化平台
4.1 风控策略控制台
策略配置/上线

策略灰度发布（%流量生效）

策略效果回溯/命中率分析

4.2 审计与监控
可视化命中日志

用户风控画像

Prometheus + Grafana 监控策略运行状态

🔒 五、安全性与容灾设计
双写数据源：主从切换保证高可用

Fail-open / Fail-close 策略

灰度发布与回滚机制

防止策略互相冲突的机制（规则编排）

高并发支持：规则预编译 + 异步日志写入

🧪 六、示例使用场景
示例1：大额提现阻断规则
条件：

用户等级为普通

提现金额大于 50,000 USDT

账户注册时间少于3天

动作：

立即阻断，发送MFA验证请求

推送告警至风控中心

🧠 七、技术栈建议
模块	推荐技术
消息队列	Kafka, Pulsar
数据存储	Redis, ClickHouse, PostgreSQL
接口层	gRPC, FastAPI, SpringBoot
特征计算	Flink, Spark（批处理）
策略引擎	自研DSL + Lua脚本执行器
模型部署	ONNX Runtime, TensorFlow Serving
控制台	React + Ant Design / Vue + Element UI

📌 八、可选增强能力
图数据库（Neo4j / TigerGraph）：识别社交网络型欺诈

设备指纹引擎：防止多账号控制

行为分析引擎（如session replay）

第三方数据接入：黑产IP库、信用评分等


【1】用户行为发生（如提现）      
         ↓
【2】行为事件流入风控系统（中间件）
         ↓
【3】提取用户特征（账户画像、行为统计）
         ↓
【4】加载并匹配风控规则（或模型评分）
         ↓
【5】策略决策（通过 / 拒绝 / 人审 / 限速等）
         ↓
【6】执行动作（拦截、标记、触发MFA）
         ↓
【7】记录审计日志、命中日志、告警日志
         ↓
【8】回流数据用于后续分析 / 模型训练 / 策略优化


