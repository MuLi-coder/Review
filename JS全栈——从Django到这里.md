
# 一、写在之前

自从上一次我尝试用django进行前后端的耦合开发（  [[Django_review]]  ），到现在已经快一年了。

曾经认真做的复盘，似乎觉得自己已经理解了全栈的数据流，然而近来因Agent部署的事情，让我不得不重新审视自己的知识，似乎我还并没有完整的刻画出Web世界的图景，哪怕是在上一个项目中再深深的追问，原来不过皮毛而已。虽说不是一窍不通，但也不过半斤八两。所以借着这一次的学习，我们进一步把Web开发这个3D点云丰富一下。

这次将会是一场漫长的旅行，而JS全栈也不过是旅程的第一站。

# 二、为什么是JS

为什么是JS？

答案很现实，单纯就是因为 Vercel 的亲儿子是JS，而 Vercel 的部署是免费的，仅此而已。不过是为了让以后自己可以随时的部署自己喜欢，自己打造的项目。

所以选择了JS作为这次全栈开发的生态。


# 三、一些前置的认识

在正式开始说明之前，我们必须首先建立一些基本的认知：

## 生态结构

前端和后端分别部署，前端由前端JS发起网络请求，后端由node.js JS发起网络请求。

![[全栈结构.png]]

# 四、Web的数据流动

我们先定下我们项目的整体架构：

我们将前端和后端分别作为两个项目独立部署在Vercel中，利用API请求后端。

下图就是数据流的大致过程，我们后续来细致的说明：

```
① User
      │
      ▼
② Browser
      │
      │ GET Frontend URL
      ▼
③ Vercel Frontend Project
      │
      │ HTML / CSS / JS
      ▼
④ Browser
      │
      │ Execute app.js
      │
      │ fetch()
      ▼
⑤ Vercel Backend Project
      │
      │ Route
      ▼
⑥ Backend Function
      │
      │ Query
      ▼
⑦ Database
      │
      │ Todo Data
      ▼
⑧ Backend Function
      │
      │ JSON
      ▼
⑨ Browser
      │
      │ JavaScript
      ▼
⑩ DOM Update
      │
      ▼
⑪ User sees Todo List
```

## 1. 故事开始在前端部署之后

当前端资源部署在 Vercel 之后，假设我们得到了一个前端项目的网址：

`https://todo-frontend.vercel.app`

在这个网址对应的服务器中，部署这我们的前端文件，比如：

```
todo-frontend/
│
├── index.html
│
├── style.css
│
└── app.js
```

## 2. 当用户按下Enter之后

当我们把URL分享给你的好大儿之后，他在地址栏输入 `https://todo-frontend.vercel.app` ，然后按下了Enter

暂时略过DNS，TCP，IP这些细节，直接关注Web应用层：

```
Browser
   │
   │ HTTP/HTTPS Request
   │
   │ GET /
   ▼
Vercel
```

浏览器向部署的服务器发出HTTP请求，请求形式是GET，这一步就相当于：

> 浏览器告诉服务器：“请把这个网址对应的资源给我”

此时，服务器往往会返回html文件，后续浏览器在解析的时候，会发现文件不完整，还会再次发起请求，所以整个过程更像：

```
① Browser
    │
    │ GET /
    ▼
② Vercel
    │
    │ index.html
    ▼
③ Browser
    │
    │ 解析 HTML
    │
    ├──────── GET /style.css ───────►
    │
    └──────── GET /app.js ──────────►
```

整体归结起来，第一部分的数据流就可以理解为（抽出总数据流的1-4部分）：

```
① User
      │
      ▼
② Browser
      │
      │ GET Frontend URL
      ▼
③ Vercel Frontend Project
      │
      │ HTML / CSS / JS
      ▼
④ Browser
```

## 3. 浏览器拿到资源之后

当浏览器拿到资源之后，就开始渲染了，此时如果JS中没有向外访问API的部分，那么浏览器就可以把完整的内容端在页面上了。就完成了一次访问。

然而，如果项目需要外部数据，那么前端JS就会通过API访问相应的后端服务器，再次发起HTTP/HTTPS请求，比如依靠 `fetch`。

就可以理解为：

> 浏览器渲染页面时发现缺少必要的数据，根据前端JS代码，向相应的后端服务器索要相关的资源

也就是：

```
④ Browser
      │
      │ Execute app.js
      │
      │ fetch()
      ▼
⑤ Vercel Backend Project
```

## 4. 前端向后端发出请求之后

比如说 `fetch` 函数中是这样写的：

`fetch(https://todo-backend.vercel.app/api/todos)`

其中：

- Domin : `https://todo-backend.vercel.app`

域名告诉浏览器：

> 请求目标是那个Web服务

- Path : `/api/todos`

路径告诉后端：

> 我想访问这个服务器的哪个接口

然后当这个请求到了后端，到了 `todo-backend.vercel.app`

Vercel接收到了，知道这是发给 todo-backend 项目的请求，那么问题来了，这个项目里面有很多的功能，例如：

```
todo-backend/
│
├── api/
│   │
│   ├── todos.js
│   │
│   ├── users.js
│   │
│   └── login.js
│
└── package.json
```

到底应该执行哪一个功能呢？

这个时候就由 ( Routing ) 路由发挥作用了

后端维护一个路由表，将URL和对应的程序进行对应。

具体而言，就是Vercel会根据 URL Path 去和路由表匹配，匹配到相应的功能程序，也就是流程中的：

```
⑤ Vercel Backend Project
      │
      │ Route
      ▼
⑥ Backend Function
```

## 5. 当后端开始工作之后

当相应功能匹配上之后，Vercel 提供的后端运行环境中相应的JS功能代码就开始运行，

