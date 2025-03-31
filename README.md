
一、核心功能与架构
Pushservice 的功能模块如下：
mermaid
graph TB
    A[推送服务] --> B[用户特征构建]
    A --> C[候选生成]
    A --> D[模型预测]
    A --> E[频率控制]
    A --> F[多渠道分发]
    
    B --> B1[基础属性]
    B --> B2[行为特征]
    B --> B3[社交图谱]
    B --> B4[设备信息]
    
    C --> C1[实时候选]
    C --> C2[历史候选]
    C --> C3[跨渠道候选]
    
    D --> D1[退订预测]
    D --> D2[活跃度预测]
    D --> D3[内容偏好]
    
    E --> E1[日推送上限]
    E --> E2[疲劳间隔]
    E --> E3[时段限制]
    
    F --> F1[应用推送]
    F --> F2[邮件通知]
    F --> F3[站内通知]
其架构采用分层设计：
前置过滤层剔除低质量内容。
轻量排序层初步筛选。
重量模型层进行深度优化。
二、关键技术特性
特征工程
系统异步加载多维度特征：
scala
Future.join(userStore.get(userId), deviceInfoStore.get(userId), historyStore.get(...))
动态频控
根据用户状态调整推送上限：
scala
def getDefaultPushCap(target: Target): Future[Int] = {
  target.targetUserState.map {
    case UserState.Inactive => 1
    case _ => configParamsBuilder.getDefaultPushCap
  }
}
多模型决策
集成多个预测模型：
scala
Future.join(optoutModelScorer.score(...), heavyRanker.score(...), pushcapModel.predict(...))
三、数据流转与计算流程
数据流转分为主动请求和系统触发两种模式：
mermaid
graph TD
    A[用户推送请求] --> B{处理器选择}
    B -->|主动请求| C[SendHandler]
    B -->|系统触发| D[RefreshForPushHandler]
    
    C --> C1[构建目标用户]
    C1 --> C2[候选项数据填充]
    C2 --> C3[特征填充]
    C3 --> C4[选择步骤/重过滤]
    C4 --> E[发送推送]
    
    D --> D1[构建目标/检查资格]
    D1 --> D2[获取候选项]
    D2 --> D3[数据填充Hydration]
    D3 --> D4[预排序过滤]
    D4 --> D5[排序阶段]
    D5 --> D6[选择步骤/重过滤]
    D6 --> E
    
    E --> F{发送渠道}
    F -->|推送通知| G[Ibis2Service]
    F -->|应用内通知| H[NotificationService]
存储体系：HDFS 存模型，Manhattan KV 存特征，Scribe 记日志。
计算流程：特征拼接 → 轻量排序 → 深度排序 → 结果筛选。
四、性能与优化
异步并行：Future.join 加速数据加载。
分级降级：高负载时禁用重排序。
监控体系：实时追踪延迟和成功率。
批量处理示例：
scala
expandCandidatesWithCopy(candidateDetails).flatMap { candidateDetailsWithCopy =>
  candidateWithCopyNumStat.add(candidateDetailsWithCopy.size)
}
五、技术栈概览
mermaid
graph LR
    A[存储] --> A1[HDFS]
    A --> A2[Manhattan KV]
    A --> A3[Memcache]
    B[计算] --> B1[TensorFlow]
    B --> B2[Finagle]
    B --> B3[XLA]
    C[服务] --> C1[Finatra]
    C --> C2[ZooKeeper]
    D[监控] --> D1[StatsReceiver]
    D --> D2[Zipkin]
六、总结
Pushservice 通过分层架构、多模型协同和动态频控，实现精准推送。其优势在于：
精准性：丰富的特征和实时反馈。
可扩展性：分布式设计支持亿级规模。
稳定性：多层次性能保障。
