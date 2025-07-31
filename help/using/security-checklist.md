---
title: Dispatcher 安全检查清单
description: 了解在投入生产环境之前应完成的 Dispatcher 安全检查清单。
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
jcr-lastmodifiedby: remove-legacypath-6-1
index: y
internal: n
snippet: y
exl-id: 49009810-b5bf-41fd-b544-19dd0c06b013
source-git-commit: c41b4026a64f9c90318e12de5397eb4c116056d9
workflow-type: ht
source-wordcount: '582'
ht-degree: 100%

---

# Dispatcher 安全检查清单{#the-dispatcher-security-checklist}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-05T05:14:35.365-0400

<p>Food for thought listed on <a href="https://jira.corp.adobe.com/browse/DOC-5649">DOC-5649</a>. To be considered while proof-reading.</p> 
<p> </p>

 -->

Adobe 建议您在开始生产前完成以下核对清单。

>[!CAUTION]
>
>在上线之前完成 AEM 版本的安全核对清单。请参阅相应的 [Adobe Experience Manager 文档](https://experienceleague.adobe.com/zh-hans/docs/experience-manager-65/content/security/security-checklist)。

## 使用最新版本的 Dispatcher {#use-the-latest-version-of-dispatcher}

安装适用于您平台的最新可用版本。升级您的 Dispatcher 实例以使用最新版本以利用产品和安全增强功能。 请参阅[安装 Dispatcher](dispatcher-install.md)。

>[!NOTE]
>
>您可以通过查看 Dispatcher 日志文件来查看 Dispatcher 安装的当前版本。
>
>`[Thu Apr 30 17:30:49 2015] [I] [23171(140735307338496)] Dispatcher initialized (build 4.1.9)`
>
>要查找日志文件，请检查 `httpd.conf` 中的 Dispatcher 配置。

## 限制可以刷新缓存的客户端 {#restrict-clients-that-can-flush-your-cache}

Adobe 建议您[限制可以刷新缓存的客户端。](dispatcher-configuration.md#limiting-the-clients-that-can-flush-the-cache)

## 为传输层安全性启用 HTTPS {#enable-https-for-transport-layer-security}

Adobe 建议在创作实例和发布实例上启用 HTTPS 传输层。

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-26T04:41:28.841-0400

<p>Recommended to have SSL termination, front end SSL.</p> 
<p>Question is do we want to have SSL communication between dispatcher and AEM instances (publish and/or author).</p> 
<p>We might want to have two items:</p> 
<ul> 
 <li>MUST HTTPS clients -&gt; dispatcher / load balancer</li> 
 <li>NICE load balancer -&gt; dispatcher<br /> </li> 
 <li>NICE dispatcher -&gt; instances if sensitive information such as credit cards / or infrastructure requirements such as DMZ</li> 
</ul>

 -->

## 限制访问权限 {#restrict-access}

在配置 Dispatcher 时应尽可能多地限制外部访问。请参阅 Dispatcher 文档中的[示例/过滤器部分](dispatcher-configuration.md#main-pars_184_1_title)。

## 确保拒绝对管理类 URL 的访问权限 {#make-sure-access-to-administrative-urls-is-denied}

请务必使用过滤器阻止外部访问所有管理类 URL，例如网页控制台。

有关必须阻止的 URL 的列表，请参阅[测试 Dispatcher 安全性](dispatcher-configuration.md#testing-dispatcher-security)。

## 使用允许列表而非阻止列表 {#use-allowlists-instead-of-blocklists}

允许列表是一种更好的访问控制方式，因为它默认拒绝所有访问请求，除非该请求已被明确列入允许列表。此模型可以对在特定配置阶段可能尚未审查或考虑的新请求进行更严格的控制。

## 以专用系统用户身份运行 Dispatcher {#run-dispatcher-with-a-dedicated-system-user}

配置 Dispatcher，确保通过一个专用的、权限最小化的用户帐户运行 Web 服务器。Adobe 建议仅授予对 Dispatcher 缓存文件夹的写入权限。

此外，IIS 用户必须按如下方式配置其网站：

1. 在网站的物理路径设置中，选择&#x200B;**以特定用户身份连接**。
1. 设置用户。

## 防御拒绝服务 (DoS) 攻击 {#prevent-denial-of-service-dos-attacks}

拒绝服务（DoS）攻击是一种试图使计算机资源无法供其预期用户使用的攻击行为。

在 Dispatcher 级别，可通过两种配置方法来防御 DoS 攻击：[过滤器](https://experienceleague.adobe.com/zh-hans/docs#/filter)

* 使用 mod_rewrite 模块（例如 [Apache 2.4](https://httpd.apache.org/docs/2.4/mod/mod_rewrite.html)）执行 URL 验证（如果 URL 模式规则不是太复杂）。

* 通过使用[过滤器](dispatcher-configuration.md#configuring-access-to-content-filter)防止 Dispatcher 缓存带假扩展的 URL。\
  例如，更改缓存规则以仅允许缓存预期的 MIME 类型，例如：

   * `.html`
   * `.jpg`
   * `.gif`
   * `.swf`
   * `.js`
   * `.doc`
   * `.pdf`
   * `.ppt`

  [限制外部访问](#restrict-access)的配置文件示例。它包括对 MIME 类型的限制。

要在发布实例上启用全部功能，请配置过滤器以阻止对以下节点的访问：

* `/etc/`
* `/libs/`

然后，配置过滤器以允许对以下节点路径的访问：

* `/etc/designs/*`
* `/etc/clientlibs/*`
* `/etc/segmentation.segment.js`
* `/libs/cq/personalization/components/clickstreamcloud/content/config.json`
* `/libs/wcm/stats/tracker.js`
* `/libs/cq/personalization/*`（JS、CSS 和 JSON）
* `/libs/cq/security/userinfo.json`（CQ 用户信息）
* `/libs/granite/security/currentuser.json`（**不得缓存数据**）

* `/libs/cq/i18n/*`（国际化）

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-26T04:38:17.016-0400

<p>We need to highlight whether a path applies to all versions or specific ones.<br /> </p>

 -->

## 配置 Dispatcher 以防御 CSRF 攻击 {#configure-dispatcher-to-prevent-csrf-attacks}

AEM 提供了一个[框架](https://experienceleague.adobe.com/zh-hans/docs/experience-manager-release-information/aem-release-updates/previous-updates/aem-previous-versions#verification-steps)，用于防止跨站点请求伪造攻击。要正确使用该框架，请执行以下操作，在 Dispatcher 中允许支持 CSRF 令牌：

1. 创建过滤器以允许 `/libs/granite/csrf/token.json` 路径；
1. 将 `CSRF-Token` 标头添加到 Dispatcher 配置的 `clientheaders` 部分。

## 防御点击劫持攻击 {#prevent-clickjacking}

若要防御点击劫持攻击，Adobe 建议您将 Web 服务器配置为将 `X-FRAME-OPTIONS` HTTP 标头集提供给 `SAMEORIGIN`。

有关点击劫持攻击的更多信息，请参阅 [OWASP 网站](https://owasp.org/www-community/attacks/Clickjacking)。

## 执行渗透测试 {#perform-a-penetration-test}

Adobe 建议在部署到生产环境之前，对您的 AEM 基础架构执行一次渗透测试。

