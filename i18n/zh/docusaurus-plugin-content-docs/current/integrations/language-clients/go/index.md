---
sidebar_label: 'Go'
sidebar_position: 1
keywords: ['clickhouse', 'go', 'client', 'golang']
slug: /integrations/go
description: 'Go客户端用来连接ClickHouse, 提供Go标准的数据库/sql接口或优化过的本地接口。'
---

import ConnectionDetails from '@site/i18n/zh/docusaurus-plugin-content-docs/current/_snippets/_gather_your_details_native.md';

# ClickHouse Go
## 一个简单的例子 {#a-simple-example}
让我们从一个简单的例子开始。这将连接到ClickHouse并从系统数据库中选择。要开始，您需要您的连接详细信息。
### 连接详情 {#connection-details}
<ConnectionDetails />
### 初始化模块 {#initialize-a-module}

```bash
mkdir clickhouse-golang-example
cd clickhouse-golang-example
go mod init clickhouse-golang-example
```
### 复制一些示例代码 {#copy-in-some-sample-code}

将此代码复制到 `clickhouse-golang-example` 目录下，命名为 `main.go`。

```go title=main.go
package main

import (
	"context"
	"crypto/tls"
	"fmt"
	"log"

	"github.com/ClickHouse/clickhouse-go/v2"
	"github.com/ClickHouse/clickhouse-go/v2/lib/driver"
)

func main() {
	conn, err := connect()
	if err != nil {
		panic((err))
	}

	ctx := context.Background()
	rows, err := conn.Query(ctx, "SELECT name,toString(uuid) as uuid_str FROM system.tables LIMIT 5")
	if err != nil {
		log.Fatal(err)
	}

	for rows.Next() {
		var (
			name, uuid string
		)
		if err := rows.Scan(
			&name,
			&uuid,
		); err != nil {
			log.Fatal(err)
		}
		log.Printf("name: %s, uuid: %s",
			name, uuid)
	}

}

func connect() (driver.Conn, error) {
	var (
		ctx       = context.Background()
		conn, err = clickhouse.Open(&clickhouse.Options{
			Addr: []string{"<CLICKHOUSE_SECURE_NATIVE_HOSTNAME>:9440"},
			Auth: clickhouse.Auth{
				Database: "default",
				Username: "default",
				Password: "<DEFAULT_USER_PASSWORD>",
			},
			ClientInfo: clickhouse.ClientInfo{
				Products: []struct {
					Name    string
					Version string
				}{
					{Name: "an-example-go-client", Version: "0.1"},
				},
			},

			Debugf: func(format string, v ...interface{}) {
				fmt.Printf(format, v)
			},
			TLS: &tls.Config{
				InsecureSkipVerify: true,
			},
		})
	)

	if err != nil {
		return nil, err
	}

	if err := conn.Ping(ctx); err != nil {
		if exception, ok := err.(*clickhouse.Exception); ok {
			fmt.Printf("Exception [%d] %s \n%s\n", exception.Code, exception.Message, exception.StackTrace)
		}
		return nil, err
	}
	return conn, nil
}
```
### 运行 go mod tidy {#run-go-mod-tidy}

```bash
go mod tidy
```
### 设置您的连接详情 {#set-your-connection-details}
之前您查看了您的连接信息。在 `main.go` 中的 `connect()` 函数中设置这些信息：

```go
func connect() (driver.Conn, error) {
  var (
    ctx       = context.Background()
    conn, err = clickhouse.Open(&clickhouse.Options{
    #highlight-next-line
      Addr: []string{"<CLICKHOUSE_SECURE_NATIVE_HOSTNAME>:9440"},
      Auth: clickhouse.Auth{
    #highlight-start
        Database: "default",
        Username: "default",
        Password: "<DEFAULT_USER_PASSWORD>",
    #highlight-end
      },
```
### 运行示例 {#run-the-example}
```bash
go run .
```
```response
2023/03/06 14:18:33 name: COLUMNS, uuid: 00000000-0000-0000-0000-000000000000
2023/03/06 14:18:33 name: SCHEMATA, uuid: 00000000-0000-0000-0000-000000000000
2023/03/06 14:18:33 name: TABLES, uuid: 00000000-0000-0000-0000-000000000000
2023/03/06 14:18:33 name: VIEWS, uuid: 00000000-0000-0000-0000-000000000000
2023/03/06 14:18:33 name: hourly_data, uuid: a4e36bd4-1e82-45b3-be77-74a0fe65c52b
```
### 深入了解 {#learn-more}
本类别中的其余文档详细介绍了ClickHouse Go客户端的细节。
## ClickHouse Go Client {#clickhouse-go-client}

ClickHouse支持两个官方的Go客户端。这些客户端是互补的，故意支持不同的用例。

