---
title: python常见问题
tags:
  - python
categories:
  - python
date: 2019-10-07 20:48:01
---

## python 常见问题

### 安装

`virtualenv -p /usr/bin/python2.7 venv`

### python 常用函数

```py
# str
s1 = str()
# in python `''` or `""` is the same
s2 = "shaunwei" # 'shaunwei'
s2len = len(s2)
# last 3 chars
s2[-3:] # wei
s2[5:8] # wei
s3 = s2[:5] # shaun
s3 += 'wei' # return 'shaunwei'
# list in python is same as ArrayList in java
s2list = list(s3)
# string at index 4
s2[4] # 'n'
# find index at first
s2.index('w') # return 5, if not found, throw ValueError
s2.find('w') # return 5, if not found, return -1

# 链表
class ListNode:
    def __init__(self, val):
        self.val = val
        self.next = None
class DListNode:
    def __init__(self, val):
        self.val = val
        self.prev = self.next = None
    def reverse(self, head):
        curt = None
        while head:
            curt = head
            head = curt.next
            curt.next = curt.prev
            curt.prev = head
        return curt

class TreeNode:
    def __init__(self, val):
        self.val = val
        self.left, self.right = None, None

queue = [] # same as list()
size = len(queue)
queue.append(1)
queue.append(2)
queue.pop(0) # return 1
queue[0] # return 2 examine the first element


s = set()
s1 = {1, 2, 3}
s.add('shaunwei')
'shaun' in s # return true
s.remove('shaunwei')

# map 在 python 中是一个keyword
hash_map = {} # or dict()
hash_map['shaun'] = 98
hash_map['wei'] = 99
exist = 'wei' in hash_map # check existence
point = hash_map['shaun'] # get value by key
point = hash_map.pop('shaun') # remove by key, return value
keys = hash_map.keys() # return key list
# iterate dictionary(map)
for key, value in hash_map.items():
    # do something with k, v
    pass

class Person(object):
    def __init__(self,name,age):
        self.name=name
        self.age=age

    def hello(self):
        print 'hello:'+self.name+":"+str(self.age)



class Tom(Person):
    def __init__(self,name):
        super(Tom,self).__init__(name,18)
    def hello2(self):#只要子类有重复的名字就会覆盖
        pass
        #print 'child say hello'
t=Tom('tom')
t.hello()
```

### 注意点

1. Python 有七个序列类型： 字符串、 Unicode 字符串、 列表、 元组、 字节数组、 缓冲区和 xrange 对象。序列类型 — str、unicode、list、 tuple、 bytearray、buffer、xrange
2. if and 执行逻辑

```py
def is_a():
    print 'a'
    return False
def is_b():
    print 'b'
    return False


if __name__=='__main__':
    #第一个条件如果为False,and后面的就直接不执行了
    if is_a() and is_b():
        print 'if中有先后顺序的'
```

### 问题

#### tornado 框架

1）异步 async+await 配合，2）效果和 yield 差不多，3）也可以使用 callback

```py
from tornado import gen

@gen.coroutine
def fetch_coroutine(url):
    http_client = AsyncHTTPClient()
    response = yield http_client.fetch(url)
    # In Python versions prior to 3.3, returning a value from
    # a generator is not allowed and you must use
    #   raise gen.Return(response.body)
    # instead.
    return response.body
```

```py
async def fetch_coroutine(url):
    http_client = AsyncHTTPClient()
    response = await http_client.fetch(url)
    return response.body
```

#### 装饰器

何时执行装饰器：导入模块时

用处：将函数添加到中央注册处，url-》相应函数

- 装饰器将函数替换成新的函数(添加一个记时功能给函数)：

```py
@clock
def haha():
	xxxxx


def clock(func):
	def clocked(*arg):
		func(*arg)
	return clocked #返回新的函数了，替换掉原来的haha()
```

- == a.\_\_eq--(b)和比较值 Java 中：equals，is 比较对象标识（地址）Java 中：==
- Copy.deepcopy(obj)会记住循环引用，优雅处理
- @classmethod 最常见的用法是构建备选的构造方法

#### 重试函数

```py
#encoding=utf8

#有参数，必须再包一层（最外层的变量大家都可以访问）
def retry(times=1,exceptions=None):
   exceptions = exceptions if exceptions is not None else Exception

   # 包一层,传入对应的函数
   def wrapper(func):
      #替换函数
      def wrapper(*args,**kwargs):
         last_exception =None
         for _ in range(times):
            try:
               return func(*args, **kwargs)
            except exceptions as e:
               last_exception = e
         raise last_exception
      return wrapper
   return wrapper

if __name__=="__main__":
   @retry(5)
   def test(uid,name):
      print("do something",uid,name)
      raise Exception
   test('123','tome')
```

#### 回调函数 callback 调用

```py
class Callback:
    def run_callback(self,fn,*args,**kwargs):
        print 'call function',fn.__name__
        print args,kwargs
        #fn(args)相当于传入一个元组参数
        #fn(*args)会把元组拆开，给对应的参数赋值
        fn(*args)


    def hi(self,name,age):
        print 'hello,',name,age


    @classmethod
    def start(cls):
        cls().run_callback(cls().hi,'tome',18)

Callback.start()
```
