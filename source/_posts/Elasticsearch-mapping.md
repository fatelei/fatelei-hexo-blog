title: Elasticsearch mapping 设计总结
date: 2016-06-29 10:15:53
tags: [Elasticsearch, Indices, Mapping]
---

该规范主要是减少创建 indices 时，由于设计不当造成的性能下降，以及造成的 Elasticsearch 集群不稳定。

<!-- more -->

+ 禁用 mapping 的 dynamic
当 Indices 的 Mapping 已经确定时（即不会改变时），强烈建议将 dynamic 设置为 false，这样可以避免非法数据被 Elasticsearch 索引。

+ 避免不必要的 tokenizer
Elasticsearch 在进行 analyzer 时，会经过以下三个步骤：character -> tokenizer -> token filer，当 text 不需要进行 tokenizer 时，需要设置 `index: not_analyzed`。考虑以下场景：搜索地理位置时，需要输入完整地理信息，这个时候对于地理信息就不需要再去做 `tokenizer`。

+ 使用 nested array 支持 custom 搜索
当需要将用户传入的自定义字段存储在 elasticsearch，作为搜索的 keyword 时，不要使用 kv，而是使用 nested array 进行存储，这样用户的输入是可以预期的，而 kv 则是无法预期的，会造成 Mapping explosion。

+ 设置合理的 number_of_shards
number_of_shards 决定 Indices 在 Elasticsearch 集群中，如何均衡的分布在各个 data node，而使用 Elasticsearch 进行搜索时，Elasticsearch 会并行的查询分布在各个 data node 的 shard（而都在同一个节点的 shards，只能进行串行的操作），最后将各个 data node 返回的数据进行聚合，并返回给客户端。举个例子，比如当前 Elasticsearch 集群有 3 个 data node，索引 foo 设置了 number_of_shards 为 3，这样 3 个 shard 就均分在 3 个节点，这样查询时，可以并行查询 3 个节点。但是这样的设计也有缺陷，单个 shard 的数据量变大了，并且当 data node 扩展到 6 个节点，该 indice 的 shard 不具备扩展性，即不能够利用到这 6 个节点。所以在设计之初，可以将 numer_of_shards 设计的大一些，这样在后续对 data node 进行扩展时，现有 indices 也可以利用到新增的节点。依旧是上面的例子，初始时 indices 的 number_of_shards 设置为 6，这样 data node 扩展到 6 个，elasticsearch 可以把这 6 个 shard，均分到 6 个节点。

+ 合理设置 routing key
可以通过合理设置 routing key，可以避免在查询时查询多个 shards。因为相同的 routing key 都在同一个 shard。**注意：单个 shard 数据最好不要超过 30 GB。**


参考链接：
1. https://www.elastic.co/blog/found-crash-elasticsearch
2. https://www.elastic.co/blog/found-beginner-troubleshooting
3. http://cpratt.co/how-many-shards-should-elasticsearch-indexes-have/