* [clickhouse-go](https://github.com/ClickHouse/clickhouse-go) - 高级语言客户端，支持Go标准的数据库/sql接口或本地接口。
* [ch-go](https://github.com/ClickHouse/ch-go) - 低级客户端，仅支持本地接口。

clickhouse-go提供了高级接口，允许用户使用面向行的语义和对于数据类型宽松的批处理来查询和插入数据- 只要没有潜在的精度损失，值将被转换。与此同时，ch-go提供了优化过的列式接口，以提供快速的数据块流式处理，并在牺牲类型严格性和更复杂使用的情况下实现低CPU和内存开销。

从2.3版本开始，Clickhouse-go利用ch-go来处理低级功能，如编码、解码和压缩。请注意，clickhouse-go也支持Go的 `database/sql` 接口标准。这两个客户端都使用本地格式进行编码，以提供最佳性能，并能够通过ClickHouse本地协议进行通信。clickhouse-go也支持HTTP作为其传输机制，以便在用户需要代理或负载平衡流量的情况下。

在选择客户端库时，用户应了解各自的优缺点 - 参见 选择客户端库。

|               | 本地格式 | 本地协议 | HTTP协议 | 面向行的API | 面向列的API | 类型灵活性 | 压缩 | 查询占位符 |
|:-------------:|:-------------:|:---------------:|:-------------:|:------------------:|:---------------------:|:----------------:|:-----------:|:------------------:|
| clickhouse-go |       ✅       |        ✅        |       ✅       |          ✅         |           ✅           |         ✅        |      ✅      |          ✅         |
|     ch-go     |       ✅       |        ✅        |               |                    |           ✅           |                  |      ✅      |                    |
## 选择客户端 {#choosing-a-client}

选择客户端库取决于您的使用模式和对最佳性能的需求。在需要每秒数百万次插入的插入重负载使用案例中，我们建议使用低级客户端 [ch-go](https://github.com/ClickHouse/ch-go)。该客户端避免了将数据从面向行的格式转换为列所需的附加开销，因为ClickHouse的本地格式要求。此外，它避免了任何反射或使用 `interface{}`（`any`）类型以简化使用。

对于专注于聚合或较低吞吐插入工作负载的查询工作负载，[clickhouse-go](https://github.com/ClickHouse/clickhouse-go)提供了一种熟悉的`database/sql`接口以及更简单的行语义。用户还可以选择使用HTTP作为传输协议，并利用帮助函数来将行和结构体之间进行值的转换。
## clickhouse-go客户端 {#the-clickhouse-go-client}

clickhouse-go客户端提供了两种API接口与ClickHouse进行通信：

* ClickHouse特定客户端API
* `database/sql`标准 - Golang提供的SQL数据库通用接口。

尽管 `database/sql` 提供了一个与数据库无关的接口，允许开发者抽象他们的数据存储，但它强制执行某些类型和查询语义，这会影响性能。因此，应该在[性能重要](https://github.com/clickHouse/clickHouse-go#benchmark)的地方使用客户端特定API。不过，想要将ClickHouse集成到支持多种数据库工具中的用户，则可能更倾向于使用标准接口。

两个接口都使用[本地格式](/native-protocol/basics.md)和本地协议进行数据编码和通信。此外，标准接口支持HTTP协议的通信。

|                    | 本地格式 | 本地协议 | HTTP协议 | 批量写入支持 | 结构体转换 | 压缩 | 查询占位符 |
|:------------------:|:-------------:|:---------------:|:-------------:|:------------------:|:-----------------:|:-----------:|:------------------:|
|   ClickHouse API   |       ✅       |        ✅        |               |          ✅         |         ✅         |      ✅      |          ✅         |
| `database/sql` API |       ✅       |        ✅        |       ✅       |          ✅         |                   |      ✅      |          ✅         |
## 安装 {#installation}

v1 驱动程序已被弃用，并将不再提供功能更新或对新ClickHouse类型的支持。用户应迁移到v2，v2提供了更高的性能。

要安装2.x版客户端，请将软件包添加到您的go.mod文件中：

`require github.com/ClickHouse/clickhouse-go/v2 main`

或者，克隆该仓库：

```bash
git clone --branch v2 https://github.com/clickhouse/clickhouse-go.git $GOPATH/src/github
```

要安装其他版本，请相应地修改路径或分支名称。

```bash
mkdir my-clickhouse-app && cd my-clickhouse-app

cat > go.mod <<-END
  module my-clickhouse-app

  go 1.18

  require github.com/ClickHouse/clickhouse-go/v2 main
END

cat > main.go <<-END
  package main

  import (
    "fmt"
    "github.com/ClickHouse/clickhouse-go/v2"
  )

  func main() {
   conn, _ := clickhouse.Open(&clickhouse.Options{Addr: []string{"127.0.0.1:9000"}})
    v, _ := conn.ServerVersion()
    fmt.Println(v.String())
  }
END

go mod tidy
go run main.go

```
### 版本控制与兼容性 {#versioning--compatibility}

客户端的发布与ClickHouse无关。2.x代表当前正在开发的主要版本。所有版本的2.x应该彼此兼容。
#### ClickHouse兼容性 {#clickhouse-compatibility}

客户端支持：

- 所有目前支持的ClickHouse版本，记录在[此处](https://github.com/ClickHouse/ClickHouse/blob/master/SECURITY.md)。随着ClickHouse版本不再受支持，它们也不再被积极测试。
- ClickHouse的所有版本在客户端发布日期后的2年内。请注意，仅对LTS版本进行积极测试。
#### Golang兼容性 {#golang-compatibility}

| 客户端版本 | Golang版本 |
|:--------------:|:---------------:|
|  => 2.0 &lt;= 2.2 |    1.17, 1.18   |
|     >= 2.3     |       1.18      |
## ClickHouse客户端API {#clickhouse-client-api}

所有关于ClickHouse客户端API的代码示例可以在[此处](https://github.com/ClickHouse/clickhouse-go/tree/main/examples)找到。
### 连接 {#connecting}

以下示例返回服务器版本，演示了如何连接到ClickHouse - 假设ClickHouse没有安全保护，可以通过默认用户访问。

注意我们使用默认的本地端口进行连接。

```go
conn, err := clickhouse.Open(&clickhouse.Options{
    Addr: []string{fmt.Sprintf("%s:%d", env.Host, env.Port)},
    Auth: clickhouse.Auth{
        Database: env.Database,
        Username: env.Username,
        Password: env.Password,
    },
})
if err != nil {
    return err
}
v, err := conn.ServerVersion()
fmt.Println(v)
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/connect.go)

**对于后续所有示例，除非明确说明，我们假设已经创建并可用ClickHouse的 `conn` 变量。**
#### 连接设置 {#connection-settings}

打开连接时，可以使用Options结构体来控制客户端行为。以下设置是可用的：

* `Protocol` - 本地或HTTP。目前HTTP仅支持[database/sql API](#databasesql-api)。
* `TLS` - TLS选项。非nil值启用TLS。请参见[使用TLS](#using-tls)。
* `Addr` - 包含端口的地址切片。
* `Auth` - 认证详细信息。请参见[认证](#authentication)。
* `DialContext` - 自定义拨号函数，用于确定如何建立连接。
* `Debug` - true/false以启用调试。
* `Debugf` - 提供函数以消费调试输出。需要将 `debug` 设置为true。
* `Settings` - ClickHouse设置的映射。这些将应用于所有ClickHouse查询。[使用上下文](#using-context)允许每个查询设置。
* `Compression` - 为块启用压缩。请参见[压缩](#compression)。
* `DialTimeout` - 建立连接的最大时间。默认为 `1s`。
* `MaxOpenConns` - 可同时使用的最大连接数。可能会有更多或更少的连接在空闲池中，但此数字是可同时使用的。默认为 `MaxIdleConns+5`。
* `MaxIdleConns` - 池中保存的连接数量。连接将被重用（如可能）。默认为 `5`。
* `ConnMaxLifetime` - 保持连接可用的最大生命周期。默认为1小时。超出这个时间连接会被销毁，并根据需要向池中添加新连接。
* `ConnOpenStrategy` - 确定如何消耗节点地址列表并用于打开连接。请参见[连接到多个节点](#connecting-to-multiple-nodes)。
* `BlockBufferSize` - 一次解码到缓冲区的最大块数。较大的值会在牺牲内存的情况下增加并行性。块大小与查询相关联，因此，尽管您可以在连接时设置此参数，我们建议根据返回数据的不同进行覆盖。默认为 `2`。

```go
conn, err := clickhouse.Open(&clickhouse.Options{
    Addr: []string{fmt.Sprintf("%s:%d", env.Host, env.Port)},
    Auth: clickhouse.Auth{
        Database: env.Database,
        Username: env.Username,
        Password: env.Password,
    },
    DialContext: func(ctx context.Context, addr string) (net.Conn, error) {
        dialCount++
        var d net.Dialer
        return d.DialContext(ctx, "tcp", addr)
    },
    Debug: true,
    Debugf: func(format string, v ...interface{}) {
        fmt.Printf(format, v)
    },
    Settings: clickhouse.Settings{
        "max_execution_time": 60,
    },
    Compression: &clickhouse.Compression{
        Method: clickhouse.CompressionLZ4,
    },
    DialTimeout:      time.Duration(10) * time.Second,
    MaxOpenConns:     5,
    MaxIdleConns:     5,
    ConnMaxLifetime:  time.Duration(10) * time.Minute,
    ConnOpenStrategy: clickhouse.ConnOpenInOrder,
    BlockBufferSize: 10,
})
if err != nil {
    return err
}
```
[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/connect_settings.go)
#### 连接池 {#connection-pooling}

客户端维护连接池，并在需要时重复使用这些连接。在连接的生命周期内，最多将使用 `MaxOpenConns`，池的最大大小由 `MaxIdleConns` 控制。客户端会为每个查询执行获取池中的一个连接，并在重用时将其返回到池中。连接在批量的生命周期内使用，并在 `Send()` 时释放。

只有在用户将 `MaxOpenConns=1` 的情况下，池中同一连接才会被用于后续查询。这通常不需要，但在用户使用临时表的情况下，可能需要。

此外，请注意，`ConnMaxLifetime` 默认值为1小时。这可能导致在节点离开集群时对ClickHouse的负载出现不平衡。发生这种情况时，如果某个节点变得不可用，连接将平衡到其他节点。这些连接将保持，并将在默认情况下持续存在1小时，即使问题节点返回集群也不被刷新。在重负载情况下，考虑减少此值。
### 使用TLS {#using-tls}

在底层，所有客户端连接方法（`DSN/OpenDB/Open`）将使用[Go tls包](https://pkg.go.dev/crypto/tls)来建立安全连接。如果Options结构体包含一个非nil的 `tls.Config` 指针，客户端将知道使用TLS。

```go
env, err := GetNativeTestEnvironment()
if err != nil {
    return err
}
cwd, err := os.Getwd()
if err != nil {
    return err
}
t := &tls.Config{}
caCert, err := ioutil.ReadFile(path.Join(cwd, "../../tests/resources/CAroot.crt"))
if err != nil {
    return err
}
caCertPool := x509.NewCertPool()
successful := caCertPool.AppendCertsFromPEM(caCert)
if !successful {
    return err
}
t.RootCAs = caCertPool
conn, err := clickhouse.Open(&clickhouse.Options{
    Addr: []string{fmt.Sprintf("%s:%d", env.Host, env.SslPort)},
    Auth: clickhouse.Auth{
        Database: env.Database,
        Username: env.Username,
        Password: env.Password,
    },
    TLS: t,
})
if err != nil {
    return err
}
v, err := conn.ServerVersion()
if err != nil {
    return err
}
fmt.Println(v.String())
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/ssl.go)

这种最小的 `TLS.Config` 通常足以连接到ClickHouse服务器的安全本地端口（通常为9440）。如果ClickHouse服务器没有有效证书（过期、错误的主机名称、未经公认的根证书颁发机构的签名），可以将 `InsecureSkipVerify` 设为true，但强烈不建议这样做。

```go
conn, err := clickhouse.Open(&clickhouse.Options{
    Addr: []string{fmt.Sprintf("%s:%d", env.Host, env.SslPort)},
    Auth: clickhouse.Auth{
        Database: env.Database,
        Username: env.Username,
        Password: env.Password,
    },
    TLS: &tls.Config{
        InsecureSkipVerify: true,
    },
})
if err != nil {
    return err
}
v, err := conn.ServerVersion()
```
[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/ssl_no_verify.go)

如果需要其他TLS参数，应用程序代码应在 `tls.Config` 结构体中设置所需字段。这可能包括特定的加密套件、强制特定的TLS版本（如1.2或1.3）、添加内部CA证书链、添加客户端证书（和私钥）如果ClickHouse服务器需要，并且与其他更专业的安全设置相关的大多数其他选项。
### 认证 {#authentication}

在连接详细信息中指定一个Auth结构体，以指定用户名和密码。

```go
conn, err := clickhouse.Open(&clickhouse.Options{
    Addr: []string{fmt.Sprintf("%s:%d", env.Host, env.Port)},
    Auth: clickhouse.Auth{
        Database: env.Database,
        Username: env.Username,
        Password: env.Password,
    },
})
if err != nil {
    return err
}
if err != nil {
    return err
}
v, err := conn.ServerVersion()
```
[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/auth.go)
### 连接到多个节点 {#connecting-to-multiple-nodes}

可以通过 `Addr` 结构体指定多个地址。

```go
conn, err := clickhouse.Open(&clickhouse.Options{
    Addr: []string{"127.0.0.1:9001", "127.0.0.1:9002", fmt.Sprintf("%s:%d", env.Host, env.Port)},
    Auth: clickhouse.Auth{
        Database: env.Database,
        Username: env.Username,
        Password: env.Password,
    },
})
if err != nil {
    return err
}
v, err := conn.ServerVersion()
if err != nil {
    return err
}
fmt.Println(v.String())
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/1c0d81d0b1388dbb9e09209e535667df212f4ae4/examples/clickhouse_api/multi_host.go#L26-L45)

提供两种连接策略：

* `ConnOpenInOrder`（默认） - 地址按顺序被消费。只有在无法连接上列表中早期的地址时，后来的地址才会被使用。这实际上是一种故障转移策略。
* `ConnOpenRoundRobin` - 负载在地址间通过轮询策略进行平衡。

这可以通过选项 `ConnOpenStrategy` 来控制。

```go
conn, err := clickhouse.Open(&clickhouse.Options{
    Addr:             []string{"127.0.0.1:9001", "127.0.0.1:9002", fmt.Sprintf("%s:%d", env.Host, env.Port)},
    ConnOpenStrategy: clickhouse.ConnOpenRoundRobin,
    Auth: clickhouse.Auth{
        Database: env.Database,
        Username: env.Username,
        Password: env.Password,
    },
})
if err != nil {
    return err
}
v, err := conn.ServerVersion()
if err != nil {
    return err
}
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/1c0d81d0b1388dbb9e09209e535667df212f4ae4/examples/clickhouse_api/multi_host.go#L50-L67)
### 执行 {#execution}

可以通过 `Exec` 方法执行任意语句。这对于DDL和简单语句很有用。它不应被用于较大插入或查询迭代。

```go
conn.Exec(context.Background(), `DROP TABLE IF EXISTS example`)
err = conn.Exec(context.Background(), `
    CREATE TABLE IF NOT EXISTS example (
        Col1 UInt8,
        Col2 String
    ) engine=Memory
`)
if err != nil {
    return err
}
conn.Exec(context.Background(), "INSERT INTO example VALUES (1, 'test-1')")
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/exec.go)

请注意，可以将上下文传递到查询中。这可以用于传递特定查询级别的设置 - 参见[使用上下文](#using-context)。
### 批量插入 {#batch-insert}

要插入大量行，客户端提供批处理语义。这要求准备一个批次以便可以附加行。最后通过 `Send()` 方法发送这些数据。批次将在发送执行之前保存在内存中。

```go
conn, err := GetNativeConnection(nil, nil, nil)
if err != nil {
    return err
}
ctx := context.Background()
defer func() {
    conn.Exec(ctx, "DROP TABLE example")
}()
conn.Exec(context.Background(), "DROP TABLE IF EXISTS example")
err = conn.Exec(ctx, `
    CREATE TABLE IF NOT EXISTS example (
            Col1 UInt8
        , Col2 String
        , Col3 FixedString(3)
        , Col4 UUID
        , Col5 Map(String, UInt8)
        , Col6 Array(String)
        , Col7 Tuple(String, UInt8, Array(Map(String, String)))
        , Col8 DateTime
    ) Engine = Memory
`)
if err != nil {
    return err
}


batch, err := conn.PrepareBatch(ctx, "INSERT INTO example")
if err != nil {
    return err
}
for i := 0; i < 1000; i++ {
    err := batch.Append(
        uint8(42),
        "ClickHouse",
        "Inc",
        uuid.New(),
        map[string]uint8{"key": 1},             // Map(String, UInt8)
        []string{"Q", "W", "E", "R", "T", "Y"}, // Array(String)
        []interface{}{ // Tuple(String, UInt8, Array(Map(String, String)))
            "String Value", uint8(5), []map[string]string{
                {"key": "value"},
                {"key": "value"},
                {"key": "value"},
            },
        },
        time.Now(),
    )
    if err != nil {
        return err
    }
}
return batch.Send()
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/batch.go)

关于ClickHouse的建议同样适用于[这里](/guides/inserting-data#best-practices-for-inserts)。批次不应在Go例程之间共享- 每个例程构造一个单独的批次。

从上述示例中，请注意附加行时变量类型需要与列类型对齐。虽然这种映射通常显而易见，但该接口力求灵活，并在没有精度损失的前提下，尽可能进行类型转换。例如，以下示例演示了将字符串插入转换为datetime64的情况。

```go
batch, err := conn.PrepareBatch(ctx, "INSERT INTO example")
if err != nil {
    return err
}
for i := 0; i < 1000; i++ {
    err := batch.Append(
        "2006-01-02 15:04:05.999",
    )
    if err != nil {
        return err
    }
}
return batch.Send()
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/type_convert.go)


有关每列类型支持的go类型的完整汇总，请参见[类型转换](#type-conversions)。
### 查询行 {#querying-rows}

用户可以使用 `QueryRow` 方法查询单行，也可以通过 `Query` 获取迭代结果集的游标。前者接受用于反序列化数据的目标，后者则要求对每一行调用 `Scan`。

```go
row := conn.QueryRow(context.Background(), "SELECT * FROM example")
var (
    col1             uint8
    col2, col3, col4 string
    col5             map[string]uint8
    col6             []string
    col7             []interface{}
    col8             time.Time
)
if err := row.Scan(&col1, &col2, &col3, &col4, &col5, &col6, &col7, &col8); err != nil {
    return err
}
fmt.Printf("row: col1=%d, col2=%s, col3=%s, col4=%s, col5=%v, col6=%v, col7=%v, col8=%v\n", col1, col2, col3, col4, col5, col6, col7, col8)
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/query_row.go)

```go
rows, err := conn.Query(ctx, "SELECT Col1, Col2, Col3 FROM example WHERE Col1 >= 2")
if err != nil {
    return err
}
for rows.Next() {
    var (
        col1 uint8
        col2 string
        col3 time.Time
    )
    if err := rows.Scan(&col1, &col2, &col3); err != nil {
        return err
    }
    fmt.Printf("row: col1=%d, col2=%s, col3=%s\n", col1, col2, col3)
}
rows.Close()
return rows.Err()
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/query_rows.go)

注意在这两种情况下，我们都需要传递指向希望将各个列值序列化的变量的指针。这些变量必须按 `SELECT` 语句中指定的顺序传递 - 默认情况下，在使用 `SELECT *` 时将使用列声明的顺序，如上所示。

与插入类似，Scan方法要求目标变量具有适当的类型。这再次旨在保持灵活性，尽可能进行类型转换，前提是没有精度损失，例如，上述示例显示UUID列被读取到字符串变量中。有关每列类型支持的go类型的完整列表，请参见[类型转换](#type-conversions)。

最后，请注意可以将 `Context` 传递给 `Query` 和 `QueryRow` 方法。这可以用于查询级别的设置 - 有关更多详细信息，请参见[使用上下文](#using-context)。
### 异步插入 {#async-insert}

通过Async方法支持异步插入。这允许用户指定客户端是否应等待服务器完成插入，或者在数据被接收后立即响应。这有效地控制参数 [wait_for_async_insert](/operations/settings/settings#wait_for_async_insert)。

```go
conn, err := GetNativeConnection(nil, nil, nil)
if err != nil {
    return err
}
ctx := context.Background()
if err := clickhouse_tests.CheckMinServerServerVersion(conn, 21, 12, 0); err != nil {
    return nil
}
defer func() {
    conn.Exec(ctx, "DROP TABLE example")
}()
conn.Exec(ctx, `DROP TABLE IF EXISTS example`)
const ddl = `
    CREATE TABLE example (
            Col1 UInt64
        , Col2 String
        , Col3 Array(UInt8)
        , Col4 DateTime
    ) ENGINE = Memory
`
if err := conn.Exec(ctx, ddl); err != nil {
    return err
}
for i := 0; i < 100; i++ {
    if err := conn.AsyncInsert(ctx, fmt.Sprintf(`INSERT INTO example VALUES (
        %d, '%s', [1, 2, 3, 4, 5, 6, 7, 8, 9], now()
    )`, i, "Golang SQL database driver"), false); err != nil {
        return err
    }
}
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/async.go)
### 列式插入 {#columnar-insert}

插入可以以列格式进行。这在数据已经以该结构进行排列时，可以提供性能优势，避免需要转换为行。

```go
batch, err := conn.PrepareBatch(context.Background(), "INSERT INTO example")
if err != nil {
    return err
}
var (
    col1 []uint64
    col2 []string
    col3 [][]uint8
    col4 []time.Time
)
for i := 0; i < 1_000; i++ {
    col1 = append(col1, uint64(i))
    col2 = append(col2, "Golang SQL database driver")
    col3 = append(col3, []uint8{1, 2, 3, 4, 5, 6, 7, 8, 9})
    col4 = append(col4, time.Now())
}
if err := batch.Column(0).Append(col1); err != nil {
    return err
}
if err := batch.Column(1).Append(col2); err != nil {
    return err
}
if err := batch.Column(2).Append(col3); err != nil {
    return err
}
if err := batch.Column(3).Append(col4); err != nil {
    return err
}
return batch.Send()
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/columnar_insert.go)
### 使用结构体 {#using-structs}

对于用户而言，Golang结构体为ClickHouse中的数据行提供了逻辑表示。为此，本地接口提供了几个方便的函数。
#### 使用序列化选择 {#select-with-serialize}

Select方法允许一组响应行通过单次调用被打包到结构体切片中。

```go
var result []struct {
    Col1           uint8
    Col2           string
    ColumnWithName time.Time `ch:"Col3"`
}

if err = conn.Select(ctx, &result, "SELECT Col1, Col2, Col3 FROM example"); err != nil {
    return err
}

for _, v := range result {
    fmt.Printf("row: col1=%d, col2=%s, col3=%s\n", v.Col1, v.Col2, v.ColumnWithName)
}
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/select_struct.go)
#### 扫描结构体 {#scan-struct}

`ScanStruct` 允许将查询的单个行打包到结构体中。

```go
var result struct {
    Col1  int64
    Count uint64 `ch:"count"`
}
if err := conn.QueryRow(context.Background(), "SELECT Col1, COUNT() AS count FROM example WHERE Col1 = 5 GROUP BY Col1").ScanStruct(&result); err != nil {
    return err
}
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/scan_struct.go)
#### Append Struct {#append-struct}

`AppendStruct` 允许将结构体附加到现有的 [batch](#batch-insert) 中，并被解释为完整的行。这要求结构体的列在名称和类型上都与表对齐。虽然所有列必须具有相应的结构字段，但某些结构字段可能没有相应的列表示。这些字段将被忽略。

```go
batch, err := conn.PrepareBatch(context.Background(), "INSERT INTO example")
if err != nil {
    return err
}
for i := 0; i < 1_000; i++ {
    err := batch.AppendStruct(&row{
        Col1:       uint64(i),
        Col2:       "Golang SQL database driver",
        Col3:       []uint8{1, 2, 3, 4, 5, 6, 7, 8, 9},
        Col4:       time.Now(),
        ColIgnored: "this will be ignored",
    })
    if err != nil {
        return err
    }
}
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/append_struct.go)
### 类型转换 {#type-conversions}

客户端旨在尽可能灵活地接受各种类型用于插入和响应的反序列化。在大多数情况下，ClickHouse 列类型都有相应的 Golang 类型，例如 [UInt64](/sql-reference/data-types/int-uint/) 对应 [uint64](https://pkg.go.dev/builtin#uint64)。这些逻辑映射应始终得到支持。用户可能希望使用可以插入到列中的变量类型，或者在首先转换变量或接收到的数据时使用。客户端旨在透明地支持这些转换，因此用户无需在插入之前精确地转换数据，并在查询时提供灵活的反序列化。这种透明转换不允许精度丧失。例如，无法使用 uint32 从 UInt64 列接收数据。相反，只要满足格式要求，可以将字符串插入到 datetime64 字段中。

目前，对于基本类型支持的类型转换可以在 [这里](https://github.com/ClickHouse/clickhouse-go/blob/main/TYPES.md) 查看。

这一努力仍在进行中，并可以分为插入（`Append`/`AppendRow`）和读取时（通过 `Scan`）。如果您需要特定转换的支持，请提出问题。
### 复杂类型 {#complex-types}
#### 日期/日期时间类型 {#datedatetime-types}

ClickHouse Go 客户端支持 `Date`、`Date32`、`DateTime` 和 `DateTime64` 日期/日期时间类型。日期可以以 `2006-01-02` 格式的字符串或使用本地 Go 的 `time.Time{}` 或 `sql.NullTime` 插入。日期时间也支持后者类型，但需要以 `2006-01-02 15:04:05` 格式的字符串传入，并可选提供时区偏移，例如 `2006-01-02 15:04:05 +08:00`。在读取时，`time.Time{}` 和 `sql.NullTime` 也是受支持的，以及任何实现了 `sql.Scanner` 接口的实现方式。

时区信息的处理依赖于 ClickHouse 类型以及值是插入还是读取：

* **DateTime/DateTime64**
    * 在 **插入** 时，值以 UNIX 时间戳格式发送到 ClickHouse。如果未提供时区，客户端将假设为客户端的本地时区。`time.Time{}` 或 `sql.NullTime` 将相应地转换为纪元时间。
    * 在 **选择** 时，将使用列的时区（如果设置了）来返回 `time.Time` 值。如果没有，则使用服务器的时区。
* **Date/Date32**
    * 在 **插入** 时，考虑任何日期的时区在将日期转换为 UNIX 时间戳时，即在存储为日期之前将按时区进行偏移，因为 Date 类型在 ClickHouse 中没有本地设置。如果这在字符串值中未指定，将使用本地时区。
    * 在 **选择** 时，日期将被扫描到 `time.Time{}` 或 `sql.NullTime{}` 实例中而不带时区信息。
#### 数组 {#array}

数组应作为切片插入。元素的类型规则与 [基本类型](#type-conversions) 中的一致，即在可行的情况下，无论元素将被转换如何。

在扫描时应提供指向切片的指针。

```go
batch, err := conn.PrepareBatch(ctx, "INSERT INTO example")
if err != nil {
    return err
}
var i int64
for i = 0; i < 10; i++ {
    err := batch.Append(
        []string{strconv.Itoa(int(i)), strconv.Itoa(int(i + 1)), strconv.Itoa(int(i + 2)), strconv.Itoa(int(i + 3))},
        [][]int64{{i, i + 1}, {i + 2, i + 3}, {i + 4, i + 5}},
    )
    if err != nil {
        return err
    }
}
if err := batch.Send(); err != nil {
    return err
}
var (
    col1 []string
    col2 [][]int64
)
rows, err := conn.Query(ctx, "SELECT * FROM example")
if err != nil {
    return err
}
for rows.Next() {
    if err := rows.Scan(&col1, &col2); err != nil {
        return err
    }
    fmt.Printf("row: col1=%v, col2=%v\n", col1, col2)
}
rows.Close()
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/array.go)
#### 映射 {#map}

映射应作为 Golang 映射插入，键和值应符合前面定义的类型规则 [earlier](#type-conversions)。

```go
batch, err := conn.PrepareBatch(ctx, "INSERT INTO example")
if err != nil {
    return err
}
var i int64
for i = 0; i < 10; i++ {
    err := batch.Append(
        map[string]uint64{strconv.Itoa(int(i)): uint64(i)},
        map[string][]string{strconv.Itoa(int(i)): {strconv.Itoa(int(i)), strconv.Itoa(int(i + 1)), strconv.Itoa(int(i + 2)), strconv.Itoa(int(i + 3))}},
        map[string]map[string]uint64{strconv.Itoa(int(i)): {strconv.Itoa(int(i)): uint64(i)}},
    )
    if err != nil {
        return err
    }
}
if err := batch.Send(); err != nil {
    return err
}
var (
    col1 map[string]uint64
    col2 map[string][]string
    col3 map[string]map[string]uint64
)
rows, err := conn.Query(ctx, "SELECT * FROM example")
if err != nil {
    return err
}
for rows.Next() {
    if err := rows.Scan(&col1, &col2, &col3); err != nil {
        return err
    }
    fmt.Printf("row: col1=%v, col2=%v, col3=%v\n", col1, col2, col3)
}
rows.Close()
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/map.go)
#### 元组 {#tuples}

元组表示任意长度的列组。列可以显式命名，也可以只是指定类型，例如：

```sql
//unnamed
Col1 Tuple(String, Int64)

//named
Col2 Tuple(name String, id Int64, age uint8)
```

在这两种方法中，命名元组提供了更大的灵活性。虽然未命名的元组必须使用切片插入和读取，但命名元组也与映射兼容。

```go
if err = conn.Exec(ctx, `
    CREATE TABLE example (
            Col1 Tuple(name String, age UInt8),
            Col2 Tuple(String, UInt8),
            Col3 Tuple(name String, id String)
        )
        Engine Memory
    `); err != nil {
    return err
}

defer func() {
    conn.Exec(ctx, "DROP TABLE example")
}()
batch, err := conn.PrepareBatch(ctx, "INSERT INTO example")
if err != nil {
    return err
}
// both named and unnamed can be added with slices. Note we can use strongly typed lists and maps if all elements are the same type
if err = batch.Append([]interface{}{"Clicky McClickHouse", uint8(42)}, []interface{}{"Clicky McClickHouse Snr", uint8(78)}, []string{"Dale", "521211"}); err != nil {
    return err
}
if err = batch.Append(map[string]interface{}{"name": "Clicky McClickHouse Jnr", "age": uint8(20)}, []interface{}{"Baby Clicky McClickHouse", uint8(1)}, map[string]string{"name": "Geoff", "id": "12123"}); err != nil {
    return err
}
if err = batch.Send(); err != nil {
    return err
}
var (
    col1 map[string]interface{}
    col2 []interface{}
    col3 map[string]string
)
// named tuples can be retrieved into a map or slices, unnamed just slices
if err = conn.QueryRow(ctx, "SELECT * FROM example").Scan(&col1, &col2, &col3); err != nil {
    return err
}
fmt.Printf("row: col1=%v, col2=%v, col3=%v\n", col1, col2, col3)
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/tuple.go)

注意：支持类型切片和映射，前提是命名元组中的所有子列都是相同的类型。
#### 嵌套 {#nested}

嵌套字段相当于命名元组的数组。嵌套的使用取决于用户是否将 [flatten_nested](/operations/settings/settings#flatten_nested) 设置为 1 或 0。

通过将 flatten_nested 设置为 0，嵌套列保留为单个元组数组。这允许用户使用映射切片进行插入和检索，并支持任意级别的嵌套。映射的键必须等于列的名称，如下面的示例所示。

注意：由于映射代表一个元组，因此它们必须为 `map[string]interface{}` 类型。目前值尚未严格定义类型。

```go
conn, err := GetNativeConnection(clickhouse.Settings{
    "flatten_nested": 0,
}, nil, nil)
if err != nil {
    return err
}
ctx := context.Background()
defer func() {
    conn.Exec(ctx, "DROP TABLE example")
}()
conn.Exec(context.Background(), "DROP TABLE IF EXISTS example")
err = conn.Exec(ctx, `
    CREATE TABLE example (
        Col1 Nested(Col1_1 String, Col1_2 UInt8),
        Col2 Nested(
            Col2_1 UInt8,
            Col2_2 Nested(
                Col2_2_1 UInt8,
                Col2_2_2 UInt8
            )
        )
    ) Engine Memory
`)
if err != nil {
    return err
}

batch, err := conn.PrepareBatch(ctx, "INSERT INTO example")
if err != nil {
    return err
}
var i int64
for i = 0; i < 10; i++ {
    err := batch.Append(
        []map[string]interface{}{
            {
                "Col1_1": strconv.Itoa(int(i)),
                "Col1_2": uint8(i),
            },
            {
                "Col1_1": strconv.Itoa(int(i + 1)),
                "Col1_2": uint8(i + 1),
            },
            {
                "Col1_1": strconv.Itoa(int(i + 2)),
                "Col1_2": uint8(i + 2),
            },
        },
        []map[string]interface{}{
            {
                "Col2_2": []map[string]interface{}{
                    {
                        "Col2_2_1": uint8(i),
                        "Col2_2_2": uint8(i + 1),
                    },
                },
                "Col2_1": uint8(i),
            },
            {
                "Col2_2": []map[string]interface{}{
                    {
                        "Col2_2_1": uint8(i + 2),
                        "Col2_2_2": uint8(i + 3),
                    },
                },
                "Col2_1": uint8(i + 1),
            },
        },
    )
    if err != nil {
        return err
    }
}
if err := batch.Send(); err != nil {
    return err
}
var (
    col1 []map[string]interface{}
    col2 []map[string]interface{}
)
rows, err := conn.Query(ctx, "SELECT * FROM example")
if err != nil {
    return err
}
for rows.Next() {
    if err := rows.Scan(&col1, &col2); err != nil {
        return err
    }
    fmt.Printf("row: col1=%v, col2=%v\n", col1, col2)
}
rows.Close()
```

[完整示例 - `flatten_tested=0`](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/nested.go#L28-L118)

如果使用了 `flatten_nested` 的默认值 1，则嵌套列被扁平化为单独的数组。这需要使用嵌套切片进行插入和检索。虽然任意级别的嵌套可能会起作用，但这并未得到官方支持。

```go
conn, err := GetNativeConnection(nil, nil, nil)
if err != nil {
    return err
}
ctx := context.Background()
defer func() {
    conn.Exec(ctx, "DROP TABLE example")
}()
conn.Exec(ctx, "DROP TABLE IF EXISTS example")
err = conn.Exec(ctx, `
    CREATE TABLE example (
        Col1 Nested(Col1_1 String, Col1_2 UInt8),
        Col2 Nested(
            Col2_1 UInt8,
            Col2_2 Nested(
                Col2_2_1 UInt8,
                Col2_2_2 UInt8
            )
        )
    ) Engine Memory
`)
if err != nil {
    return err
}


batch, err := conn.PrepareBatch(ctx, "INSERT INTO example")
if err != nil {
    return err
}
var i uint8
for i = 0; i < 10; i++ {
    col1_1_data := []string{strconv.Itoa(int(i)), strconv.Itoa(int(i + 1)), strconv.Itoa(int(i + 2))}
    col1_2_data := []uint8{i, i + 1, i + 2}
    col2_1_data := []uint8{i, i + 1, i + 2}
    col2_2_data := [][][]interface{}{
        {
            {i, i + 1},
        },
        {
            {i + 2, i + 3},
        },
        {
            {i + 4, i + 5},
        },
    }
    err := batch.Append(
        col1_1_data,
        col1_2_data,
        col2_1_data,
        col2_2_data,
    )
    if err != nil {
        return err
    }
}
if err := batch.Send(); err != nil {
    return err
}
```

[完整示例 - `flatten_nested=1`](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/nested.go#L123-L180)


注意：嵌套列必须具有相同的维度。例如，在上述示例中，`Col_2_2` 和 `Col_2_1` 必须具有相同数量的元素。

由于接口更简单并且官方支持嵌套，建议使用 `flatten_nested=0`。
#### 地理类型 {#geo-types}

客户端支持地理类型 Point、Ring、Polygon 和 Multi Polygon。这些字段在 Golang 中使用包 [github.com/paulmach/orb](https://github.com/paulmach/orb)。

```go
if err = conn.Exec(ctx, `
    CREATE TABLE example (
            point Point,
            ring Ring,
            polygon Polygon,
            mPolygon MultiPolygon
        )
        Engine Memory
    `); err != nil {
    return err
}

batch, err := conn.PrepareBatch(ctx, "INSERT INTO example")
if err != nil {
    return err
}

if err = batch.Append(
    orb.Point{11, 22},
    orb.Ring{
        orb.Point{1, 2},
        orb.Point{1, 2},
    },
    orb.Polygon{
        orb.Ring{
            orb.Point{1, 2},
            orb.Point{12, 2},
        },
        orb.Ring{
            orb.Point{11, 2},
            orb.Point{1, 12},
        },
    },
    orb.MultiPolygon{
        orb.Polygon{
            orb.Ring{
                orb.Point{1, 2},
                orb.Point{12, 2},
            },
            orb.Ring{
                orb.Point{11, 2},
                orb.Point{1, 12},
            },
        },
        orb.Polygon{
            orb.Ring{
                orb.Point{1, 2},
                orb.Point{12, 2},
            },
            orb.Ring{
                orb.Point{11, 2},
                orb.Point{1, 12},
            },
        },
    },
); err != nil {
    return err
}

if err = batch.Send(); err != nil {
    return err
}

var (
    point    orb.Point
    ring     orb.Ring
    polygon  orb.Polygon
    mPolygon orb.MultiPolygon
)

if err = conn.QueryRow(ctx, "SELECT * FROM example").Scan(&point, &ring, &polygon, &mPolygon); err != nil {
    return err
}
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/geo.go)
#### UUID {#uuid}

UUID 类型由 [github.com/google/uuid](https://github.com/google/uuid) 包支持。用户还可以将 UUID 作为字符串或任何实现了 `sql.Scanner` 或 `Stringify` 的类型进行发送和序列化。

```go
if err = conn.Exec(ctx, `
    CREATE TABLE example (
            col1 UUID,
            col2 UUID
        )
        Engine Memory
    `); err != nil {
    return err
}

batch, err := conn.PrepareBatch(ctx, "INSERT INTO example")
if err != nil {
    return err
}
col1Data, _ := uuid.NewUUID()
if err = batch.Append(
    col1Data,
    "603966d6-ed93-11ec-8ea0-0242ac120002",
); err != nil {
    return err
}

if err = batch.Send(); err != nil {
    return err
}

var (
    col1 uuid.UUID
    col2 uuid.UUID
)

if err = conn.QueryRow(ctx, "SELECT * FROM example").Scan(&col1, &col2); err != nil {
    return err
}
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/uuid.go)
#### 十进制 {#decimal}

Decimal 类型由 [github.com/shopspring/decimal](https://github.com/shopspring/decimal) 包支持。

```go
if err = conn.Exec(ctx, `
    CREATE TABLE example (
        Col1 Decimal32(3),
        Col2 Decimal(18,6),
        Col3 Decimal(15,7),
        Col4 Decimal128(8),
        Col5 Decimal256(9)
    ) Engine Memory
    `); err != nil {
    return err
}

batch, err := conn.PrepareBatch(ctx, "INSERT INTO example")
if err != nil {
    return err
}
if err = batch.Append(
    decimal.New(25, 4),
    decimal.New(30, 5),
    decimal.New(35, 6),
    decimal.New(135, 7),
    decimal.New(256, 8),
); err != nil {
    return err
}

if err = batch.Send(); err != nil {
    return err
}

var (
    col1 decimal.Decimal
    col2 decimal.Decimal
    col3 decimal.Decimal
    col4 decimal.Decimal
    col5 decimal.Decimal
)

if err = conn.QueryRow(ctx, "SELECT * FROM example").Scan(&col1, &col2, &col3, &col4, &col5); err != nil {
    return err
}
fmt.Printf("col1=%v, col2=%v, col3=%v, col4=%v, col5=%v\n", col1, col2, col3, col4, col5)
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/decimal.go)
#### 可空类型 {#nullable}

Go 的 Nil 值表示 ClickHouse 的 NULL。这可以在字段声明为 Nullable 时使用。在插入时，可以将 Nil 传递给普通列和可空列的版本。对于前者，将保留类型的默认值，例如，字符串的空字符串。对于可空版本，将在 ClickHouse 中存储一个 NULL 值。

在扫描时，用户必须传递一个指向支持 nil 的类型的指针，例如，*string，以表示可空字段的 nil 值。在下面的示例中，col1 是一个 Nullable(String)，因此接收一个 **string。这允许表示 nil。

```go
if err = conn.Exec(ctx, `
    CREATE TABLE example (
            col1 Nullable(String),
            col2 String,
            col3 Nullable(Int8),
            col4 Nullable(Int64)
        )
        Engine Memory
    `); err != nil {
    return err
}

batch, err := conn.PrepareBatch(ctx, "INSERT INTO example")
if err != nil {
    return err
}
if err = batch.Append(
    nil,
    nil,
    nil,
    sql.NullInt64{Int64: 0, Valid: false},
); err != nil {
    return err
}

if err = batch.Send(); err != nil {
    return err
}

var (
    col1 *string
    col2 string
    col3 *int8
    col4 sql.NullInt64
)

if err = conn.QueryRow(ctx, "SELECT * FROM example").Scan(&col1, &col2, &col3, &col4); err != nil {
    return err
}
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/nullable.go)

客户端还支持 `sql.Null*` 类型，例如 `sql.NullInt64`。这些类型与其等价的 ClickHouse 类型兼容。
#### 大整数 - Int128、Int256、UInt128、UInt256 {#big-ints---int128-int256-uint128-uint256}

比 64 位更大的数字类型使用本地 Go 的 [big](https://pkg.go.dev/math/big) 包表示。

```go
if err = conn.Exec(ctx, `
    CREATE TABLE example (
        Col1 Int128,
        Col2 UInt128,
        Col3 Array(Int128),
        Col4 Int256,
        Col5 Array(Int256),
        Col6 UInt256,
        Col7 Array(UInt256)
    ) Engine Memory`); err != nil {
    return err
}

batch, err := conn.PrepareBatch(ctx, "INSERT INTO example")
if err != nil {
    return err
}

col1Data, _ := new(big.Int).SetString("170141183460469231731687303715884105727", 10)
col2Data := big.NewInt(128)
col3Data := []*big.Int{
    big.NewInt(-128),
    big.NewInt(128128),
    big.NewInt(128128128),
}
col4Data := big.NewInt(256)
col5Data := []*big.Int{
    big.NewInt(256),
    big.NewInt(256256),
    big.NewInt(256256256256),
}
col6Data := big.NewInt(256)
col7Data := []*big.Int{
    big.NewInt(256),
    big.NewInt(256256),
    big.NewInt(256256256256),
}

if err = batch.Append(col1Data, col2Data, col3Data, col4Data, col5Data, col6Data, col7Data); err != nil {
    return err
}

if err = batch.Send(); err != nil {
    return err
}

var (
    col1 big.Int
    col2 big.Int
    col3 []*big.Int
    col4 big.Int
    col5 []*big.Int
    col6 big.Int
    col7 []*big.Int
)

if err = conn.QueryRow(ctx, "SELECT * FROM example").Scan(&col1, &col2, &col3, &col4, &col5, &col6, &col7); err != nil {
    return err
}
fmt.Printf("col1=%v, col2=%v, col3=%v, col4=%v, col5=%v, col6=%v, col7=%v\n", col1, col2, col3, col4, col5, col6, col7)
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/big_int.go)
### 压缩 {#compression}

对压缩方法的支持取决于所使用的底层协议。对于本地协议，客户端支持 `LZ4` 和 `ZSTD` 压缩。这是在块级别进行的。通过在连接中包含 `Compression` 配置可以启用压缩。

```go
conn, err := clickhouse.Open(&clickhouse.Options{
    Addr: []string{fmt.Sprintf("%s:%d", env.Host, env.Port)},
    Auth: clickhouse.Auth{
        Database: env.Database,
        Username: env.Username,
        Password: env.Password,
    },
    Compression: &clickhouse.Compression{
        Method: clickhouse.CompressionZSTD,
    },
    MaxOpenConns: 1,
})
ctx := context.Background()
defer func() {
    conn.Exec(ctx, "DROP TABLE example")
}()
conn.Exec(context.Background(), "DROP TABLE IF EXISTS example")
if err = conn.Exec(ctx, `
    CREATE TABLE example (
            Col1 Array(String)
    ) Engine Memory
    `); err != nil {
    return err
}
batch, err := conn.PrepareBatch(ctx, "INSERT INTO example")
if err != nil {
    return err
}
for i := 0; i < 1000; i++ {
    if err := batch.Append([]string{strconv.Itoa(i), strconv.Itoa(i + 1), strconv.Itoa(i + 2), strconv.Itoa(i + 3)}); err != nil {
        return err
    }
}
if err := batch.Send(); err != nil {
    return err
}
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/compression.go)


如果使用标准接口通过 HTTP，则还可以使用其他压缩技术。有关更多详细信息，请参见 [database/sql API - Compression](#compression)。
### 参数绑定 {#parameter-binding}

客户端支持 `Exec`、`Query` 和 `QueryRow` 方法的参数绑定。如下面示例所示，支持使用命名、编号和位置参数。以下是这些参数的示例。

```go
var count uint64
// positional bind
if err = conn.QueryRow(ctx, "SELECT count() FROM example WHERE Col1 >= ? AND Col3 < ?", 500, now.Add(time.Duration(750)*time.Second)).Scan(&count); err != nil {
    return err
}
// 250
fmt.Printf("Positional bind count: %d\n", count)
// numeric bind
if err = conn.QueryRow(ctx, "SELECT count() FROM example WHERE Col1 <= $2 AND Col3 > $1", now.Add(time.Duration(150)*time.Second), 250).Scan(&count); err != nil {
    return err
}
// 100
fmt.Printf("Numeric bind count: %d\n", count)
// named bind
if err = conn.QueryRow(ctx, "SELECT count() FROM example WHERE Col1 <= @col1 AND Col3 > @col3", clickhouse.Named("col1", 100), clickhouse.Named("col3", now.Add(time.Duration(50)*time.Second))).Scan(&count); err != nil {
    return err
}
// 50
fmt.Printf("Named bind count: %d\n", count)
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/bind.go)
#### 特殊情况 {#special-cases}

默认情况下，如果将切片作为参数传递给查询，它们将展开为逗号分隔的值。如果用户需要一组值被注入并用 `[ ]` 包裹，则应使用 `ArraySet`。

如果需要带有包裹 `()` 的组/元组，例如用于 IN 操作，用户可以使用 `GroupSet`。这对于需要多个组的情况特别有用，如下面的示例所示。

最后，DateTime64 字段在确保参数正确渲染时需要精度。客户端对此字段的精度水平是未知的，用户必须提供它。为此，我们提供 `DateNamed` 参数。

```go
var count uint64
// arrays will be unfolded
if err = conn.QueryRow(ctx, "SELECT count() FROM example WHERE Col1 IN (?)", []int{100, 200, 300, 400, 500}).Scan(&count); err != nil {
    return err
}
fmt.Printf("Array unfolded count: %d\n", count)
// arrays will be preserved with []
if err = conn.QueryRow(ctx, "SELECT count() FROM example WHERE Col4 = ?", clickhouse.ArraySet{300, 301}).Scan(&count); err != nil {
    return err
}
fmt.Printf("Array count: %d\n", count)
// Group sets allow us to form ( ) lists
if err = conn.QueryRow(ctx, "SELECT count() FROM example WHERE Col1 IN ?", clickhouse.GroupSet{[]interface{}{100, 200, 300, 400, 500}}).Scan(&count); err != nil {
    return err
}
fmt.Printf("Group count: %d\n", count)
// More useful when we need nesting
if err = conn.QueryRow(ctx, "SELECT count() FROM example WHERE (Col1, Col5) IN (?)", []clickhouse.GroupSet{{[]interface{}{100, 101}}, {[]interface{}{200, 201}}}).Scan(&count); err != nil {
    return err
}
fmt.Printf("Group count: %d\n", count)
// Use DateNamed when you need a precision in your time#
if err = conn.QueryRow(ctx, "SELECT count() FROM example WHERE Col3 >= @col3", clickhouse.DateNamed("col3", now.Add(time.Duration(500)*time.Millisecond), clickhouse.NanoSeconds)).Scan(&count); err != nil {
    return err
}
fmt.Printf("NamedDate count: %d\n", count)
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/bind_special.go)
### 使用上下文 {#using-context}

Go 上下文提供了一种跨 API 边界传递截止日期、取消信号和其他请求范围值的方法。连接上的所有方法都接受上下文作为第一个变量。虽然之前的示例使用了 context.Background()，用户可以利用此功能传递设置和截止日期以及取消查询。

传递用 `withDeadline` 创建的上下文允许在查询上设置执行时间限制。请注意，这是绝对时间，超时只会释放连接并向 ClickHouse 发送取消信号。可以使用 `WithCancel` 来明确取消查询。

帮助者 `clickhouse.WithQueryID` 和 `clickhouse.WithQuotaKey` 允许指定查询 ID 和配额密钥。查询 ID 对于跟踪日志中的查询和取消目的很有用。配额键可用于根据唯一的键值限制 ClickHouse 的使用 - 有关详细信息，请参见 [配额管理](/operations/access-rights#quotas-management)。

用户还可以使用上下文确保设置仅适用于特定查询 - 而不是整个连接，具体请参考 [连接设置](#connection-settings)。

最后，用户可以通过 `clickhouse.WithBlockSize` 控制块缓存区的大小。这会覆盖连接级别设置 `BlockBufferSize`，并控制最多同时解码和保存在内存中的块数。较大的值可能意味着更多的并行化，但代价是占用内存。

下面展示了上述内容的示例。

```go
dialCount := 0
conn, err := clickhouse.Open(&clickhouse.Options{
    Addr: []string{fmt.Sprintf("%s:%d", env.Host, env.Port)},
    Auth: clickhouse.Auth{
        Database: env.Database,
        Username: env.Username,
        Password: env.Password,
    },
    DialContext: func(ctx context.Context, addr string) (net.Conn, error) {
        dialCount++
        var d net.Dialer
        return d.DialContext(ctx, "tcp", addr)
    },
})
if err != nil {
    return err
}
if err := clickhouse_tests.CheckMinServerServerVersion(conn, 22, 6, 1); err != nil {
    return nil
}
// we can use context to pass settings to a specific API call
ctx := clickhouse.Context(context.Background(), clickhouse.WithSettings(clickhouse.Settings{
    "allow_experimental_object_type": "1",
}))

conn.Exec(ctx, "DROP TABLE IF EXISTS example")

// to create a JSON column we need allow_experimental_object_type=1
if err = conn.Exec(ctx, `
    CREATE TABLE example (
            Col1 JSON
        )
        Engine Memory
    `); err != nil {
    return err
}

// queries can be cancelled using the context
ctx, cancel := context.WithCancel(context.Background())
go func() {
    cancel()
}()
if err = conn.QueryRow(ctx, "SELECT sleep(3)").Scan(); err == nil {
    return fmt.Errorf("expected cancel")
}

// set a deadline for a query - this will cancel the query after the absolute time is reached.
// queries will continue to completion in ClickHouse
ctx, cancel = context.WithDeadline(context.Background(), time.Now().Add(-time.Second))
defer cancel()
if err := conn.Ping(ctx); err == nil {
    return fmt.Errorf("expected deadline exceeeded")
}

// set a query id to assist tracing queries in logs e.g. see system.query_log
var one uint8
queryId, _ := uuid.NewUUID()
ctx = clickhouse.Context(context.Background(), clickhouse.WithQueryID(queryId.String()))
if err = conn.QueryRow(ctx, "SELECT 1").Scan(&one); err != nil {
    return err
}

conn.Exec(context.Background(), "DROP QUOTA IF EXISTS foobar")
defer func() {
    conn.Exec(context.Background(), "DROP QUOTA IF EXISTS foobar")
}()
ctx = clickhouse.Context(context.Background(), clickhouse.WithQuotaKey("abcde"))
// set a quota key - first create the quota
if err = conn.Exec(ctx, "CREATE QUOTA IF NOT EXISTS foobar KEYED BY client_key FOR INTERVAL 1 minute MAX queries = 5 TO default"); err != nil {
    return err
}

type Number struct {
    Number uint64 `ch:"number"`
}
for i := 1; i <= 6; i++ {
    var result []Number
    if err = conn.Select(ctx, &result, "SELECT number FROM numbers(10)"); err != nil {
        return err
    }
}
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/context.go)
```yaml
title: '进度/配置/日志信息'
sidebar_label: '进度/配置/日志信息'
keywords: ['进度', '配置', '日志', 'ClickHouse', '数据库']
description: '此页面提供有关ClickHouse中查询的进度、配置和日志信息的详细说明'
```

