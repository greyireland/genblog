---
title: gorm源码分析
tags:
  - gorm
categories:
  - gorm
date: 2019-10-07 21:06:26
---
# gorm源码解析

gorm使用示例：

```
package main

import (
	"github.com/jinzhu/gorm"
	_ "github.com/jinzhu/gorm/dialects/sqlite"
)

type Product struct {
	gorm.Model
	Code  string
	Price uint
}

func main() {
	db, err := gorm.Open("sqlite3", "test.db")
	if err != nil {
		panic("failed to connect database")
	}
	defer db.Close()
	db.AutoMigrate(&Product{})
	db.Create(&Product{Code: "L1212", Price: 1000})
	var product Product
	db.First(&product, 1)                  
	db.First(&product, "code = ?", "L1212") 
	db.Model(&product).Update("Price", 2000)
	db.Delete(&product)
}
```



gorm一般的初始化方式

```
db, err := gorm.Open("mysql", "user:password@/dbname?charset=utf8&parseTime=True&loc=Local")
```

gorm中DB结构体的定义：

```
// DB的结构体
type DB struct {
    sync.RWMutex                       // 锁
    Value        interface{}           // 一般传入实际操作的表所对应的结构体
    Error        error                 // DB操作失败的error
    RowsAffected int64                 // 操作影响的行数

    // single db
    db                SQLCommon        // SQL接口，包括（Exec、Prepare、Query、QueryRow）
    blockGlobalUpdate bool             // 为true时，可以在update在没有where条件是报错，避免全局更新
    logMode           logModeValue     // 日志模式，gorm提供了三种   
    logger            logger           // 内部日志实例
    search            *search          // 查询相关的条件
    values            sync.Map         // value Map

    // global db
    parent        *DB                  // 父db，为了保存一个空的初始化后的db，也为了保存curd注册的的callback方法
    callbacks     *Callback            // callback方法
    dialect       Dialect              // 不同类型数据库对应的不同实现的相同接口 
    singularTable bool                 // 表名是否为复数形式，true时为user，false时为users
}
```

SQLCommon定义基本的查询接口

```
// SQLCommon is the minimal database connection functionality gorm requires.  Implemented by *sql.DB.
type SQLCommon interface {
	Exec(query string, args ...interface{}) (sql.Result, error)
	Prepare(query string) (*sql.Stmt, error)
	Query(query string, args ...interface{}) (*sql.Rows, error)
	QueryRow(query string, args ...interface{}) *sql.Row
}
```



gorm的Open方法：

```
func Open(dialect string, args ...interface{}) (db *DB, err error) {
    if len(args) == 0 {
        err = errors.New("invalid database source")
        return nil, err
    }
    var source string
    var dbSQL SQLCommon
    var ownDbSQL bool

    switch value := args[0].(type) {
    case string:
        var driver = dialect
        if len(args) == 1 {
            source = value
        } else if len(args) >= 2 {
            driver = value
            source = args[1].(string)
        }
        // 调用go基础库的Open方法获得db的connention附给dbSQL，
        // 此时还没有真正连接数据库
        dbSQL, err = sql.Open(driver, source)
        ownDbSQL = true
    case SQLCommon:
        dbSQL = value
        ownDbSQL = false
    default:
        return nil, fmt.Errorf("invalid database source: %v is not a valid type", value)
    }
    // 初始化DB
    db = &DB{
        db:        dbSQL,
        logger:    defaultLogger,
        callbacks: DefaultCallback,
        dialect:   newDialect(dialect, dbSQL),
    }
    // 将初始化的DB保存到db.parent中
    db.parent = db
    if err != nil {
        return
    }
    // 调用go基础库的Ping方法检测数据库connention是否可以连通
    if d, ok := dbSQL.(*sql.DB); ok {
        if err = d.Ping(); err != nil && ownDbSQL {
            d.Close()
        }
    }
    return
}
```

gorm是通过多个callbsck方法来实现curd的，具体流程以一个查询为例：

