title: sqlalchemy 使用小记（1）
date: 2016-03-26 16:48:39
tags: [sqlalchemy, orm]
---

最近本人所在的公司，偶尔会有在线上改数据库的需求，特别是移除 `table` 中的 `column`.
由于在业务中使用了 [sqlalchemy](http://www.sqlalchemy.org/)，并使用了其 `orm` 的方式进行对 database
的操作. 所以这里要删除 `table` 中的某个 `column` 时候，可以采用先更新对应的 `orm` 代码，然后再修改（或者根本不需要修改），
这样更平滑的方式进行操作，尽量减少对用户的影响。

<!-- more --> 

比如，有以下的 `table` 定义：

```
create table user (
  `id` int auto_increment,
  `name` varchar(32),
  `fullname` varchar(32),
  `password` varchar(32),
  primary key (`id`)
);
```

其对应的 `orm model` 定义如下：

```
# -*- coding: utf8 -*-

from sqlalchemy import Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class User(Base):

    __tablename__ = "user"

    id = Column(Integer, primary_key=True)
    name = Column(String)
    fullname = Column(String)
    password = Column(String)
```

假设这个时候，我们觉得 `fullname` 是多余的，想要移除它，只需要将其先从 `User` 中的定义移除即可，再加上 `fullname`
本身是可以为空的，所以在使用 `User` 进行 `insert` 操作也完全没有问题，同样的查询也不会有任何的问题。在查询的时候 `sqlalchemy`
只会查询 `User` 这个 `model` 上定义了属性，转换为 `SQL` 语句为：

+ 没有移除之前，select 语句为：

```
SELECT user.id AS user_id, user.name AS user_name, user.fullname as fullname, user.password as password FROM user
```

+ 移除 fullname 之后，select 语句为：

```
SELECT user.id AS user_id, user.name AS user_name, user.password as password FROM user
```

所以，如果需要移除一些没有约束条件的 `column` 时，完全可以先修改 `orm` 对应的代码，然后重启服务即可。