### 进度/配置/日志信息 {#progressprofilelog-information}

可以请求查询的进度、配置和日志信息。进度信息将报告在ClickHouse中已读取和处理的行和字节的统计信息。相反，配置信息提供了返回给客户端的数据摘要，包括字节（未压缩）、行和块的总数。最后，日志信息提供线程的统计信息，例如内存使用情况和数据速度。

获取这些信息需要用户使用 [Context](#using-context)，用户可以传递回调函数。

```go
totalRows := uint64(0)
// 使用上下文传递进度和配置信息的回调
ctx := clickhouse.Context(context.Background(), clickhouse.WithProgress(func(p *clickhouse.Progress) {
    fmt.Println("进度: ", p)
    totalRows += p.Rows
}), clickhouse.WithProfileInfo(func(p *clickhouse.ProfileInfo) {
    fmt.Println("配置信息: ", p)
}), clickhouse.WithLogs(func(log *clickhouse.Log) {
    fmt.Println("日志信息: ", log)
}))

rows, err := conn.Query(ctx, "SELECT number from numbers(1000000) LIMIT 1000000")
if err != nil {
    return err
}
for rows.Next() {
}

fmt.Printf("总行数: %d\n", totalRows)
rows.Close()
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/progress.go)

### 动态扫描 {#dynamic-scanning}

用户可能需要读取他们不知道架构或字段类型的表。这在进行临时数据分析或编写通用工具时很常见。为此，查询响应中提供列类型信息。这可以与Go反射结合使用，以创建正确类型变量的运行时实例，这些变量可以传递给Scan。

```go
const query = `
SELECT
        1     AS Col1
    , 'Text' AS Col2
`
rows, err := conn.Query(context.Background(), query)
if err != nil {
    return err
}
var (
    columnTypes = rows.ColumnTypes()
    vars        = make([]interface{}, len(columnTypes))
)
for i := range columnTypes {
    vars[i] = reflect.New(columnTypes[i].ScanType()).Interface()
}
for rows.Next() {
    if err := rows.Scan(vars...); err != nil {
        return err
    }
    for _, v := range vars {
        switch v := v.(type) {
        case *string:
            fmt.Println(*v)
        case *uint8:
            fmt.Println(*v)
        }
    }
}
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/dynamic_scan_types.go)

