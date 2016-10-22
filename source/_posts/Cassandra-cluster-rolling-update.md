title: Cassandra cluster rolling update.
date: 2016-10-22 21:49:26
tags: [cassandra]
---

最初部署 cassandra 时，采用 cassandra 的 2.2.1 版本，但是在实际使用中遇到了，如下 BUG：
- [[CASSANDRA-10079] LEAK DETECTED, after nodetool drain](https://issues.apache.org/jira/browse/CASSANDRA-10079)

- [[CASSANDRA-10501] Failure to start up Cassandra when temporary compaction files are not all renamed after kill/crash (FSReadError)](https://issues.apache.org/jira/browse/CASSANDRA-10501)

- [[CASSANDRA-10961] Not enough bytes error when add nodes to cluster](https://issues.apache.org/jira/browse/CASSANDRA-10961)

对线上有很严重的影响，特别是最后一个，直接导致每次新加节点到已有的集群时，都会出现加入不了的情况，基本上就不能横向扩展，T T。

既然那就升级吧，分布式数据有个好处就是，做 rolling update 特别的容易，如果不升级主版本，那就更容易了。

## 插曲

虽然 apache cassandra 2.2.x 系列的支持是到今年 11 月就截止的，但是不想一次升级太大，就选择了 2.2.x 最新的一个版本 2.2.7 不过一升级就出问题，最后不得不只升级到 2.2.5。

## 升级步骤

1. 升级某个节点的 cassandra 版本时，需要先执行

```
nodetool drain
```

该命令表示该节点不再接受写的请求，并将 memtable 进行 flush 到磁盘；

2. 进行数据的备份，可以使用

```
nodetool snapshot
```

进行 keyspace 的快照。由于目前所在的公司使用的是 AWS，这里没有使用 cassandra 自己提供的工具进行快照，而是使用的 AWS 提供的工具，对整块 EBS 进行的快照。

3. 数据备份完之后，stop cassandra 进程，可以用

```
cassandra_install_location/bin/stop-server
```

停止掉服务

4. 将老版本的中必要的配置，拷贝到新版本的目录中，比如：cassandra.yml

5. 以上完成之后，便可以直接启动新版本的 cassandra 啦，然后观察一下 cassandra 服务日志，如果没有问题，那就大功告成啦。注意，如果你升级的是主版本（1.x - 2.x）或者主版本下的一个大版本（2.1 - 2.2），在启动 cassandra 之后，还需要执行

```
nodetool upgradesstables
```

6. 然后再各个节点重复 1 - 5 的步骤。

7. 升级完之后，需要重启你的应用。

是不是很简单呢……

## 参考资料

- [Upgrading procedures for Apache Cassandra™](https://docs.datastax.com/en/latest-upgrade/upgrade/cassandra/upgrdCassandraDetails.html)