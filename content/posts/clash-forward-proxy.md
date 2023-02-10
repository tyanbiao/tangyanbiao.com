---
title: 'Clash 配置前置代理'
date: '2023-02-10T15:31:00+08:00'
---

内网环境下访问外网需要通过特定的代理服务器，但是 Clash 没有支持配置前置代理，我一直都是使用 Proxifier 这个软件为 Clash 设置代理，今天查看 Clash 的文档发现原来可以通过 Proxy Chains 策略实现前置代理的功能。

简单来说 Proxy Chains 就是链式代理，Clash 会将代理节点串成一个代理链，流量会按照顺序逐级转发到代理服务器，每个代理服务器都只能看到上一个代理服务器发送的请求。

配置起来还是挺简单的，新建一个 type 为 `relay` 的 `proxy-group`，在 `proxies` 按照顺序输入代理服务器节点。

```yml
proxies:
  - name: relay-server
	type: socks5
	server: server
	port: 1080
proxy-groups:
  - name: relay⏭
    type: relay
    proxies:
      - relay-server
      - output
  - name: output
    type: select
    proxies:
      - 香港 01
      - 香港 02
      - 香港 03
      - 日本 01
```

这是我第一次通过 `yaml` 来修改 Clash 配置，Clash for Windows 的体验做得很优秀，只需要从网站点击“导入 Clash”，一切就配置好了，压根就不需要看文档。直到遇到问题，把 [Clash for Windows 文档](https://docs.cfw.lbyczf.com/) 和 [Clash 文档](https://lancellc.gitbook.io/clash) 都过了一遍，不仅找到解决方案，还收获了许多高级技巧。