### 外部表 {#external-tables}

[外部表](/engines/table-engines/special/external-data/)允许客户端通过SELECT查询将数据发送到ClickHouse。此数据放入临时表，并可以在查询本身中用于评估。

要通过查询将外部数据发送到客户端，用户必须通过`ext.NewTable`构建外部表，然后通过上下文传递这一信息。

```go
table1, err := ext.NewTable("external_table_1",
    ext.Column("col1", "UInt8"),
    ext.Column("col2", "String"),
    ext.Column("col3", "DateTime"),
)
if err != nil {
    return err
}

for i := 0; i < 10; i++ {
    if err = table1.Append(uint8(i), fmt.Sprintf("value_%d", i), time.Now()); err != nil {
        return err
    }
}

table2, err := ext.NewTable("external_table_2",
    ext.Column("col1", "UInt8"),
    ext.Column("col2", "String"),
    ext.Column("col3", "DateTime"),
)

for i := 0; i < 10; i++ {
    table2.Append(uint8(i), fmt.Sprintf("value_%d", i), time.Now())
}
ctx := clickhouse.Context(context.Background(),
    clickhouse.WithExternalTable(table1, table2),
)
rows, err := conn.Query(ctx, "SELECT * FROM external_table_1")
if err != nil {
    return err
}
for rows.Next() {
    var (
        col1 uint8
        col2 string
        col3 time.Time
    )
    rows.Scan(&col1, &col2, &col3)
    fmt.Printf("col1=%d, col2=%s, col3=%v\n", col1, col2, col3)
}
rows.Close()

var count uint64
if err := conn.QueryRow(ctx, "SELECT COUNT(*) FROM external_table_1").Scan(&count); err != nil {
    return err
}
fmt.Printf("external_table_1: %d\n", count)
if err := conn.QueryRow(ctx, "SELECT COUNT(*) FROM external_table_2").Scan(&count); err != nil {
    return err
}
fmt.Printf("external_table_2: %d\n", count)
if err := conn.QueryRow(ctx, "SELECT COUNT(*) FROM (SELECT * FROM external_table_1 UNION ALL SELECT * FROM external_table_2)").Scan(&count); err != nil {
    return err
}
fmt.Printf("external_table_1 UNION external_table_2: %d\n", count)
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/external_data.go)

### 开放遥测 {#open-telemetry}

ClickHouse允许通过本地协议传递[跟踪上下文](/operations/opentelemetry/)。客户端允许通过`clickhouse.withSpan`函数创建一个Span，并通过Context传递以实现这一点。

```go
var count uint64
rows := conn.QueryRow(clickhouse.Context(context.Background(), clickhouse.WithSpan(
    trace.NewSpanContext(trace.SpanContextConfig{
        SpanID:  trace.SpanID{1, 2, 3, 4, 5},
        TraceID: trace.TraceID{5, 4, 3, 2, 1},
    }),
)), "SELECT COUNT() FROM (SELECT number FROM system.numbers LIMIT 5)")
if err := rows.Scan(&count); err != nil {
    return err
}
fmt.Printf("count: %d\n", count)
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/clickhouse_api/open_telemetry.go)

