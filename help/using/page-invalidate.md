---
title: 使从 AEM 中缓存的页面失效
description: 了解如何配置 Dispatcher 和 AEM 之间的交互以确保高效的缓存管理。
cmgrlastmodified: 01.11.2007 08 22 29 [aheimoz]
pageversionid: 1193211344162
template: /apps/docs/templates/contentpage
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
exl-id: 90eb6a78-e867-456d-b1cf-f62f49c91851
source-git-commit: 9be9f5935c21ebbf211b5da52280a31772993c2e
workflow-type: tm+mt
source-wordcount: '1407'
ht-degree: 100%

---

# 使从 AEM 中缓存的页面失效 {#invalidating-cached-pages-from-aem}

在将 Dispatcher 与 AEM 结合使用时，必须配置两者之间的交互以确保高效的缓存管理。根据您的环境，配置还可以提高性能。

## 设置 AEM 用户帐户 {#setting-up-aem-user-accounts}

默认 `admin` 用户帐户用于对默认安装的复制代理进行身份验证。创建一个用于复制代理的专用用户帐户。

有关更多信息，请参阅 AEM 安全检查清单的[配置复制和传输用户](https://experienceleague.adobe.com/zh-hans/docs/experience-manager-release-information/aem-release-updates/previous-updates/aem-previous-versions#VerificationSteps)部分。

<!-- OLD URL from above https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html#VerificationSteps -->

## 使创作环境中的 Dispatcher 缓存失效 {#invalidating-dispatcher-cache-from-the-authoring-environment}

发布页面时，AEM 创作实例上的复制代理会向 Dispatcher 发送缓存失效请求。Dispatcher 在发布新内容时最终刷新缓存中的文件。

<!-- 

Comment Type: draft

<p>Cache invalidation requests for a page are also generated for any aliases or vanity URLs that are configured in the page properties. </p>

 -->

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-0436B4A35714BFF67F000101@AdobeID)
Last Modified Date: 2017-05-25T10:37:23.679-0400

<p>Hiding this information due to <a href="https://jira.corp.adobe.com/browse/CQDOC-5805">CQDOC-5805</a>.</p>

 -->

使用以下步骤在 AEM 作者实例上配置复制代理。该配置会在页面激活时使 Dispatcher 缓存无效：

1. 打开 AEM 工具控制台。(`https://localhost:4502/miscadmin#/etc`)
1. 打开创作实例上的 Tools/replication/Agents 下的所需复制代理。您可以使用默认安装的 Dispatcher Flush 代理。
1. 单击“编辑”，在“设置”选项卡中，确保选中&#x200B;**“已启用”**。

1. （可选）要启用别名或虚名路径失效请求，请选择&#x200B;**“别名更新”**&#x200B;选项。
1. 在“传输”选项卡上，输入 URI 访问 Dispatcher。

   如果您使用的是标准 Dispatcher Flush 代理，则可能需要更新主机名和端口；例如，https://&lt;*dispatcherHost*>:&lt;*portApache*>/dispatcher/invalidate.cache

   **注意：**&#x200B;对于 Dispatcher Flush 代理，仅在您使用基于路径的虚拟主机条目来区分场时使用 URI 属性。您可以使用此字段来定位要使其失效的场。例如，场 #1 的虚拟主机为 `www.mysite.com/path1/*`，场 #2 的虚拟主机为 `www.mysite.com/path2/*`。您可以使用 URL `/path1/invalidate.cache` 定位第一个场，使用 `/path2/invalidate.cache` 定位第二个场。有关更多信息，请参阅[在多个域中使用 Dispatcher](dispatcher-domains.md)。

1. 根据需要配置其他参数。
1. 单击“确定”即可激活代理。

