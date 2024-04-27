---
title: 配置 Dispatcher 以防御 CSRF 攻击
description: 了解如何配置 AEM Dispatcher 以防御跨站点请求伪造攻击。
topic-tags: dispatcher
content-type: reference
exl-id: bcd38878-f977-46a6-b01a-03e4d90aef01
source-git-commit: 2d90738d01fef6e37a2c25784ed4d1338c037c23
workflow-type: tm+mt
source-wordcount: '217'
ht-degree: 46%

---

# 配置 Dispatcher 以防御 CSRF 攻击{#configuring-dispatcher-to-prevent-csrf-attacks}

AEM 提供了一个用于防御跨站点请求伪造攻击的框架。要正确使用此框架，请对Dispatcher配置进行以下更改：

>[!NOTE]
>
>请务必根据您的现有配置更新以下示例中的规则编号。请记住，调度程序使用最后一个匹配规则来授予允许或拒绝权限，因此请将规则放在现有列表的底部附近。

1. 在 `/clientheaders` 部分 `author-farm.any` 和 `publish-farm.any`，在列表底部添加以下条目：\
   `CSRF-Token`
1. 在的/filters部分中 `author-farm.any` 和 `publish-farm.any` 或 `publish-filters.any` 文件，添加以下行以允许请求 `/libs/granite/csrf/token.json` 通过Dispatcher。\
   `/0999 { /type "allow" /glob " * /libs/granite/csrf/token.json*" }`
1. 在 `/cache /rules` 部分 `publish-farm.any`，添加规则以阻止调度程序缓存 `token.json` 文件。 通常创作实例会绕过缓存，因此无需将规则添加到 `author-farm.any`.\
   `/0999 { /glob "/libs/granite/csrf/token.json" /type "deny" }`

要验证配置是否有效，请在调试模式下查看 dispatcher.log 以验证 token.json 文件是否未缓存且未被过滤器阻止。您应看到与以下内容类似的消息：\
`... checking [/libs/granite/csrf/token.json]  `
`... request URL not in cache rules: /libs/granite/csrf/token.json`\
`... cache-action for [/libs/granite/csrf/token.json]: NONE`

您还可以在Apache中验证请求是否成功 `access_log`. 对 ``/libs/granite/csrf/token.json 的请求应返回 HTTP 200 状态代码。