```
DBEngine.Table(entry.TableName).
    Select(entry.Select).
    Where(entry.sql, entry.values).
    Order(entry.order).
    Find(entry.result)
```

执行步骤：

1.执行Table方法，添加tablename条件：

```
func (s *DB) Table(name string) *DB {
    clone := s.clone()        // 执行clone方法也就是从新的db中赋值一个空的，避免交叉影响
    clone.search.Table(name)  // 赋值table name
    clone.Value = nil         // 附空
    return clone
}
```

2.执行Where方法，添加where条件：

```
// 首先也是调用clone方法，然后调用search的Where方法
func (s *DB) Where(query interface{}, args ...interface{}) *DB {
    return s.clone().search.Where(query, args...).db
}

// search的Where方法是将传进来的条件进行拼接，存入search.whereConditions
func (s *search) Where(query interface{}, values ...interface{}) *search {
    s.whereConditions = append(s.whereConditions, map[string]interface{}{"query": query, "args": values})
    return s
}
```

3.执行Order方法，添加order条件：

```
// 类似Where，reorder为true会强制刷掉gorm默认的order by
func (s *DB) Order(value interface{}, reorder ...bool) *DB {
    return s.clone().search.Order(value, reorder...).db
}

func (s *search) Order(value interface{}, reorder ...bool) *search {
    // 如果为true，先清除s.orders
    if len(reorder) > 0 && reorder[0] {
        s.orders = []interface{}{}
    }
    // 将value拼接，存入s.orders
    if value != nil && value != "" {
        s.orders = append(s.orders, value)
    }
    return s
}
```

4.执行Find方法，真正实现查询：

```
// 首先先创建一个scope（可以理解成只针对本次数据库操作有效的一个环境），再调用inlineCondition内部方法，最后执行callcallbacks一系列方法实现真正的查询操作，并将db返回
func (s *DB) Find(out interface{}, where ...interface{}) *DB {
    return s.NewScope(out).inlineCondition(where...).callCallbacks(s.parent.callbacks.queries).db
}

// NewScope方法就是初始化一个scope
func (s *DB) NewScope(value interface{}) *Scope {
    dbClone := s.clone()
    // 此时赋值value
    dbClone.Value = value
    scope := &Scope{db: dbClone, Value: value}
    if s.search != nil {
        scope.Search = s.search.clone()
    } else {
        scope.Search = &search{}
    }
    return scope
}

// inlineCondition方法是执行scope.Search.Where
func (scope *Scope) inlineCondition(values ...interface{}) *Scope {
    if len(values) > 0 {
        scope.Search.Where(values[0], values[1:]...)
    }
    return scope
}
// scope.Search.Where实际上也是执行条件拼接，由于我们在调用的时候没有在Find中传入条件，所以这个方法不会被执行
func (s *search) Where(query interface{}, values ...interface{}) *search {
    s.whereConditions = append(s.whereConditions, map[string]interface{}{"query": query, "args": values})
    return s
}

// 最重要的就是callcallbacks方法，是真正执行的地方
func (scope *Scope) callCallbacks(funcs []*func(s *Scope)) *Scope {
    defer func() {
        if err := recover(); err != nil {
            if db, ok := scope.db.db.(sqlTx); ok {
                db.Rollback()
            }
            panic(err)
        }
    }()
    // 循环里面所有的注册的funcs
    for _, f := range funcs {
        (*f)(scope)
        if scope.skipLeft {
            break
        }
    }
    return scope
}

// 这里的funcs实在程序启动时init方法注册的
func init() {
    DefaultCallback.Query().Register("gorm:query", queryCallback)
    DefaultCallback.Query().Register("gorm:preload", preloadCallback)
    DefaultCallback.Query().Register("gorm:after_query", afterQueryCallback)
}

// 比如afterQueryCallback方法还提供了反射调用结构体的AfterFind方法，如果在查询前结构体实现了AfterFind方法就会被调用，这个机制比了灵活
func afterQueryCallback(scope *Scope) {
    if !scope.HasError() {
        scope.CallMethod("AfterFind")
    }
}
```