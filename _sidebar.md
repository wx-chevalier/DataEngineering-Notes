  - [1 OLAP](/OLAP/README.md)
    - [1.1 MPP](/OLAP/MPP.md)
    - [1.2 OLAP 引擎](/OLAP/OLAP%20引擎.md)
    - 1.3 实践案例
      - [1.3.1 贝壳 OLAP 平台架构演进](/OLAP/实践案例/2021-贝壳%20OLAP%20平台架构演进.md)
    - 1.4 引擎对比
      - [1.4.1 Kylin、Druid、ClickHouse核心技术对比](/OLAP/引擎对比/2020-Kylin、Druid、ClickHouse核心技术对比.md)
      - [1.4.2 常用引擎对比与概述](/OLAP/引擎对比/2021-常用引擎对比与概述.md)
  - [2 建模与治理](/建模与治理/README.md)
    - [2.1 建模与治理](/建模与治理/建模与治理/README.md)
      - [2.1.1 记录与衍生](/建模与治理/建模与治理/记录与衍生/README.md)
        - [2.1.1.1 变更数据捕获](/建模与治理/建模与治理/记录与衍生/变更数据捕获.md)
        - [2.1.1.2 物化视图](/建模与治理/建模与治理/记录与衍生/物化视图.md)
        - [2.1.1.3 观察衍生数据状态](/建模与治理/建模与治理/记录与衍生/观察衍生数据状态.md)
        - [2.1.1.4 避免错误](/建模与治理/建模与治理/记录与衍生/避免错误.md)
      - [2.1.2 Kimball 与 Inmon](/建模与治理/建模与治理/Kimball%20与%20Inmon/README.md)
        - [2.1.2.1 Kimball](/建模与治理/建模与治理/Kimball%20与%20Inmon/Kimball.md)
      - [2.1.3 多维数据模型](/建模与治理/建模与治理/多维数据模型/README.md)
        - [2.1.3.1 Data Cube](/建模与治理/建模与治理/多维数据模型/Data%20Cube.md)
        - [2.1.3.2 元数据](/建模与治理/建模与治理/多维数据模型/元数据.md)
        - [2.1.3.3 星型与雪花模型](/建模与治理/建模与治理/多维数据模型/星型与雪花模型.md)
      - 2.1.4 数据治理
        - [2.1.4.1 原则与要素](/建模与治理/建模与治理/数据治理/原则与要素.md)
        - [2.1.4.2 数据零散化](/建模与治理/建模与治理/数据治理/数据零散化.md)
      - [2.1.5 指标体系](/建模与治理/建模与治理/指标体系/README.md)
        
      - [2.1.6 元数据管理](/建模与治理/建模与治理/元数据管理/README.md)
        - [2.1.6.1 Amundsen](/建模与治理/建模与治理/元数据管理/Amundsen.md)
      - [2.1.7 数据仓库搭建流程](/建模与治理/建模与治理/数据仓库搭建流程/README.md)
        
    - [2.2 数据湖](/建模与治理/数据湖/README.md)
      
    - [2.3 数据网格](/建模与治理/数据网格/README.md)
      
    - 2.4 数据中台
      - [2.4.1 数据栈](/建模与治理/数据中台/数据栈.md)
      - [2.4.2 评价维度](/建模与治理/数据中台/评价维度.md)
    - [2.5 数据中心](/建模与治理/数据中心/README.md)
      - [2.5.1 云数据中心](/建模与治理/数据中心/云数据中心.md)
    - [2.6 大数据](/建模与治理/大数据/README.md)
      - [2.6.1 不作恶](/建模与治理/大数据/不作恶.md)
      - [2.6.2 大数据平台](/建模与治理/大数据/大数据平台.md)
      - [2.6.3 大数据生态圈](/建模与治理/大数据/大数据生态圈.md)
      - [2.6.4 大数据的未来](/建模与治理/大数据/大数据的未来.md)
      - [2.6.5 数据的特性](/建模与治理/大数据/数据的特性.md)
  - [3 数据集成](/数据集成/README.md)
    - [3.1 Canal](/数据集成/Canal/README.md)
      - [3.1.1 架构机制](/数据集成/Canal/架构机制.md)
      - [3.1.2 部署与配置](/数据集成/Canal/部署与配置.md)
    - [3.2 DataPipeline](/数据集成/DataPipeline/README.md)
      - [3.2.1 一致性语义](/数据集成/DataPipeline/一致性语义.md)
      - [3.2.2 数据汇集层](/数据集成/DataPipeline/数据汇集层.md)
      - [3.2.3 数据源监听](/数据集成/DataPipeline/数据源监听.md)
      - [3.2.4 数据转换与检索](/数据集成/DataPipeline/数据转换与检索.md)
      - [3.2.5 运行环境与引擎](/数据集成/DataPipeline/运行环境与引擎.md)
    - [3.3 Debezium](/数据集成/Debezium.md)
    - [3.4 ETL](/数据集成/ETL/README.md)
      
  - 4 ROLAP
    - [4.1 ClickHouse](/ROLAP/ClickHouse/README.md)
      
    - [4.2 Hive](/ROLAP/Hive/README.md)
      - [4.2.1 HiveQL](/ROLAP/Hive/HiveQL.md)
      - [4.2.2 介绍与部署](/ROLAP/Hive/介绍与部署.md)
      - [4.2.3 数据类型](/ROLAP/Hive/数据类型.md)
      - [4.2.4 文件类型与存储格式](/ROLAP/Hive/文件类型与存储格式.md)
      - [4.2.5 自定义函数](/ROLAP/Hive/自定义函数.md)
      - [4.2.6 表操作](/ROLAP/Hive/表操作.md)
    - [4.3 Presto](/ROLAP/Presto/README.md)
      - [4.3.1 部署与控制](/ROLAP/Presto/部署与控制.md)
    - [4.4 QuickSQL](/ROLAP/QuickSQL/README.md)
      
    - 4.5 Sqoop
      - [4.5.1 介绍与部署](/ROLAP/Sqoop/介绍与部署.md)
  - 5 MOLAP
    - [5.1 Druid](/MOLAP/Druid/README.md)
      
    - 5.2 HBase
      - [5.2.1 CRUD](/MOLAP/HBase/CRUD.md)
      - [5.2.2 架构分析](/MOLAP/HBase/架构分析.md)
      - [5.2.3 部署与使用](/MOLAP/HBase/部署与使用.md)
    - [5.3 Kylin](/MOLAP/Kylin/README.md)
      
  - [6 HTAP](/HTAP/README.md)
    