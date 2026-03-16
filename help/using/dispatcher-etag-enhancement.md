---
title: 用于CDN重新验证的Dispatcher ETag增强功能
description: AEM as a Cloud Service中INTERNAL_AEM_DISPATCHER_ETAG_ENHANCEMENT的可用性、支持状态和行为。
source-git-commit: ac0fafd060643903735ff565072ef2c5bee970be
workflow-type: tm+mt
source-wordcount: '308'
ht-degree: 0%

---

# 用于CDN重新验证的Dispatcher ETag增强功能

## 概述

通过`INTERNAL_AEM_DISPATCHER_ETAG_ENHANCEMENT`标志，Dispatcher可以评估缓存命中数的`If-None-Match`请求标头。 当传入`If-None-Match`值与缓存的`ETag`匹配时，Dispatcher可以返回`304 Not Modified`而不是`200 OK`。

此行为旨在减少CDN和Dispatcher之间不必要的负载传输，并提高条件缓存效率。

## 可用性

- Dispatcher版本： `2.0.264`
- AEM SDK内部版本：`aem-sdk-2026.2.24464.20260214T050318Z-260100`

## AEM as a Cloud Service支持

在AEM as a Cloud Service中，此功能受支持以供客户使用。

客户可以通过在Cloud Manager中设置`INTERNAL_AEM_DISPATCHER_ETAG_ENHANCEMENT`环境变量来启用它。 Adobe还可以在需要时代表客户启用该功能。

启用后，如果CDN发送`If-None-Match`，并且Dispatcher缓存中存在相关的`ETag`，则CDN和Dispatcher之间应具有更高的`304`响应率。 这一增长是预期的结果。

## 配置示例（缓存ETag标头）

为使此增强功能生效，请确保Dispatcher缓存`ETag`响应标头，并且您的Web服务器已配置为避免生成基于文件系统的ETags。

示例`dispatcher.any`缓存部分：

```text
/cache {
  /headers {
    "Cache-Control"
    "Content-Type"
    "Expires"
    "Last-Modified"
    "ETag"
  }
}
```

Dispatcher vhost上下文中的Apache指令示例：

```apache
FileETag none
```

有关基线标头缓存的指导，请参阅[缓存HTTP响应标头](dispatcher-configuration.md#caching-http-response-headers)。

## 验证示例

启用环境变量并部署配置更改后：

1. 请求一次以预热缓存并捕获返回的`ETag`。
1. 使用`If-None-Match: <etag-value>`再次请求。
1. 确认Dispatcher返回缓存命中重新验证流的`304 Not Modified`。

## 公共引用（相关行为）

有关Dispatcher中标头缓存和`ETag`处理的面向客户的基准指南，请参阅：

- [配置Dispatcher — 缓存HTTP响应标头](https://experienceleague.adobe.com/en/docs/experience-manager-dispatcher/using/configuring/dispatcher-configuration#caching-http-response-headers)

“此功能在Dispatcher `2.0.264` (AEM SDK `2026.2.24464`)中可用。 启用后，Dispatcher可以根据缓存的`ETag`值验证`If-None-Match`，并在缓存命中时返回`304 Not Modified`。 在AEM as a Cloud Service中，这是受支持的，可以通过Cloud Manager环境配置启用它。”
