# 万字详解数仓分层设计架构 ODS-DWD-DWS-ADS

# 数仓建模的意义，为什么要对数据仓库分层？

只有数据模型将数据有序的组织和存储起来之后，大数据才能得到高性能、低成本、高效率、高质量的使用。

## 1、分层意义

1）清晰数据结构：每一个数据分层都有它的作用域，这样我们在使用表的时候能更方便地定位和理解。数据关系条理化：源系统间存在复杂的数据关系，比如客户信息同时存在于核心系统、信贷系统、理财系统、资金系统，取数时该如何决策呢？数据仓库会对相同主题的数据进行统一建模，把复杂的数据关系梳理成条理清晰的数据模型，使用时就可避免上述问题了。

2）数据血缘追踪：简单来讲可以这样理解，我们最终给业务诚信的是一能直接使用的张业务表，但是它的来源有很多，如果有一张来源表出问题了，我们希望能够快速准确地定位到问题，并清楚它的危害范围。

3）数据复用，减少重复开发：规范数据分层，开发一些通用的中间层数据，能够减少极大的重复计算。数据的逐层加工原则，下层包含了上层数据加工所需要的全量数据，这样的加工方式避免了每个数据开发人员都重新从源系统抽取数据进行加工。通过汇总层的引人，避免了下游用户逻辑的重复计算，节省了用户的开发时间和精力，同时也节省了计算和存储。极大地减少不必要的数据冗余，也能实现计算结果复用，极大地降低存储和计算成本。

4）把复杂问题简单化。讲一个复杂的任务分解成多个步骤来完成，每一层只处理单一的步骤，比较简单和容易理解。而且便于维护数据的准确性，当数据出现问题之后，可以不用修复所有的数据，只需要从有问题的步骤开始修复。

5）屏蔽原始数据的(影响) ，屏蔽业务的影响。业务或系统发生变化时，不必改一次业务就需要重新接入数据。提高数据稳定性和连续性。屏蔽源头业务系统的复杂性：源头系统可能极为繁杂，而且表命名、字段命名、字段含义等可能五花八门，通过 DW 层来规范和屏蔽所有这些复杂性，保证下游数据用户使用数据的便捷和规范。如果源头系统业务发生变更，相关的变更由 DW 层来处理，对下游用户透明，无须改动下游用户的代码和逻辑。

数据仓库的可维护性：分层的设计使得某一层的问题只在该层得到解决，无须更改下一层的代码和逻辑。大数据系统需要数据模型方法来帮助更好地组织和存储数据，以便在性能、成本、效率和质量之间取得最佳平衡！

## 2、数据仓库（ETL）的四个操作

ETL(extractiontransformation loading)负责将分散的、异构数据源中的数据抽取到临时中间层后进行清洗、转换、集成，最后加载到数据仓库或数据集市中。ETL 是实施数据仓库的核心和灵魂，ETL 规则的设计和实施约占整个数据仓库搭建工作量的 60%～ 80%。

1）数据抽取(extraction)包括初始化数据装载和数据刷新：初始化数据装载主要关注的是如何建立维表、事实表，并把相应的数据放到这些数据表中；而数据刷新关注的是当源数据发生变化时如何对数据仓库中的相应数据进行追加和更新等维护(比如可以创建定时任务，或者触发器的形式进行数据的定时刷新)。

2）数据清洗主要是针对源数据库中出现的二义性、重复、不完整、违反业务或逻辑规则等问题的数据进行统一的处理。即清洗掉不符合业务或者没用的的数据。比如通过编写 hive 或者 MR 清洗字段中长度不符合要求的数据。

3）数据转换(transformation)主要是为了将数据清洗后的数据转换成数据仓库所需要的数据：来源于不同源系统的同一数据字段的数据字典或者数据格式可能不一样(比如 A 表中叫 id,B 表中叫 ids)，在数据仓库中需要给它们提供统一的数据字典和格式，对数据内容进行归一化；另一方面，数据仓库所需要的某些字段的内容可能是源系统所不具备的，而是需要根据源系统中多个字段的内容共同确定。

4）数据加载（loading）是将最后上面处理完的数据导入到对应的存储空间里（hbase，mysql 等）以方便给数据集市提供，进而可视化。

一般大公司为了数据安全和操作方便，都是自己封装的数据平台和任务调度平台，底层封装了大数据集群比如 hadoop 集群，spark 集群，sqoop,hive,zookeepr,hbase 等只提供 web 界面，并且对于不同员工加以不同权限，然后对集群进行不同的操作和调用。以数据仓库为例，将数据仓库分为逻辑上的几个层次。这样对于不同层次的数据操作，创建不同层次的任务，可以放到不同层次的任务流中进行执行（大公司一个集群通常每天的定时任务有几千个等待执行，甚至上万个，所以划分不同层次的任务流，不同层次的任务放到对应的任务流中进行执行，会更加方便管理和维护）。

