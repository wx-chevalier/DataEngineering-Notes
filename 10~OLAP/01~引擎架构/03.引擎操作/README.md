# OLAP 引擎的常见操作

![OLAP 引擎的常见操作](https://pic.imgdb.cn/item/6185fe852ab3f51d917ddb8f.jpg)

下面所述几种 OLAP 操作，是针对 Kimball 的星型模型（Star Schema）和雪花模型（Snowflake Schema）来说的。在 Kimball 模型中，定义了事实和维度。

- 上卷（Roll Up）/聚合：选定某些维度，根据这些维度来聚合事实，如果用 SQL 来表达就是 select dim_a, aggs_func(fact_b) from fact_table group by dim_a.
- 下钻（Drill Down）：上卷和下钻是相反的操作。它是选定某些维度，将这些维度拆解出小的维度（如年拆解为月，省份拆解为城市），之后聚合事实。
- 切片（Slicing、Dicing）：选定某些维度，并根据特定值过滤这些维度的值，将原来的大 Cube 切成小 cube。如 dim_a in ('CN', 'USA')
- 旋转（Pivot/Rotate）：维度位置的互换。

![OLAP 数据常见操作](https://pic.imgdb.cn/item/61862f132ab3f51d91c2e978.jpg)
