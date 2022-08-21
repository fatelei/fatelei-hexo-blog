title: mysql driver InterpolateParams
date: 2022-08-21 13:32:32
tags: [golang, sql]
---

最近在使用 [sqlhooks](https://github.com/qustavo/sqlhooks) 进行 `sql` 执行过程中的错误
指标收集，上线后发现错误数很高，基本和执行成功数一致。为此对 `sqlhooks` 的 `onError` 增加了日志，
发现错误：`driver: skip fast-path; continue as if unimplemented`。

经过搜索引擎查询后，定位到了这个错误定义在：[driver.go](https://github.com/golang/go/blob/master/src/database/sql/driver/driver.go#L148)

这里对该错误有具体的说明：

```
ErrSkip may be returned by some optional interfaces' methods to
indicate at runtime that the fast path is unavailable and the sql
package should continue as if the optional interface was not
implemented. ErrSkip is only supported where explicitly
documented.
```

大意是：ErrSkip 可以被可选 interface 的方法返回，表面在运行时 fast path 不可用。如果 interface 未实现，sql 包应该继续执行。

SQL 执行经过了 `prepare` -> `exec` -> `close` 这三个步骤。

这里已 golang 的 `mysql driver` 的 `query` 为例：


`sql` package 在执行 `query` 的时候，会调用对应 `driver` 实现的 `QueryContext` 进行执行，`mysql` 的 `QueryContext` 会调用 `query` 方法，该方法的代码如下：

```go
func (mc *mysqlConn) query(query string, args []driver.Value) (*textRows, error) {
	if mc.closed.Load() {
		errLog.Print(ErrInvalidConn)
		return nil, driver.ErrBadConn
	}
	if len(args) != 0 {
		if !mc.cfg.InterpolateParams {
			return nil, driver.ErrSkip
		}
		// try client-side prepare to reduce roundtrip
		prepared, err := mc.interpolateParams(query, args)
		if err != nil {
			return nil, err
		}
		query = prepared
	}
	// Send command
	err := mc.writeCommandPacketStr(comQuery, query)
	if err == nil {
		// Read Result
		var resLen int
		resLen, err = mc.readResultSetHeaderPacket()
		if err == nil {
			rows := new(textRows)
			rows.mc = mc

			if resLen == 0 {
				rows.rs.done = true

				switch err := rows.NextResultSet(); err {
				case nil, io.EOF:
					return rows, nil
				default:
					return nil, err
				}
			}

			// Columns
			rows.rs.columns, err = mc.readColumns(resLen)
			return rows, err
		}
	}
	return nil, mc.markBadConn(err)
}
```

当 SQL 执行的是待有参数的，如：`SELECT * FROM foo WHERE id = ?`，会先判断 `driver` 是否启用了
`InterpolateParams`，如果没有启用，则会调用 `mysql` 执行 [prepare](https://cs.opensource.google/go/go/+/master:src/database/sql/sql.go;l=1778;drc=8d57f4dcef5d69a0a3f807afaa9625018569010b?q=QueryRow&ss=go%2Fgo) 逻辑；如果开启，则会在 client 侧进行 `prepare`，减少一次 `roundtrip`。

因此本人对于 `fast path` 理解是指是否允许在 `client` 侧进行 `prepare` 减少一次对 `mysql` 的调用。

这里有一段简单的 perf 对比：https://github.com/fatelei/go-benchmark-result/issues/3 。