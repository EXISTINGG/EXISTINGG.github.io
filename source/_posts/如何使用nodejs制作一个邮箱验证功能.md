---
title: 如何使用nodejs制作一个邮箱验证功能
date: 2023-06-18 21:08:49
tags: 
  - nodejs
categories:
  - nodejs
---
# 教程：如何使用nodejs制作一个邮箱验证功能

本教程将逐步引导您制作一个具有验证功能的应用程序。我们将使用 Express、Body Parser 和 Nodemailer 库来实现该功能。

# 准备

下载所需的库
``` bash
npm i body-parser && nodemailer
```


## 步骤 1：设置 Express 应用程序

首先，我们需要创建一个 Express 应用程序，并引入所需的库。

```javascript
const express = require('express');
const bodyParser = require('body-parser');
const nodemailer = require('nodemailer');

// 创建 Express 应用程序
const app = express();

// 解析请求体
app.use(bodyParser.urlencoded({ extended: false }));
app.use(bodyParser.json());
```

## 步骤 2：配置邮件发送

接下来，我们将配置 Nodemailer 来发送邮件。在这个例子中，我们将使用 QQ 邮箱作为邮件服务器。使用前请开通SMTP服务，简单易开通。具体步骤可参考网上教程。

```javascript
// 邮件发送配置
const transport = nodemailer.createTransport({
  host: 'smtp.qq.com', //连接的邮箱服务器
  secureConnection: true, // 使用 SSL 进行加密
  port: 465, // SMTP的端口号
  auth: {
    user: 'xxx@qq.com', // 发送邮件的邮箱
    pass: '**********', // 邮箱密码或授权码
  },
});
```

## 步骤 3：生成验证码

我们将创建一个函数来生成验证码。在这个例子中，我们将生成一个 6 位数的验证码。

```javascript
// 生成验证码
const generateCode = () => {
  const code = Math.floor(Math.random() * 1000000).toString().padStart(6, '0');
  return code;
};
```


## 步骤 4：存储验证码和有效期

我们将使用对象来存储验证码和验证码的有效期。我们创建两个空对象 `codeMap` 和 `codeExpiryMap` 来分别存储验证码和验证码的有效期，以邮箱作为键。

```javascript
const codeMap = {}; // 使用对象来存储验证码，以邮箱为键
const codeExpiryMap = {}; // 使用对象来存储验证码的有效期，以邮箱为键
```

## 步骤 5：发送验证码邮件

在这一步中，我们将创建一个路由来处理发送验证码的请求，并发送带有验证码的邮件。

```javascript
// 发送验证码邮件
app.post('/send-code', (req, res) => {
  const email = req.body.email;
  const code = generateCode();
  console.log(email, code);

  // 设置验证码有效期为 5 分钟
  const expiryTime = new Date().getTime() + 5 * 60 * 1000;
  codeExpiryMap[email] = expiryTime;

  // 邮件内容
  const mailOptions = {
    from: '小明 xxxx@qq.com', // 发送者邮箱
    to: email, // 接收者邮箱
    subject: '欢迎使用 xxx，验证码提醒', // 邮件主题
    html: `
      <div>
        <h1>欢迎您使用 xxx！</h1>
        <p>您的验证码是：<span style="color: blue; font-size: 20px;">${code}</span></p>
        <p>请注意：请勿向任何人透露此验证码，包括本网站的工作人员。本邮件是您注册 xxx 的验证码，请勿回复此邮件。</p>
        <p>如果您未在 xxx 上进行任何操作，请忽略此邮件。</p>
        <p>验证码将在5分钟内有效，过期后将无法使用。</p>
        <p>谢谢！</p>
      </div>
    `,
  };

  // 发送邮件
  transport.sendMail(mailOptions, (error, info) => {
    if (error) {
      console.log(error);
      res.status(500).send('邮件发送失败');
    } else {
      console.log(`验证码已发送至${email}`);
      codeMap[email] = code; // 存储验证码，以邮箱为键
      console.log(1, codeMap[email]); // 添加此行，验证验证码是否正确存储
      res.status(200).send('验证码已发送');
    }
  });
});
```
请注意：以上代码只是演示用途，实际应用中需要进行适当的安全性和错误处理。


## 步骤 6：验证验证码

最后一步是创建一个路由来验证用户输入的验证码是否正确。

```javascript
// 验证验证码
app.post('/verify-code', (req, res) => {
  const email = req.body.email;
  const code = req.body.code;
  const savedCode = codeMap[email];
  console.log(codeMap[email]);
  console.log(email, code, savedCode);

  // 检查验证码是否过期
  const currentTime = new Date().getTime();
  if (currentTime > codeExpiryMap[email]) {
    console.log(`${email}的验证码已过期`);
    res.status(401).send('验证码验证失败');
    return;
  }

  if (code && savedCode && code.toString() === savedCode) {
    console.log(`${email}的验证码验证通过`);
    delete codeMap[email]; // 验证通过后删除验证码
    res.status(200).send('验证码验证通过');
  } else {
    console.log(`${email}的验证码验证失败`);
    res.status(401).send('验证码验证失败');
  }
});
```


## 步骤 7：启动服务器

最后，我们通过监听端口来启动服务器。

```javascript
app.listen(3000, () => {
  console.log('服务器已启动');
});
```



