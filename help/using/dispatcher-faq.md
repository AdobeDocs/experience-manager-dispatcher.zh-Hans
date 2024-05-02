---
title: Dispatcher 热门问题
description: Adobe Experience Manager Dispatcher 的热门问题。
exl-id: 4dcc7318-aba5-4b17-8cf4-190ffefbba75
source-git-commit: 2d90738d01fef6e37a2c25784ed4d1338c037c23
workflow-type: ht
source-wordcount: '1542'
ht-degree: 100%

---

# AEM Dispatcher 常见问题解答

![配置 Dispatcher](assets/CQDispatcher_workflow_v2.png)

## 简介

### 什么是 Dispatcher？

Dispatcher 是 Adobe Experience Manager 的缓存和/或负载平衡工具，有助于实现快速且动态的 Web 创作环境。对于缓存，Dispatcher 充当 HTTP 服务器（如 Apache）的一部分。它旨在存储（即“缓存”）尽可能多的静态网站内容，并尽可能不频繁地访问网站的版面引擎。用于负载平衡时，Dispatcher 将用户请求（负载）分散在不同的 AEM 实例（渲染器）间。

对于执行缓存，Dispatcher 模块利用 Web 服务器的功能来提供静态内容。Dispatcher 将缓存文档放在 Web 服务器的文档根目录下。

### Dispatcher 如何执行缓存？

Dispatcher 利用 Web 服务器的功能来提供静态内容。Dispatcher 将缓存的文档存储在 Web 服务器的文档根目录中。Dispatcher 有两种主要的方法可在对网站作出更改时更新缓存内容。

* **内容更新**&#x200B;删除已更改的页面以及与其直接关联的文件。
* **自动失效**&#x200B;在更新后自动使缓存可能已过期的那些部分失效。例如，它可以有效地将相关页面标记为已过期，而不会删除任何内容。

### 负载平衡有哪些好处？

负载平衡将用户请求（负载）分散在多个 AEM 实例间。以下列表描述了负载平衡的优点：

* **提高了处理能力**：在实践中，此方法意味着 Dispatcher 可在若干 AEM 实例之间分摊文档请求。由于每个实例处理的文档数量减少，您的响应速度将加快。Dispatcher 保留每个文档类别的内部统计信息，以便能够估计负载并高效分发查询。
* **扩大了防故障范围**：如果 Dispatcher 没有从某个实例收到响应，则它自动将请求转发到其他实例。因此，如果一个实例变得不可用，唯一的影响就是站点速度减慢，这与计算能力损失成正比。

>[!NOTE]
>
>有关详细信息，请参阅 [Dispatcher 概述页面](dispatcher.md)

## 安装和配置

### 从何处下载 Dispatcher 模块？

您可以从 [Dispatcher 发行说明](release-notes.md)页面下载最新的 Dispatcher 模块。

### 如何安装 Dispatcher 模块？

请参阅[安装 Dispatcher](dispatcher-install.md) 页面

### 如何配置 Dispatcher 模块？

请参阅[配置 Dispatcher](dispatcher-configuration.md) 页面。

### 如何为创作实例配置 Dispatcher？

