---
title: 优化网站缓存性能
description: 了解如何设计您的网站以最大限度地提高缓存优势。
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
redirecttarget: https://helpx.adobe.com/experience-manager/6-4/sites/deploying/using/configuring-performance.html
index: y
internal: n
snippet: y
source-git-commit: 2d90738d01fef6e37a2c25784ed4d1338c037c23
workflow-type: tm+mt
source-wordcount: '1125'
ht-degree: 73%

---


# 优化网站缓存性能 {#optimizing-a-website-for-cache-performance}

<!-- 

Comment Type: remark
Last Modified By: Silviu Raiman (raiman)
Last Modified Date: 2017-10-25T04:13:34.919-0400

<p>This is a redirect to /experience-manager/6-2/sites/deploying/using/configuring-performance.html</p>

 -->

>[!NOTE]
>
>Dispatcher 版本独立于 AEM。您可能是在单击以前版本的 AEM 文档中嵌入的 Dispatcher 文档链接后重定向到此页面。

Dispatcher提供了几种内置机制，您可以使用它们来优化性能。 此部分介绍了如何设计您的网站以最大限度地提高缓存优势。

>[!NOTE]
>
>Dispatcher 将缓存存储在标准 Web 服务器上，记住这一点可能会对您有所帮助。这意味着您：
>
>* 可以缓存可存储为页面并使用URL请求的所有内容
>* 无法存储其他内容，例如HTTP标头、Cookie、会话数据和表单数据。
>
>通常，许多缓存策略涉及选择完好的URL，并且不依赖此类额外数据。

## 使用一致的页面编码 {#using-consistent-page-encoding}

由于不会缓存HTTP请求标头，因此如果您将页面编码信息存储在标头中，则会出现问题。 在此情况下，当 Dispatcher 从缓存中提供一个页面时，Web 服务器的默认编码将用于该页面。可通过两种方式避免此问题：

* 如果您只使用一种编码，请确保 Web 服务器上使用的编码与 AEM 网站的默认编码相同。
* 要设置编码，请使用 `<META>` HTML中的标记 `head` 部分，如以下示例所示：

```xml
        <META http-equiv="Content-Type" content="text/html; charset=EUC-JP">
```

## 消除 URL 参数 {#avoid-url-parameters}

如果可能，请消除要缓存的页面的 URL 参数。例如，如果您有一个图片库，则绝不会缓存以下 URL（除非对 Dispatcher 进行[相应配置](dispatcher-configuration.md#main-pars_title_24)）：

```xml
www.myCompany.com/pictures/gallery.html?event=christmas&amp;page=1
```

不过，您可以将这些参数放入页面 URL 中，如下所示：

```xml
www.myCompany.com/pictures/gallery.christmas.1.html
```

>[!NOTE]
>
>此 URL 调用与 gallery.html 相同的页面和模板。在模板定义中，您可以指定用于渲染页面的脚本，也可以对所有页面使用同一脚本。

## 按 URL 进行自定义  {#customize-by-url}

如果您允许用户更改字体大小（或任何其他版面自定义设置），请确保各种自定义设置都反映在 URL 中。

例如，由于不会缓存 cookie，因此如果您将字体大小存储在 cookie（或类似机制）中，则不会为缓存的页面保留字体大小。因此，Dispatcher 会随机返回任意字体大小的文档。

在 URL 中包含字体大小作为选择器可避免出现此问题：

```xml
www.myCompany.com/news/main.large.html
```

>[!NOTE]
>
>对于大多数版面，也可以使用样式表和/或客户端脚本。 这些通常可以与缓存很好地配合使用。
>
>这对于打印版本也很有用，您可以在其中使用URL，例如：
>
>`www.myCompany.com/news/main.print.html`
>
>通过使用模板定义的脚本通配，您可以指定单独的脚本来渲染打印页面。

## 使用作标题的图像文件失效 {#invalidating-image-files-used-as-titles}

如果您将页面标题或其他文本渲染为图片，请存储文件，以便在页面内容更新时将其删除：

1. 将图像文件放入与页面相同的文件夹中。
1. 对图像文件使用以下命名格式：

   `<page file name>.<image file name>`

例如，您可以将页面 myPage.html 的标题存储在文件 myPage.title.gif 中。由于此文件会在页面更新时自动被删除，因此对页面标题进行的任何更改都会自动反映在缓存中。

>[!NOTE]
>
>图像文件不一定会实际存在于 AEM 实例上。您可以使用动态创建图像文件的脚本。随后，Dispatcher 会将文件存储在 Web 服务器上。

## 使用于导航的图像文件失效 {#invalidating-image-files-used-for-navigation}

如果将图片用于导航条目，则方法基本上与标题相同，只是稍微复杂一点。 将所有导航图像与目标页面一起存储。如果将两张图片用于一般或活动场景，则可以使用以下脚本：

* 一个正常显示页面的脚本。
* 一个处理“.normal”请求并返回正常图片的脚本。
* 一个处理“.active”请求并返回活动图片的脚本。

请务必使用与页面相同的命名句柄创建这些图片，以确保内容更新会删除这些图片和页面。

对于未修改的页面，图片将保留在缓存中，尽管页面本身会自动失效。

## 个性化 {#personalization}

Dispatcher 无法缓存个性化数据，因此建议您仅允许在必要时进行个性化设置。原因如下：

* 如果您使用可随意自定义的开始页面，则用户每次请求该页面时都必须对它进行编辑。
* 相反，如果您提供了十个不同的开始页面以供选择，则可以缓存其中的每个页面，从而提高性能。

>[!NOTE]
>
>如果您对每个页面进行个性化设置（例如，将用户名放入标题栏中），则无法对它们进行缓存，从而会对性能造成重大影响。
>
>不过，如果您必须执行此操作，则可以：
>
>* 使用 iFrame 将页面分成两个部分，一个部分是所有用户共用的，另一个部分是用户的所有页面共用的。随后，您可以缓存这两个部分。
>* 使用客户端 JavaScript 显示个性化的信息。但您必须确保页面在用户禁用 JavaScript 后仍正常显示。
>

## 粘性连接 {#sticky-connections}

[粘性连接](dispatcher.md#TheBenefitsofLoadBalancing)可确保同一个用户的文档全部在同一服务器上撰写。如果用户在退出此文件夹不久后返回，则此连接仍保持粘性。定义一个文件夹，使其可容纳所有需要网站的粘性连接的文档。 尽量不要在该文件夹中放入其他文件。如果您使用个性化的页面和会话数据，这将影响负载平衡。

## MIME 类型 {#mime-types}

浏览器可通过两种方式确定文件的类型：

1. 按扩展名（例如.html、.gif和.jpg）
1. 按服务器随文件一起发送的 MIME 类型。

对于大多数文件，MIME 类型隐含在文件扩展名中。即：

1. 按扩展名（例如.html、.gif和.jpg）
1. 按服务器随文件一起发送的 MIME 类型。

如果文件名没有扩展名，则以纯文本形式显示。

MIME 类型是 HTTP 标头的一部分，因此 Dispatcher 不会缓存它。如果 AEM 应用程序返回的文件没有可识别的文件结尾，而是依赖 MIME 类型，则这些文件可能会错误显示。

要确保正确缓存文件，请遵循以下准则：

* 确保文件始终具有正确的扩展名。
* 避免使用通用的文件服务脚本，这些脚本具有 download.jsp?file=2214 等 URL。重写脚本，使其使用包含文件规范的URL。 对于上一个示例，这将是 `download.2214.pdf`.

