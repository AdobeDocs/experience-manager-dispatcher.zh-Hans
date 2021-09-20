---
title: 缓存受保护内容
seo-title: Caching Secured Content in AEM Dispatcher
description: 了解权限敏感缓存在 Dispatcher 中的工作方式。
seo-description: Learn how permission-sensitive caching works in AEM Dispatcher.
uuid: abfed68a-2efe-45f6-bdf7-2284931629d6
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: 4f9b2bc8-a309-47bc-b70d-a1c0da78d464
exl-id: 3d8d8204-7e0d-44ad-b41b-6fec2689c6a6
source-git-commit: 3a0e237278079a3885e527d7f86989f8ac91e09d
workflow-type: ht
source-wordcount: '748'
ht-degree: 100%

---

# 缓存受保护内容 {#caching-secured-content}

利用权限敏感型缓存，您可以缓存受保护页面。Dispatcher 会先检查用户对页面的访问权限，然后再传递缓存页面。

Dispatcher 包含实现权限敏感型缓存的 AuthChecker 模块。在激活此模块后，渲染器会调用 AEM servlet 以对请求的内容执行用户身份验证和授权。servlet 响应将确定是否将内容传送到 Web 浏览器。

由于身份验证方法和授权方法特定于 AEM 部署，因此您需要创建 servlet。

>[!NOTE]
>
>使用 `deny` 过滤器可强制实施所有安全限制。对配置为允许访问一部分用户或组的页面使用权限敏感型缓存。

下图说明了在 Web 浏览器请求权限敏感型缓存将用于的页面时发生的事件的顺序。

## 对页面进行缓存并向用户授权 {#page-is-cached-and-user-is-authorized}

![](assets/chlimage_1.png)

1. Dispatcher 确定请求的内容已缓存且有效。
1. Dispatcher 向渲染器发送请求消息。HEAD 部分包括来自浏览器请求的所有标头行。
1. 渲染器调用授权程序以执行安全检查并响应 Dispatcher。响应消息包含 HTTP 状态代码 200 以表示用户已获得授权。
1. Dispatcher 向浏览器发送一条响应消息，其中包含渲染器响应中的标头行和正文中的缓存内容。

## 不对页面进行缓存，但向用户授权 {#page-is-not-cached-and-user-is-authorized}

![](assets/chlimage_1-1.png)

1. Dispatcher 确定内容未缓存或需要更新。
1. Dispatcher 将原始请求转发到渲染器。
1. 渲染器调用授权程序 servlet 以执行安全检查。当用户获得授权时，渲染器将在响应消息的正文中包含渲染的页面。
1. Dispatcher 将响应转发到浏览器。Dispatcher 将渲染器的响应消息正文添加到缓存。

## 未向用户授权 {#user-is-not-authorized}

![](assets/chlimage_1-2.png)

1. Dispatcher 检查缓存。
1. Dispatcher 向渲染器发送一条请求消息，其中包含浏览器请求中的所有标头行。
1. 渲染器调用授权程序 servlet 以执行安全检查，如果未通过，渲染器会将原始请求转发到 Dispatcher。

## 实施权限敏感型缓存 {#implementing-permission-sensitive-caching}

要实施权限敏感型缓存，请执行以下任务：

* 开发一个执行身份验证和授权的 servlet
* 配置 Dispatcher

>[!NOTE]
>
>通常，安全资源存储在与非安全文件不同的文件夹中。例如，/content/secure/

## 创建授权 servlet {#create-the-authorization-servlet}

创建并部署一个 servlet 来对请求 Web 内容的用户执行身份验证和授权。此 servlet 可以使用任何身份验证方法和授权方法，例如 AEM 用户帐户和存储库 ACL，或 LDAP 查找服务。您将 servlet 部署到 Dispatcher 用作渲染器的 AEM 实例。

此 servlet 必须可供所有用户访问。因此，您的 servlet 应扩展 `org.apache.sling.api.servlets.SlingSafeMethodsServlet` 类，此类提供对系统的只读访问权限。

