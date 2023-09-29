---
title: js分号的坑 是真的坑
date: 2023-09-29 19:14:30
tags:
  - nodejs
categories:
  - nodejs
---

当涉及到JavaScript中分号的使用时，有一些常见的坑需要注意。分号是JavaScript语法中的重要组成部分，它表示语句的结束。虽然JavaScript解释器通常会尝试自动插入分号，但这个行为是不稳定的，可能会导致一些令人困惑和难以诊断的问题。在下面的例子中，我们将探讨一个关于分号的常见陷阱。

### 隐式分号插入

在JavaScript中，当你在不同行上编写代码时，解释器通常会自动插入分号以结束语句。这意味着你可能会在没有明确添加分号的情况下创建多个语句，从而导致代码不按预期工作。

考虑以下代码片段：

```javascript
let queryRes
let querySql
if (username) {
  // 使用用户名进行查询
  querySql = 'SELECT * FROM user WHERE username = ?'
  [queryRes] = await db.query(querySql, username)
} else {
  // 使用邮箱进行查询
  querySql = 'SELECT * FROM user WHERE email = ?'
  [queryRes] = await db.query(querySql, email)
}
```
运行结果
```shell
TypeError [ERR_INVALID_ARG_TYPE]: The first argument must be of type string or an 
instance of Buffer, ArrayBuffer, or Array or an Array-like Object. Received undefined
```

在这个示例中，JavaScript解释器会在querySql = 'SELECT * FROM user WHERE username = ?'的末尾自动插入分号，因此它实际上被解释为两个独立的语句，而不是一个赋值语句。这可能导致意外的行为和错误。

为了避免这个问题，你应该始终在语句的末尾显式添加分号，以明确语句的结束：

```javascript
let queryRes;
let querySql;
if (username) {
  // 使用用户名进行查询
  querySql = 'SELECT * FROM user WHERE username = ?';
  [queryRes] = await db.query(querySql, username);
} else {
  // 使用邮箱进行查询
  querySql = 'SELECT * FROM user WHERE email = ?';
  [queryRes] = await db.query(querySql, email);
}
```
加上分号后正常运行

