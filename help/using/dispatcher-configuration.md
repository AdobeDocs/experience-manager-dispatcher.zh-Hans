---
title: 配置 AEM Dispatcher
description: 了解如何配置 Dispatcher。了解 IPv4 和 IPv6 支持、配置文件、环境变量以及命名实例。阅读有关定义农场、识别虚拟主机等内容。
exl-id: 91159de3-4ccb-43d3-899f-9806265ff132
source-git-commit: fbfbe76b730d4037cccb400b70619fbe24b3b1bc
workflow-type: tm+mt
source-wordcount: '8938'
ht-degree: 99%

---

# 配置 Dispatcher {#configuring-dispatcher}

>[!NOTE]
>
>Dispatcher 版本独立于 AEM (Adobe Experience Manager)。如果您点击了 Dispatcher 文档的链接，则可能重定向到此页面。该链接嵌入在 AEM 先前版本的文档中。

以下各部分描述了如何配置 Dispatcher 的各个方面。

## 支持 IPv4 和 IPv6 {#support-for-ipv-and-ipv}

AEM 和 Dispatcher 的所有元素都可以安装在 IPv4 和 IPv6 网络中。请参阅 [IPV4 和 IPV6](https://experienceleague.adobe.com/zh-hans/docs/experience-manager-65/content/implementing/deploying/introduction/technical-requirements#ipv-and-ipv)。

## Dispatcher 配置文件 {#dispatcher-configuration-files}

默认情况下，Dispatcher 配置存储在 `dispatcher.any` 文本文件中，不过您可以在安装期间更改此文件的名称和位置。

配置文件包含一系列单值或多值属性，这些属性控制 Dispatcher 的行为：

* 属性名称使用正斜杠 `/` 作为前缀。
* 多值属性使用大括号 `{ }` 括起子项。

示例配置的结构如下所示：

```xml
# name of the dispatcher
/name "internet-server"

# each farm configures a set off (loadbalanced) renders
/farms
 {
  # first farm entry (label is not important, just for your convenience)
   /website
     {  
     /clientheaders
       {
       # List of headers that are passed on
       }
     /virtualhosts
       {
       # List of URLs for this Web site
       }
     /sessionmanagement
       {
       # settings for user authentification
       }
     /renders
       {
       # List of AEM instances that render the documents
       }
     /filter
       {
       # List of filters
       }
     /vanity_urls
       {
       # List of vanity URLs
       }
     /cache
       {
       # Cache configuration
       /rules
         {
         # List of cachable documents
         }
       /invalidate
         {
         # List of auto-invalidated documents
         }
       }
     /statistics
       {
       /categories
         {
         # The document categories that are used for load balancing estimates
         }
       }
     /stickyConnectionsFor "/myFolder"
     /health_check
       {
       # Page gets contacted when an instance returns a 500
       }
     /retryDelay "1"
     /numberOfRetries "5"
     /unavailablePenalty "1"
     /failover "1"
     }
 }
```

您可以包含可用于配置的其他文件：

* 如果配置文件太大，您可以将其拆分为多个较小的文件（这样易于管理），并包括每个文件。
* 可以包含自动生成的文件。

例如，要在 `/farms` 配置中包含文件 myFarm.any，请使用以下代码：

```xml
/farms
  {
  $include "myFarm.any"
  }
```

要指定一系列要包括的文件，请使用星号 (`*`) 作为通配符。

例如，如果文件 `farm_1.any` 至 `farm_5.any` 包含场 1 到 5 的配置，您可以按如下方式将其包含在内：

```xml
/farms
  {
  $include "farm_*.any"
  }
```

## 使用环境变量 {#using-environment-variables}

在 dispatcher.any 文件中，您可以在字符串值属性中使用环境变量而不是硬编码的值。要包括环境变量的值，请使用格式 `${variable_name}`。

例如，如果 dispatcher.any 文件在与缓存目录相同的目录中，则可使用 [docroot](#specifying-the-cache-directory) 属性的以下值：

```xml
/docroot "${PWD}/cache"
```

作为另一个示例，如果您创建名为 `PUBLISH_IP` 的环境变量，该变量存储 AEM 发布实例的主机名，则可以使用 [`/renders`](#defining-page-renderers-renders) 属性的以下配置：

```xml
/renders {
  /0001 {
    /hostname "${PUBLISH_IP}"
    /port "8443"
  }
}
```

## 命名 Dispatcher 实例 {#naming-the-dispatcher-instance-name}

使用 `/name` 属性指定唯一名称以标识您的 Dispatcher 实例。`/name` 属性是配置结构中的顶级属性。

## 定义场 {#defining-farms-farms}

`/farms` 属性定义一组或多组 Dispatcher 行为，每组行为与不同的网站或 URL 关联。`/farms` 属性可以包括单个场或多个场：

* 在希望 Dispatcher 以相同方式处理您的所有网页或网站时，请使用单个场。
* 当您的网站的不同区域或不同网站需要不同的 Dispatcher 行为时，请创建多个场。

`/farms` 属性是配置结构中的顶级属性。要定义场，请向 `/farms` 属性添加一个子属性。使用在 Dispatcher 实例中唯一标识场的属性名称。

`/farmname` 属性是多值属性，包含定义 Dispatcher 行为的其他属性：

* 场应用到的页面的 URL。
* 用于渲染文档的一个或多个服务 URL（通常是 AEM 发布实例）。
* 用于平衡多个文档渲染程序的负载的统计数据。
* 多种其他行为，例如缓存哪些文件以及缓存在哪里。

该值可包括任何字母数字（a 至 z、0 至 9）字符。以下示例演示了名为 `/daycom` 和 `/docsdaycom` 的两个场的主干定义：

```xml
#name of dispatcher
/name "day sites"

#farms section defines a list of farms or sites
/farms
{
   /daycom
   {
       ...
   }
   /docdaycom
   {
      ...
   }
}
```

>[!NOTE]
>
>如果您使用多个渲染场，则自下而上评估该列表。在为网站定义[虚拟主机](#identifying-virtual-hosts-virtualhosts)时，此流程相关。

每个场属性可以包含以下子属性：

| 属性名称 | 描述 |
|--- |--- |
| [/homepage](#specify-a-default-page-iis-only-homepage) | 默认主页（可选）（仅限 IIS） |
| [/clientheaders](#specifying-the-http-headers-to-pass-through-clientheaders) | 要传递的来自客户端 HTTP 请求的标头。 |
| [/virtualhosts](#identifying-virtual-hosts-virtualhosts) | 这个农场的虚拟主机。 |
| [/sessionmanagement](#enabling-secure-sessions-sessionmanagement) | 支持会话管理和身份验证。 |
| [/renders](#defining-page-renderers-renders) | 提供已渲染页面的服务器（通常是 AEM 发布实例）。 |
| [/filter](#configuring-access-to-content-filter) | 定义 Dispatcher 允许访问的 URL。 |
| [/vanity_urls](#enabling-access-to-vanity-urls-vanity-urls) | 配置对虚名 URL 的访问。 |
| [/propagateSyndPost](#forwarding-syndication-requests-propagatesyndpost) | 支持转发联合请求。 |
| [/cache](#configuring-the-dispatcher-cache-cache) | 配置缓存行为。 |
| [/statistics](#configuring-load-balancing-statistics) | 定义用于负载平衡计算的统计数据类别。 |
| [/stickyConnectionsFor](#identifying-a-sticky-connection-folder-stickyconnectionsfor) | 包含粘性文档的文件夹。 |
| [/health_check](#specifying-a-health-check-page) | 用于确定服务器可用性的 URL。 |
| [/retryDelay](#specifying-the-page-retry-delay) | 重试失败的连接之前的延迟。 |
| [/unavailablePenalty](#reflecting-server-unavailability-in-dispatcher-statistics) | 影响负载平衡计算的统计数据的惩罚。 |
| [/failover](#using-the-failover-mechanism) | 在原始请求失败时将请求重新发送到不同的渲染。 |
| [/auth_checker](permissions-cache.md) | 有关对权限敏感的缓存，请参阅[缓存受保护内容](permissions-cache.md)。 |

## 指定默认页面（仅限IIS） — /homepage {#specify-a-default-page-iis-only-homepage}

>[!CAUTION]
>
>`/homepage` 参数（仅限 IIS）不再有效。您应该改用 [IIS URL 重写模块](https://learn.microsoft.com/zh-cn/iis/extensions/url-rewrite-module/using-the-url-rewrite-module)。
>
>如果您在使用 Apache，则应该使用 `mod_rewrite` 模块。有关 `mod_rewrite`（例如，[Apache 2.4](https://httpd.apache.org/docs/current/mod/mod_rewrite.html)）的信息，请参阅 Apache 网站文档。在使用 `mod_rewrite` 时，建议使用“passthrough|PT”（传递到下个处理程序）标志强制重写引擎以将内部 `uri` 结构的 `request_rec` 字段设置为 `filename` 字段的值。

<!-- 

Comment Type: draft

<p>The optional /homepage parameter specifies the page that Dispatcher returns when a client requests an undeterminable page or file.</p> 
<p>Typically this situation occurs when a user specifies an URL for which neither IIS or AEM provides an automatic redirection target. For example, if the AEM render instance is shut down after the content is cached, the content redirect URL is unavailable.</p> 
<p>The following example configuration displays the <span class="code">index.html</span> page in such circumstances:</p>

 -->

<!-- 

Comment Type: draft

<codeblock gutter="true" class="syntax xml">
  /homepage&nbsp;"/index.html" 
</codeblock>

 -->

<!-- 

Comment Type: draft

<p>The <span class="code">/homepage</span> section is located inside the <span class="code">/farms</span> section, for example:<br /> </p>

 -->

<!-- 

Comment Type: draft

<codeblock gutter="true" class="syntax xml">
  #name&nbsp;of&nbsp;dispatcher!!discoiqbr!!/name&nbsp;"day&nbsp;sites"!!discoiqbr!!!!discoiqbr!!#farms&nbsp;section&nbsp;defines&nbsp;a&nbsp;list&nbsp;of&nbsp;farms&nbsp;or&nbsp;sites!!discoiqbr!!/farms!!discoiqbr!!{!!discoiqbr!!&nbsp;&nbsp;&nbsp;/daycom!!discoiqbr!!&nbsp;&nbsp;&nbsp;{!!discoiqbr!!&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/homepage&nbsp;"/index.html"!!discoiqbr!!&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;...!!discoiqbr!!&nbsp;&nbsp;&nbsp;}!!discoiqbr!!&nbsp;&nbsp;&nbsp;/docdaycom!!discoiqbr!!&nbsp;&nbsp;&nbsp;{!!discoiqbr!!&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;...!!discoiqbr!!&nbsp;&nbsp;&nbsp;}!!discoiqbr!!} 
</codeblock>

 -->

## 指定要传递的 HTTP 标头 {#specifying-the-http-headers-to-pass-through-clientheaders}

`/clientheaders` 属性定义 Dispatcher 从客户端 HTTP 请求传递到渲染程序（AEM 实例）的 HTTP 标头的列表。

默认情况下，Dispatcher 将标准 HTTP 标头转发到 AEM 实例。在一些实例中，您可能希望转发额外的标头或者移除特定标头：

* 在 HTTP 请求中添加 AEM 实例期望的标头，例如自定义标头。
* 删除只与 Web 服务器相关的标头，例如身份验证标头。

如果自定义要传递的那组标头，则必须指定详尽的标头列表，其中包括一般默认包括的那些标头。

例如，为发布实例处理页面激活请求的 Dispatcher 实例要求在 `/clientheaders` 部分中有 `PATH` 标头。`PATH` 标头使复制代理与 Dispatcher 之间可进行通信。

以下代码是 `/clientheaders` 的示例配置：

```shell
/clientheaders
  {
  "CSRF-Token"
  "X-Forwarded-Proto"
  "referer"
  "user-agent"
  "authorization"
  "from"
  "content-type"
  "content-length"
  "accept-charset"
  "accept-encoding"
  "accept-language"
  "accept"
  "host"
  "max-forwards"
  "proxy-authorization"
  "proxy-connection"
  "range"
  "cookie"
  "cq-action"
  "cq-handle"
  "handle"
  "action"
  "cqstats"
  "depth"
  "translate"
  "expires"
  "date"
  "dav"
  "ms-author-via"
  "if"
  "lock-token"
  "x-expected-entity-length"
  "destination"
  "PATH"
  }
```

## 识别虚拟主机 {#identifying-virtual-hosts-virtualhosts}

`/virtualhosts` 属性定义 Dispatcher 为此场接受的所有主机名和 URI 组合的列表。可以使用星号 (`*`) 字符作为通配符。/`virtualhosts` 属性的值使用以下格式：

```xml
[scheme]host[uri][*]
```

* `scheme`：（可选）`https://` 或 `https://.`
* `host`：主机的名称或 IP 地址，以及端口号（如有必要）。（请参阅 [https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23)）
* `uri`：（可选）资源的路径。

以下示例配置处理针对 myCompany 的 `.com` 和 `.ch` 域以及所有 mySubDivision 域的请求：

```xml
   /virtualhosts
    {
    "www.myCompany.com"
    "www.myCompany.ch"
    "www.mySubDivison.*"
    }
```

以下配置处理&#x200B;*所有*&#x200B;请求：

```xml
   /virtualhosts
    {
    "*"
    }
```

### 解析虚拟主机 {#resolving-the-virtual-host}

当 Dispatcher 收到 HTTP 或 HTTPS 请求时，它会查找与 `host,` `uri` 很好地匹配的虚拟主机值，以及请求的 `scheme` 标头。Dispatcher 按照以下顺序评估 `virtualhosts` 属性中的值：

* Dispatcher 从 dispatcher.any 中最低的场开始，然后依次往上。
* 对于每个场，Dispatcher 从 `virtualhosts` 属性中最顶部的值开始，然后在值列表中依次往下。

Dispatcher 按照以下方法查找很好地匹配的虚拟主机值：

* 使用遇到的第一个与请求的全部三个 `host`、`scheme` 和 `uri` 匹配的虚拟主机。
* 如果任何 `virtualhosts` 值的 `scheme` 和 `uri` 部分均与请求的 `scheme` 和 `uri` 不匹配，则将使用遇到的第一个与请求的 `host` 匹配的虚拟主机。
* 如果任何 `virtualhosts` 值的主机部分均与请求主机不匹配，则使用最顶部场的最顶部虚拟主机。

因此，应将默认虚拟主机放在 `virtualhosts` 属性的顶部。将其放置在您的 `dispatcher.any` 文件的最顶部场中。

### 虚拟主机解析示例 {#example-virtual-host-resolution}

以下示例演示了来自 `dispatcher.any` 文件的代码段，其中定义了两个 Dispatcher 场，并为每个场定义了一个 `virtualhosts` 属性。

```xml
/farms
  {
  /myProducts
    {
    /virtualhosts
      {
      "www.mycompany.com/products/*"
      }
    /renders
      {
      /hostname "server1.myCompany.com"
      /port "80"
      }
    }
  /myCompany
    {
    /virtualhosts
      {
      "www.mycompany.com"
      }
    /renders
      {
      /hostname "server2.myCompany.com"
      /port "80"
      }
    }
  }
```

对于此示例，下表显示了对于给定 HTTP 请求解析的虚拟主机：

| 请求 URL | 解析的虚拟主机 |
|---|---|
| `https://www.mycompany.com/products/gloves.html` | `www.mycompany.com/products/` |
| `https://www.mycompany.com/about.html` | `www.mycompany.com` |

## 启用安全会话 - `/sessionmanagement` {#enabling-secure-sessions-sessionmanagement}

>[!CAUTION]
>
>将 `/cache` 部分中的 `/allowAuthorized` 设置为 `"0"` 以启用此功能。正如[“使用身份验证时的缓存”](#caching-when-authentication-is-used)一节所述，设置 `/allowAuthorized 0 ` 后，包含身份验证信息的请求&#x200B;**不会**&#x200B;被缓存。如果需要在缓存时区分权限，请参阅[缓存受保护内容](https://experienceleague.adobe.com/zh-hans/docs/experience-manager-dispatcher/using/configuring/permissions-cache)页面。

创建安全会话以供访问渲染场，以使用户必须登录才能访问场中的任何页面。在登录之后，用户可以访问场中的各个页面。有关将此功能与 CUG 一起使用的信息，请参阅[创建封闭用户组](https://experienceleague.adobe.com/zh-hans/docs/experience-manager-65/content/security/cug#creating-the-user-group-to-be-used)。此外，在上线之前，请查看 Dispatcher [安全检查清单](/help/using/security-checklist.md)。

`/sessionmanagement` 属性是 `/farms` 的子属性。

>[!CAUTION]
>
>如果网站的各个部分使用不同的访问要求，则您需要定义多个场。

**/sessionmanagement** 有多个子参数：

**/directory**（必需）

存储会话信息的目录。如果目录不存在，则创建目录。

>[!CAUTION]
>
> 配置目录的子参数时，**请勿**&#x200B;指向根文件夹 (`/directory "/"`)，因为这可能会导致严重问题。请始终指定存储了会话信息的文件夹的路径。例如：

```xml
/sessionmanagement
  {
  /directory "/usr/local/apache/.sessions"
  }
```

**/encode**（可选）

如何对会话信息进行编码。使用 `md5` 可利用 md5 算法加密，或者使用 `hex` 进行十六进制加密。如果您加密了会话数据，则具有文件系统访问权限的用户也无法读取会话内容。默认为 `md5`。

**/header**（可选）

存储授权信息的 HTTP 标头或 Cookie 的名称。如果您要将信息存储在 http 标头中，请使用 `HTTP:<header-name>`。要将信息存储在 Cookie 中，请使用 `Cookie:<header-name>`。如果您未指定值，则使用 `HTTP:authorization`。

**/timeout**（可选）

会话在上次使用之后，进入超时的秒数。如果未指定，则使用 `"800"`，这样会话在用户的上个请求之后，13 分钟多一点超时。

示例配置如下所示：

```xml
/sessionmanagement
  {
  /directory "/usr/local/apache/.sessions"
  /encode "md5"
  /header "HTTP:authorization"
  /timeout "800"
  }
```

## 定义页面渲染程序 {#defining-page-renderers-renders}

`/renders` 属性定义 URL，Dispatcher 将请求发送到该 URL 以渲染文档。以下示例 `/renders` 部分标识了单个 AEM 示例用于渲染：

```xml
/renders
  {
    /myRenderer
      {
      # hostname or IP of the renderer
      /hostname "aem.myCompany.com"
      # port of the renderer
      /port "4503"
      # connection timeout in milliseconds, "0" (default) waits indefinitely
      /timeout "0"
      }
  }
```

以下示例 `/renders` 部分标识与 Dispatcher 在同一台计算机上运行的 AEM 实例：

```xml
/renders
  {
    /myRenderer
     {
     /hostname "127.0.0.1"
     /port "4503"
     }
  }
```

以下示例 `/renders` 部分在两个 AEM 实例之间均匀地分发渲染请求：

```xml
/renders
  {
    /myFirstRenderer
      {
      /hostname "aem.myCompany.com"
      /port "4503"
      }
    /mySecondRenderer
      {
      /hostname "127.0.0.1"
      /port "4503"
      }
  }
```

### 渲染选项 {#renders-options}

**/timeout**

指定访问 AEM 实例的连接超时，以毫秒为单位。默认值为 `"0"`，这会导致 Dispatcher 无限期等待。

**/receiveTimeout**

指定允许的响应时间，以毫秒为单位。默认值为 `"600000"`，这会导致 Dispatcher 等待 10 分钟。设置为 `"0"` 可消除超时。

如果在解析响应标头时出现超时，则返回 HTTP 状态 504（网关错误）。如果在读取响应正文时出现超时，则 Dispatcher 向客户端返回不完整的响应。它还会删除任何可能已写入的缓存文件。

**/ipv4**

指定 Dispatcher 使用 `getaddrinfo` 函数（用于 IPv6）还是 `gethostbyname` 函数（用于 IPv4）来获取渲染的 IP 地址。值为 0 会导致使用 `getaddrinfo`。值为 `1` 会导致使用 `gethostbyname`。默认值为 `0`。

`getaddrinfo` 函数返回 IP 地址列表。Dispatcher 迭代地址列表，直至建立了 TCP/IP 连接。因此，当渲染主机名与多个 IP 地址相关联时，`ipv4` 属性非常重要。而且，主机响应 `getaddrinfo` 函数时，会返回始终按相同顺序排列的 IP 地址列表。在这种情况下，您应使用 `gethostbyname` 函数，这样 Dispatcher 可以随机连接到 IP 地址。

Amazon Elastic Load Balancing (ELB) 就是这样一种服务，可以使用相同顺序的 IP 地址列表响应 getaddrinfo。

**/secure**

如果 `/secure` 属性的值为 `"1"`，则 Dispatcher 使用 HTTPS 与 AEM 实例通信。请参阅[配置 Dispatcher 使用 SSL](dispatcher-ssl.md#configuring-dispatcher-to-use-ssl)，了解详细信息。

**/always-resolve**

使用 Dispatcher 版本 **4.1.6**，您可以如下所示配置 `/always-resolve` 属性：

* 设置为 `"1"` 时，它在每次请求时都解析主机名（Dispatcher 从不缓存任何 IP 地址）。由于每个请求都需要额外的调用以获取主机信息，因此可能会稍有影响性能。
* 如果未设置该属性，则默认缓存 IP 地址。

此外，此属性也可以在您遇到动态 IP 解析问题时使用，如以下示例所示：

```xml
/renders {
  /0001 {
     /hostname "host-name-here"
     /port "4502"
     /ipv4 "1"
     /always-resolve "1"
     }
  }
```

## 配置内容访问权限 {#configuring-access-to-content-filter}

使用 `/filter` 部分指定 Dispatcher 接受的 HTTP 请求。所有其他请求使用 404 错误代码（页面未找到）发送回 Web 服务器。如果没有 `/filter` 部分，则接受所有请求。

**注意：** 始终拒绝对 [statfile](#naming-the-statfile) 的请求。

>[!CAUTION]
>
>请参阅 [AEM Dispatcher 安全检查清单](security-checklist.md)，了解使用 Dispatcher 限制访问时的更多注意事项。有关 AEM 安装的其他安全详细信息，另请阅读 [AEM 安全检查清单](https://experienceleague.adobe.com/zh-hans/docs/experience-manager-65/content/security/security-checklist#security)。

`/filter` 部分由一系列规则组成，这些规则根据 HTTP 请求的请求行部分中的模式拒绝或允许访问内容。为 `/filter` 部分使用允许列表策略：

* 首先，拒绝对一切的访问。
* 在需要时允许访问内容。

>[!NOTE]
>
>只要筛选规则中有变，就要清除缓存。

### 定义过滤器 {#defining-a-filter}

`/filter` 部分中的每一项包含一个类型和一个模式，该项与请求行的特定元素或整个请求行匹配。每个过滤器可以包含以下项：

* **类型**：`/type` 指示是允许还是拒绝与模式匹配的请求的访问。该值可以为 `allow` 或 `deny`。

* **请求行的元素：**&#x200B;包括 `/method`、`/url`、`/query` 或 `/protocol`。此外，还包括一种过滤请求的模式。根据 HTTP 请求中请求行部分的特定内容对其进行过滤。根据请求行的元素进行筛选（而不是整个请求行）是首选的筛选模式。

* **请求行的高级元素：** 从 Dispatcher 4.2.0 开始，有 4 个新的过滤器元素可供使用。这些新元素分别是 `/path`、`/selectors`、`/extension` 和 `/suffix`。包括这些项中的一个或多个可以进一步控制 URL 模式。

>[!NOTE]
>
>有关上述每个元素所引用的请求行部分的更多信息，请参阅 [Sling URL 分解](https://sling.apache.org/documentation/the-sling-engine/url-decomposition.html) wiki 页面。

* **glob 属性**：`/glob` 属性用于匹配 HTTP 请求的整个请求行。

>[!CAUTION]
>
>使用 glob 进行筛选的方式在 Dispatcher 中已弃用。因此，您应该避免在 `/filter` 部分中使用 glob，因为这会导致安全问题。因此，不应使用：
>
>`/glob "* *.css *"`
>
>使用
>
>`/url "*.css"`

#### HTTP 请求的 request-line 部分 {#the-request-line-part-of-http-requests}

HTTP/1.1 如下所示定义[请求行](https://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html)：

`Method Request-URI HTTP-Version<CRLF>`

`<CRLF>` 字符表示回车后跟换行符。以下示例是客户端请求 WKND 网站的美国英语页面时收到的请求行：

`GET /content/wknd/us/en.html HTTP.1.1<CRLF>`

您的模式必须考虑到请求行中的空格字符和 `<CRLF>` 字符。

#### 双引号与单引号 {#double-quotes-vs-single-quotes}

创建筛选规则时，为简单模式使用双引号 `"pattern"`。如果您使用 Dispatcher 4.2.0 或更高版本并且模式包含正则表达式，则必须将正则表达式模式 `'(pattern1|pattern2)'` 括在单引号中。

#### 正则表达式 {#regular-expressions}

在版本高于 4.2.0 的 Dispatcher 中，您可以在筛选模式中包含 POSIX 扩展正则表达式。

#### 过滤器疑难解答 {#troubleshooting-filters}

如果过滤器未按预期触发，请在 Dispatcher 上启用[追踪日志记录](#trace-logging)，以便查看是哪个过滤器拦截了请求。

#### 示例过滤器：全部拒绝 {#example-filter-deny-all}

以下示例过滤器部分导致 Dispatcher 拒绝对所有文件的请求。拒绝访问所有文件，然后允许访问特定区域。

```xml
/0001  { /type "deny" /url "*"  }
```

对被明确拒绝访问的区域发出的请求将返回 404 错误代码（页面未找到）。

#### 示例过滤器：拒绝对特定区域的访问 {#example-filter-deny-access-to-specific-areas}

通过过滤器，还可拒绝访问各种元素，例如 ASP 页面和发布实例中的敏感区域。以下过滤器拒绝访问 ASP 页面：

```xml
/0002  { /type "deny" /url "*.asp"  }
```

#### 示例过滤器：启用 POST 请求 {#example-filter-enable-post-requests}

以下示例过滤器允许由 POST 方法提交表单数据：

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002 { /type "allow" /method "POST" /url "/content/[.]*.form.html" }
}
```

#### 示例过滤器：允许对工作流程控制台的访问 {#example-filter-allow-access-to-the-workflow-console}

以下示例显示用于允许从外部访问工作流控制台的过滤器：

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002  {  /type "allow"  /url "/libs/cq/workflow/content/console*"  }
}
```

如果您的发布实例使用 Web 应用程序上下文（例如，发布），则还可将它添加到您的过滤器定义。

```xml
/0003   { /type "deny"  /url "/publish/libs/cq/workflow/content/console/archive*"  }
```

如果必须访问受限区域中的单个页面，则可允许访问这些页面。例如，要允许对工作流程控制台中“存档”选项卡的访问，请添加以下部分：

```xml
/0004  { /type "allow"  /url "/libs/cq/workflow/content/console/archive*"   }
```

>[!NOTE]
>
>当多个过滤器模式同时用于一个请求时，最后应用的过滤器模式将生效。

#### 示例过滤器：使用正则表达式 {#example-filter-using-regular-expressions}

此过滤器使用正则表达式（如下以单引号括起）启用对非公开内容目录中指定扩展名的访问权限：

```xml
/005  {  /type "allow" /extension '(css|gif|ico|js|png|swf|jpe?g)' }
```

#### 示例过滤器：过滤某个请求 URL 中的额外元素 {#example-filter-filter-additional-elements-of-a-request-url}

以下是一个规则示例，它使用路径、选择器和扩展名过滤器阻止从 `/content` 路径及其子树抓取内容：

```xml
/006 {
        /type "deny"
        /path "/content/*"
        /selectors '(feed|rss|pages|languages|blueprint|infinity|tidy|sysview|docview|query|jcr:content|_jcr_content|search|childrenlist|ext|assets|assetsearch|[0-9-]+)'
        /extension '(json|xml|html|feed))'
        }
```

### 示例 `/filter` 部分 {#example-filter-section}

在配置 Dispatcher 时应尽可能多地限制外部访问。以下示例为外部访客提供了最小限度的访问权限：

* `/content`
* 其他内容，例如设计资源和客户端库。例如：

   * `/etc/designs/default*`
   * `/etc/designs/mydesign*`

创建过滤器之后，[测试页面访问权限](#testing-dispatcher-security)以确保您的 AEM 实例安全无虞。

`dispatcher.any` 文件的以下 `/filter` 部分可用作您的 [Dispatcher 配置文件的基础。](#dispatcher-configuration-files)

此示例基于随 Dispatcher 提供的默认配置文件，其目的是作为在生产环境中使用的示例。具有 `#` 前缀的项被停用（被注释掉）。如果决定（通过删除该行上的 `#`）激活这些项的其中之一，请谨慎操作。这样做可能会影响安全性。

拒绝访问一切，然后允许访问特定（有限）元素：

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-26T04:32:37.986-0400

<p>We should mention the config files that are shipped with the dispatcher distribution and only give a few examples here. This aims to avoid confusion and reduce content maintenance.<br /> </p>

 -->

```xml
  /filter
      {
      # Deny everything first and then allow specific entries
      /0001  { /type "deny" /url "*"  }

      # Open consoles
#     /0011 { /type "allow" /url "/admin/*"  }  # allow servlet engine admin
#     /0012 { /type "allow" /url "/crx/*"    }  # allow content repository
#     /0013 { /type "allow" /url "/system/*" }  # allow OSGi console

      # Allow non-public content directories
#     /0021 { /type "allow" /url "/apps/*"   }  # allow apps access
#     /0022 { /type "allow" /url "/bin/*"    }
      /0023 { /type "allow" /url "/content*" }  # disable this rule to allow mapped content only

#     /0024 { /type "allow" /url "/libs/*"   }
#     /0025 { /type "deny"  /url "/libs/shindig/proxy*" } # if you enable /libs close access to proxy

#     /0026 { /type "allow" /url "/home/*"   }
#     /0027 { /type "allow" /url "/tmp/*"    }
#     /0028 { /type "allow" /url "/var/*"    }

      # Enable extensions in non-public content directories, using a regular expression
      /0041
        {
        /type "allow"
        /extension '(css|gif|ico|js|png|swf|jpe?g)'
        }

      # Enable features
      /0062 { /type "allow" /url "/libs/cq/personalization/*"  }  # enable personalization

      # Deny content grabbing, on all accessible pages, using regular expressions
      /0081
        {
        /type "deny"
        /selectors '((sys|doc)view|query|[0-9-]+)'
        /extension '(json|xml)'
        }
      # Deny content grabbing for /content and its subtree
      /0082
        {
        /type "deny"
        /path "/content/*"
        /selectors '(feed|rss|pages|languages|blueprint|infinity|tidy)'
        /extension '(json|xml|html)'
        }

#     /0087 { /type "allow" /method "GET" /extension 'json' "*.1.json" }  # allow one-level json requests
}
```

>[!NOTE]
>
>在用于 Apache 时，根据 Dispatcher 模块的 DispatcherUseProcessedURL 属性设计过滤器 URL 模式。（请参阅[Apache Web Server - 为 Dispatcher 配置 Apache Web Server](dispatcher-install.md##apache-web-server-configure-apache-web-server-for-dispatcher)。）

<!----
>[!NOTE]
>
>Filters `0030` and `0031` regarding Dynamic Media are applicable to AEM 6.0 and higher. -->

如果确实决定扩大访问范围，请考虑以下建议：

* 如果使用 CQ 5.4 版或更低版本，请禁止外部访问 `/admin`。

* 在允许访问 `/libs` 中的文件时务必谨慎。应该按各个文件授予访问权限。
* 拒绝对复制配置的访问，这样就无法看到它：

   * `/etc/replication.xml*`
   * `/etc/replication.infinity.json*`

* 拒绝对 Google Gadgets 反向代理的访问：

   * `/libs/opensocial/proxy*`

根据您的安装，`/libs`、`/apps` 下或其他位置可能会必须提供更多资源。您可以使用 `access.log` 文件作为一种方法来确定被外部访问的资源。

>[!CAUTION]
>
>对控制台和目录的访问可能会对生产环境带来安全风险。除非您有明确的理由，否则它们应保持停用（注释掉）。

>[!CAUTION]
>
>如果您在[发布环境中使用报告](https://experienceleague.adobe.com/zh-hans/docs/experience-manager-65/content/sites/administering/operations/reporting#using-reports-in-a-publish-environment)，则应将 Dispatcher 配置为拒绝外部访客访问 `/etc/reports`。

### 限制查询字符串 {#restricting-query-strings}

自 Dispatcher 版本 4.1.5 开始，使用 `/filter` 部分来限制查询字符串。建议通过 `allow` 过滤器元素明确允许查询字符串并排除普遍的允许。

一个项可以具有 `glob` 或者 `method`、`url`、`query` 和 `version` 的一些组合，但不能同时具有。以下示例针对解析到 `/etc` 节点的 URL，允许 `a=*` 查询字符串并拒绝所有其他查询字符串：

```xml
/filter {
 /0001 { /type "deny" /method "POST" /url "/etc/*" }
 /0002 { /type "allow" /method "GET" /url "/etc/*" /query "a=*" }
}
```

>[!NOTE]
>
>如果某个规则包含 `/query`，则它仅匹配包含查询字符串并与所提供的查询模式匹配的请求。
>
>在上例中，如果也要允许对不带查询字符串的 `/etc` 的请求，则需要添加以下规则：
>

```xml
/filter {  
>/0001 { /type "deny" /method "*" /url "/path/*" }  
>/0002 { /type "allow" /method "GET" /url "/path/*" }  
>/0003 { /type "deny" /method "GET" /url "/path/*" /query "*" }  
>/0004 { /type "allow" /method "GET" /url "/path/*" /query "a=*" }  
}  
```

### 测试 Dispatcher 安全性 {#testing-dispatcher-security}

Dispatcher 过滤器在 AEM 发布实例上应该阻止对以下页面和脚本的访问。使用 Web 浏览器尝试以网站访客身份打开以下页面，并验证是否返回了代码 404。如果获得了其他结果，请调整过滤器。

您应看到正常渲染 `/content/add_valid_page.html?debug=layout` 页面。

* `/admin`
* `/system/console`
* `/dav/crx.default`
* `/crx`
* `/bin/crxde/logs`
* `/jcr:system/jcr:versionStorage.json`
* `/_jcr_system/_jcr_versionStorage.json`
* `/libs/wcm/core/content/siteadmin.html`
* `/libs/collab/core/content/admin.html`
* `/libs/cq/ui/content/dumplibs.html`
* `/var/linkchecker.html`
* `/etc/linkchecker.html`
* `/home/users/a/admin/profile.json`
* `/home/users/a/admin/profile.xml`
* `/libs/cq/core/content/login.json`
* `/content/../libs/foundation/components/text/text.jsp`
* `/content/.{.}/libs/foundation/components/text/text.jsp`
* `/apps/sling/config/org.apache.felix.webconsole.internal.servlet.OsgiManager.config/jcr%3acontent/jcr%3adata`
* `/libs/foundation/components/primary/cq/workflow/components/participants/json.GET.servlet`
* `/content.pages.json`
* `/content.languages.json`
* `/content.blueprint.json`
* `/content.-1.json`
* `/content.10.json`
* `/content.infinity.json`
* `/content.tidy.json`
* `/content.tidy.-1.blubber.json`
* `/content/dam.tidy.-100.json`
* `/content/content/geometrixx.sitemap.txt `
* `/content/add_valid_page.query.json?statement=//*`
* `/content/add_valid_page.qu%65ry.js%6Fn?statement=//*`
* `/content/add_valid_page.query.json?statement=//*[@transportPassword]/(@transportPassword%20|%20@transportUri%20|%20@transportUser)`
* `/content/add_valid_path_to_a_page/_jcr_content.json`
* `/content/add_valid_path_to_a_page/jcr:content.json`
* `/content/add_valid_path_to_a_page/_jcr_content.feed`
* `/content/add_valid_path_to_a_page/jcr:content.feed`
* `/content/add_valid_path_to_a_page/pagename._jcr_content.feed`
* `/content/add_valid_path_to_a_page/pagename.jcr:content.feed`
* `/content/add_valid_path_to_a_page/pagename.docview.xml`
* `/content/add_valid_path_to_a_page/pagename.docview.json`
* `/content/add_valid_path_to_a_page/pagename.sysview.xml`
* `/etc.xml`
* `/content.feed.xml`
* `/content.rss.xml`
* `/content.feed.html`
* `/content/add_valid_page.html?debug=layout`
* `/projects`
* `/tagging`
* `/etc/replication.html`
* `/etc/cloudservices.html`
* `/welcome`

要确定是否启用了匿名写入权限，请在终端或命令提示符下发出以下命令。不可能向节点写入数据。

`curl -X POST "https://anonymous:anonymous@hostname:port/content/usergenerated/mytestnode"`

若要尝试使 Dispatcher 缓存失效并确认返回 403 响应代码，请在终端或命令提示符中输入以下命令：

`curl -H "CQ-Handle: /content" -H "CQ-Path: /content" https://yourhostname/dispatcher/invalidate.cache`

## 启用对虚名 URL 的访问权限 {#enabling-access-to-vanity-urls-vanity-urls}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (sbroders@adobe.com)
Last Modified Date: 2015-03-25T14:23:05.185-0400

<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">For https://jira.corp.adobe.com/browse/DOC-4812</p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">The "com.adobe.granite.dispatcher.vanityurl.content" package needs to be made public before publishing this contnet.</p>
 -->

配置 Dispatcher 以启用对为 AEM 页面配置的虚名 URL 的访问。

启用了对虚名 URL 的访问时，Dispatcher 定期调用在渲染实例上运行的服务以获取虚名 URL 的列表。Dispatcher 将此列表存储在本地文件中。由于 `/filter` 部分中的过滤器而拒绝了对页面的请求时，Dispatcher 会参考虚名 URL 的列表。如果被拒绝的 URL 位于列表上，Dispatcher 将允许对虚名 URL 的访问。

要启用对虚名 URL 的访问，请将 `/vanity_urls` 部分添加到 `/farms` 部分中，类似于以下实例：

```xml
 /vanity_urls {
      /url "/libs/granite/dispatcher/content/vanityUrls.html"
      /file "/tmp/vanity_urls"
      /delay 300
 }
```

`/vanity_urls` 部分包含以下属性：

* `/url`：在渲染实例上运行的虚名 URL 服务的路径。此属性的值必须为 `"/libs/granite/dispatcher/content/vanityUrls.html"`。

* `/file`：Dispatcher 存储虚名 URL 列表的本地文件的路径。确保 Dispatcher 对此文件具有读写访问权限。
* `/delay`：对虚名 URL 服务调用所间隔的时间（秒）。

>[!NOTE]
>
>如果渲染器为 AEM 实例，则必须[安装软件发行版中的 VanityURLS-Components 包](https://experience.adobe.com/#/downloads/content/software-distribution/en/aem.html?package=/content/software-distribution/en/details.html/content/dam/aem/public/adobe/packages/granite/vanityurls-components)以启用虚 URL 服务。（有关更多详细信息，请参阅[软件分发](https://experienceleague.adobe.com/zh-hans/docs/experience-manager-65/content/sites/administering/contentmanagement/package-manager#software-distribution)。）

使用以下过程启用对虚名 URL 的访问。

1. 如果渲染服务是 AEM 实例，请在发布实例上安装 `com.adobe.granite.dispatcher.vanityurl.content` 程序包（参见以上说明）。
1. 对于您为 AEM 或 CQ 页面配置的每个虚名 URL，请确保 [`/filter`](#configuring-access-to-content-filter) 配置拒绝该 URL。如有必要，请添加拒绝该 URL 的过滤器。
1. 在 `/farms` 下添加 `/vanity_urls` 部分。
1. 重新启动 Apache Web Server。

Dispatcher **版本 4.3.6** 中增加了一个新 `/loadOnStartup` 参数。通过使用该参数，您可以按如下方式配置在启动时加载虚名 URL：

通过添加 `/loadOnStartup 0`（参见下方示例），可以禁止在启动时加载虚名 URL。

```
/vanity_urls {
        /url "/libs/granite/dispatcher/content/vanityUrls.html"
        /file "/tmp/vanity_urls"
        /loadOnStartup 0
        /delay 60
      } 
```

`/loadOnStartup 1` 会在启动时加载虚名 URL，请注意 `/loadOnStartup 1` 是该参数的当前默认值。

## 转发联合请求 - `/propagateSyndPost` {#forwarding-syndication-requests-propagatesyndpost}

联合请求仅用于 Dispatcher，因此默认不将此类请求发送到渲染器（例如，AEM 实例）。

如有必要，请将 `/propagateSyndPost` 属性设置为 `"1"` 以将联合请求转发到 Dispatcher。如果设置此属性，您必须确保在过滤器部分中没有拒绝 POST 请求。

## 配置 Dispatcher 缓存 - `/cache` {#configuring-the-dispatcher-cache-cache}

`/cache` 部分控制 Dispatcher 如何缓存文档。配置多个子属性以实施缓存策略：

* `/docroot`
* `/statfile`
* `/serveStaleOnError`
* `/allowAuthorized`
* `/rules`
* `/statfileslevel`
* `/invalidate`
* `/invalidateHandler`
* `/allowedClients`
* `/ignoreUrlParams`
* `/headers`
* `/mode`
* `/gracePeriod`
* `/enableTTL`

示例缓存部分如下所示：

```xml
/cache
  {
  /docroot "/opt/dispatcher/cache"
  /statfile  "/tmp/dispatcher-website.stat"
  /allowAuthorized "0"

  /rules
    {
    # List of files that are cached
    }

  /invalidate
    {
    # List of files that are auto-invalidated
    }
  }
  
```

>[!NOTE]
>
>有关对权限敏感的缓存，请阅读[缓存受保护内容](permissions-cache.md)。

### 指定缓存目录 {#specifying-the-cache-directory}

`/docroot` 属性用于指定缓存文件的存储目录。

>[!NOTE]
>
>该值必须是与 Web 服务器的文档根相同的路径，以便 Dispatcher 和 Web 服务器处理相同的文件。\
>在使用 Dispatcher 缓存文件时，由 Web 服务器负责传送正确的状态代码，正因如此，服务器能够找到该文件非常重要。

如果您使用多个场，则每个场必须使用不同的文档根目录。

### 命名状态文件 {#naming-the-statfile}

`/statfile` 属性标识要用作 statfile 的文件。Dispatcher 使用此文件来注册最近更新内容的时间。statfile 可以是 Web 服务器上的任意文件。

statfile 不包括内容。内容有更新时，Dispatcher 会更新时间戳。默认 statfile 的名称为 `.stat`，存储在 docroot 中。Dispatcher 阻止对 statfile 的访问。

>[!NOTE]
>
>如果配置了 `/statfileslevel`，则 Dispatcher 忽略 `/statfile` 属性并使用 `.stat` 作为名称。

### 在发生错误时提供过期文档 {#serving-stale-documents-when-errors-occur}

`/serveStaleOnError` 属性用于控制当渲染服务器返回错误时，Dispatcher 是否返回已失效的文档。默认情况下，当触及 statfile 并使缓存内容失效时，Dispatcher 会删除缓存的内容。此操作将在下次请求时执行。

如果将 `/serveStaleOnError` 设置为 `"1"`，则 Dispatcher 不会从缓存中删除已失效的内容。也就是说，除非渲染服务器返回成功响应。来自 AEM 的 502、503 或 504 响应或者连接超时导致 Dispatcher 提供过期的内容，并使用 HTTP 状态 111（重新验证失败）响应。

### 在使用身份验证时进行缓存 {#caching-when-authentication-is-used}

`/allowAuthorized` 属性控制是否缓存包含以下任意身份验证信息的请求：

* `authorization` 标头
* 名为 `authorization` 的 Cookie
* 名为 `login-token` 的 Cookie

默认情况下，不缓存包含此身份验证信息的请求，因为缓存的文档在返回到客户端时不执行身份验证。此配置防止 Dispatcher 向没有所需权限的用户提供缓存的文档。

但是，如果要求允许缓存经过身份验证的文档，请将 `/allowAuthorized` 设置为 1：

`/allowAuthorized "1"`

>[!NOTE]
>
>为了启用会话管理（使用 `/sessionmanagement` 属性），`/allowAuthorized` 属性必须设置为 `"0"`。

### 指定要缓存的文档 {#specifying-the-documents-to-cache}

`/rules` 属性根据文档路径控制要缓存哪些文档。不论 `/rules` 属性如何，在以下情况下，Dispatcher 绝不会缓存文档：

* 请求 URI 包含问号 (`?`)。
   * 指示无需缓存的动态页面，如搜索结果。
* 缺失文件扩展名。
   * Web 服务器需要扩展名以确定文档类型（MIME 类型）。
* 设置了身份验证标头（可配置）。
* AEM 实例使用以下标头进行响应：

   * `no-cache`
   * `no-store`
   * `must-revalidate`

>[!NOTE]
>
>GET 或 HEAD（针对 HTTP 标头）方法可由 Dispatcher 缓存。有关响应标头缓存的其他信息，请参阅[缓存 HTTP 响应标头](#caching-http-response-headers)部分。

`/rules` 属性中的每一项包含一个 [`glob`](#designing-patterns-for-glob-properties) 模式和一个类型：

* `glob` 模式用于匹配文档的路径。
* 该类型指示是否缓存与 `glob` 模式匹配的文档。该值可以是 `allow`（缓存文档）或者 `deny`（渲染文档）。

如果您（在上述规则已排除的动态页面之外）没有动态页面，则可配置 Dispatcher 以缓存一切。规则部分如下所示：

```xml
/rules
  {
    /0000  {  /glob "*"   /type "allow" }
  }
```

有关 Glob 属性的信息，请参阅[为 Glob 属性设计模式](#designing-patterns-for-glob-properties)。

如果您的页面中有一些部分是动态的（例如，新闻应用程序）或者位于封闭用户组中，您可以定义例外情况：

>[!NOTE]
>
>不要缓存封闭的用户组。原因是未检查缓存页面的用户权限。

```xml
/rules
  {
   /0000  { /glob "*" /type "allow" }
   /0001  { /glob "/en/news/*" /type "deny" }
   /0002  { /glob "*/private/*" /type "deny"  }
  }
```

**压缩**

可在 Apache Web Server 上压缩缓存的文档。压缩使得 Apache 可以在客户端这样请求时，返回压缩格式的文档。压缩通过启用 Apache 模块 `mod_deflate` 自动完成，例如：

```xml
AddOutputFilterByType DEFLATE text/plain
```

该模块默认随 Apache 2.x 安装。

<!-- 

Comment Type: draft

<note type="note"> 
 <p>Depending on the Apache web server version you can enable <span class="code">gzip</span> compression as follows:</p> 
 <ul> 
  <li>For Apache 1.3 you can enable the <span class="code">mod_gzip </span>module. The module must be downloaded and installed.</li> 
  <li>For Apache 2.x you can enable the <span class="code">mod_deflate</span> module. The module is installed by default with Apache 2.x.<br /> </li> 
 </ul> 
 <p> </p> 
</note>

 -->

<!-- 

Comment Type: draft

<p>The following rule caches all documents in compressed form; Apache can return either the uncompressed or the compressed form to the client:</p>

 -->

<!-- 

Comment Type: draft

<codeblock gutter="true" class="syntax xml">
  /rules!!discoiqbr!!&nbsp;&nbsp;{!!discoiqbr!!&nbsp;&nbsp;&nbsp;/rulelabel&nbsp;&nbsp;{&nbsp;&nbsp;/glob&nbsp;"*"&nbsp;/type&nbsp;"allow"&nbsp;&nbsp;/compress&nbsp;"gzip"&nbsp;}!!discoiqbr!!&nbsp;&nbsp;} 
</codeblock>

 -->

<!-- 

Comment Type: remark
Last Modified By: Silviu Raiman (raiman)
Last Modified Date: 2017-11-13T09:23:24.326-0500

<p>Hidden the <span class="code">mod_gzip</span> content as requested in CQDOC-11124.</p>

 -->

### 按文件夹级别使文件失效 {#invalidating-files-by-folder-level}

使用 `/statfileslevel` 属性，根据其路径使缓存的文件失效：

* Dispatcher 在从 docroot 文件夹到您指定的文件夹级别的每个文件夹中创建 `.stat` 文件。docroot 文件夹是级别 0。
* 文件通过接触 `.stat` 文件而失效。`.stat` 文件的上次修改日期与缓存的文档的上次修改日期相比较。如果 `.stat` 文件较新，则重新提取文档。

* 当特定级别上的文件失效时，将接触从 docroot **到**&#x200B;失效文件所在的级别或所配置的 `statsfilevel`（其中较小者）的&#x200B;**所有** `.stat` 文件。

   * 例如：如果将 `statfileslevel` 属性设置为 6，而 5 级上的某个文件失效，则将接触从 docroot 到 5 级的每个 `.stat` 文件。仍以此为例，如果 7 级上的某个文件无效，则将接触从 docroot 到 6 级的每个 `stat` 文件（因 `/statfileslevel = "6"`）。

只有&#x200B;**沿着路径**&#x200B;到失效文件的资源不受影响。请考虑以下示例：网站使用结构 `/content/myWebsite/xx/.`。如果您将 `statfileslevel` 设置为 3，则系统如下所示创建 `.stat` 文件：

* `docroot`
* `/content`
* `/content/myWebsite`
* `/content/myWebsite/*xx*`

当 `/content/myWebsite/xx` 中的某个文件失效时，将接触从 docroot 下至 `/content/myWebsite/xx` 的每个 `.stat` 文件。此场景仅适用于 `/content/myWebsite/xx`，而不适用于示例 `/content/myWebsite/yy` 或 `/content/anotherWebSite`。

>[!NOTE]
>
>可以通过发送额外的标头 `CQ-Action-Scope:ResourceOnly` 防止失效。此方法可用于刷新特定资源而不使缓存的其他部分失效。有关其他详细信息，请参阅[此页面](https://adobe-consulting-services.github.io/acs-aem-commons/features/dispatcher-flush-rules/index.html)和[手动使 Dispatcher 缓存失效](https://experienceleague.adobe.com/zh-hans/docs/experience-manager-dispatcher/using/configuring/page-invalidate#configuring)。

>[!NOTE]
>
>如果您为 `/statfileslevel` 属性指定值，则忽略 `/statfile` 属性。

### 自动使缓存的文件失效 {#automatically-invalidating-cached-files}

`/invalidate` 属性定义在内容更新时自动失效的文档。

使用自动失效，Dispatcher 不在内容更新之后删除缓存的文件，而是在下次请求时检查其有效性。缓存中不自动失效的文档留在缓存中，直至内容更新显式地删除这些文档为止。

自动失效通常用于 HTML 页面。HTML 页面通常包含其他页面的链接，使其难于确定内容更新是否影响页面。为了确保所有相关的页面在内容更新时失效，请自动失效所有 HTML 页面。以下配置使所有 HTML 页面失效：

```xml
  /invalidate
  {
   /0000  { /glob "*" /type "deny" }
   /0001  { /glob "*.html" /type "allow" }
  }
```

有关 Glob 属性的信息，请参阅[为 Glob 属性设计模式](#designing-patterns-for-glob-properties)。

此配置导致在激活 `/content/wknd/us/en` 时出现以下活动：

* 所有具有模式 en.* 的文件从 `/content/wknd/us` 文件夹中删除。
* 删除 `/content/wknd/us/en./_jcr_content` 文件夹。
* 与 `/invalidate` 配置匹配的所有其他文件不立即删除。这些文件在下次出现请求时删除。在该示例中并不删除 `/content/wknd.html`，而是在请求 `/content/wknd.html` 时将其删除。

如果您的产品建议自动生成了 PDF 和 ZIP 文件以供下载，则您可能还需要自动使这些文件失效。配置示例如下所示：

```xml
/invalidate
  {
   /0000 { /glob "*" /type "deny" }
   /0001 { /glob "*.html" /type "allow" }
   /0002 { /glob "*.zip" /type "allow" }
   /0003 { /glob "*.pdf" /type "allow" }
  }
```

AEM 与 Adobe Analytics 集成，在网站的 `analytics.sitecatalyst.js` 文件中提供配置数据。随 Dispatcher 提供的示例 `dispatcher.any` 文件包括此文件的以下失效规则：

```xml
{
   /glob "*/analytics.sitecatalyst.js"  /type "allow"
}
```

### 使用自定义失效脚本 {#using-custom-invalidation-scripts}

`/invalidateHandler` 属性可让您定义会在 Dispatcher 收到每个无效请求时调用的脚本。

使用以下参数调用脚本：

* 句柄 - 失效的内容路径
* 操作 - 复制操作（例如，激活、停用）
* 操作范围 - 复制操作的范围（除非发送了 `CQ-Action-Scope: ResourceOnly` 标头，否则为空。有关详细信息，请参阅[使从 AEM 中缓存的页面失效](page-invalidate.md)）

此方法可用于涵盖多个不同的用例。例如，使其他应用程序特有的缓存失效，或处理页面的外部化 URL 及其在 docroot 中的位置与内容路径不匹配的情况。

以下示例脚本将每个失效请求记录到一个文件。

```xml
/invalidateHandler "/opt/dispatcher/scripts/invalidate.sh"
```

#### 失效处理器脚本示例 {#sample-invalidation-handler-script}

```shell
#!/bin/bash

printf "%-15s: %s %s" $1 $2 $3>> /opt/dispatcher/logs/invalidate.log
```

### 限制可以刷新缓存的客户端 {#limiting-the-clients-that-can-flush-the-cache}

`/allowedClients` 属性定义允许刷新缓存的特定客户端。通配模式根据 IP 进行匹配。

以下示例：

1. 拒绝对任意客户端的访问
1. 明确允许对本地主机的访问

```xml
/allowedClients
  {
   /0001 { /glob "*.*.*.*"  /type "deny" }
   /0002 { /glob "127.0.0.1" /type "allow" }
  }
```

有关 Glob 属性的信息，请参阅[为 Glob 属性设计模式](#designing-patterns-for-glob-properties)。

>[!CAUTION]
>
>建议您定义 `/allowedClients`。
>
>如果不这样做，则任何客户端均可发出清除缓存的调用。如果重复这样做，则可能会严重影响网站性能。

### 忽略 URL 参数 {#ignoring-url-parameters}

`ignoreUrlParams` 部分定义在确定是否缓存页面或者从缓存提供页面时，忽略哪些 URL 参数：

* 当请求 URL 包含全部忽略的参数时，将缓存页面。
* 当请求 URL 包含一个或多个未忽略的参数时，不缓存页面。

忽略页面的某个参数时，在首次请求页面时缓存该页面。对该页面后续的请求提供缓存的页面，不论请求中的参数值如何。

>[!NOTE]
>
>建议您以允许列表的方式配置`ignoreUrlParams`设置。因此，所有查询参数都会被忽略，只有已知或预期的查询参数才不会被忽略（“拒绝”）。有关更多详细信息和示例，请参阅[此页面](https://github.com/adobe/aem-dispatcher-optimizer-tool/blob/main/docs/Rules.md#dot---the-dispatcher-publish-farm-cache-should-have-its-ignoreurlparams-rules-configured-in-an-allow-list-manner)。

要指定忽略哪些参数，请将 glob 规则添加到 `ignoreUrlParams` 属性：

* 要缓存页面而不管请求中是否包含 URL 参数，请创建一个允许（忽略）该参数的 Glob 属性。
* 要防止缓存该页面，请创建一个拒绝（忽略）该参数的 glob 属性。

>[!NOTE]
>
>在配置该 glob 属性时，它应与查询参数名称一致。例如，如果要从 URL `http://example.com/path/test.html?p1=test&p2=v2` 中忽略“p1”参数，则 glob 属性应为：
>> `/0002 { /glob "p1" /type "allow" }`

以下示例导致 Dispatcher 忽略除 `nocache` 参数之外的所有参数。因此，Dispatcher 从不缓存包括 `nocache` 参数的请求 URL：

```xml
/ignoreUrlParams
{
    # ignore-all-url-parameters-by-dispatcher-and-requests-are-cached
    /0001 { /glob "*" /type "allow" }
    # allow-the-url-parameter-nocache-to-bypass-dispatcher-on-every-request
    /0002 { /glob "nocache" /type "deny" }
}
```

在上述 `ignoreUrlParams` 配置示例的上下文中，由于 `willbecached` 参数被忽略，以下 HTTP 请求会导致页面被缓存：

```xml
GET /mypage.html?willbecached=true
```

在上述 `ignoreUrlParams` 配置示例的上下文中，由于 `nocache` 参数不会被忽略，因此以下 HTTP 请求&#x200B;**不**&#x200B;会导致页面被缓存：

```xml
GET /mypage.html?nocache=true
GET /mypage.html?nocache=true&willbecached=true
```

有关 Glob 属性的信息，请参阅[为 Glob 属性设计模式](#designing-patterns-for-glob-properties)。

### 缓存 HTTP 请求标头 {#caching-http-response-headers}

>[!NOTE]
>
>Dispatcher **4.1.11** 版本提供此功能。

`/headers` 属性可让您定义 Dispatcher 要缓存的 HTTP 标头类型。在对未缓存资源的首个请求中，与配置的值之一匹配的所有标头（参见以下配置示例）存储在缓存文件旁的单独文件中。在对缓存的资源的后续请求中，存储的标头添加到响应。

以下是默认配置的示例：

```xml
/cache {
  ...
  /headers {
    "Cache-Control"
    "Content-Disposition"
    "Content-Type"
    "Expires"
    "Last-Modified"
    "X-Content-Type-Options"
    "Last-Modified"
  }
}
```

>[!NOTE]
>
>不允许使用文件通配字符。有关更多详细信息，请参阅[为 Glob 属性设计模式](#designing-patterns-for-glob-properties)。

>[!NOTE]
>
>如果您需要 Dispatcher 从 AEM 存储和传输 ETag 响应标头，请执行以下操作：
>
>* 在 `/cache/headers` 部分中添加标头名称。
>* 在与 Dispatcher 相关的部分中添加以下 [Apache 指令](https://httpd.apache.org/docs/2.4/mod/core.html#fileetag)：
>
>```xml
>FileETag none
>```

### Dispatcher 缓存文件权限 {#dispatcher-cache-file-permissions}

`mode` 属性用于指定缓存中新建的目录和文件应用哪些权限。调用进程的 `umask` 限制此设置。这是一个八进制数，由以下一个或多个值之和构成：

* `0400` 允许所有者读取。
* `0200` 允许所有者写入。
* `0100` 允许所有者在目录中搜索。
* `0040` 允许组成员读取。
* `0020` 允许组成员写入。
* `0010` 允许组成员在目录中搜索。
* `0004` 允许其他人读取。
* `0002` 允许其他人写入。
* `0001` 允许其他人在目录中搜索。

默认值为 `0755`，它允许所有者读取、写入或搜索以及组成员和其他人读取或搜索。

### 限制 .stat 文件的更新频率 {#throttling-stat-file-touching}

如果使用默认的 `/invalidate` 属性，每次激活都会使所有 `.html` 文件失效（前提是其路径与 `/invalidate` 部分匹配）。在流量很大的网站上，后续激活将加大后端的 CPU 负载。在此类场景中，最好“限制”接触 `.stat` 文件以保持网站可响应。可以使用 `/gracePeriod` 属性完成此操作。

`/gracePeriod` 属性定义在上次激活之后几秒内仍可从缓存提供某个陈旧、自动失效的资源。该属性可用于的设置情况是，如果不这样设置，一批激活将会反复地使整个缓存失效。推荐值为 2 秒。

有关更多详细信息，请参阅 `/invalidate` 和更早些的 `/statfileslevel`。

### 配置基于时间的缓存失效方法 - `/enableTTL` {#configuring-time-based-cache-invalidation-enablettl}

基于时间的缓存失效依靠 `/enableTTL` 属性以及存在来自 HTTP 标准的常规到期标头。如果将该属性设置为 1 (`/enableTTL "1"`)，则它从后端求响应标头的值。如果标头包含 `Cache-Control`、`max-age` 或 `Expires` 日期，则在缓存的文件旁创建一个辅助的空文件，其修改时间与到期日期相同。当在修改时间之后请求缓存的文件时，将自动从后端重新请求该文件。

在 Dispatcher 4.3.5 之前，TTL 失效逻辑仅基于所配置的 TTL 值。在 Dispatcher 4.3.5 中，将所设置的 TTL **和** Dispatcher 缓存失效规则都考虑在内。因此，对于缓存的文件：

1. 如果将 `/enableTTL` 设置为 1，则检查该文件是否过期。如果根据所设置的 TTL，该文件已到期，则不执行其他检查，并从后端重新请求缓存的文件。
2. 如果该文件未到期或未配置 `/enableTTL`，则应用标准的缓存失效规则，如 [`/statfileslevel`](#invalidating-files-by-folder-level) 和 [`/invalidate`](#automatically-invalidating-cached-files) 设置的规则。此流程意味着 Dispatcher 可使其 TTL 尚未到期的文件失效。

此新实施支持文件具有较长 TTL 的用例（例如，在 CDN 上）。但是，即使 TTL 没有过期，这些文件仍然可以失效。它更注重内容新鲜度而不是 Dispatcher 上的缓存命中率。

相反，如果&#x200B;**只**&#x200B;需将过期逻辑应用于某个文件，请将 `/enableTTL` 设置为 1，并从标准缓存失效机制中排除该文件。例如，您可以：

* 配置缓存部分中的[失效规则](#automatically-invalidating-cached-files)以忽略该文件。在下面的代码片段中，所有以 `.example.html` 结尾的文件均被忽略，并且只有所设置的 TTL 已过，这些文件才到期。

```xml
  /invalidate
  {
   /0000  { /glob "*" /type "deny" }
   /0001  { /glob "*.html" /type "allow" }
   /0002  { /glob "*.example.html" /type "deny" }
  }
```

* 将内容结构设计得让您可设置较大的 [`/statfilelevel`](#invalidating-files-by-folder-level)，以便不自动使该文件失效。

这样做确保不使用 `.stat` 文件失效，而只有 TTL 过期作用于指定的文件。

>[!NOTE]
>
>请记住，将 `/enableTTL` 设置为 1 将仅在 Dispatcher 端启用 TTL 缓存。因此，附加文件（见上文）中包含的 TTL 信息不会提供给任何其他从 Dispatcher 请求此类文件类型的用户代理。如果您想向下游系统（如 CDN 或浏览器）提供缓存标头，则应该相应地配置 `/cache/headers` 分区。

>[!NOTE]
>
>此功能在 Dispatcher 版本 **4.1.11** 或更高版本上可用。

## 配置负载平衡 - `/statistics` {#configuring-load-balancing-statistics}

`/statistics` 部分定义文件类别，Dispatcher 针对这样的文件为每次渲染的响应性打分。Dispatcher 使用分数来确定向哪个渲染发送请求。

您创建的每个类别定义一个 glob 模式。Dispatcher 将所请求内容的 URI 与这些模式进行比较，以确定所请求的内容的类别：

* 类别的顺序确定它们与 URI 进行比较的顺序。
* 与 URI 匹配的第一个类别模式是文件的类别。不评估其他类别模式。

Dispatcher 支持最多 8 个统计类别。如果您定义的类别超过了 8 个，则只会使用前 8 个。

**渲染选择**

Dispatcher 每次请求渲染的页面时，它使用以下算法来选择渲染：

1. 如果请求在 `renderid` cookie 中包含渲染名称，Dispatcher 使用该渲染。
1. 如果请求不包含 `renderid` cookie，Dispatcher 比较渲染统计数据：

   1. Dispatcher 确定请求 URI 的类别。
   1. Dispatcher 确定该类别中什么渲染具有最低的响应得分，并选择该渲染。

1. 如果尚未选择渲染，则使用列表中的第一个渲染。

渲染类别的分数基于以前的响应时间以及 Dispatcher 以前尝试连接失败和成功的次数。对于每次尝试，将更新所请求 URI 的类别的分数。

>[!NOTE]
>
>如果不使用负载平衡，您可以忽略此部分。

### 定义统计数据类别 {#defining-statistics-categories}

为要保留统计数据用于渲染选择的每个文档类型定义一个类别。`/statistics` 部分包含 `/categories` 部分。要定义类别，请在 `/categories` 部分下添加具有以下格式的行：

`/name { /glob "pattern"}`

类别 `name` 必须对场唯一。`pattern` 在[为 glob 属性设计模式](#designing-patterns-for-glob-properties)部分中有介绍。

为确定 URI 的类别，Dispatcher 将 URI 与各个类别模式比较，直至找到匹配。Dispatcher 从列表中的第一个类别开始，然后按顺序继续。因此，将具有更具体模式的类别放在最前。

例如，Dispatcher 的默认`dispatcher.any` 文件定义 HTML 类别和一个其他类别。HTML 类别更具体，因此显示在最前：

```xml
/statistics
  {
  /categories
    {
      /html { /glob "*.html" }
      /others  { /glob "*" }
    }
  }
```

以下示例还包括用于搜索页面的类别：

```xml
/statistics
  {
  /categories
    {
      /search { /glob "*search.html" }
      /html { /glob "*.html" }
      /others  { /glob "*" }
    }
  }
```

### 在 Dispatcher 统计数据中反映服务器不可用状态 {#reflecting-server-unavailability-in-dispatcher-statistics}

`/unavailablePenalty` 属性设置时间（以十分之一秒为单位），该时间在连接到渲染失败时应用到渲染统计数据。Dispatcher 将时间添加到与所请求 URI 匹配的统计数据类别。

例如，当与指定主机名/端口的 TCP/IP 连接无法建立时，就会受到惩罚。原因可能是 AEM 没有运行（也没有监听），或者存在与网络有关的问题。

`/unavailablePenalty` 属性是 `/farm` 部分（`/statistics` 部分的同级）的直接子级。

如果没有 `/unavailablePenalty` 属性，则使用值 `"1"`。

```xml
/unavailablePenalty "1"
```

## 标识粘性连接文件夹 - `/stickyConnectionsFor` {#identifying-a-sticky-connection-folder-stickyconnectionsfor}

`/stickyConnectionsFor` 属性定义了一个包含粘性文档的文件夹。使用 URL 访问此属性。Dispatcher 从此文件夹中的单个用户，将所有请求发送到相同的渲染实例。粘性连接确保会话数据存在并对所有文档一致。此机制使用 `renderid` Cookie。

以下示例定义与 /products 文件夹的粘性连接：

```xml
/stickyConnectionsFor "/products"
```

当某个页面由来自多个内容节点的内容组成时，请包括列出了指向内容的路径的 `/paths` 属性。例如，一个页面包含来自 `/content/image`、`/content/video` 和 `/var/files/pdfs` 的内容。以下配置为页面上的所有内容启用粘性连接：

```xml
/stickyConnections {
  /paths {
    "/content/image"
    "/content/video"
    "/var/files/pdfs"
  }
}
```

### `httpOnly` {#httponly}

启用粘性连接后，Dispatcher 模块将设置 `renderid` Cookie。此 Cookie 没有 `httponly` 标志，但应添加该标志以增强安全性。通过在 `dispatcher.any` 配置文件的 `/stickyConnections` 节点中设置 `httpOnly` 属性而添加 `httponly` 标志。该属性的值（`0` 或 `1`）定义 `renderid` Cookie 是否追加了 `HttpOnly` 属性。默认值为 `0`，这意味着不添加该属性。

有关 `httponly` 标记的更多信息，请阅读[此页面](https://owasp.org/www-community/HttpOnly)。

### `secure` {#secure}

启用粘性连接后，Dispatcher 模块将设置 `renderid` Cookie。此 Cookie 没有 `secure` 标志，但应添加该标志以增强安全性。通过在 `dispatcher.any` 配置文件的 `/stickyConnections` 节点中设置 `secure` 属性而添加 `secure` 标志。该属性的值（`0` 或 `1`）定义 `renderid` Cookie 是否追加了 `secure` 属性。默认值为 `0`，这意味着&#x200B;**如果**&#x200B;传入请求是安全的，则添加该属性。如果将该值设置为 `1`，则无论传入请求是否安全，都将添加 secure 标志。

## 处理渲染连接错误 {#handling-render-connection-errors}

配置当渲染服务器返回 500 错误或不可用时的 Dispatcher 行为。

### 指定一个健康检查页面 {#specifying-a-health-check-page}

使用 `/health_check` 属性指定在出现 500 状态码时用于检查的 URL。如果此页面也返回 500 状态代码，则将该实例视为不可用，并在重试之前将一个可配置的时间惩罚 (`/unavailablePenalty`) 应用于渲染。

```xml
/health_check
  {
  # Page gets contacted when an instance returns a 500
  /url "/health_check.html"
  }
```

### 指定页面重试延迟时间 {#specifying-the-page-retry-delay}

`/retryDelay` 属性用于设置 Dispatcher 在多次尝试与场渲染服务器连接时两次尝试之间的等待时间（单位为秒）。对于每一轮，Dispatcher 连接到渲染的最大尝试次数是场中的渲染数量。

如果没有明确定义 `/retryDelay`，Dispatcher 使用值 `"1"`。默认值适用于大多数情况。

```xml
/retryDelay "1"
```

### 配置重试次数 {#configuring-the-number-of-retries}

`/numberOfRetries` 属性设置 Dispatcher 对渲染执行的连接尝试的最大轮数。如果 Dispatcher 在尝试了这个次数之后无法成功连接到渲染，Dispatcher 返回失败的响应。

对于每一轮，Dispatcher 连接到渲染的最大尝试次数是场中的渲染数量。因此，Dispatcher 的最大连接尝试次数是 (`/numberOfRetries`) x（渲染数）。

如果没有明确定义该值，默认值为 `5`。

```xml
/numberOfRetries "5"
```

### 使用故障转移机制 {#using-the-failover-mechanism}

要在原始请求失败时将请求重新发送到渲染，请在 Dispatcher 场上启用故障转移机制。启用了故障转移时，Dispatcher 具有以下行为：

* 对渲染的请求返回 HTTP 状态 503 (UNAVAILABLE) 时，Dispatcher 将请求发送到不同的渲染。
* 对渲染的请求返回 HTTP 状态 50x（503 除外）时，Dispatcher 为针对 `health_check` 属性配置的页面发送请求。
   * 如果运行状况检查返回 500 (INTERNAL_SERVER_ERROR)，Dispatcher 将原始请求发送给不同的渲染。
   * 如果运行状况检查返回 HTTP 状态 200，则 Dispatcher 向客户端返回初始 HTTP 500 错误。

要启用故障转移，请向场（或网站）添加以下行：

```xml
/failover "1"
```

>[!NOTE]
>
>为重试包含正文的 HTTP 请求，Dispatcher 在假脱机实际内容之前发送 `Expect: 100-continue` 请求标头到渲染。随后，带有 CQSE 的 CQ 5.5 立即使用 100 (CONTINUE) 或错误代码应答。也支持其他 servlet 容器。

## 忽略中断错误 - `/ignoreEINTR` {#ignoring-interruption-errors-ignoreeintr}

>[!CAUTION]
>
>此选项并非必需。只有在看到以下日志消息时才使用此选项：
>
>`Error while reading response: Interrupted system call`

任何面向文件系统的系统调用，只要该系统调用的对象在通过 NFS 访问的远程系统上，即可用 `EINTR` 中断该系统调用。这些系统调用是否会超时或被中断，取决于本地机器上底层文件系统的挂载方式。

如果您的实例采用了此类配置，并且日志中包含以下消息，请使用 `/ignoreEINTR` 参数：

`Error while reading response: Interrupted system call`

Dispatcher 在内部使用可表示为以下形式的循环，从远程服务器（即 AEM）读取响应：

```text
while (response not finished) {  
read more data  
}
```

当 `EINTR` 出现在 `read more data` 部分时，就会生成此类消息。这个问题是由于在收到任何数据之前就接收到信号而引发的。

要忽略此类中断，可将以下参数添加到 `dispatcher.any`（在 `/farms` 之前）：

`/ignoreEINTR "1"`

将 `/ignoreEINTR` 设置为 `"1"` 会导致 Dispatcher 继续尝试读取数据，直至读取了完整的响应。默认值为 `0`，将停用选项。

## glob 属性的设计模式 {#designing-patterns-for-glob-properties}

Dispatcher 配置文件中的多个部分使用 `glob` 属性作为客户端请求的选择标准。`glob` 属性的值是 Dispatcher 与请求的某个方面进行比较的模式，例如所请求资源的路径或者客户端的 IP 地址。例如，`/filter` 部分中的项使用 `glob` 模式来确定 Dispatcher 所操作或拒绝的页面的路径。

`glob` 值可以包括通配符字符和字母数字字符来定义模式。

| 通配符 | 描述 | 示例 |
|--- |--- |--- |
| `*` | 匹配字符串中连续出现的任意零个或多个字符。以下任意情况决定匹配的最后一个字符：<br/>字符串中的字符匹配了模式中的下一个字符，并且模式字符具有以下特征：<br/><ul><li>不是 `*`</li><li>不是 `?`</li><li>一个文本字符（包括空格）或字符类。</li><li>达到模式的结尾。</li></ul>在字符类中，字符按字面解释。 | `*/geo*` 匹配 `/content/geometrixx` 节点和 `/content/geometrixx-outdoors` 节点下的任意页面。以下 HTTP 请求与 glob 模式匹配：<br/><ul><li>`"GET /content/geometrixx/en.html"`</li><li>`"GET /content/geometrixx-outdoors/en.html"` </li></ul><br/> `*outdoors/*` <br/>匹配 `/content/geometrixx-outdoors` 节点下的任意页面。例如，以下 HTTP 请求与 glob 模式匹配：<br/><ul><li>`"GET /content/geometrixx-outdoors/en.html"`</li></ul> |
| `?` | 匹配任意单个字符。使用外部字符类。在字符类中，此字符按字面解释。 | `*outdoors/??/*`<br/> 匹配 geometrixx-outdoors 网站中任意语言的页面。例如，以下 HTTP 请求与 glob 模式匹配：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>以下请求不与 glob 模式匹配：<br/><ul><li>&quot;GET /content/geometrixx-outdoors/en.html&quot;</li></ul> |
| `[ and ]` | 取消字符类的开始和结尾的标记。字符类可以包括一个或多个字符范围以及单个字符。<br/>如果目标字符与字符类中的任意字符匹配或者在定义的范围内，则出现匹配。<br/>如果不包含右括号，则模式不会生成匹配。 | `*[o]men.html*`<br/> 匹配以下 HTTP 请求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/> 不匹配以下 HTTP 请求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/> `*[o/]men.html*` <br/>匹配以下 HTTP 请求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `-` | 它表示一系列字符。用于字符类中。在字符类之外，此字符按字面解释。 | `*[m-p]men.html*` 匹配以下 HTTP 请求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul>它不匹配以下 HTTP 请求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `!` | 使后续的字符或字符类无效。仅用于否定字符类中的字符和字符范围。等同于 `^ wildcard`。<br/>在字符类之外，此字符按字面解释。 | `*[!o]men.html*`<br/> 匹配以下 HTTP 请求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/> 不匹配以下 HTTP 请求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>`*[!o!/]men.html*`<br/> 不匹配以下 HTTP 请求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"` 或 `"GET /content/geometrixx-outdoors/en/men. html"`</li></ul> |
| `^` | 使后续的字符或字符范围无效。仅用于否定字符类中的字符和字符范围。等同于 `!` 通配符。<br/>在字符类之外，此字符按字面解释。 | 应用 `!` 通配符的示例，将示例模式中的 `!` 字符替换为 `^` 字符。 |


<!--- need to troubleshoot table

The following table describes the wildcard characters.

<table border="1" cellpadding="1" cellspacing="0" width="100%"> 
 <tbody> 
  <tr> 
   <th>Wildcard character</th> 
   <th>Description</th> 
   <th>Examples</th> 
  </tr> 
  <tr> 
   <td>*</td> 
   <td><p>Matches zero or more contiguous instances of any character in the string. The final character of the match is determined by either of the following situations:</p> 
    <ul> 
     <li>A character in the string matches the next character in the pattern, and the pattern character has the following characteristics: 
      <ul> 
       <li>Not a *</li> 
       <li>Not a ?</li> 
       <li>A literal character (including a space) or a character class.</li> 
      </ul> </li> 
     <li>The end of the pattern is reached.</li> 
    </ul> <p>Inside a character class, the character is interpreted literally.</p> </td> 
   <td><p>*/geo*</p> <p>Matches any page below the /content/geometrixx node and the /content/geometrixx-outdoors node. The following HTTP requests match the glob pattern:</p> 
    <ul> 
     <li>"GET /content/geometrixx/en.html"</li> 
     <li>"GET /content/geometrixx-outdoors/en.html" </li> 
    </ul> <p>*outdoors/*</p> <p>Matches any page below the /content/geometrixx-outdoors node. For example, the following HTTP request matches the glob pattern:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en.html"</li> 
    </ul> </td> 
  </tr> 
  <tr> 
   <td>?</td> 
   <td><p>Matches any single character. Use outside character classes.</p> <p>Inside a character class, this character is interpreted literally.</p> </td> 
   <td><p>*outdoors/??/*</p> <p>Matches the pages for any language in the geometrixx-outdoors site. For example, the following HTTP request matches the glob pattern:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/men.html"</li> 
    </ul> <p>The following request does not match the glob pattern:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en.html" </li> 
    </ul> </td> 
  </tr> 
  <tr> 
   <td>[ and ]</td> 
   <td><p>Demarks the beginning and end of a character class.</p> <p>Character classes can include one or more character ranges and single characters.</p> <p>A match occurs if the target character matches any of the characters in the character class, or within a defined range.</p> <p>If the closing bracket is not included, the pattern produces no matches.</p> <p></p> <p></p> <p></p> </td> 
   <td><p>*[o]men.html*</p> <p>Matches the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html"</li> 
    </ul> <p>Does not match the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/men.html" </li> 
    </ul> <p>*[o/]men.html*</p> <p>Matches the following HTTP requests: </p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html" </li> 
     <li> "GET /content/geometrixx-outdoors/en/men.html" </li> 
    </ul> <p></p> </td> 
  </tr> 
  <tr> 
   <td>-</td> 
   <td><p>Denotes a range of characters. For use in character classes.<br /> </p> <p>Outside of a character class, this character is interpreted literally.</p> <p></p> </td> 
   <td><p>*[m-p]men.html*</p> <p>Matches the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html"</li> 
    </ul> <p>Does not match the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/men.html"</li> 
    </ul> <p></p> </td> 
  </tr> 
  <tr> 
   <td>!</td> 
   <td><p>Negates the character or character class that follows. Use only for negating characters and character ranges inside character classes. Equivalent to the ^ wildcard.</p> <p>Outside of a character class, this character is interpreted literally.</p> <p></p> </td> 
   <td><p>*[!o]men.html*</p> <p>Matches the following HTTP request: </p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/men.html"</li> 
    </ul> <p>Does not match the following HTTP request</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html" </li> 
    </ul> <p>*[!o!/]men.html*</p> <p>Does not match the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html" or "GET /content/geometrixx-outdoors/en/men. html" </li> 
    </ul> </td> 
  </tr> 
  <tr> 
   <td>^</td> 
   <td><p>Negates the character or character range that follows. Use for negating only characters and character ranges inside character classes. Equivalent to the ! wildcard character.</p> <p>Outside of a character class, this charcter is interpreted literally.</p> </td> 
   <td>The examples for the ! wildcard character apply, substituting the ! characters in the example patterns with ^ characters.</td> 
  </tr> 
 </tbody> 
</table>
-->

## 日志 {#logging}

在 Web 服务器配置中，您可以设置：

* Dispatcher 日志文件的位置。
* 日志级别。

有关更多信息，请参阅 Web 服务器文档和 Dispatcher 实例的自述文件。

**Apache 轮转日志或管道日志**

如果使用 **Apache** Web Server，则可使用日志轮换和/或管道传输日志的标准功能。例如，使用管道传输日志：

`DispatcherLog "| /usr/apache/bin/rotatelogs logs/dispatcher.log%Y%m%d 604800"`

此功能自动轮换：

* Dispatcher 日志文件，在扩展名中带有时间戳 (`logs/dispatcher.log%Y%m%d`)。
* 每周（60 x 60 x 24 x 7 = 604800 秒）轮换一次。

请参阅关于日志轮换和管道传输日志的 Apache Web Server 文档。例如，[Apache 2.4](https://httpd.apache.org/docs/2.4/logs.html)。

>[!NOTE]
>
>安装后，默认日志级别为高（即，级别 3 = 调试），因此 Dispatcher 记录所有错误和警告。此级别在初始阶段有用。
>
>但是，这样的级别需要更多的资源。当 Dispatcher *按您的需求正常运行*&#x200B;时，您可以适当降低日志级别。

### 追踪日志记录 {#trace-logging}

除了 Dispatcher 的其他增强之外，4.2.0 版还引入了跟踪日志记录。

此功能比在日志中显示附加信息的调试日志记录高一级。它添加以下日志记录：

* 转发的标头的值；
* 应用于某个特定操作的规则。

您可通过在 Web 服务器上将日志级别设置为 `4` 来启用跟踪日志记录。

以下是启用了跟踪的日志示例：

```xml
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[Host] = "localhost:8443"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[User-Agent] = "curl/7.43.0"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[Accept] = "*/*"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-SSL-Client-Cert] = "(null)"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[Via] = "1.1 localhost:8443 (dispatcher)"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-For] = "::1"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-SSL] = "on"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-SSL-Cipher] = "DHE-RSA-AES256-SHA"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-SSL-Session-ID] = "ba931f5e4925c2dde572d766fdd436375e15a0fd24577b91f4a4d51232a934ae"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-Port] = "8443"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[Server-Agent] = "Communique-Dispatcher"
```

在请求了与阻止规则匹配的文件时将记录事件：

```xml
[Thu Mar 03 14:42:45 2016] [T] [11831] 'GET /content.infinity.json HTTP/1.1' was blocked because of /0082
```

## 确认基本操作 {#confirming-basic-operation}

要确认基本操作和与 Web 服务器、Dispatcher 以及 AEM 实例的交互，您可以使用以下步骤：

1. 将 `loglevel` 设置为 `3`。

1. 启动 Web 服务器。这样做还将启动 Dispatcher。
1. 启动 AEM 实例。
1. 查看 Web 服务器和 Dispatcher 日志和错误文件。
   * 根据您的 Web 服务器，您应看到类似于下文的消息：
      * `[Thu May 30 05:16:36 2002] [notice] Apache/2.0.50 (Unix) configured` 和
      * `[Fri Jan 19 17:22:16 2001] [I] [19096] Dispatcher initialized (build XXXX)`

1. 通过 Web 服务器浏览网站。确认按要求显示了内容。\
   例如，在本地安装上，如果 AEM 运行在端口 `4502` 并且 Web 服务器运行在端口 `80`，则可以同时使用这两个端口访问网站控制台：
   * `https://localhost:4502/libs/wcm/core/content/siteadmin.html`
   * `https://localhost:80/libs/wcm/core/content/siteadmin.html`
   * 结果应相同。确认使用相同的机制访问其他页面。

1. 检查是否在填充缓存目录。
1. 要检查是否正确刷新缓存，请激活某个页面。
1. 如果一切工作，可将 `loglevel` 降至 `0`。

## 使用多个 Dispatcher {#using-multiple-dispatchers}

在复杂设置中，您可以使用多个 Dispatcher。例如，您可以使用：

* 一个 Dispatcher 用于在内联网上发布网站
* 第二个 Dispatcher，通过不同的地址和不同的安全设置，在内联网上发布相同的内容。

在这种情况下，请确保每个请求只通过一个 Dispatcher。一个 Dispatcher 不能处理来自另一个 Dispatcher 的请求。因此，请确保两个 Dispatcher 都能直接访问 AEM 网站。

## 调试 {#debugging}

将标头 `X-Dispatcher-Info` 添加到请求时，Dispatcher 应答是缓存了目标、从缓存返回还是完全不可缓存。响应标头 `X-Cache-Info` 以可读格式包含此信息。您可以使用响应标头调试涉及由 Dispatcher 缓存的响应的问题。

默认情况下不启用此功能，因此要包括响应标头 `X-Cache-Info`，该场必须包含以下条目：

```xml
/info "1"
```

例如，

```xml
/farm
{
    /mywebsite
    {
        # Include X-Cache-Info response header if X-Dispatcher-Info is in request header
        /info "1"
    }
}
```

此外，`X-Dispatcher-Info` 标头不需要值，但是，如果使用 `curl` 进行测试，则必须提供值以发送到该标头，例如：

```xml
curl -v -H "X-Dispatcher-Info: true" https://localhost/content/wknd/us/en.html
```

以下是包含 `X-Dispatcher-Info` 返回的响应标头的列表：

* **目标文件缓存**\
  目标文件包含在缓存中，Dispatcher 已确定可有效地传送它。
* **caching**\
  目标文件未包含在缓存中，Dispatcher 已确定可有效地缓存输出并传送它。
* **缓存：stat 文件较新**
目标文件包含在缓存中。不过，最新的 stat 文件可以使其失效。Dispatcher 删除目标文件，从输出重新创建它并传送它。
* **不可缓存：文档根不存在**
场的配置不包含文档根（配置元素 `cache.docroot`）。
* **not cacheable: cache file path too long**\
  目标文件（文档根与 URL 文件的连接）超过了系统上允许的最长文件名。
* **not cacheable: temporary file path too long**\
  临时文件名模板超过了系统上允许的最长文件名。Dispatcher 首先创建一个临时文件，然后再真正地创建或覆盖缓存的文件。该临时文件名是目标文件名追加 `_YYYYXXXXXX` 字符，其中替换 `Y` 和 `X` 以创建唯一名称。
* **不可缓存：请求 URL 缺少扩展名**\
  请求的 URL 没有扩展名，或者文件扩展名后跟路径，例如：`/test.html/a/path`。
* **不可缓存：请求必须是 GET 或 HEAD**
HTTP 方法既不是 GET，也不是 HEAD。Dispatcher 假定输出包含不应缓存的动态数据。
* **not cacheable: request contained a query string**\
  请求包含查询字符串。Dispatcher 假定输出取决于给出的查询字符串，并因此不缓存。
* **不可缓存：会话管理器需要验证**\
  会话管理器控制（该配置包含一个 `sessionmanagement` 节点）场的缓存，请求不包含相应的身份验证信息。
* **not cacheable: request contains authorization**\
  场不允许缓存输出 (`allowAuthorized 0`) 并且请求包含身份验证信息。
* **not cacheable: target is a directory**\
  目标文件是目录。此位置可能会指向某种概念性的错误，其中 URL 和某些子 URL 都包含可缓存的输出。例如，如果对 `/test.html/a/file.ext` 的请求排在第一位，并且包含可缓存的输出，则 Dispatcher 无法缓存对 `/test.html` 的后续请求的输出。
* **not cacheable: request URL has a trailing slash**\
  请求 URL 有结尾斜杠。
* **不可缓存：缓存规则中缺少请求 URL**\
  场的缓存规则明确拒绝了缓存某些请求 URL 的输出。
* **not cacheable: authorization checker denied access**\
  场的授权检查程序拒绝访问缓存的文件。
* **不可缓存：会话无效**
会话管理器控制（配置包含一个 `sessionmanagement` 节点）场的缓存，而且用户的会话无效或不再有效。
* **not cacheable: response contains`no_cache`**
远程服务器返回了一个 `Dispatcher: no_cache` 标头，表示禁止 Dispatcher 缓存输出。
* **not cacheable: response content length is zero**
响应的内容长度为零，Dispatcher 不创建长度为零的文件。

