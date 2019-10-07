---
title: boltdb使用
tags:
  - boltdb
categories:
  - boltdb
date: 2019-04-19 13:46:16
---

# golang boltdb的学习和实践

## 1. 安装

```
go get github.com/boltdb/bolt
```

## 2.创建和启动数据库

```
db, err := bolt.Open("my.db", 0600, nil)
```

其中`open`的第一个参数为路径,如果数据库不存在则会创建名为my.db的数据库， 第二个为文件操作，第三个参数是可选参数， 内部可以配置只读和超时时间等，
特别需要注意的地方就是因为boltdb是文件操作类型的数据库，所以只能单点写入和读取，如果多个同时操作的话后者会被挂起直到前者关闭操作为止， boltdb一次只允许一个读写事务，但一次允许多个只读事务。所以数据具有较强的一致性。

因此单个事务和从它们创建的所有对象（例如桶、键）都不是线程安全的。与数据在多个概念你必须为每一个或使用锁机制来保证只有一个goroutine里操作改变数据。
只读事务和读写事物通常不应该在同一个goroutine里同时打开。由于读写事务需要周期性地重新映射数据文件，这可能导致死锁。

## 3.读写事务

boltdb的读写事务操作我们可以使用`DB.Update()`来完成形如：

```
err := db.Update(func(tx *bolt.Tx) error {
    ...
    return nil
})
```

在闭包fun中,在结束时返回nil来提交事务。您还可以通过返回一个错误在任何点回滚事务。所有数据库操作都允许在读写事务中进行。
始终要关注err返回，因为它将报告导致您的事务不能完成的所有磁盘故障。

## 4.批量读写事物

每一次新的事物都需要等待上一次事物的结束，这种开销我们可以通过`DB.Batch()`批处理来完成

```
err := db.Batch(func(tx *bolt.Tx) error {
    ...
    return nil
})
```

在批处理过程中如果某个事务失败了,批处理会多次调用这个函数函数返回成功则成功。如果中途失败了，则整个事务会回滚。

## 5.只读事务

只读事务可以使用`DB.View()`来完成

```
err := db.View(func(tx *bolt.Tx) error {
    ...
    return nil
})
```

不改变数据的操作都可以通过只读事务来完成， 您只能检索桶、检索值，或在只读事务中复制数据库。

## 6.启动事务

`DB.Begin()`启动函数包含在db.update和db.batch中,该函数启动事务开始执行事务并返回结果关闭事务，这是boltdb推荐的方式，有时候你可能需要手动启动事物你可以使用`Tx.Begin()`来开始，切记不要忘记关闭事务。

```
// Start a writable transaction.
tx, err := db.Begin(true)
if err != nil {
    return err
}
defer tx.Rollback()

// Use the transaction...
_, err := tx.CreateBucket([]byte("MyBucket"))
if err != nil {
    return err
}

// Commit the transaction and check for error.
if err := tx.Commit(); err != nil {
    return err
}
```

## 7.使用桶

桶是数据库中键/值对的集合。桶中的所有键必须是唯一的。您可以使用`DB.CreateBucket()`创建一个桶：

```
db.Update(func(tx *bolt.Tx) error {
    b, err := tx.CreateBucket([]byte("MyBucket"))
    if err != nil {
        return fmt.Errorf("create bucket: %s", err)
    }
    return nil
})
```

你也可以是实用`Tx.CreateBucketIfNotExists()`来创建桶，该函数会先判断是否已经存在该桶不存在即创建， 删除桶可以使用`Tx.DeleteBucket()`来完成

## 8.使用k-v对

存储键值对到桶里可以使用`Bucket.Put()`来完成：

```
db.Update(func(tx *bolt.Tx) error {
    b := tx.Bucket([]byte("MyFriendsBucket"))
    err := b.Put([]byte("one"), []byte("zhangsan"))
    return err
})
```

获取键值`Bucket.Get()`：

```
db.View(func(tx *bolt.Tx) error {
    b := tx.Bucket([]byte("MyFriendsBucket"))
    v := b.Get([]byte("one"))
    fmt.Printf("The answer is: %s\n", v)
    return nil
})
```

`get()`函数不返回一个错误，因为它的运行是保证工作（除非有某种系统故障）。如果键存在，那么它将返回它的值。如果它不存在，那么它将返回nil。
还需要注意的是当事务打开都get返回的值时唯一有效的，如果你需要将该值用于其他事务，你可以通过`copy`拷贝到其他的byte slice中

## 9.桶的自增

利用`nextsequence()`功能，你可以让boltdb生成序列作为你键值对的唯一标识。见下面的示例。

