---
slug: /guides/sre/scaling-clusters
sidebar_label: 重新平衡分片
sidebar_position: 20
description: ClickHouse 不支持自动分片重新平衡，因此我们提供了一些最佳实践来重新平衡分片。
---


# 数据重新平衡

ClickHouse 不支持自动分片重新平衡。然而，按照偏好顺序，有几种方法可以重新平衡分片：

1. 调整 [分布式表](/engines/table-engines/special/distributed.md) 的分片，从而允许写入偏向新的分片。这可能会导致集群的负载不平衡和热点，但在写入吞吐量不是极高的大多数场景中是可行的。用户无需更改其写入目标，即可以保持为分布式表。这对于重新平衡现有数据没有帮助。

2. 作为对（1）的替代方案，修改现有集群并仅对新分片进行写入，直到集群达到平衡 - 手动加权写入。这与（1）具有相同的限制。

3. 如果您需要重新平衡现有数据，并且您已经对数据进行了分区，请考虑分离分区并手动将其迁移到另一节点，然后再重新附加到新分片。这比后续技术更手动，但可能更快且资源消耗更少。这是一个手动操作，因此需要考虑数据的重新平衡。

4. 通过 [INSERT FROM SELECT](/sql-reference/statements/insert-into.md/#inserting-the-results-of-select) 将数据从源集群导出到新集群。对于非常大的数据集，这将表现不佳，并可能在源集群上产生显著的 IO，并使用大量网络资源。这是最后的手段。
