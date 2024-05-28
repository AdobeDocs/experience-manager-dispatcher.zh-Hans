---
title: 配置Adobe Experience Manager Dispatcher以防御CSRF攻击
description: 了解如何配置Adobe Experience Manager Dispatcher以防御跨站点请求伪造攻击。
topic-tags: dispatcher
content-type: reference
exl-id: bcd38878-f977-46a6-b01a-03e4d90aef01
source-git-commit: 0a1aa854ea286a30c3527be8fc7c0998726a663f
workflow-type: tm+mt
source-wordcount: '235'
ht-degree: 48%

---

# 配置Adobe Experience Manager Dispatcher以防御CSRF攻击{#configuring-dispatcher-to-prevent-csrf-attacks}

AEM (Adobe Experience Manager)提供了一个用于防御跨站点请求伪造攻击的框架。 要正确使用此框架，请对Dispatcher配置进行以下更改：

>[!NOTE]
>
>请务必根据您的现有配置更新以下示例中的规则编号。请记住，Dispatcher使用最后一个匹配规则来授予允许或拒绝权限，因此请将规则放在现有列表的底部附近。

1. 在 `author-farm.any` 和 `publish-farm.any` 的 `/clientheaders` 部分中，将以下条目添加到列表底部：\
   `CSRF-Token`
1. 在 `author-farm.any` 和 `publish-farm.any` 或 `publish-filters.any` 文件的 /filters 部分中，添加以下行以允许通过 Dispatcher 请求 `/libs/granite/csrf/token.json`。\
   `/0999 { /type "allow" /glob " * /libs/granite/csrf/token.json*" }`

1. 在 `publish-farm.any` 的 `/cache /rules` 部分下，添加规则以阻止 Dispatcher 缓存 `token.json` 文件。通常创作实例会绕过缓存，因此您无需将规则添加到 `author-farm.any` 中。

   `/0999 { /glob "/libs/granite/csrf/token.json" /type "deny" }`

要验证配置是否正常工作，请在调试模式下查看dispatcher.log 。 它可以帮助您验证 `token.json` 文件，以确保不会被过滤器缓存或阻止。 您应看到与以下内容类似的消息：\
`... checking [/libs/granite/csrf/token.json]  `
`... request URL not in cache rules: /libs/granite/csrf/token.json`\
`... cache-action for [/libs/granite/csrf/token.json]: NONE`

您也可以在 Apache `access_log` 中验证请求是否成功。对 ``/libs/granite/csrf/token.json 的请求应返回 HTTP 200 状态代码。
