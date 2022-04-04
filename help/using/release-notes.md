---
title: AEM Dispatcher 发行说明
seo-title: AEM Dispatcher Release Notes
description: 特定于 Adobe Experience Manager Dispatcher 的发行说明
seo-description: Release notes specific to Adobe Experience Manager Dispatcher
uuid: ae3ccf62-0514-4c03-a3b9-71799a482cbd
topic-tags: release-notes
content-type: reference
products: SG_EXPERIENCEMANAGER/6.4
discoiquuid: ff3d38e0-71c9-4b41-85f9-fa896393aac5
exl-id: b55c7a34-d57b-4d45-bd83-29890f1524de
source-git-commit: 3f040a4150bc398d25ffa2426f9dd9de99a0b8fc
workflow-type: tm+mt
source-wordcount: '977'
ht-degree: 94%

---

# AEM Dispatcher 发行说明{#aem-dispatcher-release-notes}

## 版本信息 {#release-information}

|  |  |
|--- |--- |
| 产品 | Adobe Experience Manager (AEM) Dispatcher |
| 版本 | 4.3.5 |
| 类型 | 次要版本 |
| 日期 | 2022 年 4 月 04 日 |
| 下载 URL | <ul><li>[Apache 2.4](release-notes.md#apache)</li><li>[Microsoft Internet Information Services (IIS)](release-notes.md#iis)</li></ul> |
| 兼容性 | AEM 6.1 或更高版本 |

## 系统要求和先决条件 {#system-requirements-and-prerequisites}

有关要求和先决条件的更多信息，请参阅[支持的平台](https://helpx.adobe.com/cn/experience-manager/6-4/sites/deploying/using/technical-requirements.html)页面。

Adobe强烈建议使用最新版本的AEM Dispatcher，以从最新功能、最新错误修复和最佳性能中受益。

## 安装说明 {#installation-instructions}

有关详细说明，请参阅[安装 Dispatcher](dispatcher-install.md)。

## 版本历史记录 {#release-history}

### 4.3.5版（2022年4月29日） {#apr}

**改进功能**：

* DISP-954 — 即使未过期，也支持失效
* DISP-949 — 即使过滤器阻止POST请求，Dispatcher仍会返回200而不是404

### 版本 4.3.4（2021 年 11 月 29 日） {#nov}

**错误修复**：

* DISP-833 - X-Forwarded-Host 标题可能包含以逗号分隔的主机名列表
* DISP-835 - DispatcherUseForwardedHost 忽略最后出现的主机标头

**改进功能**：

* DISP-874 - 创建 Dispatcher 配置以通过 `DispatcherRestrictUncacheableContent` 标志启用或禁用 DISP-818 的实施。默认值为“禁用”。当设置为“启用”时，系统会删除 mod expires 为不可缓存内容设置的任何缓存标头。这与 4.3.3 版本中的行为（默认值为“启用”）不同，但与 4.3.3 之前的版本（默认值为“禁用”）相同。推荐的方法是保留 `DispatcherRestrictUncacheableContent` 的“禁用”默认值，以便浏览器缓存具有更大的灵活性。从 4.3.3 版升级到 4.3.4 版时，如果您希望保持与 4.3.3 版相同的行为，则必须将 `DispatcherRestrictUncacheableContent` 明确设置为“启用”。
* DISP-841 - Dispatcher 不遵循 504 响应代码的 /serverStaleOnError
* DISP-874 - 创建 Dispatcher 配置以启用或禁用 DISP-818 的实施
* DISP-883 - 在 Dispatcher 中显示 URL 请求分解的跟踪
* DISP-944 - 预加载虚名 URL

### 版本 4.3.3（2019 年 10 月 18 日） {#october}

**错误修复**：

* DISP-739 - LogLevel Dispatcher：**级别**&#x200B;无效
* DISP-749 - Alpine Linux Dispatcher 发生崩溃，提供跟踪日志级别

**改进功能**：

* DISP-813 - Dispatcher 支持 openssl 1.1.x
* DISP-814 - 缓存刷新期间出现 Apache 40x 错误
* DISP-818 - mod_expires 为不可缓存的内容添加 Cache-Control 标头
* DISP-821 - 不在套接字中存储日志上下文
* DISP-822 - Dispatcher 应使用 ppoll 而非 pselect
* DISP-824 - 安全 DispatcherUseForwardedHost
* DISP-825 - 当磁盘上没有更多空间时记录特殊消息
* DISP-826 - 支持使用查询字符串重新获取 URI

**新增功能**：

* DISP-703 - 场特定的缓存命中率
* DISP-827 - 用于测试的本地服务器
* DISP-828 - 为 Dispatcher 创建测试 docker 镜像

### 版本 4.3.2（2019 年 1 月 31 日） {#jan}

**错误修复**：

* DISP-734 - 如果未设置为处理程序，Dispatcher 会导致 insert_output_filter 崩溃
* DISP-735 - RE 在 Alpine Linux 上不起作用
* DISP-740 - 默认情况下禁止在 macOS Mojave 中加载 Dispatcher
* DISP-742 - 被阻止的请求可能会将信息泄露给受 auth checker 保护的资源

**改进功能**：

* DISP-746 - dispatcher.any 中未标记的字符串将生成警告

**新增功能**：

* DISP-747 - 提供 Apache 环境中的请求信息

### 版本 4.3.1（2018 年 10 月 16 日） {#oct}

**错误修复**：

* DISP-656 - Dispatcher 提供错误的 ETag 标头
* DISP-694 - 禁止在保持活动连接失效时显示警告
* DISP-714 - 基于 Cookie 的会话管理在 IIS 中不起作用
* DISP-715 - renderid cookie 的安全标志
* DISP-720 - 临时文件未关闭会导致内存耗尽（打开的文件过多）
* DISP-721 - 当 Apache 正常重启子进程时，Dispatcher 会中断 poll()
* DISP-722 - 使用八进制模式 0600 创建缓存文件
* DISP-723 - 当渲染超时设置为 0 时，默认为 10 分钟超时（并重试）
* DISP-725 - 以静默方式将字符串后面的尾随字符转换为未命名的值
* DISP-726 - 在任何场均不实际匹配传入主机时记录警告
* DISP-727 - Dispatcher 检查空缓存文件的请求内容长度
* DISP-730 - 尝试通过 Dispatcher 访问头文件时出现 404
* DISP-731 - Dispatcher 易受日志注入攻击
* DISP-732 - Dispatcher 将删除 URL 中的连续“/”
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
* DISP-685 - 32 位 Solaris SPARC 二进制文件具有对 __divdi3 的未定义引用
* DISP-688 - Dispatcher 不会在 404 响应中返回“X-Cache-Info”标头
* DISP-690 - Last-Modified 标头不可缓存
* DISP-691 - w3wp.exe 中的访问违规
* DISP-693 - 需要在 Dispatcher 下载页面上更新 solaris 服务器的架构详细信息
* DISP-695 - Dispatcher 模块 4.2.3 中的 DispatcherLog 级别问题
* DISP-698 - Dispatcher TTL 需要支持 s-maxage 和私有指令
* DISP-700 - 模块在 Alpine Linux 上不起作用
* DISP-704 - 包含 %2b 的浏览器请求将发送到未编码的发布者
* DISP-705 - 双重释放或损坏 (fasttop) 导致 Dispatcher 发生崩溃
* DISP-706 - 在失效期间，Dispatcher 将回溯引用可能导致无限循环的符号链接
* DISP-709 - 阻止一些虚名 URL 扩展
* DISP-710 - 针对 Linux 的构建无法在 Cent OS 6 上使用

**改进功能**：

* DISP-652 - Dispatcher 提供错误的 Date 标头

## 有用资源 {#helpful-resources}

* [AEM Dispatcher 概述](dispatcher.md)

## 下载 {#downloads}

### Apache 2.4 {#apache}

| 平台 | 架构 | OpenSSL 支持 | 下载 |
|---|---|---|---|
| Linux | i686（32 位） | 无 | [dispatcher-apache2.4-linux-i686-4.3.5.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-4.3.5.tar.gz) |
| Linux | i686（32 位） | 1.0 | [dispatcher-apache2.4-linux-i686-ssl1.0-4.3.5.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-ssl1.0-4.3.5.tar.gz) |
| Linux | i686（32 位） | 1.1 | [dispatcher-apache2.4-linux-i686-ssl1.1-4.3.5.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-ssl1.1-4.3.5.tar.gz) |
| Linux | x86_64（64 位） | 无 | [dispatcher-apache2.4-linux-x86_64-4.3.5.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-4.3.5.tar.gz) |
| Linux | x86_64（64 位） | 1.0 | [dispatcher-apache2.4-linux-x86_64-ssl1.0-4.3.5.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-ssl1.0-4.3.5.tar.gz) |
| Linux | x86_64（64 位） | 1.1 | [dispatcher-apache2.4-linux-x86_64-ssl1.1-4.3.5.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-ssl1.1-4.3.5.tar.gz) |
| macOS | x86_64（64 位） | 无 | [dispatcher-apache2.4-darwin-x86_64-4.3.5.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-darwin-x86_64-4.3.5.tar.gz) |

### IIS {#iis}

| 平台 | 架构 | OpenSSL 支持 | 下载 |
|---|---|---|---|
| Windows | x86（32 位） | 无 | [dispatcher-iis-windows-x86-4.3.5.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-4.3.5.zip) |
| Windows | x86（32 位） | 1.0 | [dispatcher-iis-windows-x86-ssl1.0-4.3.5.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-ssl1.0-4.3.5.zip) |
| Windows | x86（32 位） | 1.1 | [dispatcher-iis-windows-x86-ssl1.1-4.3.5.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-ssl1.1-4.3.5.zip) |
| Windows | x64（64 位） | 无 | [dispatcher-iis-windows-x64-4.3.5.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-4.3.5.zip) |
| Windows | x64（64 位） | 1.0 | [dispatcher-iis-windows-x64-ssl1.0-4.3.5.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-ssl1.0-4.3.5.zip) |
| Windows | x64（64 位） | 1.1 | [dispatcher-iis-windows-x64-ssl1.1-4.3.5.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-ssl1.1-4.3.5.zip) |
