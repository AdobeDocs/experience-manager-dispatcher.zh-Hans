---
title: 在多个域中使用 Dispatcher
description: 了解如何使用 Dispatcher 处理多个 Web 域中的页面请求。
contentOwner: User
cq-exporttemplate: /etc/contentsync/templates/geometrixx/page/rewrite
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
exl-id: 1470b636-7e60-48cc-8c31-899f8785dafa
source-git-commit: c41b4026a64f9c90318e12de5397eb4c116056d9
workflow-type: ht
source-wordcount: '2986'
ht-degree: 100%

---

# 在多个域中使用 Dispatcher {#using-dispatcher-with-multiple-domains}

<!-- This article is missing required metadata image ALT tags throughout -->

>[!NOTE]
>
>Dispatcher 版本独立于 AEM。您可能是在单击 AEM 或 CQ 文档中嵌入的 Dispatcher 文档链接后重定向到此页面。

使用 Dispatcher 可处理多个 Web 域中的页面请求，并支持以下条件：

* 两个域的 Web 内容都存储在一个 AEM 存储库中。
* 可单独为每个域使 Dispatcher 缓存中的文件失效。

例如，一家公司针对其两个品牌发布网站：品牌 A 和品牌 B。网站页面的内容是在 AEM 中创作的，并且将存储在同一存储库工作区中：

```
/
| - content  
   | - sitea  
       | - content nodes  
   | - siteb  
       | - content nodes
```

`BrandA.com` 的页面存储在 `/content/sitea` 下方。针对 URL `https://BrandA.com/en.html` 的客户端请求将返回到 `/content/sitea/en` 节点的渲染页面。同样，`BrandB.com` 的页面存储在 `/content/siteb` 下方。

在使用 Dispatcher 缓存内容时，在客户端 HTTP 请求中的页面 URL 与缓存中对应文件的路径/存储库中对应文件的路径之间建立关联。

## 客户端请求

当客户端向 Web 服务器发送 HTTP 请求时，请求页面的 URL 必须解析为 Dispatcher 缓存中的内容，并最终解析为存储库中的内容。

![](assets/chlimage_1-8.png)

1. 域名系统会发现为 HTTP 请求中的域名注册的 Web 服务器的 IP 地址。
1. HTTP 请求将发送到 Web 服务器。
1. HTTP 请求将传递到 Dispatcher。
1. Dispatcher 确定缓存的文件是否有效。如果有效，则将缓存的文件提供给客户端。
1. 如果缓存的文件无效，Dispatcher 会从 AEM 发布实例请求新渲染的页面。

## 使缓存失效

当 `Dispatcher Flush` 复制代理请求 Dispatcher 使缓存文件失效时，存储库中内容的路径必须能够解析到缓存中的对应内容。

![](assets/chlimage_1-9.png)

* a - 在 AEM 作者实例上激活一个页面后，内容会被复制到发布实例。
* b - `Dispatcher Flush` 代理调用 Dispatcher，使被复制内容的缓存失效。
* c - Dispatcher 会更新一个或多个 .stat 文件，使缓存文件失效。

要在多个域中使用 Dispatcher，必须配置 AEM、Dispatcher 和 Web 服务器。此页面上描述的解决方案是通用的，适用于大多数环境。由于一些 AEM 拓扑的复杂性，您的解决方案可能需要进一步的自定义配置才能解决特定问题。您可能必须调整示例以符合现有的 IT 基础架构和管理策略。

## URL 映射 {#url-mapping}

要使域 URL 和内容路径能够解析为缓存的文件，必须在此过程中转换文件路径或页面 URL。提供了以下常见策略的说明，其中将在此流程的不同时间点转换路径或 URL：