或者，您也可以从 [AEM Touch UI](https://experienceleague.adobe.com/zh-hans/docs/experience-manager-65/content/implementing/deploying/configuring/replication#configuring-a-dispatcher-flush-agent) 访问和配置 Dispatcher Flush 代理。

有关如何启用对虚名 URL 的访问的详细信息，请参阅[启用对虚名 URL 的访问](dispatcher-configuration.md#enabling-access-to-vanity-urls-vanity-urls)。

>[!NOTE]
>
>刷新 Dispatcher 缓存的代理无需具有用户名和密码，但如果已配置用户名和密码，则将通过基本身份验证发送它们。

此方法可能存在两个问题：

* 必须可从创作实例访问 Dispatcher。如果您的网络（例如防火墙）配置为限制两者之间的此类访问，则可能不会出现这种情况。

* 发布和缓存失效同时发生。根据时机的不同，用户可能会在刚从缓存中删除一个页面之后、在发布新页面之前请求该页面。AEM 此时会返回旧页面，并且 Dispatcher 会再次缓存它。对于大型网站来说，这种情况更常见。

## 使发布实例中的 Dispatcher 缓存失效 {#invalidating-dispatcher-cache-from-a-publishing-instance}

在某些情况下，可以通过将缓存管理从创作环境转移到发布实例来提高性能。随后，发布环境（而不是 AEM 创作环境）会在收到发布的页面时向 Dispatcher 发送缓存失效请求。

此类情况包括：

<!-- 

Comment Type: draft

<p>Cache invalidation requests for a page are also generated for any aliases or vanity URLs that are configured in the page properties. </p>

 -->

* 防止 AEM Dispatcher 和发布实例之间可能发生的时机冲突（请参阅[使创作环境中的 Dispatcher 缓存失效](#invalidating-dispatcher-cache-from-the-authoring-environment)）。
* 系统包括多个驻留在高性能服务器上的发布实例，并且仅包含一个作者实例。

>[!NOTE]
>
>经验丰富的 AEM 管理员应决定使用此方法。

在发布实例上运行的复制代理控制着 Dispatcher 的刷新。不过，将在创作环境中进行配置，然后通过激活代理来传输配置：

1. 打开 AEM 工具控制台。
1. 打开发布实例上的 Tools/replication/Agents 下的所需复制代理。您可以使用默认安装的 Dispatcher Flush 代理。
1. 单击“编辑”，在“设置”选项卡中，确保选中&#x200B;**“已启用”**。
1. （可选）要启用别名或虚名路径失效请求，请选择&#x200B;**“别名更新”**&#x200B;选项。
1. 在“传输”选项卡上，通过输入所需的 URI 访问 Dispatcher。\
   如果您使用标准 Dispatcher Flush 代理，请更新主机名和端口。例如，`http://<dispatcherHost>:<portApache>/dispatcher/invalidate.cache`

   **注意：**&#x200B;对于 Dispatcher Flush 代理，仅在您使用基于路径的虚拟主机条目来区分场时使用 URI 属性。您可以使用此字段来定位要使其失效的场。例如，场 #1 的虚拟主机为 `www.mysite.com/path1/*`，场 #2 的虚拟主机为 `www.mysite.com/path2/*`。您可以使用 URL `/path1/invalidate.cache` 定位第一个场，使用 `/path2/invalidate.cache` 定位第二个场。有关更多信息，请参阅[在多个域中使用 Dispatcher](dispatcher-domains.md)。

1. 根据需要配置其他参数。
1. 登录到发布实例并验证刷新代理配置。此外，还要确保启用了它。
1. 对每个受影响的发布实例重复此操作。

配置后，在激活创作中的页面以进行发布时，此代理将启动标准复制。日志包含指示来自发布服务器的请求的消息，类似于以下示例：

1. `<publishserver> 13:29:47 127.0.0.1 POST /dispatcher/invalidate.cache 200`

## 手动使 Dispatcher 缓存失效 {#manually-invalidating-the-dispatcher-cache}

要在不激活页面的情况下使 Dispatcher 缓存失效（或进行刷新），您可以向 Dispatcher 发出 HTTP 请求。例如，您可以创建一个 AEM 应用程序，以便管理员或其他应用程序能够刷新缓存。

HTTP 请求促使 Dispatcher 从缓存中删除特定文件。（可选）Dispatcher 随后使用新副本刷新缓存。

### 删除缓存的文件 {#delete-cached-files}

发出 HTTP 请求以促使 Dispatcher 从缓存中删除文件。Dispatcher 仅在收到对页面的客户端请求时才重新缓存文件。对于不太可能同时收到对同一页面的请求的网站，可通过此方式删除缓存的文件。

HTTP 请求具有以下形式：

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
CQ-Handle: path-pattern
Content-Length: 0
```

Dispatcher 刷新（删除）名称与 `CQ-Handler` 标头值匹配的缓存的文件和文件夹。例如，`/content/geomtrixx-outdoors/en` 的 `CQ-Handle` 与以下项匹配：

* `geometrixx-outdoors` 目录中名称（任何文件扩展名）为 `en` 的所有文件

* `en` 目录下任何名为“`_jcr_content`”的目录（如果存在，则包含页面的子节点的缓存渲染）

通过接触 `.stat` 文件使 Dispatcher 缓存中的所有其他文件（或向上至特定级别，取决于 `/statfileslevel` 设置）失效。将此文件的上次修改日期与缓存的文档的上次修改日期进行比较，如果 `.stat` 文件更新，则重新获取该文档。有关详细信息，请参阅[按文件夹级别使文件失效](dispatcher-configuration.md#main-pars_title_26)。

可以通过发送额外的标头 `CQ-Action-Scope: ResourceOnly` 防止失效（即接触 .stat 文件）。该功能可用于刷新特定资源。所有这些都不会使缓存的其他部分（如 JSON 数据）失效。 该数据是动态创建的，需要定期刷新，与缓存无关。例如，表示从第三方系统获取的数据以显示新闻和股票行情等等。

### 删除和重新缓存文件 {#delete-and-recache-files}

发出 HTTP 请求，促使 Dispatcher 删除缓存的文件，并立即检索和重新缓存文件。在网站可能同时收到对同一页面的客户端请求时，删除并立即重新缓存文件。立即重新缓存可确保 Dispatcher 仅检索和缓存页面一次，而不是为每个并发客户端请求检索和缓存页面一次。

**注意：**&#x200B;应仅在发布实例上执行文件删除和重新缓存操作。在从创作实例执行时，在资源发布之前尝试重新缓存资源时会发生争用情况。

HTTP 请求具有以下形式：

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate   
`Content-Type: text/plain  
CQ-Handle: path-pattern  
Content-Length: numchars in bodypage_path0

page_path1 
...  
page_pathn
```

要立即重新缓存的页面路径将在消息正文的单独行中列出。`CQ-Handle` 的值是使要重新缓存的页面失效的页面路径。（请参阅[缓存](dispatcher-configuration.md#main-pars_146_44_0010)配置项的 `/statfileslevel` 参数。）以下示例 HTTP 请求消息指示删除并重新缓存 `/content/geometrixx-outdoors/en.html page`：

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
Content-Type: text/plain   
CQ-Handle: /content/geometrixx-outdoors/en/men.html  
Content-Length: 36

/content/geometrixx-outdoors/en.html
```

### 示例刷新 servlet {#example-flush-servlet}

以下代码实施一个向 Dispatcher 发送失效请求的 servlet。此 servlet 接收包含 `handle` 和 `page` 参数的请求消息。这些参数分别提供要重新缓存的页面的 `CQ-Handle` 标头和路径的值。此 servlet 使用这些值为 Dispatcher 构造 HTTP 请求。

在将 servlet 部署到发布实例时，以下 URL 会促使 Dispatcher 删除 /content/geometrixx-outdoors/en.html 页面，然后缓存一个新副本。

`10.36.79.223:4503/bin/flushcache/html?page=/content/geometrixx-outdoors/en.html&handle=/content/geometrixx-outdoors/en/men.html`

>[!NOTE]
>
>此示例 servlet 不安全，并且仅演示对 HTTP Post 请求消息的使用。您的解决方案应保护对 servlet 的访问。
>

```java
package com.adobe.example;

import org.apache.felix.scr.annotations.Component;
import org.apache.felix.scr.annotations.Service;
import org.apache.felix.scr.annotations.Property;

import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.api.SlingHttpServletResponse;
import org.apache.sling.api.servlets.SlingSafeMethodsServlet;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import org.apache.commons.httpclient.*;
import org.apache.commons.httpclient.methods.PostMethod;
import org.apache.commons.httpclient.methods.StringRequestEntity;

@Component(metatype=true)
@Service
public class Flushcache extends SlingSafeMethodsServlet {

 @Property(value="/bin/flushcache")
 static final String SERVLET_PATH="sling.servlet.paths";

 private Logger logger = LoggerFactory.getLogger(this.getClass());

 public void doGet(SlingHttpServletRequest request, SlingHttpServletResponse response) {
  try{ 
      //retrieve the request parameters
      String handle = request.getParameter("handle");
      String page = request.getParameter("page");

      //hard-coding connection properties is a bad practice, but is done here to simplify the example
      String server = "localhost"; 
      String uri = "/dispatcher/invalidate.cache";

      HttpClient client = new HttpClient();

      PostMethod post = new PostMethod("https://"+host+uri);
      post.setRequestHeader("CQ-Action", "Activate");
      post.setRequestHeader("CQ-Handle",handle);
   
      StringRequestEntity body = new StringRequestEntity(page,null,null);
      post.setRequestEntity(body);
      post.setRequestHeader("Content-length", String.valueOf(body.getContentLength()));
      client.executeMethod(post);
      post.releaseConnection();
      //log the results
      logger.info("result: " + post.getResponseBodyAsString());
      }
  }catch(Exception e){
      logger.error("Flushcache servlet exception: " + e.getMessage());
  }
 }
}
```