有关详细步骤，请参阅[将 Dispatcher 与创作实例结合使用](dispatcher.md#using-a-dispatcher-with-an-author-server)。

### 如何配置 Dispatcher 以在多个域中使用？

您可以配置 CQ Dispatcher 以在多个域中使用，前提是这些域满足以下条件：

* 这些域的 Web 内容都存储在一个 AEM 存储库中
* 可单独为每个域使 Dispatcher 缓存中的文件失效

有关详细信息，请参阅[在多个域中使用 Dispatcher](dispatcher-domains.md)。

### 如何配置 Dispatcher，以便将来自某个用户的所有请求路由到同一个发布实例？

您可以使用[粘性连接](dispatcher-configuration.md#identifying-a-sticky-connection-folder-stickyconnectionsfor)功能，从而确保在同一 AEM 实例上处理某个用户的所有文档。如果您使用个性化的页面和会话数据，则此功能很重要。数据存储在实例上。因此，来自同一用户的后续请求必须返回到该实例，否则数据将丢失。

由于粘性连接会限制 Dispatcher 优化请求的能力，因此应仅在需要时使用此方法。您可以指定包含“粘性”文档的文件夹，从而确保在同一个实例上处理该文件夹中某个用户的所有文档。

### 是否能将粘性连接和缓存结合使用？

对于大多数使用粘性连接的页面，您应禁用缓存。否则，无论会话内容如何，都会向所有用户显示同一页面实例。

对于某些应用程序，可以同时使用粘性连接和缓存。例如，如果您显示的表单将数据写入会话中，则可以将粘性连接和缓存结合使用。

### Dispatcher 和 AEM 发布实例是否能驻留在同一台物理计算机上？

可以，但前提是计算机的功能足够强大。但是，您应该在不同的计算机上设置 Dispatcher 和 AEM Publish 实例。

通常，发布实例位于防火墙内，而 Dispatcher 位于 DMZ 中。如果您决定让发布实例和 Dispatcher 驻留在同一台物理计算机上，请确保防火墙设置禁止从外部网络直接访问发布实例。

### 是否能仅缓存带特定扩展名的文件？

可以。例如，如果您只想缓存 GIF 文件，请在 dispatcher.any 配置文件的缓存部分中指定 *.gif。

### 如何从缓存中删除文件？

您可以使用 HTTP 请求从缓存中删除文件。在收到 HTTP 请求时，Dispatcher 会从缓存中删除文件。Dispatcher 仅在收到对页面的客户端请求时才重新缓存文件。对于不太可能同时收到对同一页面的请求的网站，可通过此方式删除缓存的文件。

HTTP 请求具有以下语法：

```
POST /dispatcher/invalidate.cache HTTP/1.1
CQ-Action: Activate
CQ-Handle: path-pattern
Content-Length: 0
```

Dispatcher 删除名称与 CQ-Handle 标头值匹配的缓存的文件和文件夹。例如，`/content/geomtrixx-outdoors/en` 的 CQ-Handle 与以下项匹配：

geometrixx-outdoors 目录中名称（任何文件扩展名）为 en 的所有文件。
en 目录下任何名为“`_jcr_content`”的目录（如果存在，则包含页面子节点的缓存渲染）。
只有 `CQ-Action` 为 `Delete` 或 `Deactivate`，才能删除 `en` 目录。

有关此主题的其他详细信息，请参阅[手动使 Dispatcher 缓存失效](page-invalidate.md)。

### 如何实施权限敏感型缓存？

请参阅[缓存安全内容](permissions-cache.md)页面。

### 如何保护 Dispatcher 和 CQ 实例之间的通信？

请参阅 [Dispatcher 安全核对清单](security-checklist.md)和 [AEM 安全核对清单](https://experienceleague.adobe.com/en/docs/experience-manager-64/administering/security/security-checklist)页面。

### Dispatcher 问题 `jcr:content` 已更改为 `jcr%3acontent`

**问题**：业务最近在 Dispatcher 级别面临一个问题。从 CQ 存储库获取一些数据的 AJAX 调用之一包含 `jcr:content`。它被编码为 `jcr%3acontent`，导致那个错误的结果集。

**回答**：通过 `ResourceResolver.map()` 方法，使用/发出从中获取请求的“友好”URL 并解决与 Dispatcher 相关的缓存问题。Map() 方法将 `:` 冒号编码为下划线，而 resolve() 方法将其解码回 SLING JCR 可读格式。使用 map() 方法生成在 Ajax 调用中使用的 URL。

有关详细信息，请参阅：[https://sling.apache.org/documentation/the-sling-engine/mappings-for-resource-resolution.html#namespace-mangling](https://sling.apache.org/documentation/the-sling-engine/mappings-for-resource-resolution.html#namespace-mangling)

## 刷新 Dispatcher

### 如何在发布实例上配置 Dispatcher 刷新代理？

请参阅[复制](https://experienceleague.adobe.com/en/docs/experience-manager-65/content/implementing/deploying/configuring/replication#configuring-your-replication-agents)页面。

### 如何解决 Dispatcher 刷新问题？

[请参阅这些故障排除文章](https://experienceleague.adobe.com/search.html?lang=zh-Hans#q=troubleshooting%20dispatcher%20flushing%20issues&amp;sort=relevancy&amp;f:el_product=[Experience%20Manager])。

如果删除操作促使 Dispatcher 进行刷新，请[使用由 Sensei Martin 撰写的本社区博客帖子中提供的解决方法](https://mkalugin-cq.blogspot.com/2012/04/i-have-been-working-on-following.html)。

### 如何从 Dispatcher 缓存刷新 DAM 资产？

您可以使用“链复制”功能。启用此功能后，当收到来自作者的复制时，Dispatcher的刷新代理会发送一个刷新请求。

要启用该功能：

1. [执行此处的步骤](page-invalidate.md#invalidating-dispatcher-cache-from-a-publishing-instance)以在发布实例上创建刷新代理
1. 转到每个代理的配置，并在&#x200B;**触发器**&#x200B;选项卡上选中&#x200B;**接收时**&#x200B;框。

## 其他

Dispatcher 如何确定文档是否为最新版本？
为了确定文档是否为最新版本，Dispatcher 会执行以下操作：

它检查文档是否遵循自动失效机制。如果不是，则认为该文档是最新的。
如果该文档配置为自动失效，则 Dispatcher 检查它比最后一次可用更改旧还是新。如果较旧，则 Dispatcher 从 AEM 实例请求当前版本，并替换缓存中的版本。

### Dispatcher 如何返回文档？

您可以使用 [Dispatcher 配置](dispatcher-configuration.md)文件 `dispatcher.any` 来定义 Dispatcher 是否缓存文档。Dispatcher 根据可缓存文档列表检查请求。如果文档不在此列表中，则 Dispatcher 从 AEM 实例中请求该文档。

`/rules` 属性根据文档路径控制要缓存哪些文档。不论 `/rules` 属性如何，在以下情况下，Dispatcher 绝不会缓存文档：

* 请求 URI 包含 `(?)` 问号。
* 它指示无需缓存的动态页面，如搜索结果。
* 缺失文件扩展名。
* Web 服务器需要扩展名以确定文档类型（MIME 类型）。
* 设置了身份验证标头（可配置）。
* AEM 实例使用以下标头进行响应：
   * no-cache
   * no-store
   * must-revalidate

Dispatcher 将缓存文件存储在 Web 服务器上，当做静态网站的一部分。如果用户请求一个缓存文档，则 Dispatcher 会检查该文档是否存在于 Web 服务器的文件系统中。如果存在，则 Dispatcher 将返回该文档。如果不存在，则 Dispatcher 将从 AEM 实例请求该文档。

>[!NOTE]
>
>GET 或 HEAD（针对 HTTP 标头）方法可由 Dispatcher 缓存。有关响应标头缓存的其他信息，请参阅[缓存 HTTP 响应标头](dispatcher-configuration.md#caching-http-response-headers)部分。

### 是否能在设置中实施多个 Dispatcher？

是。在此类情况下，请确保两个 Dispatcher 都能直接访问 AEM 网站。一个 Dispatcher 无法处理来自另一个 Dispatcher 的请求。