* （推荐）AEM 发布实例使用 Sling 映射进行资源解析，以实施内部 URL 重写规则。域 URL 将转换为内容存储库路径。请参阅 [AEM 重写传入 URL](#aem-rewrites-incoming-urls)。
* Web 服务器使用内部 URL 重写规则将域 URL 转换为缓存路径。请参阅 [Web 服务器重写传入 URL](#the-web-server-rewrites-incoming-urls)。

最好是使用网页的短 URL。通常，页面 URL 反映了包含 Web 内容的存储库文件夹的结构。但是，URL 不会显示最顶层的存储库节点，例如 `/content`。客户端不一定了解 AEM 存储库的结构。

## 一般要求 {#general-requirements}

您的环境必须实施以下配置来支持在多个域中使用 Dispatcher：

* 每个域的内容均驻留在单独的存储库分支中（请参阅下面的示例环境）。
* Dispatcher Flush 复制代理是在 AEM 发布实例上配置的。（请参阅[使发布实例中的 Dispatcher 缓存失效](page-invalidate.md)。）
* 域名系统将域名解析为 Web 服务器的 IP 地址。
* Dispatcher 缓存反映了 AEM 内容存储库的目录结构。Web 服务器的文档根目录下的文件路径与存储库中文件的路径相同。

## 所提供示例的环境 {#environment-for-the-provided-examples}

提供的示例解决方案适用于具有以下特征的环境：

* 在 Linux® 系统上部署 AEM 创作和发布实例。
* Apache HTTPD 是 Linux® 系统上部署的 Web 服务器。
* AEM 内容存储库和 Web 服务器的文档根目录使用以下文件结构（Apache Web Server 的文档根目录为 /`usr/lib/apache/httpd-2.4.3/htdocs)`：

  **存储库**

```
  | - /content  
    | - sitea  
  |    | - content nodes
    | - siteb  
       | - content nodes
```

**Web 服务器的文档根目录**

```
  | - /usr  
    | - lib  
      | - apache  
        | - httpd-2.4.3  
          | - htdocs  
            | - content  
              | - sitea  
                 | - content nodes 
              | - siteb  
                 | - content nodes
```

## AEM 重写传入 URL {#aem-rewrites-incoming-urls}

用于资源解析的 Sling 映射可让您将传入 URL 与 AEM 内容路径相关联。在 AEM 发布实例上创建映射，以使来自 Dispatcher 的渲染请求解析为存储库中的正确内容。

针对页面渲染的 Dispatcher 请求使用从 Web 服务器传递的 URL 标识页面。当 URL 包含域名时，Sling 映射会将 URL 解析为内容。下图说明了 `branda.com/en.html` URL 与 `/content/sitea/en` 节点之间的映射。

![](assets/chlimage_1-10.png)

Dispatcher 缓存反映了存储库节点结构。因此，在页面激活时，产生的使缓存页面失效的请求不需要 URL 或路径转换。

![](assets/chlimage_1-11.png)

## 在 Web 服务器上定义虚拟主机 {#define-virtual-hosts-on-the-web-server}

在 Web 服务器上定义虚拟主机，以便向每个 Web 域分配不同的文档根目录：

* Web 服务器必须为每个 Web 域定义一个虚拟域。
* 对于每个域，将文档根目录配置为与存储库中包含域的 Web 内容的文件夹一致。
* 每个虚拟域还必须包含与 Dispatcher 相关的配置，如[安装 Dispatcher](dispatcher-install.md) 页面上所述。

以下示例 `httpd.conf` 文件为 Apache Web Server 配置两个虚拟域：

* 服务器名称（与域名一致）是 branda.com（第 16 行）和 brandb.com（第 30 行）。
* 每个虚拟域的文档根目录是 Dispatcher 缓存中包含站点页面的目录。（第 17 行和第 31 行）

对于此配置，Web 服务器会在收到 `https://branda.com/en/products.html` 的请求时执行以下操作：

* 将 URL 与 `ServerName` 为 `branda.com.` 的虚拟主机关联。

* 将 URL 转发给 Dispatcher。

### httpd.conf {#httpd-conf}

```xml
# load the Dispatcher module
LoadModule dispatcher_module modules/mod_dispatcher.so
# configure the Dispatcher module
<IfModule disp_apache2.c>
 DispatcherConfig conf/dispatcher.any
 DispatcherLog    logs/dispatcher.log  
 DispatcherLogLevel 3
 DispatcherNoServerHeader 0 
 DispatcherDeclineRoot 0
 DispatcherUseProcessedURL 0
 DispatcherPassError 0
</IfModule>

# Define virtual host for brandA.com
<VirtualHost *:80>
  ServerName branda.com
  DocumentRoot /usr/lib/apache/httpd-2.4.3/htdocs/content/sitea
   <Directory /usr/lib/apache/httpd-2.4.3/htdocs/content/sitea>
     <IfModule disp_apache2.c>
       SetHandler dispatcher-handler
       ModMimeUsePathInfo On
     </IfModule>
     Options FollowSymLinks
     AllowOverride None
   </Directory>
</VirtualHost>

# define virtual host for brandB.com
<VirtualHost *:80>
  ServerName brandB.com
  DocumentRoot /usr/lib/apache/httpd-2.4.3/htdocs/content/siteb
   <Directory /usr/lib/apache/httpd-2.4.3/htdocs/content/siteb>
     <IfModule disp_apache2.c>
       SetHandler dispatcher-handler
       ModMimeUsePathInfo On
     </IfModule>
     Options FollowSymLinks
     AllowOverride None
   </Directory>
</VirtualHost>

# document root for web server
DocumentRoot "/usr/lib/apache/httpd-2.4.3/htdocs"
```

虚拟主机将继承主服务器部分中配置的 [DispatcherConfig](dispatcher-install.md#main-pars-67-table-7) 属性值。虚拟主机可以包括自己的 DispatcherConfig 属性来覆盖主服务器配置。

>[!NOTE]
>
>在 AEM as a Cloud Service 中，必须使用单独的虚拟主机配置，并且 DocumentRoot 必须比每一个子页面都高一级。在原型中已默认实施了该过程。但如果使用多个 DocumentRoot，就需要采用优先级更高的虚拟主机配置用于完成使整个缓存失效的操作，因为无法为每个站点单独配置缓存失效机制。此新配置中的 ServerAlias 必须接受主机标头“localhost”。

### 配置 Dispatcher 以处理多个域 {#configure-dispatcher-to-handle-multiple-domains}

要支持包含域名及其相应的虚拟主机的 URL，请定义以下 Dispatcher 场：

* 为每个虚拟主机配置 Dispatcher 场。这些场处理来自每个域的 Web 服务器的请求，检查缓存的文件，并从渲染器请求页面。
* 配置用于使缓存的内容失效的 Dispatcher 场，而不管内容属于哪个域。该场处理来自 Flush Dispatcher 复制代理的文件失效请求。

### 为虚拟主机创建 Dispatcher 场

虚拟主机的场必须具有以下配置，以使客户端 HTTP 请求中的 URL 解析为 Dispatcher 缓存中的正确文件：

* `/virtualhosts` 属性设置为域名。利用此属性，Dispatcher 能够将场与域相关联。
* 利用 `/filter` 属性可访问域名部分后截断的请求 URL 的路径。例如，对于 `https://branda.com/en.html` URL，路径将解释为 `/en.html`，因此过滤器必须允许对此路径的访问。

* `/docroot` 属性设置为根目录的路径。即 Dispatcher 缓存中域网站内容的根目录。此路径用作原始请求中的连接的 URL 的前缀。例如，docroot 为 `/usr/lib/apache/httpd-2.4.3/htdocs/sitea` 会导致针对 `https://branda.com/en.html` 的请求解析为 `/usr/lib/apache/httpd-2.4.3/htdocs/sitea/en.html` 文件。

此外，必须将 AEM 发布实例指定为虚拟主机的渲染器。根据需要配置其他场属性。以下代码是 branda.com 域的缩写场配置：

```xml
/farm_sitea  {     
    ...
    /virtualhosts { "branda.com" }
    /renders {
      /rend01  { /hostname "127.0.0.1"  /port "4503" }
    }
    /filter {
      /0001 { /type "deny"  /glob "*" }
      /0023 { /type "allow" /glob "*/en*" }  
      ...
     }
    /cache {
      /docroot "/usr/lib/apache/httpd-2.4.3/htdocs/content/sitea"
      ...
   }
   ...
}
```

### 创建用于缓存失效的 Dispatcher 场

需要 Dispatcher 场来处理使缓存文件失效的请求。该场必须能够访问每个虚拟主机的 `docroot` 目录中的 .stat 文件。

利用以下属性配置，Dispatcher 能够从缓存中的文件解析 AEM 内容存储库中的文件：

* `/docroot` 属性设置为 Web 服务器的默认 `docroot`。通常，/`docroot` 是在其中创建 `/content` 文件夹的目录。Linux® 上的 Apache 示例值为 `/usr/lib/apache/httpd-2.4.3/htdocs`。
* `/filter` 属性允许对 `/content` 目录下的文件的访问。

`/statfileslevel` 属性必须足够高，才会在每个虚拟主机的根目录中创建 .stat 文件。利用此属性，可单独使每个域的缓存失效。对于示例设置，`/statfileslevel` 值为 `2` 将在 `*docroot*/content/sitea` 目录和 `*docroot*/content/siteb` 目录中创建 .stat 文件。

此外，必须将 发布实例指定为虚拟主机的渲染器。根据需要配置其他场属性。以下代码是用于使缓存失效的场的缩写配置：

```xml
/farm_flush {  
    ...
    /virtualhosts   { "invalidation_only" }
    /renders  {
      /rend01  { /hostname "127.0.0.1" /port "4503" }
    }
    /filter   {
      /0001 { /type "deny"  /glob "*" }
      /0023 { /type "allow" /glob "*/content*" } 
      ...
      }
    /cache  {
       /docroot "/usr/lib/apache/httpd-2.4.3/htdocs"
       /statfileslevel "2"
       ...
   }
   ...
}
```

在启动 Web 服务器时，Dispatcher 日志（在调试模式下）指示所有场的初始化：

```shell
Dispatcher initializing (build 4.1.2)
[Fri Nov 02 16:27:18 2012] [D] [24974(140006182991616)] farms[farm_sitea].cache.docroot = /usr/lib/apache/httpd-2.4.3/htdocs/content/sitea
[Fri Nov 02 16:27:18 2012] [D] [24974(140006182991616)] farms[farm_siteb].cache.docroot = /usr/lib/apache/httpd-2.4.3/htdocs/content/siteb
[Fri Nov 02 16:27:18 2012] [D] [24974(140006182991616)] farms[farm_flush].cache.docroot = /usr/lib/apache/httpd-2.4.3/htdocs
[Fri Nov 02 16:27:18 2012] [I] [24974(140006182991616)] Dispatcher initialized (build 4.1.2)
```

### 配置 Sling 映射以实现资源解析 {#configure-sling-mapping-for-resource-resolution}

使用 `Sling` 映射进行资源解析，使基于域的 URL 能够解析至 AEM 发布实例上的内容。资源映射会将来自 Dispatcher（最初源自客户端 HTTP 请求）的传入 URL 转换为对应的内容节点。

要了解 `Sling` 资源映射，请参阅 `Sling` 文档中的[用于资源解析的映射](https://sling.apache.org/documentation/the-sling-engine/mappings-for-resource-resolution.html)。

通常，以下资源需要映射，但也可能需要其他的映射：

* 内容页面的根节点（`/content` 的下方）
* 页面使用的设计节点（`/etc/designs` 的下方）
* `/libs` 文件夹

创建内容页面的映射后，要发现更多所需的映射，请使用 Web 浏览器打开 Web 服务器上的页面。在发布实例的 error.log 文件中，找到有关未找到的资源的消息。以下示例消息指示需要 `/etc/clientlibs` 的映射：

```shell
01.11.2012 15:59:24.601 *INFO* [10.36.34.243 [1351799964599] GET /etc/clientlibs/foundation/jquery.js HTTP/1.1] org.apache.sling.engine.impl.SlingRequestProcessorImpl service: Resource /content/sitea/etc/clientlibs/foundation/jquery.js not found
```

>[!NOTE]
>
>默认的 Apache `Sling` 重写器中的 Linkchecker 转换器会自动更改页面中的超链接，以防止链接失效。但是，仅当链接目标是 HTML 或 HTM 文件时才执行链接重写。要更新指向其他文件类型的链接，请创建转换器组件并将它添加到 HTML 重写器管道中。

### 示例资源映射节点

下表列出了实施 branda.com 域的资源映射的节点。为 `brandb.com` 域创建类似的节点，例如 `/etc/map/http/brandb.com`。在所有情况下，只要无法在 Sling 的上下文中正确解析 HTML 页面中的引用，就需要映射。

| 节点路径 | 类型 | 属性 |
|--- |--- |--- |
| `/etc/map/http/branda.com` | `sling:Mapping` | `Name: sling:internalRedirect Type: String Value: /content/sitea` |
| `/etc/map/http/branda.com/libs` | `sling:Mapping` | `Name: sling:internalRedirect <br/>Type: String <br/>Value: /libs` |
| `/etc/map/http/branda.com/etc` | `sling:Mapping` | |
| `/etc/map/http/branda.com/etc/designs` | `sling:Mapping` | `Name: sling:internalRedirect <br/>VType: String <br/>VValue: /etc/designs` |
| `/etc/map/http/branda.com/etc/clientlibs` | `sling:Mapping` | `Name: sling:internalRedirect <br/>VType: String <br/>VValue: /etc/clientlibs` |

## 配置 Dispatcher Flush 复制代理 {#configuring-the-dispatcher-flush-replication-agent}

AEM 发布实例上的 `Dispatcher Flush` 复制代理必须将失效请求发送到正确的 Dispatcher 场。要将场作为目标，请使用 `Dispatcher Flush` 复制代理的 URI 属性（在“传输”选项卡上）。包含为使缓存失效而配置的 Dispatcher 场的 `/virtualhost` 属性值：

`https://*webserver_name*:*port*/*virtual_host*/dispatcher/invalidate.cache`

例如，要使用上一示例的 `farm_flush` 场，URI 应为 `https://localhost:80/invalidation_only/dispatcher/invalidate.cache`。

![](assets/chlimage_1-12.png)

## Web 服务器重写传入 URL {#the-web-server-rewrites-incoming-urls}

使用 Web 服务器的内部 URL 重写功能将基于域的 URL 转换为 Dispatcher 缓存中的文件路径。例如，`https://brandA.com/en.html` 页面的客户端请求将转换为 Web 服务器的文档根目录中的 `content/sitea/en.html` 文件。

![](assets/chlimage_1-13.png)

Dispatcher 缓存反映了存储库节点结构。因此，在页面激活时，产生的使缓存页面失效的请求不需要 URL 或路径转换。

![](assets/chlimage_1-14.png)

## 在 Web 服务器上定义虚拟主机并重写规则 {#define-virtual-hosts-and-rewrite-rules-on-the-web-server}

在 Web 服务器上进行如下配置：

* 为每个 Web 域定义一个虚拟主机。
* 对于每个域，将文档根目录配置为与存储库中包含域的 Web 内容的文件夹一致。
* 对于每个虚拟域，创建一个 URL 重命名规则来将传入 URL 转换为缓存文件的路径。
* 每个虚拟域还必须包含与 Dispatcher 相关的配置，如[安装 Dispatcher](dispatcher-install.md) 页面上所述。
* 必须将 Dispatcher 模块配置为使用 Web 服务器已重写的 URL。（请参阅 [安装 Dispatcher](dispatcher-install.md) 中的 `DispatcherUseProcessedURL` 属性。）

以下示例 httpd.conf 文件为 Apache Web Server 配置两个虚拟主机：

* 服务器名称（与域名一致）是 `brandA.com`（第 16 行）和 `brandB.com`（第 32 行）。

* 每个虚拟域的文档根目录是 Dispatcher 缓存中包含站点页面的目录。（第 20 行和第 33 行）
* 每个虚拟域的 URL 重写规则都是正则表达式。正则表达式为请求的页面的路径添加前缀。其前缀是缓存中页面的路径。（第 19 行和第 35 行）
* `DispatcherUseProcessedURL` 属性设置为 `1`。（第 10 行）

例如，Web 服务器会在收到带 `https://brandA.com/en/products.html` URL 的请求时执行以下操作：

* 将 URL 与 `ServerName` 为 `brandA.com.` 的虚拟主机关联。
* 将 URL 重写为 `/content/sitea/en/products.html.`
* 将 URL 转发给 Dispatcher。

### httpd.conf {#httpd-conf-1}

```xml
# load the Dispatcher module
LoadModule dispatcher_module modules/mod_dispatcher.so
# configure the Dispatcher module
<IfModule disp_apache2.c>
 DispatcherConfig conf/dispatcher.any
 DispatcherLog    logs/dispatcher.log  
 DispatcherLogLevel 3
 DispatcherNoServerHeader 0 
 DispatcherDeclineRoot 0
 DispatcherUseProcessedURL 1
 DispatcherPassError 0
</IfModule>

# Define virtual host for brandA.com
<VirtualHost *:80>
  ServerName branda.com
  DocumentRoot /usr/lib/apache/httpd-2.4.3/htdocs/content/sitea
  RewriteEngine  on
  RewriteRule    ^/(.*)\.html$  /content/sitea/$1.html [PT]
   <Directory /usr/lib/apache/httpd-2.4.3/htdocs/content/sitea>
     <IfModule disp_apache2.c>
       SetHandler dispatcher-handler
       ModMimeUsePathInfo On
     </IfModule>
     Options FollowSymLinks
     AllowOverride None
   </Directory>
</VirtualHost>

# define virtual host for brandB.com
<VirtualHost *:80>
  ServerName brandB.com
  DocumentRoot /usr/lib/apache/httpd-2.4.3/htdocs/content/siteb
  RewriteEngine  on
  RewriteRule    ^/(.*)\.html$  /content/siteb/$1.html [PT]
   <Directory /usr/lib/apache/httpd-2.4.3/htdocs/content/siteb>
     <IfModule disp_apache2.c>
       SetHandler dispatcher-handler
       ModMimeUsePathInfo On
     </IfModule>
     Options FollowSymLinks
     AllowOverride None
   </Directory>
</VirtualHost>

# document root for web server
DocumentRoot "/usr/lib/apache/httpd-2.4.3/htdocs"
```

### 配置 Dispatcher 场 {#configure-a-dispatcher-farm}

在 Web 服务器重写 URL 时，Dispatcher 需要根据[配置 Dispatcher](dispatcher-configuration.md) 定义的单个场。需要以下配置才能支持 Web 服务器虚拟主机和 URL 重命名规则：

* `/virtualhosts` 属性必须包含所有 VirtualHost 定义的 ServerName 值。
* `/statfileslevel` 属性必须足够高，才会在包含每个域的内容文件的目录中创建 .stat 文件。

以下示例配置文件基于随 Dispatcher 一起安装的示例 `dispatcher.any` 文件。需要进行以下更改才能支持上一个 `httpd.conf` 文件的 Web 服务器配置：

* `/virtualhosts` 属性促使 Dispatcher 处理 `brandA.com` 和 `brandB.com` 域的请求。（第 12 行）
* `/statfileslevel` 属性设置为 2，以便在每个包含域的 Web 内容的目录中创建 stat 文件（第 41 行）：`/statfileslevel "2"`

像往常一样，缓存的文档根目录与 Web 服务器的文档根目录相同（第 40 行）：`/usr/lib/apache/httpd-2.4.3/htdocs`

### `dispatcher.any` {#dispatcher-any}

```xml
/name "testDispatcher"
/farms
  {
  /dispfarm0
    {  
    /clientheaders
      {
      "*"
      }      
    /virtualhosts
      {
      "brandA.com" "brandB.com"
      }
    /renders
      {
      /rend01    {  /hostname "127.0.0.1"   /port "4503"  }
      }
    /filter
      {
      /0001 { /type "deny"  /glob "*" }
      /0023 { /type "allow" /glob "*/content*" }  # disable this rule to allow mapped content only
      /0041 { /type "allow" /glob "* *.css *"   }  # enable css
      /0042 { /type "allow" /glob "* *.gif *"   }  # enable gifs
      /0043 { /type "allow" /glob "* *.ico *"   }  # enable icos
      /0044 { /type "allow" /glob "* *.js *"    }  # enable javascript
      /0045 { /type "allow" /glob "* *.png *"   }  # enable png
      /0046 { /type "allow" /glob "* *.swf *"   }  # enable flash
      /0061 { /type "allow" /glob "POST /content/[.]*.form.html" }  # allow POSTs to form selectors under content
      /0062 { /type "allow" /glob "* /libs/cq/personalization/*"  }  # enable personalization
      /0081 { /type "deny"  /glob "GET *.infinity.json*" }
      /0082 { /type "deny"  /glob "GET *.tidy.json*"     }
      /0083 { /type "deny"  /glob "GET *.sysview.xml*"   }
      /0084 { /type "deny"  /glob "GET *.docview.json*"  }
      /0085 { /type "deny"  /glob "GET *.docview.xml*"  }      
      /0086 { /type "deny"  /glob "GET *.*[0-9].json*" }
      /0090 { /type "deny"  /glob "* *.query.json*" }
      }
    /cache
      {
      /docroot "/usr/lib/apache/httpd-2.4.3/htdocs"
      /statfileslevel "2"
      /allowAuthorized "0"
      /rules
        {
        /0000  { /glob "*"     /type "allow"  }
        }
      /invalidate
        {
        /0000  {   /glob "*" /type "deny"  }
        /0001 {  /glob "*.html" /type "allow"  }
        }
      /allowedClients
        {
        }     
      }
    /statistics
      {
      /categories
        {
        /html  { /glob "*.html" }
        /others  {  /glob "*"  }
        }
      }
    }
  }
```

>[!NOTE]
>
>由于定义了单个 Dispatcher 场，因此 AEM 发布实例上的 `Dispatcher Flush` 复制代理不需要专门配置。

## 重写指向非 HTML 文件的链接 {#rewriting-links-to-non-html-files}

要重写对具有 .html 或 .htm 以外的扩展名的文件的引用，请创建 Sling 重写器转换器组件并将它添加到默认重写器管道中。

当资源路径无法在 Web 服务器上下文中正确解析时重写引用。例如，当图像生成组件创建链接（例如 /content/sitea/en/products.navimage.png）时，需要使用转换器。[如何创建功能完善的 Internet 网站](https://experienceleague.adobe.com/zh-hans/docs/experience-manager-65/content/implementing/developing/introduction/the-basics)的 `topnav` 组件将创建此类链接。

[`Sling`重写器](https://sling.apache.org/documentation/bundles/output-rewriting-pipelines-org-apache-sling-rewriter.html)模块用于对 `Sling` 输出进行后处理。重写器的 SAX 管道实施由一个生成器、一个或多个转换器以及一个序列化器组成：

* **生成器**：解析 `Sling` 输出流（HTML 文档）并在遇到特定元素类型时生成 SAX 事件。
* **转换器：**&#x200B;侦听 SAX 事件，从而修改事件目标（一个 HTML 元素）。重写器管道包含零个或多个转换器。转换器按顺序执行，并将 SAX 事件传递到序列中的下一个转换器。
* **序列化器：**&#x200B;序列化输出，并包含每个转换器中的修改。

![](assets/chlimage_1-15.png)

### AEM 默认重写器管道 {#the-aem-default-rewriter-pipeline}

AEM 使用默认管道重写器来处理 text/html 类型的文档：

* 生成器解析 HTML 文档并在遇到 a、img、area、form、base、link、script 和 body 元素时生成 SAX 事件。生成器的别名为 `htmlparser`。
* 管道包含以下转换器：`linkchecker`、`mobile`、`mobiledebug`、`contentsync`。`linkchecker` 转换器将引用的 HTML 或 HTM 文件的路径外部化，以防止链接失效。
* 序列化器写入 HTML 输出。序列化器别名为 htmlwriter。

`/libs/cq/config/rewriter/default` 节点定义管道。

### 创建转换器 {#creating-a-transformer}

执行以下任务可创建转换器组件并将它用于管道：

1. 实施 `org.apache.sling.rewriter.TransformerFactory` 接口。此类创建转换器类的实例。指定 `transformer.type` 属性的值（转换器别名），并将类配置为 OSGi 服务组件。
1. 实施 `org.apache.sling.rewriter.Transformer` 接口。要最大限度地减少工作量，可以扩展 `org.apache.cocoon.xml.sax.AbstractSAXPipe` 类。重写 startElement 方法可自定义重写行为。为传递给转换器的每个 SAX 事件调用此方法。
1. 捆绑和部署这些类。
1. 将配置节点添加到 AEM 应用程序可将转换器添加到管道。

>[!TIP]
>您可以改为将 TransformerFactory 配置为将转换器插入定义的每个重写器中。因此，无需配置管道：
>
>* 将 `pipeline.mode` 属性设置为 `global`。
>* 将 `service.ranking` 属性设置为正整数。
>* 请勿包含 `pipeline.type` 属性。

>[!NOTE]
>
>使用[多模块](https://experienceleague.adobe.com/zh-hans/docs/experience-manager-release-information/aem-release-updates/previous-updates/aem-previous-versions)原型的 Content Package Maven Plugin 可创建您的 Maven 项目。POM 自动创建并安装内容包。

以下示例实施了重写对图像文件的引用的转换器。

* MyRewriterTransformerFactory 类实例化 MyRewriterTransformer 对象。管道类型属性将转换器别名设置为 `mytransformer`。要在管道中包含别名，管道配置节点应将此别名包含在转换器列表中。
* MyRewriterTransformer 类覆盖 AbstractSAXTransformer 类的 startElement 方法。startElement 方法重写 img 元素的 src 属性的值。

这些例子不够有力。请勿在生产环境中使用它们。

### TransformerFactory 实施示例 {#example-transformerfactory-implementation}

```java
package com.adobe.example;

import org.apache.felix.scr.annotations.Component;
import org.apache.felix.scr.annotations.Service;
import org.apache.felix.scr.annotations.Property;

import org.apache.sling.rewriter.Transformer;
import org.apache.sling.rewriter.TransformerFactory;

@Component
@Service
public class MyRewriterTransformerFactory implements TransformerFactory {
    /* Define the alias */
    @Property(value="mytransformer")
    static final String PIPELINE_TYPE ="pipeline.type";
 
    public Transformer createTransformer() {
        
        return new MyRewriterTransformer ();
    }
}
```

### 转换器实施示例 {#example-transformer-implementation}

```java
package com.adobe.example;

import java.io.IOException;

import org.apache.cocoon.xml.sax.AbstractSAXPipe;

import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.rewriter.ProcessingComponentConfiguration;
import org.apache.sling.rewriter.ProcessingContext;
import org.apache.sling.rewriter.Transformer;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import org.xml.sax.Attributes;
import org.xml.sax.SAXException;
import org.xml.sax.helpers.AttributesImpl;

import javax.servlet.http.HttpServletRequest;

public class MyRewriterTransformer extends AbstractSAXPipe implements Transformer {

 private static final Logger log = LoggerFactory.getLogger(MyRewriterTransformer.class);
 private SlingHttpServletRequest httpRequest; 
 /* The element and attribute to act on  */
 private static final String ATT_NAME = new String("src");
 private static final String EL_NAME = new String("img");

 public MyRewriterTransformer () {
 }
 public void dispose() {
 }
 public void init(ProcessingContext context, ProcessingComponentConfiguration config) throws IOException {
  this.httpRequest = context.getRequest();
  log.debug("Transforming request {}.", httpRequest.getRequestURI());
 }
 @Override
 public void startElement (String nsUri, String localname, String qname, Attributes atts) throws SAXException {
  /* copy the element attributes */
  AttributesImpl linkAtts = new AttributesImpl(atts); 
  /* Only interested in EL_NAME elements */
  if(EL_NAME.equalsIgnoreCase(localname)){

   /* iterate through the attributes of the element and act only on ATT_NAME attributes */
   for (int i=0; i < linkAtts.getLength(); i++) {
    if (ATT_NAME.equalsIgnoreCase(linkAtts.getLocalName(i))) {
     String path_in_link = linkAtts.getValue(i);

     /* use the resource resolver of the http request to reverse-resolve the path  */
     String mappedPath = httpRequest.getResourceResolver().map(httpRequest, path_in_link);

     log.info("Tranformed {} to {}.", path_in_link,mappedPath);

     /* update the attribute value */
     linkAtts.setValue(i,mappedPath);
    }
   }

  }
        /* return updated attributes to super and continue with the transformer chain */
 super.startElement(nsUri, localname, qname, linkAtts);
 }
}
```

### 将转换器添加到重写器管道 {#adding-the-transformer-to-a-rewriter-pipeline}

创建一个 JCR 节点，该节点定义使用您的转换器的管道。以下节点定义创建一个处理 text/html 文件的管道。使用 HTML 的默认 AEM 生成器和解析器。

>[!NOTE]
>
>如果您将 Transformer 属性 `pipeline.mode` 设置为 `global`，则无需配置管道。`global` 模式将转换器插入所有管道中。

### 重写器配置节点 - XML 表示形式 {#rewriter-configuration-node-xml-representation}

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:jcr="https://www.jcp.org/jcr/1.0" xmlns:nt="https://www.jcp.org/jcr/nt/1.0"
    jcr:primaryType="nt:unstructured"
    contentTypes="[text/html]"
    enabled="{Boolean}true"
    generatorType="htmlparser"
    order="5"
    serializerType="htmlwriter"
    transformerTypes="[mytransformer]">
</jcr:root>
```

下图显示了节点的 CRXDE Lite 表示形式：

![](assets/chlimage_1-16.png)