关于如何利用跟踪的更多详细信息可以在[开放遥测支持](/operations/opentelemetry/)中找到。
## 数据库/SQL API {#databasesql-api}

`database/sql` 或 "标准" API 允许用户在应用代码应该对底层数据库保持无关的场景中使用客户端，通过符合标准接口。这样做会带来一些代价 - 额外的抽象和间接层以及一些不一定与ClickHouse对齐的原语。然而，这些成本在需要连接多个数据库的工具场景中通常是可以接受的。

此外，该客户端支持使用HTTP作为传输层 - 数据仍将以本地格式编码，以获得最佳性能。

以下旨在镜像ClickHouse API的文档结构。

标准API的完整代码示例可以在[这里](https://github.com/ClickHouse/clickhouse-go/tree/main/examples/std)找到。
### 连接 {#connecting-1}

可以通过格式为`clickhouse://<host>:<port>?<query_option>=<value>`的DSN字符串和`Open`方法，或通过`clickhouse.OpenDB`方法实现连接。后者不是`database/sql`规范的一部分，但返回`sql.DB`实例。此方法提供诸如配置的功能，后者在`database/sql`规范中没有明显的公开手段。

```go
func Connect() error {
	env, err := GetStdTestEnvironment()
	if err != nil {
		return err
	}
	conn := clickhouse.OpenDB(&clickhouse.Options{
		Addr: []string{fmt.Sprintf("%s:%d", env.Host, env.Port)},
		Auth: clickhouse.Auth{
			Database: env.Database,
			Username: env.Username,
			Password: env.Password,
		},
	})
	return conn.Ping()
}

func ConnectDSN() error {
	env, err := GetStdTestEnvironment()
	if err != nil {
		return err
	}
	conn, err := sql.Open("clickhouse", fmt.Sprintf("clickhouse://%s:%d?username=%s&password=%s", env.Host, env.Port, env.Username, env.Password))
	if err != nil {
		return err
	}
	return conn.Ping()
}
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/std/connect.go)

**对于所有后续示例，除非明确说明，否则我们假定使用ClickHouse `conn` 变量已创建并可用。**
#### 连接设置 {#connection-settings-1}

在DSN字符串中可以传递以下参数：

* `hosts` - 用于负载均衡和故障切换的逗号分隔单地址主机列表 - 参见 [连接多个节点](#connecting-to-multiple-nodes)。
* `username/password` - 验证凭据 - 参见 [身份验证](#authentication)
* `database` - 选择当前默认数据库
* `dial_timeout` - 持续时间字符串可能是带符号的十进制数序列，每个数字后可选分数和单位后缀，如 `300ms`、`1s`。有效的时间单位为`ms`、`s`、`m`。
* `connection_open_strategy` - `random/in_order`（默认`random`） - 参见 [连接多个节点](#connecting-to-multiple-nodes)
    - `round_robin` - 从集合中选择轮询服务器
    - `in_order` - 按指定顺序选择第一个活动服务器
* `debug` - 启用调试输出（布尔值）
* `compress` - 指定压缩算法 - `none`（默认）、`zstd`、`lz4`、`gzip`、`deflate`、`br`。如果设置为`true`，则将使用`lz4`。仅支持`lz4`和`zstd`用于本地通信。
* `compress_level` - 压缩级别（默认是`0`）。参见压缩。这是算法特定的：
    - `gzip` - `-2`（最佳速度）到 `9`（最佳压缩）
    - `deflate` - `-2`（最佳速度）到 `9`（最佳压缩）
    - `br` - `0`（最佳速度）到 `11`（最佳压缩）
    - `zstd`、`lz4` - 被忽略
* `secure` - 建立安全的SSL连接（默认是`false`）
* `skip_verify` - 跳过证书验证（默认是`false`）
* `block_buffer_size` - 允许用户控制块缓冲区大小。参见 [`BlockBufferSize`](#connection-settings)。 （默认是`2`）

```go
func ConnectSettings() error {
	env, err := GetStdTestEnvironment()
	if err != nil {
		return err
	}
	conn, err := sql.Open("clickhouse", fmt.Sprintf("clickhouse://127.0.0.1:9001,127.0.0.1:9002,%s:%d/%s?username=%s&password=%s&dial_timeout=10s&connection_open_strategy=round_robin&debug=true&compress=lz4", env.Host, env.Port, env.Database, env.Username, env.Password))
	if err != nil {
		return err
	}
	return conn.Ping()
}
```
[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/std/connect_settings.go)

#### 连接池 {#connection-pooling-1}

用户可以影响所提供的节点地址列表的使用，如在 [连接多个节点](#connecting-to-multiple-nodes) 中所述。不过，连接管理和池化设计上委托给了`sql.DB`。
#### 通过HTTP连接 {#connecting-over-http}

默认情况下，连接是通过本地协议建立的。对于需要HTTP的用户，可以通过修改DSN以包括HTTP协议或在连接选项中指定协议来启用此功能。

```go
func ConnectHTTP() error {
	env, err := GetStdTestEnvironment()
	if err != nil {
		return err
	}
	conn := clickhouse.OpenDB(&clickhouse.Options{
		Addr: []string{fmt.Sprintf("%s:%d", env.Host, env.HttpPort)},
		Auth: clickhouse.Auth{
			Database: env.Database,
			Username: env.Username,
			Password: env.Password,
		},
		Protocol: clickhouse.HTTP,
	})
	return conn.Ping()
}

func ConnectDSNHTTP() error {
	env, err := GetStdTestEnvironment()
	if err != nil {
		return err
	}
	conn, err := sql.Open("clickhouse", fmt.Sprintf("http://%s:%d?username=%s&password=%s", env.Host, env.HttpPort, env.Username, env.Password))
	if err != nil {
		return err
	}
	return conn.Ping()
}
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/std/connect_http.go)

