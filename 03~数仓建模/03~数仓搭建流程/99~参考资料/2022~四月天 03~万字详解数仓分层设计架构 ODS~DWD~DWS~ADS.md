# 数仓分层设计架构 ODS-DWD-DWS-ADS 精简版

## 一、数仓分层的核心意义

数据分层是为了在**性能、成本、效率和质量**之间取得最佳平衡，核心价值体现在：

1. **清晰数据结构**：每一层有明确作用域，便于定位和理解数据
2. **追踪数据血缘**：快速定位问题并评估影响范围
3. **数据复用**：开发通用中间层，减少重复计算和存储成本
4. **简化复杂问题**：将复杂任务分解为多个步骤，便于维护
5. **屏蔽业务变化**：源头系统变更时，对下游用户透明

## 二、ETL 核心操作

ETL 占数据仓库搭建工作量的 60%-80%，包含四个关键步骤：

- **抽取(Extraction)**：初始化装载和增量刷新源数据
- **清洗(Cleaning)**：处理二义性、重复、不完整和违反规则的数据
- **转换(Transformation)**：统一数据格式和字典，生成派生字段
- **加载(Loading)**：将处理后的数据导入目标存储系统

## 三、技术架构

![横向数据流架构](https://ngte-superbed.oss-cn-beijing.aliyuncs.com/item/20230325144117.png)

![纵向数据流架构](https://ngte-superbed.oss-cn-beijing.aliyuncs.com/item/20230325144139.png)

数据中台核心组成：

- 系统架构：Hadoop、Spark 等组件体系
- 数据架构：顶层设计、主题域划分、分层设计
- 数据建模：维度建模、业务过程-粒度-维度-事实表
- 数据管理：资产、元数据、质量、主数据、标准、安全
- 辅助系统：调度、ETL、监控系统
- 数据服务：门户、报表、可视化、数据挖掘

## 四、数仓分层架构详解

![数仓分层架构](https://ngte-superbed.oss-cn-beijing.aliyuncs.com/item/20230325144258.png)

![四层数据模型](https://ngte-superbed.oss-cn-beijing.aliyuncs.com/item/20230325144333.png)

![四层逻辑分层](https://ngte-superbed.oss-cn-beijing.aliyuncs.com/item/20230325144445.png)

### 1. 贴源层（ODS, Operational Data Store）

![数据源层](https://ngte-superbed.oss-cn-beijing.aliyuncs.com/item/20230325144550.png)

**核心定义**：原始数据几乎无处理地存储，结构与源系统基本一致，是数据仓库的准备区。

**数据来源**：

- 业务数据库（MySQL、Oracle 等）
- 客户端埋点和后端日志
- 外部合作数据和爬虫数据

**存储策略**：

- **增量存储**：按日分区存储日增量数据，适用于交易、日志等事务性强的大表
- **全量存储**：每个分区存放截止到当日的全量数据，适用于小数据量的维度表
- **拉链存储**：通过 start_dt 和 end_dt 记录数据变更历史，适用于缓慢变化维度

![拉链存储](https://ngte-superbed.oss-cn-beijing.aliyuncs.com/item/20230325145711.png)

### 2. 数仓层（DW, Data Warehouse）

DW 层是数据仓库的核心，细分为三个子层：

#### 公共维度层（DIM, Dimension）

**核心定义**：建立企业级一致性维度，降低数据计算口径不统一的风险。

**维度分类**：
- 高基数维度：用户表、商品表（千万级以上）
- 低基数维度：日期表、枚举值表（几千条以内）

**设计原则**：
- 采用星型模型，维表单表不超过 1000 万条
- 与事实表 Join 时使用 Map Join
- 缓慢变化维采用拉链表实现

#### 数据明细层（DWD, Data Warehouse Detail）

![数据明细层](https://ngte-superbed.oss-cn-beijing.aliyuncs.com/item/20230325151518.png)

**核心定义**：对 ODS 层数据进行清洗、规范化和维度退化，构建最细粒度的明细事实表。

**主要处理**：
- 数据清洗：去噪、去重、过滤异常值、敏感数据脱敏
- 数据转换：统一格式、单位和数据字典
- 维度退化：将低基数维度属性冗余到事实表中，减少关联

![维度退化示例](https://ngte-superbed.oss-cn-beijing.aliyuncs.com/item/20230325144429.png)

**命名规范**：`dwd_{业务板块}_{数据域}_{业务过程}_{单分区标识}`

- 示例：`dwd_asale_trd_ordcrt_trip_di`（电商航旅机票订单下单事实表，日增量）

![创建基础明细表](https://ngte-superbed.oss-cn-beijing.aliyuncs.com/item/20230325152423.png)

#### 数据服务层（DWS, Data Warehouse Summary）

![数据轻汇总层](https://ngte-superbed.oss-cn-beijing.aliyuncs.com/item/20230325152521.png)

**核心定义**：按主题对 DWD 明细数据进行轻度汇总，构建公共粒度的宽表，为上层提供统一指标。

**主要工作**：

- 按业务主题划分（用户、商品、订单、流量等）
- 预计算常用指标（日活、留存、GMV、复购率等）
- 构建宽表，覆盖 80%以上的业务查询需求

**命名规范**：`dws_{业务板块}_{数据域}_{数据粒度}_{统计周期}`

- 示例：`dws_asale_trd_byr_subpay_1d`（买家粒度交易分阶段付款一日汇总表）

![DWS 层宽表示例](https://ngte-superbed.oss-cn-beijing.aliyuncs.com/item/20230325153322.png)

### 3. 应用层（ADS, Application Data Store）

![数据应用层](https://ngte-superbed.oss-cn-beijing.aliyuncs.com/item/20230325153527.png)

**核心定义**：存放面向具体业务场景的个性化统计指标和报表数据，直接对接数据产品和分析应用。

**主要用途**：

- 生成业务报表和仪表盘
- 提供 OLAP 分析和自助取数
- 支撑数据挖掘和机器学习
- 对外提供数据服务

**存储系统**：MySQL、HBase、Kylin、Doris、Elasticsearch 等

![ADS 层复购率统计](https://ngte-superbed.oss-cn-beijing.aliyuncs.com/item/20230325153713.png)

![计算 GMV](https://ngte-superbed.oss-cn-beijing.aliyuncs.com/item/20230325153743.png)

## 五、分层设计原则

1. **不是为了分层而分层**：根据业务需求和公司规模灵活调整
2. **数据单向流动**：上层数据只能依赖下层数据，避免循环依赖
3. **公共数据下沉**：通用指标和维度尽量放在下层，提高复用性
4. **粒度由细到粗**：从 ODS 到 ADS，数据粒度逐渐变粗
5. **命名规范统一**：各层表名和字段名遵循统一的命名规则

需要我把这份精简版进一步压缩成**一页纸速查手册**，只保留各层的核心定义、关键操作和命名规范吗？