```
func (s *Store) CreateUser(u *User) error {
    return s.db.Update(func(tx *bolt.Tx) error {
        // 创建users桶
        b := tx.Bucket([]byte("users"))

        // 生成自增序列
        id, _ = b.NextSequence()
        u.ID = int(id)

        // Marshal user data into bytes.
        buf, err := json.Marshal(u)
        if err != nil {
            return err
        }

        // Persist bytes to users bucket.
        return b.Put(itob(u.ID), buf)
    })
}

// itob returns an 8-byte big endian representation of v.
func itob(v int) []byte {
    b := make([]byte, 8)
    binary.BigEndian.PutUint64(b, uint64(v))
    return b
}

type User struct {
    ID int
    ...
}
```

## 10. 迭代键

boltdb以桶中的字节排序顺序存储键。这使得在这些键上的顺序迭代非常快。要遍历键，我们将使用游标`Cursor()`：

```
db.View(func(tx *bolt.Tx) error {
    // Assume bucket exists and has keys
    b := tx.Bucket([]byte("MyBucket"))

    c := b.Cursor()

    for k, v := c.First(); k != nil; k, v = c.Next() {
        fmt.Printf("key=%s, value=%s\n", k, v)
    }

    return nil
})
```

游标`Cursor()`允许您移动到键列表中的特定点，并一次一个地通过操作键前进或后退。
光标上有以下函数：

```
First()  移动到第一个健.
Last()   移动到最后一个健.
Seek()   移动到特定的一个健.
Next()   移动到下一个健.
Prev()   移动到上一个健.
```

这些函数中的每一个都返回一个包含(key []byte, value []byte)的签名。当你有光标迭代结束，next()将返回一个nil。在调用next()或prev()之前，你必须寻求一个位置使用first()，last()，或seek()。如果您不寻求位置，则这些函数将返回一个nil键。
在迭代过程中，如果键为非零，但值为0，则意味着键指向一个桶而不是一个值。用桶.bucket()访问子桶。

## 11.前缀扫描

遍历一个key的前缀，你可以结合`seek()`和`bytes.hasprefix()`：

```
db.View(func(tx *bolt.Tx) error {
    // Assume bucket exists and has keys
    c := tx.Bucket([]byte("MyBucket")).Cursor()

    prefix := []byte("1234")
    for k, v := c.Seek(prefix); bytes.HasPrefix(k, prefix); k, v = c.Next() {
        fmt.Printf("key=%s, value=%s\n", k, v)
    }

    return nil
})
```

## 12.范围扫描

另一个常见的用例是扫描范围，例如时间范围。如果你使用一个合适的时间编码，如rfc3339然后可以查询特定日期范围的数据：

```
db.View(func(tx *bolt.Tx) error {
    // Assume our events bucket exists and has RFC3339 encoded time keys.
    c := tx.Bucket([]byte("Events")).Cursor()

    // Our time range spans the 90's decade.
    min := []byte("1990-01-01T00:00:00Z")
    max := []byte("2000-01-01T00:00:00Z")

    // Iterate over the 90's.
    for k, v := c.Seek(min); k != nil && bytes.Compare(k, max) <= 0; k, v = c.Next() {
        fmt.Printf("%s: %s\n", k, v)
    }

    return nil
})
```

## 13.循环遍历每一个

如果你知道所在桶中拥有键，你也可以使用`ForEach()`来迭代：

```
db.View(func(tx *bolt.Tx) error {
    // Assume bucket exists and has keys
    b := tx.Bucket([]byte("MyBucket"))

    b.ForEach(func(k, v []byte) error {
        fmt.Printf("key=%s, value=%s\n", k, v)
        return nil
    })
    return nil
})
```

## 14.嵌套桶

还可以在一个键中存储一个桶，以创建嵌套的桶：

```
func (*Bucket) CreateBucket(key []byte) (*Bucket, error)
func (*Bucket) CreateBucketIfNotExists(key []byte) (*Bucket, error)
func (*Bucket) DeleteBucket(key []byte) error
```

## 15.数据库备份

boltdb是一个单一的文件，所以很容易备份。你可以使用`TX.writeto()`函数写一致的数据库。如果从只读事务调用这个函数，它将执行热备份，而不会阻塞其他数据库的读写操作。
默认情况下，它将使用一个常规文件句柄，该句柄将利用操作系统的页面缓存。有关优化大于RAM数据集的信息，请参见`Tx`文档。
一个常见的用例是在HTTP上进行备份，这样您就可以使用像`cURL`这样的工具来进行数据库备份：