#### 连接多个节点 {#connecting-to-multiple-nodes-1}

如果使用`OpenDB`，可以使用与ClickHouse API相同的选项方法连接多个主机 - 可选择指定`ConnOpenStrategy`。

对于基于DSN的连接，该字符串接受多个主机和一个`connection_open_strategy`参数，其值可以设置为`round_robin`或`in_order`。

```go
func MultiStdHost() error {
	env, err := GetStdTestEnvironment()
	if err != nil {
		return err
	}
	conn, err := clickhouse.Open(&clickhouse.Options{
		Addr: []string{"127.0.0.1:9001", "127.0.0.1:9002", fmt.Sprintf("%s:%d", env.Host, env.Port)},
		Auth: clickhouse.Auth{
			Database: env.Database,
			Username: env.Username,
			Password: env.Password,
		},
		ConnOpenStrategy: clickhouse.ConnOpenRoundRobin,
	})
	if err != nil {
		return err
	}
	v, err := conn.ServerVersion()
	if err != nil {
		return err
	}
	fmt.Println(v.String())
	return nil
}

func MultiStdHostDSN() error {
	env, err := GetStdTestEnvironment()
	if err != nil {
		return err
	}
	conn, err := sql.Open("clickhouse", fmt.Sprintf("clickhouse://127.0.0.1:9001,127.0.0.1:9002,%s:%d?username=%s&password=%s&connection_open_strategy=round_robin", env.Host, env.Port, env.Username, env.Password))
	if err != nil {
		return err
	}
	return conn.Ping()
}
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/std/multi_host.go)

### 使用TLS {#using-tls-1}

如果使用DSN连接字符串，则可以通过参数"secure=true"启用SSL。`OpenDB`方法采用与[本地API中的TLS](#using-tls)相同的方法，依赖于指定一个非空的TLS结构。虽然DSN连接字符串支持参数skip_verify来跳过SSL验证，但要求使用`OpenDB`方法进行更高级的TLS配置，因为它允许传递配置。

```go
func ConnectSSL() error {
	env, err := GetStdTestEnvironment()
	if err != nil {
		return err
	}
	cwd, err := os.Getwd()
	if err != nil {
		return err
	}
	t := &tls.Config{}
	caCert, err := ioutil.ReadFile(path.Join(cwd, "../../tests/resources/CAroot.crt"))
	if err != nil {
		return err
	}
	caCertPool := x509.NewCertPool()
	successful := caCertPool.AppendCertsFromPEM(caCert)
	if !successful {
		return err
	}
	t.RootCAs = caCertPool


	conn := clickhouse.OpenDB(&clickhouse.Options{
		Addr: []string{fmt.Sprintf("%s:%d", env.Host, env.SslPort)},
		Auth: clickhouse.Auth{
			Database: env.Database,
			Username: env.Username,
			Password: env.Password,
		},
		TLS: t,
	})
	return conn.Ping()
}