## 3、分层的误区

数仓层内部的划分不是为了分层而分层，分层是为了解决 ETL 任务及工作流的组织、数据的流向、读写权限的控制、不同需求的满足等各类问题。

业界较为通行的做法将整个数仓层又划分成了 DWD、DWT、DWS、DIM、DM 等很多层。然而我们却始终说不清楚这几层之间清晰的界限是什么，或者说我们能说清楚它们之间的界限，复杂的业务场景却令我们无法真正落地执行。

所以数据分层这块一般来说三层是最基础的，至于 DW 层如何进行切分，是根据具体的业务需求和公司场景自己去定义。

# 二、技术架构

![横向数据流架构](https://assets.ng-tech.icu/item/20230325144117.png)

![纵向数据流架构](https://assets.ng-tech.icu/item/20230325144139.png)

数据中台包含的内容很多，对应到具体工作中的话，它可以包含下面的这些内容：

- **系统架构：**以 Hadoop、Spark 等组件为中心的架构体系
- **数据架构：**顶层设计，主题域划分，分层设计，ODS-DW-ADS
- **数据建模：**维度建模，业务过程-确定粒度-维度-事实表
- **数据管理：**资产管理，元数据管理、质量管理、主数据管理、数据标准、数据安全管理
- **辅助系统：**调度系统、ETL 系统、监控系统
- **数据服务：**数据门户、机器学习数据挖掘、数据查询、分析、报表系统、可视化系统、数据交换分享下载

# 三、数仓分层架构

![数仓分层架构](https://assets.ng-tech.icu/item/20230325144258.png)

数据仓库标准上可以分为四层。但是注意这种划分和命名不是唯一的，一般数仓都是四层，但是不同公司可能叫法不同。但是核心的理念都是从四层数据模型而来。

![四层数据模型](https://assets.ng-tech.icu/item/20230325144333.png)

![四层数据模型详解](https://assets.ng-tech.icu/item/20230325144429.png)

![四层逻辑分层](https://assets.ng-tech.icu/item/20230325144445.png)

## 1、贴源层（ODS, Operational Data Store）

![数据源层](https://assets.ng-tech.icu/item/20230325144550.png)

数据引入层（ODS，Operational Data Store，又称数据基础层）：将原始数据几乎无处理地存放在数据仓库系统中，结构上与源系统基本保持一致，是数据仓库的数据准备区。这一层的主要职责是将基础数据同步、存储。一般来说 ODS 层的数据和源系统的数据是同构的，主要目的是简化后续数据加工处理的工作。从数据粒度上来说 ODS 层的数据粒度是细的。ODS 层的表通常包括两类，一个用于存储当前需要加载的数据，一个用于存储处理完后的历史数据。历史数据一般保存 3-6 个月后需要清除，以节省空间。但不同的项目要区别对待，如果源系统的数据量不大，可以保留更长的时间，甚至全量保存。

注意：在这层，理应不是简单的数据接入，而是要考虑一定的数据清洗，比如异常字段的处理、字段命名规范化、时间字段的统一等，一般这些很容易会被忽略，但是却至关重要。特别是后期我们做各种特征自动生成的时候，会十分有用。

注意：有的公司 ODS 层不会做太多数据过滤处理,会放到 DWD 层来处理。有的公司会在一开始时就在 ODS 层做数据相对精细化的过滤.这个并没有明确规定,看每个公司自己的想法和技术规范。一般企业开发时,都会对原始数据存入到 ODS 时,做一些最基本的处理。

- 数据来源区分：数据按照时间分区存储,一般是按照天,也有公司使用年、月、日三级分区做存储的。进行最基本的数据处理,如格式错误的丢弃,关键信息丢失的过滤掉等等。

- 数据实时离线

  - 离线方面：每日定时任务型：跑批任务，业务库，比如我们典型的日计算任务，这里经常会使用 Sqoop 来抽取，比如我们每天定时抽取一次。每天凌晨算前一天的数据，早上起来看报表。这种任务经常使用 Hive、Spark 来计算，最终结果写入 Hive、Hbase、Mysql、Es 或者 Redis 中。
  - 实时数据：日志埋点数据或者业务库，这部分主要是各种实时的系统使用，比如我们的实时推荐、实时用户画像，一般我们会用 Spark Streaming、Flink 来计算，最后会落入 Es、Hbase 或者 Redis 中。数据源是业务数据库，可以考虑用 Canal 监听 Mysql 的 Binlog，实时接入即可，然后也是收集到消息队列中，最终再由 Camus 拉取到 HDFS。

- 数据主要来源
  - 数据源是业务数据库，公司所有的系统产生的数据
  - 是通过在客户端埋点上报，收集用户的行为日志，以及一些后端日志的日志类型数据源。对于埋点行为日志来说，一般会经过一个这样的流程，首先数据会上报到 Nginx 然后经过 Flume 收集，然后存储到 Kafka 这样的消息队列，然后再由实时或者离线的一些拉取的任务，拉取到我们的离线数据仓库 HDFS
  - 外部数据（包括合作数据以及爬虫获得的数据），将所采集的数据汇总到一起

### 数据存储策略（增量、全量）

实际应用中，可以选择采用增量、全量存储或拉链存储的方式。

#### 增量存储

为了满足历史数据分析需求，您可以在 ODS 层表中添加时间维度作为分区字段。以天为单位的增量存储，以业务日期作为分区，每个分区存放日增量的业务数据。举例如下：

- 1 月 1 日，用户 A 访问了 A 公司电商店铺 B，A 公司电商日志产生一条记录 t1。1 月 2 日，用户 A 又访问了 A 公司电商店铺 C，A 公司电商日志产生一条记录 t2。采用增量存储方式，t1 将存储在 1 月 1 日这个分区中，t2 将存储在 1 月 2 日这个分区中。
- 1 月 1 日，用户 A 在 A 公司电商网购买了 B 商品，交易日志将生成一条记录 t1。1 月 2 日，用户 A 又将 B 商品退货了，交易日志将更新 t1 记录。

采用增量存储方式，初始购买的 t1 记录将存储在 1 月 1 日这个分区中，更新后的 t1 将存储在 1 月 2 日这个分区中。交易、日志等事务性较强的 ODS 表适合增量存储方式。这类表数据量较大，采用全量存储的方式存储成本压力大。此外，这类表的下游应用对于历史全量数据访问的需求较小（此类需求可通过数据仓库后续汇总后得到）。例如，日志类 ODS 表没有数据更新的业务过程，因此所有增量分区 UNION 在一起就是一份全量数据。

#### 全量存储

以天为单位的全量存储，以业务日期作为分区，每个分区存放截止到业务日期为止的全量业务数据。

例如，1 月 1 日，卖家 A 在 A 公司电商网发布了 B、C 两个商品，前端商品表将生成两条记录 t1、t2。1 月 2 日，卖家 A 将 B 商品下架了，同时又发布了商品 D，前端商品表将更新记录 t1，同时新生成记录 t3。采用全量存储方式，在 1 月 1 日这个分区中存储 t1 和 t2 两条记录，在 1 月 2 日这个分区中存储更新后的 t1 以及 t2、t3 记录。

对于小数据量的缓慢变化维度数据，例如商品类目，可直接使用全量存储。

#### 拉链存储

拉链存储通过新增两个时间戳字段（start_dt 和 end_dt），将所有以天为粒度的变更数据都记录下来，通常分区字段也是这两个时间戳字段。

![拉链存储](https://assets.ng-tech.icu/item/20230325145711.png)

## 2、数仓层（DW,data warehouse）

数据仓库层(DW)层：数据仓库层是我们在做数据仓库时要核心设计的一层，本层将从 ODS 层中获得的数据按照主题建立各种数据模型，每一个主题对应一个宏观的分析领域，数据仓库层排除对决策无用的数据，提供特定主题的简明视图。在 DW 层会保存 BI 系统中所有的历史数据，例如保存 10 年的数据。DW 存放明细事实数据、维表数据及公共指标汇总数据。其中，明细事实数据、维表数据一般根据 ODS 层数据加工生成。公共指标汇总数据一般根据维表数据和明细事实数据加工生成。

DW 层又细分为维度层（DIM）、明细数据层（DWD）和汇总数据层（DWS），采用维度模型方法作为理论基础，可以定义维度模型主键与事实模型中外键关系，减少数据冗余，也提高明细数据表的易用性。在汇总数据层同样可以关联复用统计粒度中的维度，采取更多的宽表化手段构建公共指标数据层，提升公共指标的复用性，减少重复加工。

- 维度层（DIM，Dimension）：以维度作为建模驱动，基于每个维度的业务含义，通过添加维度属性、关联维度等定义计算逻辑，完成属性定义的过程并建立一致的数据分析维表。为了避免在维度模型中冗余关联维度的属性，基于雪花模型构建维度表。
- 明细数据层（DWD，Data Warehouse Detail）：以业务过程作为建模驱动，基于每个具体的业务过程特点，构建最细粒度的明细事实表。可将某些重要属性字段做适当冗余，也即宽表化处理。
- 汇总数据层（DWS，Data Warehouse Summary）：以分析的主题对象作为建模驱动，基于上层的应用和产品的指标需求，构建公共粒度的汇总指标表。以宽表化手段物理化模型，构建命名规范、口径一致的统计指标，为上层提供公共指标，建立汇总宽表、明细事实表。
- 主题域：面向业务过程，将业务活动事件进行抽象的集合，如下单、支付、退款都是业务过程。针对公共明细层（DWD）进行主题划分。
- 数据域：面向业务分析，将业务过程或者维度进行抽象的集合。针对公共汇总层（DWS） 进行数据域划分。

其中 DWD 层是以业务过程为驱动，DWS 层、DWT 层和 ADS 层都是以需求为驱动。

### 公共维度层（DIM，Dimension）

DIM：这一层比较单纯，举个例子就明白，比如国家代码和国家名、地理位置、中文名、国旗图片等信息就存在 DIM 层中。基于维度建模理念思想，建立整个企业的一致性维度。降低数据计算口径和算法不统一风险。

公共维度汇总层（DIM）主要由维度表（维表）构成。维度是逻辑概念，是衡量和观察业务的角度。维表是根据维度及其属性将数据平台上构建的表物理化的表，采用宽表设计的原则。因此，构建公共维度汇总层（DIM）首先需要定义维度。

- 高基数维度数据：一般是用户资料表、商品资料表类似的资料表。数据量可能是千万级或者上亿级别。
- 低基数维度数据：一般是配置表，比如枚举值对应的中文含义，或者日期维表。数据量可能是个位数或者几千几万。

完成维度定义后，您就可以对维度进行补充，进而生成维表了。维表的设计需要注意：

- 建议维表单表信息不超过 1000 万条。
- 维表与其他表进行 Join 时，建议您使用 Map Join。
- 避免过于频繁的更新维表的数据。缓慢变化维：拉链表

公共维度汇总层（DIM）维表规范：公共维度汇总层（DIM）维表命名规范：`dim_{业务板块名称/pub}_{维度定义}[_{自定义命名标签}]`，所谓 pub 是与具体业务板块无关或各个业务板块都可公用的维度，如时间维度。例如：公共区域维表 dim_pub_area 商品维表 dim_asale_itm。事实表中一条记录所表达的业务细节程度被称为粒度。通常粒度可以通过两种方式来表述：一种是维度属性组合所表示的细节程度，一种是所表示的具体业务含义。通透！数据仓库领域常见建模方法及实例演示。

#### 建模方式及原则

需要构建维度模型，一般采用星型模型，呈现的状态一般为星座模型（由多个事实表组合，维表是公共的，可被多个事实表共享）；为支持数据重跑可额外增加数据业务日期字段，可按日进行分表，用增量 ODS 层数据和前一天 DWD 相关表进行 merge 处理。

粒度是一行信息代表一次行为，例如一次下单。

#### 维度建模步骤

1. 选择业务过程：在业务系统中，挑选感兴趣的业务线，比如下单业务，支付业务，退款业务，物流业务，一条业务线对应一张事实表。如果是中小公司，尽量把所有业务过程都选择。DWD 如果是大公司（1000 多张表），选择和需求相关的业务线。

2. 声明粒度：数据粒度指数据仓库的数据中保存数据的细化程度或综合程度的级别。声明粒度意味着精确定义事实表中的一行数据表示什么，应该尽可能选择最小粒度，以此来应各种各样的需求。典型的粒度声明如下：订单当中的每个商品项作为下单事实表中的一行，粒度为每次。每周的订单次数作为一行，粒度为每周。每月的订单次数作为一行，粒度为每月。如果在 DWD 层粒度就是每周或者每月，那么后续就没有办法统计细粒度的指标了。所以建议采用最小粒度。

3. 确定维度：维度的主要作用是描述业务是事实，主要表示的是“谁，何处，何时”等信息。确定维度的原则是：后续需求中是否要分析相关维度的指标。例如，需要统计，什么时间下的订单多，哪个地区下的订单多，哪个用户下的订单多。需要确定的维度就包括：时间维度、地区维度、用户维度。维度表：需要根据维度建模中的星型模型原则进行维度退化。

4. 确定事实：此处的“事实”一词，指的是业务中的度量值（次数、个数、件数、金额，可以进行累加），例如订单金额、下单次数等。在 DWD 层，以业务过程为建模驱动，基于每个具体业务过程的特点，构建最细粒度的明细层事实表。事实表可做适当的宽表化处理。

DWD 层是以业务过程为驱动。DWS 层、DWT 层和 ADS 层都是以需求为驱动，和维度建模已经没有关系了。DWS 和 DWT 都是建宽表，按照主题去建表。主题相当于观察问题的角度。对应着维度表。

- 关于主题：数据仓库中的数据是面向主题组织的，主题是在较高层次上将企业信息系统中的数据进行综合、归类和分析利用的一个抽象概念，每一个主题基本对应一个宏观的分析领域。如财务分析就是一个分析领域，因此这个数据仓库应用的主题就为“财务分析”。

- 关于主题域：主题域通常是联系较为紧密的数据主题的集合。可以根据业务的关注点，将这些数据主题划分到不同的主题域(也说是对某个主题进行分析后确定的主题的边界)

主题域的确定必须由最终用户（业务）和数据仓库的设计人员共同完成的，而在划分主题域时，大家的切入点不同可能会造成一些争论、重构等的现象，考虑的点可能会是下方的某些方面：

- 按照业务或业务过程划分：比如一个靠销售广告位置的门户网站主题域可能会有广告域，客户域等，而广告域可能就会有广告的库存，销售分析、内部投放分析等主题；
- 根据需求方划分：比如需求方为财务部，就可以设定对应的财务主题域，而财务主题域里面可能就会有员工工资分析，投资回报比分析等主题；
- 按照功能或应用划分：比如微信中的朋友圈数据域、群聊数据域等，而朋友圈数据域可能就会有用户动态信息主题、广告主题等；
- 按照部门划分：比如可能会有运营域、技术域等，运营域中可能会有工资支出分析、活动宣传效果分析等主题；

总而言之，切入的出发点逻辑不一样，就可以存在不同的划分逻辑。在建设过程中可采用迭代方式，不纠结于一次完成所有主题的抽象，可先从明确定义的主题开始，后续逐步归纳总结成自身行业的标准模型。主题：当事人、营销、财务、合同协议、机构、地址、渠道、产品、

金融业务主题可分为四个主题：

- 用户主题（用户年龄、性别、收货地址、电话、省份等）
- 交易主题（订单数据、账单数据等）
- 风控主题（用户的风控等级，第三方征信数据）
- 营销主题（营销活动名单，活动配置信息等）

### DWD（data warehouse detail）数据明细层，明细粒度事实层

![数据明细层](https://assets.ng-tech.icu/item/20230325151518.png)

DWD 是业务层与数据仓库的隔离层，这一层主要解决一些数据质量问题和数据的完整度问题。明细表用于存储 ODS 层原始表转换过来的明细数据，DWD 层的数据应该是一致的、准确的、干净的数据，即对源系统数据 ODS 层数据进行清洗（去除空值，脏数据，超过极限范围的数据，行式存储改为列存储，改压缩格式）、规范化、维度退化、脱敏等操作。比如用户的资料信息来自于很多不同表，而且经常出现延迟丢数据等问题，为了方便各个使用方更好的使用数据，我们可以在这一层做一个屏蔽。这一层也包含统一的维度数据。

明细粒度事实层（DWD）：以业务过程作为建模驱动，基于每个具体的业务过程特点，构建最细粒度的明细层事实表。可以结合企业的数据使用特点，将明细事实表的某些重要维度属性字段做适当冗余，即宽表化处理。明细粒度事实层的表通常也被称为逻辑事实表。

负责数据的最细粒度的数据，在 DWD 层基础上，进行轻度汇总，结合常用维度（时间，地点，组织层级，用户，商品等），该层一般保持和 ODS 层一样的数据粒度，并且提供一定的数据质量保证，在 ODS 的基础上对数据进行加工处理，提供更干净的数据。同时，为了提高数据明细层的易用性，该层会采用一些维度退化手法，当一个维度没有数据仓库需要的任何数据时，就可以退化维度，将维度退化至事实表中，减少事实表和维表的关联。

例如：订单 ID，这种量级很大的维度，没必要用一张维度表来进行存储，而我们一般在进行数据分析时订单 id 又非常重要，所以我们将订单 id 冗余在事实表中，这种维度就是退化维度。这一层的数据一般是遵循数据库第三范式或者维度建模，其数据粒度通常和 ODS 的粒度相同。在 PDW 层会保存 BI 系统中所有的历史数据，例如保存 10 年的数据。

数据在装入本层前需要做以下工作：去噪、去重、提脏、业务提取、单位统一、砍字段、业务判别。清洗的数据种类：

- 不完整数据
- 错误数据
- 重复的数据

数据清洗的任务是过滤那些不符合要求的数据，将过滤的结果交给业务主管部门，确认是否过滤掉还是由业务单位修正之后再进行抽取。

#### 数据清洗过滤

- 去除废弃字段,去除格式错误的信息
- 去除丢失了关键字段的信息
- 过滤核心字段无意义的数据，比如订单表中订单 id 为 null，支付表中支付 id 为空
- 对手机号、身份证号等敏感数据脱敏
- 去除不含时间信息的数据(这个看公司具体业务,但一般数据中都会带上时间戳,这样方便后续处理时,进行时间维度上信息分析处理和提取)

有些公司还会在这一层将数据打平，不过这具体要看业务需求。这是因为 kylin 适合处理展平后数据，不适合处理嵌套的表数据信息，有些公司还会将数据 session 做切割，这个一般是 app 的日志数据，其他业务场景不一定适合。这是因为 app 有进入后台模式，例如用户上午打开 app 用了 10 分钟,然后 app 切入后台，晚上再打开，这时候 session 还是一个，实际上应该做切割才对。(也有公司会记录 app 进入后台,再度进入前台的记录,这样来做 session 切割)

#### 数据映射，转换

将 GPS 经纬度转换为省市区详细地址。业界常见 GPS 快速查询一般将地理位置知识库使用 geohash 映射,然后将需要比对的 GPS 转换为 geohash 后跟知识库中 geohash 比对,查找出地理位置信息当然,也有公司使用 open api,如高德地图,百度地图的 api 进行 GPS 和地理位置信息映射,但这个达到一定次数需要花钱,所以大家都懂的。

- 会将 IP 地址也转换为省市区详细地址。这个有很多快速查找库,不过基本原理都是二分查找,因为 ip 地址可以转换为长整数.典型的如 ip2region 库
- 将时间转换为年,月,日甚至周,季度维度信息
- 数据规范化,因为大数据处理的数据可能来资源公司不同部门,不同项目,不同客户端,这时候可能相同业务数据字段,数据类型,空值等都不一样,这时候需要在 DWD 层做抹平.否则后续处理使用时,会造成很大的困扰
- 如 boolean,有使用 0 1 标识,也有使用 true false 标识的
- 如字符串空值,有使用"",也有使用 null,的,统一为 null 即可
- 如日期格式,这种就差异性更大,需要根据实际业务数据决定,不过一般都是格式化为 YYYY-MM-dd HH:mm:ss 这类标准格式
- 维度退化：对业务数据传过来的表进行维度退化和降维。（商品一级二级三级、省市县、年月日）订单 id 冗余在事实表

#### 明细粒度事实表设计原则

- 一个明细粒度事实表仅和一个维度关联。
- 尽可能包含所有与业务过程相关的事实。
- 只选择与业务过程相关的事实。
- 分解不可加性事实为可加的组件。
- 在选择维度和事实之前必须先声明粒度。
- 在同一个事实表中不能有多种不同粒度的事实。
- 事实的单位要保持一致。粒度
- 谨慎处理 Null 值。
- 使用退化维度提高事实表的易用性。

命名规范为：`dwd_{业务板块/pub}_{数据域缩写}_{业务过程缩写}[_{自定义表命名标签缩写}] _{单分区增量全量标识}`，pub 表示数据包括多个业务板块的数据。单分区增量全量标识通常为：i 表示增量，f 表示全量。例如：dwd_asale_trd_ordcrt_trip_di（A 电商公司航旅机票订单下单事实表，日刷新增量） dwd_asale_itm_item_df（A 电商商品快照事实表，日刷新全量）。

本教程中，DWD 层主要由三个表构成：

- 交易商品信息事实表：dwd_asale_trd_itm_di。
- 交易会员信息事实表：ods_asale_trd_mbr_di。
- 交易订单信息事实表：dwd_asale_trd_ord_di。

![ODS 表](https://assets.ng-tech.icu/item/20230325152246.png)

```sql

CREATE TABLE IF NOT EXISTS dwd_asale_trd_itm_di
(
    item_id BIGINT COMMENT '商品ID',
    item_title STRING COMMENT '商品名称',
    item_price DOUBLE COMMENT '商品价格',
    item_stuff_status BIGINT COMMENT '商品新旧程度_0全新1闲置2二手',
    item_prov STRING COMMENT '商品省份',
    item_city STRING COMMENT '商品城市',
    cate_id BIGINT COMMENT '商品类目ID',
    cate_name STRING COMMENT '商品类目名称',
    commodity_id BIGINT COMMENT '品类ID',
    commodity_name STRING COMMENT '品类名称',
    buyer_id BIGINT COMMENT '买家ID',
)
COMMENT '交易商品信息事实表'
PARTITIONED BY (ds STRING COMMENT '日期')
LIFECYCLE 400;
```

![创建基础明细表](https://assets.ng-tech.icu/item/20230325152423.png)

### DWS（data warehouse service）数据服务层，汇总层宽表

![数据轻汇总层](https://assets.ng-tech.icu/item/20230325152521.png)

基于 DWD 明细数据层，我们会按照一些分析场景、分析实体等去组织我们的数据，组织成一些分主题的汇总数据层 DWS。DWS 层（数据汇总层）宽表，面向主题的汇总，维度相对来说比较少，DWS 是根据 DWD 层基础数据按各个维度 ID 进行粗粒度汇总聚合，如按交易来源，交易类型进行汇合。整合汇总成分析某一个主题域的服务数据，一般是宽表。

以 DWD 为基础，按天进行轻度汇总。统计各个主题对象的当天行为，（例如，购买行为，统计商品复购率）。该层数据表会相对比较少，大多都是宽表(一张表会涵盖比较多的业务内容，表中的字段较多)。按照主题划分，如订单、用户等，生成字段比较多的宽表，用于提供后续的业务查询，OLAP 分析，数据分发等。融合多个中间层数据，基于主题形成事实表，比如用户事实表、渠道事实表、终端事实表、资产事实表等等，事实表一般是宽表，在本层上实现企业级数据的一致性。

首先划分业务主题，将主题划分为销售域、库存域、客户域、采购域 等，其次就是 确定每个主题域的事实表和维度表。通常根据业务需求，划分成流量、订单、用户等，生成字段比较多的宽表，用于提供后续的业务查询，OLAP 分析，数据分发等。

最近一天某个类目（例如：厨具）商品在各省的销售总额、该类目 Top10 销售额商品名称、各省用户购买力分布。因此，我们可以以最终交易成功的商品、类目、买家等角度对最近一天的数据进行汇总。比如用户每个时间段在不同登录 ip 购买的商品数等。这里做一层轻度的汇总会让计算更加的高效，在此基础上如果计算仅 7 天、30 天、90 天的行为的话会快很多。我们希望 80%的业务都能通过我们的 DWS 层计算，而不是 ODS。

#### DWS 层做了哪些事？

DWS 将 DWD 层的数据按主题进行汇总，按照主题放到一个表中，比如用户主题下会将用户注册信息、用户收货地址、用户的征信数据放到同一张表中，而这些在 DWD 层是对应多张表的,按照业务划分，如流量、订单、用户等，生成字段比较多的宽表

主题建模,围绕某一个业务主题进行数据建模,将相关数据抽离提取出来.如：

- 将流量会话按照天,月进行聚合
- 将每日新用户进行聚合
- 将每日活跃用户进行聚合
- 维度建模,其实也差不多,不过是根据业务需要,提前将后续数据查询处理需要的维度数据抽离处理出来,方便后续查询使用.
- 如将运营位维度数据聚合
- 将渠道拉新维度数据聚合

以电商用户域为例：

- DWS 层每个主题 1-3 张宽表（处理 100-200 个指标 70%以上的需求）。具体宽表名称：用户行为宽表，用户购买商品明细行为宽表，商品宽表，物流宽表、售后等。
- 哪个宽表最宽？大概有多少个字段？最宽的是用户行为宽表。大概有 60-200 个字段
- 具体用户行为宽表字段名称：评论、打赏、收藏、关注--商品、关注--人、点赞、分享、好价爆料、文章发布、活跃、签到、补签卡、幸运屋、礼品、金币、电商点击、gmv
- 分析过的指标：日活、月活、周活、留存、留存率、新增（日、周、年）、转化率、流失、回流、七天内连续 3 天登录（点赞、收藏、评价、购买、加购、下单、活动）、连续 3 周（月）登录、GMV（成交金额，下单）、复购率、复购率排行、点赞、评论、收藏、领优惠价人数、使用优惠价、沉默、值不值得买、退款人数、退款率 topn 热门商品

#### 公共汇总事实表规范

公共汇总事实表命名规范：`dws_{业务板块缩写/pub}_{数据域缩写}_{数据粒度缩写}[_{自定义表命名标签缩写}]_{统计时间周期范围缩写}`。关于统计实际周期范围缩写，缺省情况下，离线计算应该包括最近一天（`_1d`），最近 N 天（`_nd`）和历史截至当天（`_td`）三个表。如果出现`_nd`的表字段过多需要拆分时，只允许以一个统计周期单元作为原子拆分。即一个统计周期拆分一个表，例如最近 7 天（`_1w`）拆分一个表。不允许拆分出来的一个表存储多个统计周期。对于小时表（无论是天刷新还是小时刷新），都用 `_hh` 来表示。对于分钟表（无论是天刷新还是小时刷新），都用 `_mm` 来表示。

举例如下：

- dws_asale_trd_byr_subpay_1d（买家粒度交易分阶段付款一日汇总事实表）
- dws_asale_trd_byr_subpay_td（买家粒度分阶段付款截至当日汇总表）
- dws_asale_trd_byr_cod_nd（买家粒度货到付款交易汇总事实表）
- dws_asale_itm_slr_td（卖家粒度商品截至当日存量汇总表）
- dws_asale_itm_slr_hh（卖家粒度商品小时汇总表）---维度为小时
- dws_asale_itm_slr_mm（卖家粒度商品分钟汇总表）---维度为分钟

```sql
-- 用户维度：用户主题

drop table
if exists dws_sale_detail_daycount;
create external table dws_sale_detail_daycount(
user_id string comment '用户 id',
--用户信息
user_gender string comment '用户性别',
user_age string comment '用户年龄',
user_level string comment '用户等级',
buyer_nick string comment '买家昵称',
mord_prov string comment '地址',
--下单数、商品数量，金额汇总
login_count bigint comment '当日登录次数',
cart_count bigint comment '加入购物车次数',
order_count bigint comment '当日下单次数',
order_amount decimal(16,2) comment '当日下单金额',
payment_count bigint comment '当日支付次数',
payment_amount decimal(16,2) comment '当日支付金额',
confirm_paid_amt_sum_1d double comment '最近一天订单已经确认收货的金额总和'
order_detail_stats array<struct<sku_id:string,sku_num:bigint,order_count:bigint,order_amount:decimal(20,2)>> comment '下单明细统计') comment '每日购买行为'
partitioned by(`dt`
string)
stored as parquet
location '/warehouse/gmall/dws/dws_sale_detail_daycount/'
tblproperties("parquet.compression" = "lzo");

-- 商品维度：商品主题

CREATE TABLE IF NOT EXISTS dws_asale_trd_itm_ord_1d
(
item_id BIGINT COMMENT '商品ID',
--商品信息，产品信息
item_title STRING COMMENT '商品名称',
cate_id BIGINT COMMENT '商品类目ID',
cate_name STRING COMMENT '商品类目名称',
--mord_prov STRING COMMENT '收货人省份',
--商品售出金额汇总
confirm_paid_amt_sum_1d DOUBLE COMMENT '最近一天订单已经确认收货的金额总和'
)
COMMENT '商品粒度交易最近一天汇总事实表'
PARTITIONED BY (ds STRING COMMENT '分区字段YYYYMMDD')
LIFECYCLE 36000;
```

![DWS 层宽表](https://assets.ng-tech.icu/item/20230325153322.png)

## 3、应用层（ADS）applicationData Service 应用数据服务

![数据应用层](https://assets.ng-tech.icu/item/20230325153527.png)

数据应用层（ADS，Application Data Store）：存放数据产品个性化的统计指标数据，报表数据。主要是提供给数据产品和数据分析使用的数据，通常根据业务需求，划分成流量、订单、用户等，生成字段比较多的宽表，用于提供后续的业务查询，OLAP 分析，数据分发等。从数据粒度来说，这层的数据是汇总级的数据，也包括部分明细数据。从数据的时间跨度来说，通常是 DW 层的一部分，主要的目的是为了满足用户分析的需求，而从分析的角度来说，用户通常只需要分析近几年的即可。从数据的广度来说，仍然覆盖了所有业务数据。

在 DWS 之上，我们会面向应用场景去做一些更贴近应用的 APP 应用数据层，这些数据应该是高度汇总的，并且能够直接导入到我们的应用服务去使用。应用层(ADS)：应用层主要是各个业务方或者部门基于 DWD 和 DWS 建立的数据集市(Data Market, DM)，一般来说应用层的数据来源于 DW 层，而且相对于 DW 层，应用层只包含部门或者业务方面自己关心的明细层和汇总层的数据。该层主要是提供数据产品和数据分析使用的数据。一般就直接对接 OLAP 分析,或者业务层数据调用接口了。

数据应用层 APP：面向业务定制的应用数据主要提供给数据铲平和数据分析使用的数据，一般会放在 ES，MYSQL，Oracle，Redis 等系统供线上系统使用，也可以放在 Hive 或者 Druid 中供数据分析和数据挖掘使用。APP 层：为应用层，这层数据是完全为了满足具体的分析需求而构建的数据，也是星形或雪花结构的数据。如我们经常说的报表数据，或者说那种大宽表，一般就放在这里。包括前端报表、分析图表、KPI、仪表盘、OLAP、专题等分析，面向最终结果用户；

ADS 层复购率统计：

![ADS 层复购率](https://assets.ng-tech.icu/item/20230325153713.png)

![计算 GMV](https://assets.ng-tech.icu/item/20230325153743.png)

```sql

CREATE TABLE app_usr_interact( user_id string COMMENT '用户id',
nickname string COMMENT '用户昵称',
register_date string COMMENT '注册日期',
register_from string COMMENT '注册来源',
remark string COMMENT '细分渠道',
province string COMMENT '注册省份',
pl_cnt bigint COMMENT '评论次数',
ds_cnt bigint COMMENT '打赏次数',
sc_add bigint COMMENT '添加收藏',
sc_cancel bigint COMMENT '取消收藏',
gzg_add bigint COMMENT '关注商品',
gzg_cancel bigint COMMENT '取消关注商品',
gzp_add bigint COMMENT '关注人',
gzp_cancel bigint COMMENT '取消关注人',
buzhi_cnt bigint COMMENT '点不值次数',
zhi_cnt bigint COMMENT '点值次数',
zan_cnt bigint COMMENT '点赞次数',
share_cnts bigint COMMENT '分享次数',
bl_cnt bigint COMMENT '爆料数',
fb_cnt bigint COMMENT '好价发布数',
online_cnt bigint COMMENT '活跃次数',
checkin_cnt bigint COMMENT '签到次数',
fix_checkin bigint COMMENT '补签次数',
house_point bigint COMMENT '幸运屋金币抽奖次数',
house_gold bigint COMMENT '幸运屋积分抽奖次数',
pack_cnt bigint COMMENT '礼品兑换次数',
gold_add bigint COMMENT '获取金币',
gold_cancel bigint COMMENT '支出金币',
surplus_gold bigint COMMENT '剩余金币',
event bigint COMMENT '电商点击次数',
gmv_amount bigint COMMENT 'gmv',
gmv_sales bigint COMMENT '订单数'
)
PARTITIONED BY( dt string)
--stat_dt
date COMMENT '互动日期',
```