```
func BackupHandleFunc(w http.ResponseWriter, req *http.Request) {
    err := db.View(func(tx *bolt.Tx) error {
        w.Header().Set("Content-Type", "application/octet-stream")
        w.Header().Set("Content-Disposition", `attachment; filename="my.db"`)
        w.Header().Set("Content-Length", strconv.Itoa(int(tx.Size())))
        _, err := tx.WriteTo(w)
        return err
    })
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
    }
}
```

然后您可以使用此命令进行备份：
`$ curl http://localhost/backup > my.db`
或者你可以打开你的浏览器以<http://localhost/backup>，它会自动下载。
如果你想备份到另一个文件，你可以使用`TX.copyfile()`辅助功能。

## 16.统计

数据库对运行的许多内部操作保持一个运行计数，这样您就可以更好地了解发生了什么。通过捕捉这些数据的快照，我们可以看到在这个时间范围内执行了哪些操作。
例如，我们可以开始一个goroutine里记录统计每10秒：

```
go func() {
    // Grab the initial stats.
    prev := db.Stats()

    for {
        // Wait for 10s.
        time.Sleep(10 * time.Second)

        // Grab the current stats and diff them.
        stats := db.Stats()
        diff := stats.Sub(&prev)

        // Encode stats to JSON and print to STDERR.
        json.NewEncoder(os.Stderr).Encode(diff)

        // Save stats for the next loop.
        prev = stats
    }
```

## 17.只读模式

有时创建一个共享的只读boltdb数据库是有用的。对此，设置options.readonly国旗打开数据库时。只读模式使用共享锁允许多个进程从数据库中读取，但它将阻塞任何以读写方式打开数据库的进程。

```
db, err := bolt.Open("my.db", 0666, &bolt.Options{ReadOnly: true})
if err != nil {
    log.Fatal(err)
}
```

## 18.移动端支持（ios/android）

boltdb能够运行在移动设备上利用的工具结合特征GoMobile。创建一个结构体，包含您的数据库逻辑和参考一个bolt.db与初始化contstructor需要在文件路径，数据库文件将存储。使用这种方法，Android和iOS都不需要额外的权限或清理。

```
func NewBoltDB(filepath string) *BoltDB {
    db, err := bolt.Open(filepath+"/demo.db", 0600, nil)
    if err != nil {
        log.Fatal(err)
    }

    return &BoltDB{db}
}

type BoltDB struct {
    db *bolt.DB
    ...
}

func (b *BoltDB) Path() string {
    return b.db.Path()
}

func (b *BoltDB) Close() {
    b.db.Close()
}
```

数据库逻辑应定义为此包装器结构中的方法。
要从本机语言初始化此结构（两个平台现在都将本地存储与云同步）。这些片段禁用数据库文件的功能）：
Android

```
String path;
if (android.os.Build.VERSION.SDK_INT >=android.os.Build.VERSION_CODES.LOLLIPOP){
    path = getNoBackupFilesDir().getAbsolutePath();
} else{
    path = getFilesDir().getAbsolutePath();
}
Boltmobiledemo.BoltDB boltDB = Boltmobiledemo.NewBoltDB(path)
```

IOS

```
- (void)demo {
    NSString* path = [NSSearchPathForDirectoriesInDomains(NSLibraryDirectory,
                                                          NSUserDomainMask,
                                                          YES) objectAtIndex:0];
    GoBoltmobiledemoBoltDB * demo = GoBoltmobiledemoNewBoltDB(path);
    [self addSkipBackupAttributeToItemAtPath:demo.path];
    //Some DB Logic would go here
    [demo close];
}

- (BOOL)addSkipBackupAttributeToItemAtPath:(NSString *) filePathString
{
    NSURL* URL= [NSURL fileURLWithPath: filePathString];
    assert([[NSFileManager defaultManager] fileExistsAtPath: [URL path]]);

    NSError *error = nil;
    BOOL success = [URL setResourceValue: [NSNumber numberWithBool: YES]
                                  forKey: NSURLIsExcludedFromBackupKey error: &error];
    if(!success){
        NSLog(@"Error excluding %@ from backup %@", [URL lastPathComponent], error);
    }
    return success;
}
```

## 19.查看工具

1.下载工具
`go get github.com/boltdb/boltd`
然后编译cmd下的main文件生成可执行文件改名为boltd
拷贝boltd到 *.db同级目录，执行如下：`boltd my.db `

2.命令行工具
<https://github.com/hasit/bolter>

`bolter -f my.db`

boltdb源码解析TODO