func ConnectDSNSSL() error {
	env, err := GetStdTestEnvironment()
	if err != nil {
		return err
	}
	conn, err := sql.Open("clickhouse", fmt.Sprintf("https://%s:%d?secure=true&skip_verify=true&username=%s&password=%s", env.Host, env.HttpsPort, env.Username, env.Password))
	if err != nil {
		return err
	}
	return conn.Ping()
}
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/std/ssl.go)

### 身份验证 {#authentication-1}

如果使用`OpenDB`，可以通过常规选项传递身份验证信息。对于基于DSN的连接，可以在连接字符串中传递用户名和密码 - 可以作为参数或作为编码在地址中的凭据。

```go
func ConnectAuth() error {
	env, err := GetStdTestEnvironment()
	if err != nil {
		return err
	}
	conn := clickhouse.OpenDB(&clickhouse.Options{
		Addr: []string{fmt.Sprintf("%s:%d", env.Host, env.Port)},
		Auth: clickhouse.Auth{
			Database: env.Database,
			Username: env.Username,
			Password: env.Password,
		},
	})
	return conn.Ping()
}

func ConnectDSNAuth() error {
	env, err := GetStdTestEnvironment()
	conn, err := sql.Open("clickhouse", fmt.Sprintf("http://%s:%d?username=%s&password=%s", env.Host, env.HttpPort, env.Username, env.Password))
	if err != nil {
		return err
	}
	if err = conn.Ping(); err != nil {
		return err
	}
	conn, err = sql.Open("clickhouse", fmt.Sprintf("http://%s:%s@%s:%d", env.Username, env.Password, env.Host, env.HttpPort))
	if err != nil {
		return err
	}
	return conn.Ping()
}
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/std/auth.go)

### 执行 {#execution-1}

一旦获取了连接，用户可以通过Exec方法发出`sql`语句以执行。

```go
conn.Exec(`DROP TABLE IF EXISTS example`)
_, err = conn.Exec(`
    CREATE TABLE IF NOT EXISTS example (
        Col1 UInt8,
        Col2 String
    ) engine=Memory
`)
if err != nil {
    return err
}
_, err = conn.Exec("INSERT INTO example VALUES (1, 'test-1')")
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/std/exec.go)


此方法不支持接收上下文 - 默认情况下，它使用后台上下文执行。用户可以使用`ExecContext`，如果需要的话 - 参见 [使用上下文](#using-context)。
### 批量插入 {#batch-insert-1}

通过使用`Being`方法创建`sql.Tx`可以实现批量语义。由此，可以使用`Prepare`方法与`INSERT`语句获得一个批量。这返回一个`sql.Stmt`，可以使用`Exec`方法将行附加到其中。批量将被积累在内存中，直到在原始`sql.Tx`上执行`Commit`。

```go
batch, err := scope.Prepare("INSERT INTO example")
if err != nil {
    return err
}
for i := 0; i < 1000; i++ {
    _, err := batch.Exec(
        uint8(42),
        "ClickHouse", "Inc",
        uuid.New(),
        map[string]uint8{"key": 1},             // Map(String, UInt8)
        []string{"Q", "W", "E", "R", "T", "Y"}, // Array(String)
        []interface{}{ // Tuple(String, UInt8, Array(Map(String, String)))
            "String Value", uint8(5), []map[string]string{
                map[string]string{"key": "value"},
                map[string]string{"key": "value"},
                map[string]string{"key": "value"},
            },
        },
        time.Now(),
    )
    if err != nil {
        return err
    }
}
return scope.Commit()
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/std/batch.go)

### 查询行 {#querying-rows-1}

通过`QueryRow`方法可以查询单行。这返回一个*sql.Row，可以在其上调用Scan，并传入指向变量的指针以将列映射到这些变量中。`QueryRowContext`变体允许在后台以外的上下文。

```go
row := conn.QueryRow("SELECT * FROM example")
var (
    col1             uint8
    col2, col3, col4 string
    col5             map[string]uint8
    col6             []string
    col7             interface{}
    col8             time.Time
)
if err := row.Scan(&col1, &col2, &col3, &col4, &col5, &col6, &col7, &col8); err != nil {
    return err
}
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/std/query_row.go)

迭代多个行需要`Query`方法。这返回一个`*sql.Rows`结构，可以在其上调用Next以迭代这些行。`QueryContext`等效方法允许传递上下文。

```go
rows, err := conn.Query("SELECT * FROM example")
if err != nil {
    return err
}
var (
    col1             uint8
    col2, col3, col4 string
    col5             map[string]uint8
    col6             []string
    col7             interface{}
    col8             time.Time
)
for rows.Next() {
    if err := rows.Scan(&col1, &col2, &col3, &col4, &col5, &col6, &col7, &col8); err != nil {
        return err
    }
    fmt.Printf("行: col1=%d, col2=%s, col3=%s, col4=%s, col5=%v, col6=%v, col7=%v, col8=%v\n", col1, col2, col3, col4, col5, col6, col7, col8)
}
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/std/query_rows.go)

### 异步插入 {#async-insert-1}

可以通过`ExecContext`方法执行插入以实现异步插入。这里应该传递一个启用了异步模式的上下文，如下所示。这允许用户指定客户端是否应等待服务器完成插入，或在数据被接收后立即响应。这有效地控制参数 [wait_for_async_insert](/operations/settings/settings#wait_for_async_insert)。

```go
const ddl = `
    CREATE TABLE example (
            Col1 UInt64
        , Col2 String
        , Col3 Array(UInt8)
        , Col4 DateTime
    ) ENGINE = Memory
    `
if _, err := conn.Exec(ddl); err != nil {
    return err
}
ctx := clickhouse.Context(context.Background(), clickhouse.WithStdAsync(false))
{
    for i := 0; i < 100; i++ {
        _, err := conn.ExecContext(ctx, fmt.Sprintf(`INSERT INTO example VALUES (
            %d, '%s', [1, 2, 3, 4, 5, 6, 7, 8, 9], now()
        )`, i, "Golang SQL database driver"))
        if err != nil {
            return err
        }
    }
}
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/std/async.go)

### 列式插入 {#columnar-insert-1}

不支持使用标准接口。
### 使用结构体 {#using-structs-1}

不支持使用标准接口。
### 类型转换 {#type-conversions-1}

