---
title: 产品笔记：Mixpanel 埋点
date: '2023-12-07T23:36:00+08:00'
tags: ['产品笔记']
---

## 为什么需要埋点
简单来说，因为需要了解用户，知道他们是谁、如何使用我们的产品，从而改进产品。每一个埋点就像城市里的监控摄像头，悄无声息地记录用户的行为，通过分析这些数据来为产品决策提供数据支撑。

拿到这些数据后可以干什么，举一个很简单的例子，你能知道有多少人访问了产品的落地页，其中又有多少人点击了购买按钮，然后计算出一个转化率，可以对比同行，看看自己的转化率是高还是低，另外如果你想要优化落地页、改进产品，这些数据就可以用来衡量你的改进有没有效果。

没有数据，就像摸黑走夜路。

## Mixpanel 是什么
[Mixpanel](https://mixpanel.com/) 是一款分析工具，可以帮助你跟踪用户使用产品过程中的互动数据。类似的工具还有友盟和 Google Analytics。

Mixpanel 有三个关键概念：Events, Users, Properties。

Events（事件）表示用户与产品的交互行为，例如浏览了某个页面，点击了某个按钮，注册成功，购买成功等。

Users（用户）表示 Events 的主体，事件都是用户触发的，用户是我们需要研究的主体。Mixpanel 中每个上报的事件都有一个 `distinct_id` 属性，代表了一个唯一的用户。

Properties（属性）表示一个事件或用户的具体特征，一个页面浏览事件，就有 `URL Path`, `Country`, `Browser`, `Operating System` 这些属性，对于后续的分析很有用。

## 埋点策略
埋点策略，就是确定在产品中需要跟踪哪些事件，而这背后，其实是你想通过数据来解决什么问题，Mixpanel 的文档中给出了三个方向：

**参与度**：衡量用户与产品互动频率和深度的指标。它可以通过页面浏览次数，点击次数，完成的任务数量，社交互动（例如“喜欢”，评论，分享）等方式来测量。高参与度通常表示用户对产品非常满意，并且经常使用它。

**增长**：衡量产品用户基数或活跃用户数量增长的指标。增长可能通过口碑，社交媒体营销，搜索引擎优化（SEO），付费广告等方式来实现，或者通过提高产品的参与度和用户满意度来保留现有用户。

**留存**：衡量用户在一段时间后是否仍然使用产品的指标。留存率可以分为短期留存和长期留存。短期留存通常指的是用户在下载或注册产品后的几天或几周内是否仍然在使用产品。长期留存则是指用户在几个月甚至几年后是否仍然在使用产品。高留存率意味着产品能够持续满足用户的需求，从而使他们持续使用。常使用的指标包括：次日留存、三日留存、七日留存、半月留存和月留存。
## Chrome Extension 接入 Mixpanel SDK
接入 Mixpanel 的 SDK 还是很简单的，官方提供了非常多平台的 SDK，照着文档基本上不会有啥问题。但是官方的 SDK 偏偏不支持浏览器插件，准确地说，Manifest v3 以后，background 中去掉了 localStorage 和 XMLHttpRequest 的支持，导致 Mixpanle SDK 无法在 background 中正常使用。

我的解决办法是将 `chrome.storage.local` 包装成 `localStorage` 并挂载到 `widnow` 对象上。至于无法使用 XHR 的问题，在 GitHub 上有人提了 [PR](https://github.com/mixpanel/mixpanel-js/pull/342), 但是一直没有被合并，所以只能暂时使用这位老哥的 fork ，好在它确实是正常工作的。

## 数据收集和分析
产品接入埋点之后，接下来要做的就是耐心等待积累数据并进行分析，这部分的内容打算之后再单独写一篇文章进行介绍。

## 最后
产品笔记这个栏目会记录我在开发产品过程中学习到的知识，主要是出海、产品、营销方面的内容。这个栏目会长期更新，通过将学到的知识写下来，一方面可以确认自己是否真的理解了，另一方面也许会对别人有帮助。