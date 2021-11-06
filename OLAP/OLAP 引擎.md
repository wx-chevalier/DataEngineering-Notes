# OLAP 引擎

一个普遍的共识是，没有一个 OLAP 引擎能同时在数据量，灵活性和性能这三个方面做到完美，用户需要基于自己的需求进行取舍和选型。预计算模式的 OLAP 引擎在查询响应时间上相较于 MPP 引擎（Impala、SparkSQL、Presto 等）有一定优势，但相对限制了灵活性。

![OLTP ETL OLAP](https://pic.imgdb.cn/item/6185f7a92ab3f51d9175a4cc.jpg)

OLAP（On-line Analytical Processing，联机分析处理）是在基于数据仓库多维模型的基础上实现的面向分析的各类操作的集合。可以比较下其与传统的 OLTP（On-line Transaction Processing，联机事务处理）的区别来看一下它的特点：

![OLAP 与 OLTP 对比](https://pic.imgdb.cn/item/6185f7fd2ab3f51d91761ae1.jpg)