标准`database/sql`接口应支持与[ClickHouse API](#type-conversions)相同的类型。存在一些例外情况，主要是对于复杂类型，详细信息如下。与ClickHouse API类似，客户端旨在尽可能灵活地接受插入和响应的变量类型。有关更多详细信息，请参见[类型转换](#type-conversions)。
### 复杂类型 {#complex-types-1}

除非另有说明，否则复杂类型处理应与[ClickHouse API](#complex-types)相同。差异源于`database/sql`内部。
#### 映射 {#maps}

与ClickHouse API不同，标准API要求在扫描类型时强类型映射。例如，用户不能为`Map(String,String)`字段传递`map[string]interface{}`，而必须使用`map[string]string`。`interface{}`变量将始终兼容，并可以用于更复杂的结构。读取时不支持结构体。

```go
var (
    col1Data = map[string]uint64{
        "key_col_1_1": 1,
        "key_col_1_2": 2,
    }
    col2Data = map[string]uint64{
        "key_col_2_1": 10,
        "key_col_2_2": 20,
    }
    col3Data = map[string]uint64{}
    col4Data = []map[string]string{
        {"A": "B"},
        {"C": "D"},
    }
    col5Data = map[string]uint64{
        "key_col_5_1": 100,
        "key_col_5_2": 200,
    }
)
if _, err := batch.Exec(col1Data, col2Data, col3Data, col4Data, col5Data); err != nil {
    return err
}
if err = scope.Commit(); err != nil {
    return err
}
var (
    col1 interface{}
    col2 map[string]uint64
    col3 map[string]uint64
    col4 []map[string]string
    col5 map[string]uint64
)
if err := conn.QueryRow("SELECT * FROM example").Scan(&col1, &col2, &col3, &col4, &col5); err != nil {
    return err
}
fmt.Printf("col1=%v, col2=%v, col3=%v, col4=%v, col5=%v", col1, col2, col3, col4, col5)
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/std/map.go)

插入行为与ClickHouse API相同。
### 压缩 {#compression-1}

标准API支持与本地 [ClickHouse API](#compression) 相同的压缩算法，即区块级的`lz4`和`zstd`压缩。此外，HTTP连接还支持gzip、deflate和br压缩。如果启用这些，将在插入和查询响应时对块进行压缩。其他请求，例如心跳或查询请求，将保持未压缩。这与`lz4`和`zstd`选项一致。

如果使用`OpenDB`方法建立连接，可以传递压缩配置。这包括指定压缩级别的能力（请参见下面）。如果通过包含DSN连接使用`sql.Open`，请使用参数`compress`。这可以是特定的压缩算法，例如`gzip`、`deflate`、`br`、`zstd`或`lz4`，也可以是布尔标志。如果设置为true，将使用`lz4`。默认为`none`，即禁用压缩。

```go
conn := clickhouse.OpenDB(&clickhouse.Options{
    Addr: []string{fmt.Sprintf("%s:%d", env.Host, env.HttpPort)},
    Auth: clickhouse.Auth{
        Database: env.Database,
        Username: env.Username,
        Password: env.Password,
    },
    Compression: &clickhouse.Compression{
        Method: clickhouse.CompressionBrotli,
        Level:  5,
    },
    Protocol: clickhouse.HTTP,
})
```
[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/std/compression.go#L27-L76)


```go
conn, err := sql.Open("clickhouse", fmt.Sprintf("http://%s:%d?username=%s&password=%s&compress=gzip&compress_level=5", env.Host, env.HttpPort, env.Username, env.Password))
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/std/compression.go#L78-L115)

施加的压缩级别可以通过DSN参数compress_level或Compression选项的Level字段来控制。默认值为0，但特定于算法：

* `gzip` - `-2`（最佳速度）到 `9`（最佳压缩）
* `deflate` - `-2`（最佳速度）到 `9`（最佳压缩）
* `br` - `0`（最佳速度）到 `11`（最佳压缩）
* `zstd`、`lz4` - 被忽略
### 参数绑定 {#parameter-binding-1}

标准API支持与[ClickHouse API](#parameter-binding)相同的参数绑定能力，允许将参数传递给`Exec`、`Query`和`QueryRow`方法（及其等效的[Context](#using-context)变体）。支持位置参数、命名参数和编号参数。

```go
var count uint64
// 位置绑定
if err = conn.QueryRow(ctx, "SELECT count() FROM example WHERE Col1 >= ? AND Col3 < ?", 500, now.Add(time.Duration(750)*time.Second)).Scan(&count); err != nil {
    return err
}
// 250
fmt.Printf("位置绑定计数: %d\n", count)
// 数字绑定
if err = conn.QueryRow(ctx, "SELECT count() FROM example WHERE Col1 <= $2 AND Col3 > $1", now.Add(time.Duration(150)*time.Second), 250).Scan(&count); err != nil {
    return err
}
// 100
fmt.Printf("数字绑定计数: %d\n", count)
// 命名绑定
if err = conn.QueryRow(ctx, "SELECT count() FROM example WHERE Col1 <= @col1 AND Col3 > @col3", clickhouse.Named("col1", 100), clickhouse.Named("col3", now.Add(time.Duration(50)*time.Second))).Scan(&count); err != nil {
    return err
}
// 50
fmt.Printf("命名绑定计数: %d\n", count)
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/std/bind.go)

注意[特殊情况](#special-cases)仍然适用。
### 使用上下文 {#using-context-1}

标准API支持与[ClickHouse API](#using-context)相同的通过上下文传递截止日期、取消信号和其他请求范围值的能力。与ClickHouse API不同，这是通过使用方法的`Context`变体实现的。例如，像`Exec`这样的使用后台上下文的方法有一个变体`ExecContext`，可以将上下文作为第一个参数传递。这允许在应用程序流程的任何阶段传递上下文。例如，用户可以在通过`ConnContext`建立连接时或通过`QueryRowContext`请求查询行时传递上下文。所有可用方法的示例如下。

有关使用上下文传递截止日期、取消信号、查询ID、配额键和连接设置的更多详细信息，请参见[ClickHouse API](#using-context)中的使用上下文。

```go
ctx := clickhouse.Context(context.Background(), clickhouse.WithSettings(clickhouse.Settings{
    "allow_experimental_object_type": "1",
}))
conn.ExecContext(ctx, "DROP TABLE IF EXISTS example")
// 要创建JSON列，我们需要allow_experimental_object_type=1
if _, err = conn.ExecContext(ctx, `
    CREATE TABLE example (
            Col1 JSON
        )
        Engine Memory
    `); err != nil {
    return err
}

// 查询可以使用上下文取消
ctx, cancel := context.WithCancel(context.Background())
go func() {
    cancel()
}()
if err = conn.QueryRowContext(ctx, "SELECT sleep(3)").Scan(); err == nil {
    return fmt.Errorf("预期为取消")
}

// 为查询设置截止日期 - 这将在绝对时间达到后取消查询。再次终止连接只会结束，查询将在ClickHouse中继续完成
ctx, cancel = context.WithDeadline(context.Background(), time.Now().Add(-time.Second))
defer cancel()
if err := conn.PingContext(ctx); err == nil {
    return fmt.Errorf("预期截止超时")
}

// 设置查询ID以帮助在日志中跟踪查询，例如，参见system.query_log
var one uint8
ctx = clickhouse.Context(context.Background(), clickhouse.WithQueryID(uuid.NewString()))
if err = conn.QueryRowContext(ctx, "SELECT 1").Scan(&one); err != nil {
    return err
}

conn.ExecContext(context.Background(), "DROP QUOTA IF EXISTS foobar")
defer func() {
    conn.ExecContext(context.Background(), "DROP QUOTA IF EXISTS foobar")
}()
ctx = clickhouse.Context(context.Background(), clickhouse.WithQuotaKey("abcde"))
// 设置配额键 - 首先创建配额
if _, err = conn.ExecContext(ctx, "CREATE QUOTA IF NOT EXISTS foobar KEYED BY client_key FOR INTERVAL 1 minute MAX queries = 5 TO default"); err != nil {
    return err
}

// 查询可以使用上下文取消
ctx, cancel = context.WithCancel(context.Background())
// 我们将在取消之前获得一些结果
ctx = clickhouse.Context(ctx, clickhouse.WithSettings(clickhouse.Settings{
    "max_block_size": "1",
}))
rows, err := conn.QueryContext(ctx, "SELECT sleepEachRow(1), number FROM numbers(100);")
if err != nil {
    return err
}
var (
    col1 uint8
    col2 uint8
)

for rows.Next() {
    if err := rows.Scan(&col1, &col2); err != nil {
        if col2 > 3 {
            fmt.Println("预期为取消")
            return nil
        }
        return err
    }
    fmt.Printf("行: col2=%d\n", col2)
    if col2 == 3 {
        cancel()
    }
}
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/std/context.go)
```

### 会话 {#sessions}

尽管本地连接本身具有会话，但通过 HTTP 的连接需要用户创建一个会话 id 以将上下文作为设置传递。这允许使用绑定到会话的功能，例如临时表。

```go
conn := clickhouse.OpenDB(&clickhouse.Options{
    Addr: []string{fmt.Sprintf("%s:%d", env.Host, env.HttpPort)},
    Auth: clickhouse.Auth{
        Database: env.Database,
        Username: env.Username,
        Password: env.Password,
    },
    Protocol: clickhouse.HTTP,
    Settings: clickhouse.Settings{
        "session_id": uuid.NewString(),
    },
})
if _, err := conn.Exec(`DROP TABLE IF EXISTS example`); err != nil {
    return err
}
_, err = conn.Exec(`
    CREATE TEMPORARY TABLE IF NOT EXISTS example (
            Col1 UInt8
    )
`)
if err != nil {
    return err
}
scope, err := conn.Begin()
if err != nil {
    return err
}
batch, err := scope.Prepare("INSERT INTO example")
if err != nil {
    return err
}
for i := 0; i < 10; i++ {
    _, err := batch.Exec(
        uint8(i),
    )
    if err != nil {
        return err
    }
}
rows, err := conn.Query("SELECT * FROM example")
if err != nil {
    return err
}
var (
    col1 uint8
)
for rows.Next() {
    if err := rows.Scan(&col1); err != nil {
        return err
    }
    fmt.Printf("row: col1=%d\n", col1)
}
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/std/session.go)
### 动态扫描 {#dynamic-scanning-1}

与 [ClickHouse API](#dynamic-scanning) 类似，列类型信息可用于允许用户创建具有正确类型变量的运行时实例，并可以传递给 Scan。这允许在类型未知的情况下读取列。

```go
const query = `
SELECT
        1     AS Col1
    , 'Text' AS Col2
`
rows, err := conn.QueryContext(context.Background(), query)
if err != nil {
    return err
}
columnTypes, err := rows.ColumnTypes()
if err != nil {
    return err
}
vars := make([]interface{}, len(columnTypes))
for i := range columnTypes {
    vars[i] = reflect.New(columnTypes[i].ScanType()).Interface()
}
for rows.Next() {
    if err := rows.Scan(vars...); err != nil {
        return err
    }
    for _, v := range vars {
        switch v := v.(type) {
        case *string:
            fmt.Println(*v)
        case *uint8:
            fmt.Println(*v)
        }
    }
}
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/std/dynamic_scan_types.go)
### 外部表 {#external-tables-1}

[外部表](/engines/table-engines/special/external-data/) 允许客户端将数据发送到 ClickHouse，通过 `SELECT` 查询。这些数据被放置在临时表中，可以在查询本身中用于评估。

要通过查询将外部数据发送到客户端，用户必须通过 `ext.NewTable` 构建外部表，然后将其通过上下文传递。

```go
table1, err := ext.NewTable("external_table_1",
    ext.Column("col1", "UInt8"),
    ext.Column("col2", "String"),
    ext.Column("col3", "DateTime"),
)
if err != nil {
    return err
}

for i := 0; i < 10; i++ {
    if err = table1.Append(uint8(i), fmt.Sprintf("value_%d", i), time.Now()); err != nil {
        return err
    }
}

table2, err := ext.NewTable("external_table_2",
    ext.Column("col1", "UInt8"),
    ext.Column("col2", "String"),
    ext.Column("col3", "DateTime"),
)

for i := 0; i < 10; i++ {
    table2.Append(uint8(i), fmt.Sprintf("value_%d", i), time.Now())
}
ctx := clickhouse.Context(context.Background(),
    clickhouse.WithExternalTable(table1, table2),
)
rows, err := conn.QueryContext(ctx, "SELECT * FROM external_table_1")
if err != nil {
    return err
}
for rows.Next() {
    var (
        col1 uint8
        col2 string
        col3 time.Time
    )
    rows.Scan(&col1, &col2, &col3)
    fmt.Printf("col1=%d, col2=%s, col3=%v\n", col1, col2, col3)
}
rows.Close()

var count uint64
if err := conn.QueryRowContext(ctx, "SELECT COUNT(*) FROM external_table_1").Scan(&count); err != nil {
    return err
}
fmt.Printf("external_table_1: %d\n", count)
if err := conn.QueryRowContext(ctx, "SELECT COUNT(*) FROM external_table_2").Scan(&count); err != nil {
    return err
}
fmt.Printf("external_table_2: %d\n", count)
if err := conn.QueryRowContext(ctx, "SELECT COUNT(*) FROM (SELECT * FROM external_table_1 UNION ALL SELECT * FROM external_table_2)").Scan(&count); err != nil {
    return err
}
fmt.Printf("external_table_1 UNION external_table_2: %d\n", count)
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/std/external_data.go)
### 开放遥测 {#open-telemetry-1}

ClickHouse 允许通过本地协议传递 [追踪上下文](/operations/opentelemetry/)。客户端允许通过函数 `clickhouse.withSpan` 创建 Span，并通过上下文传递以实现此功能。使用 HTTP 作为传输方式时不支持此功能。

```go
var count uint64
rows := conn.QueryRowContext(clickhouse.Context(context.Background(), clickhouse.WithSpan(
    trace.NewSpanContext(trace.SpanContextConfig{
        SpanID:  trace.SpanID{1, 2, 3, 4, 5},
        TraceID: trace.TraceID{5, 4, 3, 2, 1},
    }),
)), "SELECT COUNT() FROM (SELECT number FROM system.numbers LIMIT 5)")
if err := rows.Scan(&count); err != nil {
    return err
}
fmt.Printf("count: %d\n", count)
```

[完整示例](https://github.com/ClickHouse/clickhouse-go/blob/main/examples/std/open_telemetry.go)
## 性能提示 {#performance-tips}

* 尽可能利用 ClickHouse API，特别是在处理基本类型时。这可以避免大量的反射和间接调用。
* 如果读取大数据集，考虑修改 [`BlockBufferSize`](#connection-settings)。这将增加内存占用，但意味着在行迭代期间可以并行解码更多的块。默认值为 2 是保守的，并且最小化内存开销。更高的值将意味着更多的块驻留在内存中。这需要测试，因为不同的查询可能会产生不同的块大小。因此可以在查询级别通过上下文进行设置。
* 插入数据时要明确类型。虽然客户端旨在灵活，例如允许字符串解析为 UUID 或 IP，但这需要数据验证，并在插入时产生开销。
* 在可能的情况下，使用面向列的插入。这些应强类型化，避免客户端转换值的需要。
* 遵循 ClickHouse [建议](/sql-reference/statements/insert-into/#performance-considerations) 以获得最佳插入性能。
