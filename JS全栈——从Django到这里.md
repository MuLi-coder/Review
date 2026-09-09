
# 一、写在之前

自从上一次我尝试用django进行前后端的耦合开发（  [[Django_review]]  ），到现在已经快一年了。

曾经认真做的复盘，似乎觉得自己已经理解了全栈的数据流，然而近来因Agent部署的事情，让我不得不重新审视自己的知识，似乎我还并没有完整的刻画出Web世界的图景，哪怕是在上一个项目中再深深的追问，原来不过皮毛而已。虽说不是一窍不通，但也不过半斤八两。所以借着这一次的学习，我们进一步把Web开发这个3D点云丰富一下。

这次将会是一场漫长的旅行，而JS全栈也不过是旅程的第一站。

# 二、为什么是JS

为什么是JS？

答案很现实，单纯就是因为 Vercel 的亲儿子是JS，而 Vercel 的部署是免费的，仅此而已。不过是为了让以后自己可以随时的部署自己喜欢，自己打造的项目。

所以选择了JS作为这次全栈开发的生态。


# 三、一些前置的认识

在正式开始说明之前，我们必须首先建立一些基本的认知
##

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