servlet 仅接收来自渲染器的 HEAD 请求，因此您只需实施 `doHead` 方法。

渲染器包含请求的资源的 URI 作为 HTTP 请求的参数。例如，通过 `/bin/permissioncheck` 访问授权 servlet。要对 /content/geometrixx-outdoors/en.html 页面执行安全检查，渲染器在 HTTP 请求中包含以下 URL：

`/bin/permissioncheck?uri=/content/geometrixx-outdoors/en.html`

servlet 响应消息必须包含以下 HTTP 状态代码：

* 200：已通过身份验证和授权。

以下示例 servlet 从 HTTP 请求中获取所请求资源的 URL。代码使用 Felix SCR `Property` 注释以将 `sling.servlet.paths` 属性的值设置为 /bin/permissioncheck。在 `doHead` 方法中，servlet 获取会话对象并使用 `checkPermission` 方法确定适当的响应代码。

>[!NOTE]
>
>必须在 Sling Servlet Resolver (org.apache.sling.servlets.resolver.SlingServletResolver) 服务中启用 sling.servlet.paths 属性的值。

### 示例 servlet {#example-servlet}

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

import javax.jcr.Session;

@Component(metatype=false)
@Service
public class AuthcheckerServlet extends SlingSafeMethodsServlet {
 
    @Property(value="/bin/permissioncheck")
    static final String SERVLET_PATH="sling.servlet.paths";
    
    private Logger logger = LoggerFactory.getLogger(this.getClass());
    
    public void doHead(SlingHttpServletRequest request, SlingHttpServletResponse response) {
     try{ 
      //retrieve the requested URL
      String uri = request.getParameter("uri");
      //obtain the session from the request
      Session session = request.getResourceResolver().adaptTo(javax.jcr.Session.class);     
      //perform the permissions check
      try {
       session.checkPermission(uri, Session.ACTION_READ);
       logger.info("authchecker says OK");
       response.setStatus(SlingHttpServletResponse.SC_OK);
      } catch(Exception e) {
       logger.info("authchecker says READ access DENIED!");
       response.setStatus(SlingHttpServletResponse.SC_FORBIDDEN);
      }
     }catch(Exception e){
      logger.error("authchecker servlet exception: " + e.getMessage());
     }
    }
}
```

## 配置 Dispatcher 以进行权限敏感型缓存 {#configure-dispatcher-for-permission-sensitive-caching}

dispatcher.any 文件的 auth_checker 部分控制权限敏感型缓存的行为。auth_checker 部分包含以下子部分：

* `url`：执行安全检查的 servlet 的 `sling.servlet.paths` 属性值。

* `filter`：指定将权限敏感型缓存应用于的文件夹的过滤器。通常，`deny` 过滤器将应用于所有文件夹，`allow` 过滤器将应用于安全文件夹。

* `headers`：指定授权 servlet 包含在响应中的 HTTP 标头。

在 Dispatcher 启动时，Dispatcher 日志文件包含以下调试级别消息：

`AuthChecker: initialized with URL 'configured_url'.`

以下示例 auth_checker 部分将 Dispatcher 配置为使用上一个主题的 servlet。过滤器部分可促使仅对安全 HTML 资源执行权限检查。

### 示例配置 {#example-configuration}

```xml
/auth_checker
  {
  # request is sent to this URL with '?uri=<page>' appended
  /url "/bin/permissioncheck"
      
  # only the requested pages matching the filter section below are checked,
  # all other pages get delivered unchecked
  /filter
    {
    /0000
      {
      /glob "*"
      /type "deny"
      }
    /0001
      {
      /glob "/content/secure/*.html"
      /type "allow"
      }
    }
  # any header line returned from the auth_checker's HEAD request matching
  # the section below will be returned as well
  /headers
    {
    /0000
      {
      /glob "*"
      /type "deny"
      }
    /0001
      {
      /glob "Set-Cookie:*"
      /type "allow"
      }
    }
  }
```
