---
date: 2022-11-25T18:02:00+08:00
title: 我的独立开发技术栈

---

刚开始做自己的 SideProject 时，我花了很多时间在纠结选择哪些编程语言和框架这类问题上，我尝试过 Python + FastAPI，最后还是受不了 Python 难用的类型系统，又换回了 TypeScript + NestJS，所以决定写一篇文章分享一下我做 SideProject 所使用的技术栈。

## 选择技术栈的原则

技术需要为业务服务，自己的 SideProject 意味着体量小，灵活度高，需要快速完成市场验证，而且没有过多的限制，可以用你最喜欢的技术栈。

SideProject 选择技术栈并没有什么明确的原则，我的原则是：使用你最舒服的技术栈。要做一个网站？React、 Vue Svelte、jQuery 都可以完成，要开发一个 APP?    原生、Flutter、React Native 也能胜任，选择一种你最熟悉的技术栈，不要过于跨平台性能不好，体验不好什么的，先做出来比什么都重要。

## 编程语言

虽然我们常说编程语言只是工具，但实际上编程语言的影响不止于此。

> 我的语言之局限，即我的世界之局限。

这句话不仅适用于我们日常说的语言，也同样适用于编程语言，一门语言会悄无声息地影响着你的思维方式。写 Java 的开发者即便使用其他的编程语言，大概率也会情不自禁写出 Model，Service，Controller，Dao 这样充满 Java 味的代码。

对于 SideProject 来说，我会选择自己主业所使用的语言，这意味着你可以熟练地上手，快速完成 MVP，得到反馈。 我是一位前端开发者，所以 TypeScript 既是我主业使用的语言，也是我的 SideProject 所使用的语言。 TypeScript 相当全面，前端、后端,、微信小程序、Android APP、iOS APP 都可以用它来开发。

## 框架

### 后端

**[NestJS](https://nestjs.com/)**

NestJS 是我最喜欢的后端框架，提供了开箱即用的架构，设计上采用了依赖注入的方式，和 Java Spring 非常相似，使用 NestJS 开发后端体验非常舒服，代码条理清晰，有一种美感。

**[Prisma](https://www.prisma.io/)**

Prisma 是一个 ORM 库，定义好 Model 后能直接生成 CRUD 方法，可以极大提高生产力，非常强大。

### 前端

**[React](https://reactjs.org/)**

最流行的前端框架。

**[Tailwind CSS](https://tailwindcss.com/)**

Tailwind CSS 最大的好处在于它出自设计师之手，自带了一套设计系统，使用Tailwind CSS 可以帮助你快速做出不丑的 UI，同时还极大提升了 CSS 的开发体验。此外 Tailwind CSS 拥有丰富的生态,有非常多的 UI Kit 基于 Tailwind CSS，例如 [Flobite](https://flowbite.com/)、 [HyperUI](https://www.hyperui.dev/) 、 [daisyUI](https://daisyui.com/)。

**[Svelte](https://svelte.dev/)**

我使用 Svelte 开发浏览器插件，开发过程十分丝滑，不像 React 那样庞大复杂，它小巧又实用，用来开发浏览器插件再合适不过。建议使用 [Svelte Typescript Chrome Extension Boilerplate](https://github.com/NekitCorp/chrome-extension-svelte-typescript-boilerplate) 这个 Starter 一起使用

**[Next.js](https://nextjs.org/)**

Next.js 的好处是大幅降低了 React 的配置难度，开箱即用, 让你不用关注业务无关的东西。Next.js是一个全栈框架，不仅能写前端，也能写后端，对于后端简单的产品，Next.js的后端功能完全够用，快速开发完，一键部署到 Vercel。

## 工具

- 统计分析：[Plausible](https://plausible.io/)
- 海外收款：[Paddle](https://www.paddle.com/)
- 用户反馈收集：[腾讯兔小巢](https://support.qq.com/)

