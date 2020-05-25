---
title: go-函数式可选项
tags:
  - go
  - 函数式选项
categories:
  - 设计模式
date: 2019-03-10 21:52:22
---

## 函数式可选项

通过参数创建实例，常见方式

常用方式

```go
func NewStuffClient(conn Connection, timeout, retries int) StuffClient {
    return &stuffClient{
        conn:    conn,
        timeout: timeout,
        retries: retries,
    }
}
func NewStuffClient(conn Connection) StuffClient {
    return &stuffClient{
        conn:    conn,
        timeout: DEFAULT_TIMEOUT,
        retries: DEFAULT_RETRIES,
    }
}
func NewStuffClientWithOptions(conn Connection, timeout, retries int) StuffClient {
    return &stuffClient{
        conn:    conn,
        timeout: timeout,
        retries: retries,
    }
}
```

或者

```go
type StuffClientOptions struct {
    Retries int
    Timeout int
}
//传入对象
func NewStuffClient(conn Connection, options StuffClientOptions) StuffClient {
    return &stuffClient{
        conn:    conn,
        timeout: options.Timeout,
        retries: options.Retries,
    }
}

```

更高级的方式，通过闭包函数注入

```go
type StuffClientOptions struct {
    Retries int //number of times to retry the request before giving up
    Timeout int //connection timeout in seconds
}
type StuffClientOption func(*StuffClientOptions)
func WithRetries(r int) StuffClientOption {
    return func(o *StuffClientOptions) {
        o.Retries = r
    }
}
func WithTimeout(t int) StuffClientOption {
    return func(o *StuffClientOptions) {
        o.Timeout = t
    }
}

var defaultStuffClientOptions = StuffClientOptions{
    Retries: 3,
    Timeout: 2,
}
//使用可选函数注入选项
func NewStuffClient(conn Connection, opts ...StuffClientOption) StuffClient {
    options := defaultStuffClientOptions
    for _, o := range opts {
        o(&options)
    }
    return &stuffClient{
        conn:    conn,
        timeout: options.Timeout,
        retries: options.Retries,
    }
}

```

使用方式

```go
x := NewStuffClient(Connection{})
fmt.Println(x) // prints &{{} 2 3}
x = NewStuffClient(
    Connection{},
    WithRetries(1),
)
fmt.Println(x) // prints &{{} 2 1}
x = NewStuffClient(
    Connection{},
    WithRetries(1),
    WithTimeout(1),
)
fmt.Println(x) // prints &{{} 1 1}

```

### 模式步骤

1. 定义 options
2. 定义 option
3. 定义闭包函数
4. 传入函数选项
5. 调用函数,注入配置到选项

### 实际项目使用

redigo go 的 redis 连接客户端代码片段

```go

//1 声明options
type dialOptions struct {
	readTimeout  time.Duration
	writeTimeout time.Duration
	dialer       *net.Dialer
	dial         func(network, addr string) (net.Conn, error)
	db           int
	password     string
	useTLS       bool
	skipVerify   bool
	tlsConfig    *tls.Config
}
//2 声明option
type DialOption struct {
	f func(*dialOptions)
}
// DialReadTimeout specifies the timeout for reading a single command reply
//3 声明闭包函数
func DialReadTimeout(d time.Duration) DialOption {
	return DialOption{func(do *dialOptions) {
		do.readTimeout = d
	}}
}

// DialWriteTimeout specifies the timeout for writing a single command.
func DialWriteTimeout(d time.Duration) DialOption {
	return DialOption{func(do *dialOptions) {
		do.writeTimeout = d
	}}
}

// DialConnectTimeout specifies the timeout for connecting to the Redis server when
// no DialNetDial option is specified.
func DialConnectTimeout(d time.Duration) DialOption {
	return DialOption{func(do *dialOptions) {
		do.dialer.Timeout = d
	}}
}

// DialKeepAlive specifies the keep-alive period for TCP connections to the Redis server
// when no DialNetDial option is specified.
// If zero, keep-alives are not enabled. If no DialKeepAlive option is specified then
// the default of 5 minutes is used to ensure that half-closed TCP sessions are detected.
func DialKeepAlive(d time.Duration) DialOption {
	return DialOption{func(do *dialOptions) {
		do.dialer.KeepAlive = d
	}}
}

// DialNetDial specifies a custom dial function for creating TCP
// connections, otherwise a net.Dialer customized via the other options is used.
// DialNetDial overrides DialConnectTimeout and DialKeepAlive.
func DialNetDial(dial func(network, addr string) (net.Conn, error)) DialOption {
	return DialOption{func(do *dialOptions) {
		do.dial = dial
	}}
}

// DialDatabase specifies the database to select when dialing a connection.
func DialDatabase(db int) DialOption {
	return DialOption{func(do *dialOptions) {
		do.db = db
	}}
}

// DialPassword specifies the password to use when connecting to
// the Redis server.
func DialPassword(password string) DialOption {
	return DialOption{func(do *dialOptions) {
		do.password = password
	}}
}

// DialTLSConfig specifies the config to use when a TLS connection is dialed.
// Has no effect when not dialing a TLS connection.
func DialTLSConfig(c *tls.Config) DialOption {
	return DialOption{func(do *dialOptions) {
		do.tlsConfig = c
	}}
}

// DialTLSSkipVerify disables server name verification when connecting over
// TLS. Has no effect when not dialing a TLS connection.
func DialTLSSkipVerify(skip bool) DialOption {
	return DialOption{func(do *dialOptions) {
		do.skipVerify = skip
	}}
}

// DialUseTLS specifies whether TLS should be used when connecting to the
// server. This option is ignore by DialURL.
func DialUseTLS(useTLS bool) DialOption {
	return DialOption{func(do *dialOptions) {
		do.useTLS = useTLS
	}}
}

// Dial connects to the Redis server at the given network and
// address using the specified options.
//4传入参数
func Dial(network, address string, options ...DialOption) (Conn, error) {
	do := dialOptions{
		dialer: &net.Dialer{
			KeepAlive: time.Minute * 5,
		},
  }
  //5 注入
	for _, option := range options {
		option.f(&do)
	}
	if do.dial == nil {
		do.dial = do.dialer.Dial
	}

	netConn, err := do.dial(network, address)
	if err != nil {
		return nil, err
	}

	if do.useTLS {
		var tlsConfig *tls.Config
		if do.tlsConfig == nil {
			tlsConfig = &tls.Config{InsecureSkipVerify: do.skipVerify}
		} else {
			tlsConfig = cloneTLSConfig(do.tlsConfig)
		}
		if tlsConfig.ServerName == "" {
			host, _, err := net.SplitHostPort(address)
			if err != nil {
				netConn.Close()
				return nil, err
			}
			tlsConfig.ServerName = host
		}

		tlsConn := tls.Client(netConn, tlsConfig)
		if err := tlsConn.Handshake(); err != nil {
			netConn.Close()
			return nil, err
		}
		netConn = tlsConn
	}

	c := &conn{
		conn:         netConn,
		bw:           bufio.NewWriter(netConn),
		br:           bufio.NewReader(netConn),
		readTimeout:  do.readTimeout,
		writeTimeout: do.writeTimeout,
	}

	if do.password != "" {
		if _, err := c.Do("AUTH", do.password); err != nil {
			netConn.Close()
			return nil, err
		}
	}

	if do.db != 0 {
		if _, err := c.Do("SELECT", do.db); err != nil {
			netConn.Close()
			return nil, err
		}
	}

	return c, nil
}
```
