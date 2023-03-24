# OLAP

![OLTP ETL OLAP](https://pic.imgdb.cn/item/6185f7a92ab3f51d9175a4cc.jpg)

OLAP（On-line Analytical Processing，联机分析处理）是在基于数据仓库多维模型的基础上实现的面向分析的各类操作的集合。可以比较下其与传统的 OLTP（On-line Transaction Processing，联机事务处理）的区别来看一下它的特点：

![OLAP 与 OLTP 对比](https://pic.imgdb.cn/item/6185f7fd2ab3f51d91761ae1.jpg)

OLAP 的优势是基于数据仓库面向主题、集成的、保留历史及不可变更的数据存储，以及多维模型多视角多层次的数据组织形式，如果脱离的这两点，OLAP 将不复存在，也就没有优势可言。

# OLAP 引擎的常见操作

![OLAP 引擎的常见操作](https://pic.imgdb.cn/item/6185fe852ab3f51d917ddb8f.jpg)

下面所述几种 OLAP 操作，是针对 Kimball 的星型模型（Star Schema）和雪花模型（Snowflake Schema）来说的。在 Kimball 模型中，定义了事实和维度。

- 上卷（Roll Up）/聚合：选定某些维度，根据这些维度来聚合事实，如果用 SQL 来表达就是 select dim_a, aggs_func(fact_b) from fact_table group by dim_a.
- 下钻（Drill Down）：上卷和下钻是相反的操作。它是选定某些维度，将这些维度拆解出小的维度（如年拆解为月，省份拆解为城市），之后聚合事实。
- 切片（Slicing、Dicing）：选定某些维度，并根据特定值过滤这些维度的值，将原来的大 Cube 切成小 cube。如 dim_a in ('CN', 'USA')
- 旋转（Pivot/Rotate）：维度位置的互换。

![OLAP 数据常见操作](https://pic.imgdb.cn/item/61862f132ab3f51d91c2e978.jpg)

# OLAP 分类

![OLAP 分类](https://pic.imgdb.cn/item/61862ffe2ab3f51d91c47c5a.jpg)

OLAP 是一种让用户可以用从不同视角方便快捷的分析数据的计算方法。主流的 OLAP 可以分为 3 类：多维 OLAP (Multi-dimensional OLAP)、关系型 OLAP(Relational OLAP) 和混合 OLAP(Hybrid OLAP) 三大类。

## MOLAP 的优点和缺点

MOLAP 的典型代表是：Druid，Kylin，Doris，MOLAP 一般会根据用户定义的数据维度、度量（也可以叫指标）在数据写入时生成预聚合数据；Query 查询到来时，实际上查询的是预聚合的数据而不是原始明细数据，在查询模式相对固定的场景中，这种优化提速很明显。

MOLAP 的优点和缺点都来自于其数据预处理（pre-processing）环节。数据预处理，将原始数据按照指定的计算规则预先做聚合计算，这样避免了查询过程中出现大量的即使计算，提升了查询性能。但是这样的预聚合处理，需要预先定义维度，会限制后期数据查询的灵活性；如果查询工作涉及新的指标，需要重新增加预处理流程，损失了灵活度，存储成本也很高；同时，这种方式不支持明细数据的查询，仅适用于聚合型查询（如：sum，avg，count）。

因此，MOLAP 适用于查询场景相对固定并且对查询性能要求非常高的场景。如广告主经常使用的广告投放报表分析。

## ROLAP 的优点和缺点

ROLAP 的典型代表是：Presto，Impala，GreenPlum，ClickHouse，Elasticsearch，Hive，Spark SQL，Flink SQL

数据写入时，ROLAP 并未使用像 MOLAP 那样的预聚合技术；ROLAP 收到 Query 请求时，会先解析 Query，生成执行计划，扫描数据，执行关系型算子，在原始数据上做过滤(Where)、聚合(Sum, Avg, Count)、关联(Join)，分组（Group By)、排序（Order By）等，最后将结算结果返回给用户，整个过程都是即时计算，没有预先聚合好的数据可供优化查询速度，拼的都是资源和算力的大小。

ROLAP 不需要进行数据预处理 ( pre-processing )，因此查询灵活，可扩展性好。这类引擎使用 MPP 架构 ( 与 Hadoop 相似的大型并行处理架构，可以通过扩大并发来增加计算资源 )，可以高效处理大量数据。

但是当数据量较大或 query 较为复杂时，查询性能也无法像 MOLAP 那样稳定。所有计算都是即时触发 ( 没有预处理 )，因此会耗费更多的计算资源，带来潜在的重复计算。

因此，ROLAP 适用于对查询模式不固定、查询灵活性要求高的场景。如数据分析师常用的数据分析类产品，他们往往会对数据做各种预先不能确定的分析，所以需要更高的查询灵活性。

## HOLAP

混合 OLAP，是 MOLAP 和 ROLAP 的一种融合。当查询聚合性数据的时候，使用 MOLAP 技术；当查询明细数据时，使用 ROLAP 技术。在给定使用场景的前提下，以达到查询性能的最优化。

顺便提一下，国内外有一些闭源的商业 OLAP 引擎，没有在这里归类和介绍，主要是因为使用的公司不多并且源码不可见、资料少，很难分析学习其中的源码和技术点。在一二线的互联网公司中，应用较为广泛的还是上面提到的各种 OLAP 引擎，如果你希望能够通过掌握一种 OLAP 技术，学习这些就够了。