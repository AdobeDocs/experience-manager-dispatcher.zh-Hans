---
title: Dispatcher 问题疑难解答
description: 了解如何解决 Dispatcher 问题。
cmgrlastmodified: 01.11.2007 08 22 29 [aheimoz]
pageversionid: 1193211344162
template: /apps/docs/templates/contentpage
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
exl-id: 29f338ab-5d25-48a4-9309-058e0cc94cff
source-git-commit: 2d90738d01fef6e37a2c25784ed4d1338c037c23
workflow-type: ht
source-wordcount: '538'
ht-degree: 100%

---

# Dispatcher 问题疑难解答 {#troubleshooting-dispatcher-problems}

>[!NOTE]
>
>虽然 Dispatcher 版本独立于 AEM，但 Dispatcher 文档会嵌入到 AEM 文档中。始终使用嵌入到最新版本的 AEM 文档中的 Dispatcher 文档。
>
>您可能是在单击以前版本的 AEM 文档中嵌入的 Dispatcher 文档链接后重定向到此页面。

>[!NOTE]
>
>请查看 [Dispatcher 知识库](https://helpx.adobe.com/cn/experience-manager/kb/index/dispatcher.html)、[解决 Dispatcher 刷新问题](https://experienceleague.adobe.com/search.html?lang=zh-Hans#q=troubleshooting%20dispatcher%20flushing%20issues&amp;sort=relevancy&amp;f:el_product=[Experience%20Manager])和 [Dispatcher 常见问题解答](dispatcher-faq.md)，了解更多信息。

## 检查基本配置 {#check-the-basic-configuration}

与往常一样，首要步骤是检查基本情况：

* [确认基本操作](/help/using/dispatcher-configuration.md#confirming-basic-operation)
* 查看 Web 服务器和 Dispatcher 的所有日志文件。如有必要，可提高用于 Dispatcher [日志记录](/help/using/dispatcher-configuration.md#logging)的 `loglevel`。

* [检查您的配置](/help/using/dispatcher-configuration.md)：

   * 您是否有多个 Dispatcher？

      * 您是否已确定哪个 Dispatcher 正在处理您所调查的网站/页面？

   * 您是否已实施过滤器？

      * 这些筛选条件是否影响您调查的事情？

## IIS 诊断工具 {#iis-diagnostic-tools}

IIS 提供了各种跟踪工具，具体取决于实际版本：

* IIS 6 - 可以下载和配置 IIS 诊断工具
* IIS 7 - 跟踪已完全集成

这些工具可以帮助您监控活动。

## IIS 和 404 Not Found {#iis-and-not-found}

在使用 IIS 时，您可能在多种场景下遇到返回 `404 Not Found` 的情况。如果是这样，请参阅以下知识库文章。

* [IIS 6/7：POST 方法返回 404](https://helpx.adobe.com/cn/experience-manager/kb/IIS6IsapiFilters.html)
* [IIS 6：对包含基本路径 `/bin` 的 URL 的请求返回 `404 Not Found`](https://helpx.adobe.com/cn/experience-manager/kb/RequestsToBinDirectoryFailInIIS6.html)

还应检查 Dispatcher 缓存根目录和 IIS 文档根目录是否已设为同一目录。

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
   &quot; `ERROR 'Could not delete workflow model!!`&quot;.

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
