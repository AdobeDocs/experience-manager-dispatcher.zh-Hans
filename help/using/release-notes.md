---
title: AEM Dispatcher 发行说明
description: 特定于Adobe Experience Manager调度程序的发行说明。
topic-tags: release-notes
content-type: reference
products: SG_EXPERIENCEMANAGER/6.4
exl-id: b55c7a34-d57b-4d45-bd83-29890f1524de
source-git-commit: 2d90738d01fef6e37a2c25784ed4d1338c037c23
workflow-type: tm+mt
source-wordcount: '1048'
ht-degree: 53%

---

# AEM Dispatcher 发行说明{#aem-dispatcher-release-notes}

## 版本信息 {#release-information}

|  |  |
|--- |--- |
| 产品 | Adobe Experience Manager (AEM) Dispatcher |
| 版本 | 4.3.7 |
| 类型 | 次要版本 |
| 日期 | 2024 年 3 月 27 日 |
| 下载 URL | <ul><li>[Apache 2.4](#apache)</li><li>[Microsoft® Internet Information Services (IIS)](#iis)</li></ul> |
| 兼容性 | AEM 6.1 或更高版本 |

## 系统要求和先决条件 {#system-requirements-and-prerequisites}

请参阅 [支持的平台](https://experienceleague.adobe.com/en/docs/experience-manager-64/deploying/introduction/technical-requirements) 有关要求和先决条件的更多信息。

Adobe建议使用最新版本的AEM Dispatcher以从最新功能、最新错误修复和最佳性能中受益。

## 安装说明 {#installation-instructions}

有关详细说明，请参阅[安装 Dispatcher](dispatcher-install.md)。

## 版本历史记录 {#release-history}

### 4.3.7 版（2024 年 3 月 27 日） {#march}

**改进功能**：

* DISP-1009 — 再次设置标头长度。
* DISP-1013 — 添加对Linux®的openssl 3.0支持。
* DISP-1014 - response.location处理导致无效重定向。
* disp-1017 — 更改DTD定义。

### 4.3.6 版本（2023 年 7 月 25 日） {#jyly}

**改进功能**：

* DISP-911 AEM-05 - X-Edge-Key可能会在disp_apache2.c中泄漏。
* DISP-937记录所有选择器。
* DISP-998使启动时加载虚名url成为可配置状态。

### 版本 4.3.5（2022 年 4 月 04 日） {#apr}

**改进功能**：

* DISP-954 - 在未过期的情况下支持失效
* DISP-949 — 即使过滤器阻止了POST请求，Dispatcher也会返回200，而不是404。

### 版本 4.3.4（2021 年 11 月 29 日） {#nov}

**错误修复**：

* DISP-833 - X-Forwarded-Host标头可能包含以逗号分隔的主机名列表。
* DISP-835 - DispatcherUseForwardedHost忽略最后出现的主机标头。

**改进功能**：

* DISP-874 — 创建Dispatcher配置以通过标志打开或关闭DISP-818的实施 `DispatcherRestrictUncacheableContent`. 默认值为“禁用”。当设置为“启用”时，系统会删除 mod expires 为不可缓存内容设置的任何缓存标头。这与 4.3.3 版本中的行为（默认值为“启用”）不同，但与 4.3.3 之前的版本（默认值为“禁用”）相同。推荐的方法是保留 `DispatcherRestrictUncacheableContent` 的“禁用”默认值，以便浏览器缓存具有更大的灵活性。从版本4.3.3升级到4.3.4时，如果想保留与版本4.3.3相同的行为，则必须明确设置 `DispatcherRestrictUncacheableContent` 到On。
* DISP-841 - Dispatcher 不遵循 504 响应代码的 /serverStaleOnError
* DISP-874 — 创建Dispatcher配置以打开或关闭DISP-818的实施
* DISP-883 - 在 Dispatcher 中显示 URL 请求分解的跟踪
* DISP-944 — 预加载虚名URL

### 版本 4.3.3（2019 年 10 月 18 日） {#october}

**错误修复**：

* DISP-739 - LogLevel Dispatcher： **级别** 不起作用。
* DISP-749 - Alpine Linux® Dispatcher发生崩溃，提供跟踪日志级别。

**改进功能**：

* DISP-813 - Dispatcher 支持 openssl 1.1.x
* DISP-814 - 缓存刷新期间出现 Apache 40x 错误
* DISP-818 - mod_expires 为不可缓存的内容添加 Cache-Control 标头
* DISP-821 - 不在套接字中存储日志上下文
* DISP-822 - Dispatcher应使用 `ppoll` 而不是 `pselect`
* DISP-824 - 安全 DispatcherUseForwardedHost
* DISP-825 - 当磁盘上没有更多空间时记录特殊消息
* DISP-826 — 支持使用查询字符串重新获取URI

**新增功能**：

* DISP-703 - 场特定的缓存命中率
* DISP-827 - 用于测试的本地服务器
* DISP-828 — 为Dispatcher创建测试docker映像

### 版本 4.3.2（2019 年 1 月 31 日） {#jan}

**错误修复**：

* DISP-734 - 如果未设置为处理程序，Dispatcher 会导致 insert_output_filter 崩溃
* DISP-735 - RE在Alpine Linux上不起作用®
* DISP-740 — 默认情况下禁止在macOS Mojave中加载Dispatcher
* DISP-742 — 被阻止的请求可能会将信息泄露给受授权检查器保护的资源

**改进功能**：

* DISP-746 - dispatcher.any中未标记的字符串将生成警告

**新增功能**：

* DISP-747 - 提供 Apache 环境中的请求信息

### 版本 4.3.1（2018 年 10 月 16 日） {#oct}

**错误修复**：

* DISP-656 - Dispatcher 提供错误的 ETag 标头
* DISP-694 - 禁止在保持活动连接失效时显示警告
* DISP-714 — 基于Cookie的会话管理在IIS中不起作用
* DISP-715 - renderid cookie 的安全标志
* DISP-720 - 临时文件未关闭会导致内存耗尽（打开的文件过多）
* DISP-721 - 当 Apache 正常重启子进程时，Dispatcher 会中断 poll()
* DISP-722 - 使用八进制模式 0600 创建缓存文件
* DISP-723 — 当渲染超时设置为0时，默认为10分钟超时（并重试）
* DISP-725 - 以静默方式将字符串后面的尾随字符转换为未命名的值
* DISP-726 - 在任何场均不实际匹配传入主机时记录警告
* DISP-727 - Dispatcher 检查空缓存文件的请求内容长度
* DISP-730 — 尝试通过Dispatcher访问头文件时出现404
* DISP-731 - Dispatcher 易受日志注入攻击
* DISP-732 - Dispatcher应删除URL中的连续“/”
* DISP-733 - Dispatcher 将设置（计算）Age 标头

**改进功能**：

* DISP-656 - Dispatcher 提供错误的 ETag 标头
* DISP-694 - 禁止在保持活动连接失效时显示警告
* DISP-715 - renderid cookie 的安全标志
* DISP-722 - 使用八进制模式 0600 创建缓存文件
* DISP-726 - 在任何场均不实际匹配传入主机时记录警告

### 版本 4.3.0（2018 年 6 月 13 日） {#jun}

**错误修复**：

* DISP-682 - 数字日志级别应用不正确
* DISP-685 - 32位Solaris™ SPARC®二进制文件具有对__divdi3的未定义引用
* DISP-688 - Dispatcher不会在404响应中返回“X-Cache-Info”标头
* DISP-690 - Last-Modified 标头不可缓存
* DISP-691 - w3wp.exe 中的访问违规
* DISP-693 — 在Dispatcher下载页面上更新Solaris™服务器的架构详细信息
* DISP-695 - Dispatcher 模块 4.2.3 中的 DispatcherLog 级别问题
* DISP-698 - Dispatcher TTL必须支持s-maxage和私有指令
* DISP-700 — 模块在Alpine Linux上无法正常工作®
* DISP-704 - 包含 %2b 的浏览器请求将发送到未编码的发布者
* DISP-705 — 双重释放或损坏导致Dispatcher崩溃(`fasttop`)
* DISP-706 — 在失效期间，Dispatcher将回溯引用可能导致无限循环的符号链接
* DISP-709 - 阻止一些虚名 URL 扩展
* DISP-710 — 针对Linux的构建®在Cent OS 6上不可用

**改进功能**：

* DISP-652 - Dispatcher 提供错误的 Date 标头

## 有用资源 {#helpful-resources}

* [AEM Dispatcher 概述](dispatcher.md)

## 下载 {#downloads}

### Apache 2.4 {#apache}

| 平台 | 架构 | OpenSSL支持 | 单击以下载 |
|---|---|---|---|
| Linux® | i686（32位） | 无 | [`dispatcher-apache2.4-linux-i686-4.3.7.tar.gz`](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-4.3.7.tar.gz) |
| Linux® | i686（32位） | 1.0 | [`dispatcher-apache2.4-linux-i686-ssl1.0-4.3.7.tar.gz`](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-ssl1.0-4.3.7.tar.gz) |
| Linux® | i686（32位） | 1.1 | [`dispatcher-apache2.4-linux-i686-ssl1.1-4.3.7.tar.gz`](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-ssl1.1-4.3.7.tar.gz) |
| Linux® | i686（32位） | 3.0 | [`dispatcher-apache2.4-linux-i686-ssl3.0-4.3.7.tar.gz`](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-ssl3.0-4.3.7.tar.gz) |
| Linux® | x86_64（64位） | 无 | [`dispatcher-apache2.4-linux-x86_64-4.3.7.tar.gz`](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-4.3.7.tar.gz) |
| Linux® | x86_64（64位） | 1.0 | [`dispatcher-apache2.4-linux-x86_64-ssl1.0-4.3.7.tar.gz`](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-ssl1.0-4.3.7.tar.gz) |
| Linux® | x86_64（64位） | 1.1 | [`dispatcher-apache2.4-linux-x86_64-ssl1.1-4.3.7.tar.gz`](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-ssl1.1-4.3.7.tar.gz) |
| Linux® | x86_64（64位） | 3.0 | [`dispatcher-apache2.4-linux-x86_64-ssl3.0-4.3.7.tar.gz`](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-ssl3.0-4.3.7.tar.gz) |
| Linux® | aarch64（64位） | 无 | [`dispatcher-apache2.4-linux-aarch64-4.3.7.tar.gz`](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-aarch64-4.3.7.tar.gz) |
| Linux® | aarch64（64位） | 1.0 | [`dispatcher-apache2.4-linux-aarch64-ssl1.0-4.3.7.tar.gz`](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-aarch64-ssl1.0-4.3.7.tar.gz) |
| Linux® | aarch64（64位） | 1.1 | [`dispatcher-apache2.4-linux-aarch64-ssl1.1-4.3.7.tar.gz`](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-aarch64-ssl1.1-4.3.7.tar.gz) |
| Linux® | aarch64（64位） | 3.0 | [`dispatcher-apache2.4-linux-aarch64-ssl3.0-4.3.7.tar.gz`](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-aarch64-ssl3.0-4.3.7.tar.gz) |
| macOS | arm64（64位） | 无 | [`dispatcher-apache2.4-darwin-arm64-4.3.7.tar.gz`](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-darwin-arm64-4.3.7.tar.gz) |
| macOS | x86_64（64位） | 无 | [`dispatcher-apache2.4-darwin-x86_64-4.3.7.tar.gz`](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-darwin-x86_64-4.3.7.tar.gz) |

### IIS {#iis}

| 平台 | 架构 | OpenSSL支持 | 单击以下载 |
|---|---|---|---|
| Windows | x86（32位） | 无 | [`dispatcher-iis-windows-x86-4.3.7.zip`](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-4.3.7.zip) |
| Windows | x86（32位） | 1.0 | [`dispatcher-iis-windows-x86-ssl1.0-4.3.7.zip`](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-ssl1.0-4.3.7.zip) |
| Windows | x86（32位） | 1.1 | [`dispatcher-iis-windows-x86-ssl1.1-4.3.7.zip`](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-ssl1.1-4.3.7.zip) |
| Windows | x64（64位） | 无 | [`dispatcher-iis-windows-x64-4.3.7.zip`](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-4.3.7.zip) |
| Windows | x64（64位） | 1.0 | [`dispatcher-iis-windows-x64-ssl1.0-4.3.7.zip`](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-ssl1.0-4.3.7.zip) |
| Windows | x64（64位） | 1.1 | [`dispatcher-iis-windows-x64-ssl1.1-4.3.7.zip`](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-ssl1.1-4.3.7.zip) |
| Windows | x64（64位） | 3.0 | [`dispatcher-iis-windows-x64-ssl3.0-4.3.7.zip`](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-ssl3.0-4.3.7.zip) |