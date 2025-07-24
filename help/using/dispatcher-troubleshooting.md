---
title: Dispatcher问题疑难解答
description: 了解如何解决 Dispatcher 问题。
cmgrlastmodified: 01.11.2007 08 22 29 [aheimoz]
pageversionid: 1193211344162
template: /apps/docs/templates/contentpage
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
exl-id: 29f338ab-5d25-48a4-9309-058e0cc94cff
source-git-commit: c41b4026a64f9c90318e12de5397eb4c116056d9
workflow-type: tm+mt
source-wordcount: '472'
ht-degree: 93%

---

# Dispatcher问题疑难解答 {#troubleshooting-dispatcher-problems}

>[!NOTE]
>
>Dispatcher 版本独立于 AEM。不过，Dispatcher 文档已嵌入到 AEM 文档中。始终使用嵌入到最新版本的 AEM 文档中的 Dispatcher 文档。
>
>如果您点击了 Dispatcher 文档的链接，则可能重定向到此页面。该链接嵌入在 AEM 先前版本的文档中。

>[!NOTE]
>
>有关详细信息，请查看<!-- URL is 404[Dispatcher Knowledge Base](https://helpx.adobe.com/experience-manager/kb/index/dispatcher.html), -->[Dispatcher刷新问题疑难解答](https://experienceleague.adobe.com/search.html?lang=zh-Hans#q=troubleshooting%20dispatcher%20flushing%20issues&sort=relevancy&f:el_product=[Experience%20Manager])和[Dispatcher常见问题解答](dispatcher-faq.md)。

## 检查基本配置 {#check-the-basic-configuration}

与往常一样，首要步骤是检查基本情况：

* [确认基本操作](/help/using/dispatcher-configuration.md#confirming-basic-operation)
* 查看 Web 服务器和 Dispatcher 的所有日志文件。如有必要，可提高用于 Dispatcher [日志记录](/help/using/dispatcher-configuration.md#logging)的 `loglevel`。

* [检查您的配置](/help/using/dispatcher-configuration.md)：

   * 您是否有多个 Dispatcher？

      * 您是否已确定哪个 Dispatcher 正在处理您所调查的网站/页面？

   * 您是否已实施过滤器？

      * 这些筛选条件是否影响您调查的事情？

## IIS诊断工具 {#iis-diagnostic-tools}

IIS 提供了各种跟踪工具，具体取决于实际版本：

* IIS 6 - 可以下载和配置 IIS 诊断工具
* IIS 7 - 跟踪已完全集成

这些工具可以帮助您监控活动。

<!-- Both URLs in this topic 404! >
## IIS and 404 Not Found {#iis-and-not-found}

When using IIS, you might experience `404 Not Found` being returned in various scenarios. If so, see the following Knowledge Base articles.

* [IIS 6/7: POST method returns 404](https://helpx.adobe.com/experience-manager/kb/IIS6IsapiFilters.html)
* [IIS 6: Requests to URLs that contain the base path `/bin` return a `404 Not Found`](https://helpx.adobe.com/experience-manager/kb/RequestsToBinDirectoryFailInIIS6.html)

Also check that the Dispatcher cache root and the IIS document root are set to the same directory. -->

## 删除工作流模型时出现问题 {#problems-deleting-workflow-models}

**症状**

在通过 Dispatcher 访问 AEM 创作实例的情况下尝试删除工作流模型时出现问题。

**重现步骤：**

1. 登录到您的创作实例（确认正在通过 Dispatcher 路由请求）。
1. 创建一个工作流；例如，将“标题”设置为 workflowToDelete。
1. 确认已成功创建该工作流。
1. 选择并右键单击该工作流，然后单击&#x200B;**删除**。

1. 单击&#x200B;**是**&#x200B;以确认。
1. 随后将出现一个错误消息框，其中显示以下内容：\
   `ERROR 'Could not delete workflow model!!`。

**解决方法**

将以下标头添加到 `dispatcher.any` 文件的 `/clientheaders` 部分：

* `x-http-method-override`
* `x-requested-with`

```
{  
{  
/clientheaders  
{  
...  
"x-http-method-override"  
"x-requested-with"  
}
```

## 对 mod_dir 的干预 (Apache) {#interference-with-mod-dir-apache}

此过程描述 Dispatcher 如何与 Apache Web Server 中的 `mod_dir` 进行交互，因为这可能会导致多种有可能意想不到的效果：

### Apache 1.3 {#apache}

在 Apache 1.3 中，`mod_dir` 处理每个这样的请求：其中 URL 映射到文件系统中的某个目录。

它会：

* 将请求重定向到现有 `index.html` 文件
* 生成目录列表

启用 Dispatcher 后，它通过将自身注册为 `httpd/unix-directory` 内容类型的处理程序而处理此类请求。

### Apache 2.x {#apache-x}

在 Apache 2.x 中，情况则有所不同。模块可以处理请求的不同阶段，例如 URL 修复。`mod_dir` 通过将请求（当 URL 映射到目录时）重定向到追加了 `/` 的 URL 而处理此阶段。

Dispatcher 不拦截 `mod_dir` 修复，而是完整地处理对重定向的 URL（即追加了 `/`）的请求。如果远程服务器（例如 AEM）处理对 `/a_path` 的请求的方式与处理对 `/a_path/` 的请求（当 `/a_path` 映射到现有目录时）的方式不同，则此过程可能会引发问题。

如果出现这种情况，您必须：

* 对 Dispatcher 处理的 `Directory` 或 `Location` 子树禁用 `mod_dir`

* 使用 `DirectorySlash Off` 配置 `mod_dir` 以不追加 `/`
