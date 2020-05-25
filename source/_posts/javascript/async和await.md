---
title: async和await
tags:
  - await
  - async
categories:
  - js
date: 2017-12-04 15:41:49
---

## async和await示例

为什么异步执行？  
> 不影响UI渲染  

为什么await？  
> 函数执行前后顺序保证

```js
function resolveAfter2Seconds() {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve('3.resolved');
    }, 2000);
  });
}

async function asyncCall() {
  console.log('1.calling');
  // 阻塞等待resolveAfter2Seconds函数返回的Promise解析完成
  var result = await resolveAfter2Seconds();
  console.log(result);
  // expected output: 'resolved'
}

asyncCall(); // 直接调用
console.log("2.hello");

// 等待所有任务执行完成才结束 exit(0)
```
