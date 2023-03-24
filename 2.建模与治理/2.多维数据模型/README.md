# 多维数据模型

在 OLAP 中，我们通常会通过 Schema 来定义一个多维数据库，它是一个逻辑概念上的模型，其中包含 Cube（立方体）、Dimension（维度）、Hierarchy（层次）、Level（级别）、Measure（度量），这些被映射到数据库物理模型。

- Cube（立方体）是一系列 Dimension 和 Measure 的集合区域，它们共用一个事实表。

- Dimension（维度）是一个 Hierarchy 的集合，维度一般有其相对应的维度表，它由 Hierarchy（层次）组成，而 Hierarchy（层次）又是由组成 Level（级别）的。

- Hierarchy（层次）是指定维度的层级关系的，如果没有指定，默认 Hierarchy 里面装的是来自立方体中的真实表。

- Level（级别）是 Hierarchy 的组成部分，使用它可以构成一个结构树，Level 的先后顺序决定了 Level 在结构树上的位置，最顶层的 Level 位于树的第一级，依次类推。

- Measure（度量）是我们要进行度量计算的数值，支持的操作有 sum、count、avg、distinct-count、max、min 等。

概括总结一下：在多维分析中，关注的内容通常被称为度量(Measure)，而把限制条件称为维度(Dimension)。多维分析就是对同时满足多种限制条件的所有度量值做汇总统计。包含度量值的表被称为事实表(Fact Table)，描述维度具体信息的表被称为维表(Dimension Table)，同时有一点需要注意：并不是所有的维度都要有维表，对于取值简单的维度，可以直接使用事实表中的一列作为维度展示。
