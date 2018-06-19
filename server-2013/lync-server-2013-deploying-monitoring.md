﻿---
title: 部署监控
TOCTitle: 部署监控
ms:assetid: 117f4a3e-0670-4388-a553-b9854921145f
ms:mtpsurl: https://technet.microsoft.com/zh-cn/library/Gg398199(v=OCS.15)
ms:contentKeyID: 49312042
ms.date: 05/19/2016
mtps_version: v=OCS.15
ms.translationtype: HT
---

# 部署监控

 

_**上一次修改主题：** 2013-12-17_

自弃用监控服务器角色以来，已对 Microsoft Lync Server 2013 监控基础结构进行了重大更改。不再采用不同的监控服务器角色（通常需要组织设置专用计算机来充当监控服务器），现在监控服务已并置到每台前端服务器中。除了其他优点外，此更改还有助于：

  - 减少实现 Lync Server 2013 时所需的服务器角色数。在此情况下，减少监控服务器角色数还可帮助降低成本，因为无需维护用于监控的专用服务器。

  - 降低 Lync Server 安装和管理的复杂性。通过自动在每台前端服务器上并置监控服务，您不必再安装、配置和管理监控服务器角色。

<table>
<thead>
<tr class="header">
<th><img src="images/Dn783119.note(OCS.15).gif" title="note" alt="note" />注意：</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>存档服务器角色在 Lync Server 2013 中也已弃用。与监控服务一样，Lync Server 2013 存档服务现在也并置在每台前端服务器上。注意到这一点非常重要，只是因为监控和存档通常共享同一 SQL Server 数据库实例。</td>
</tr>
</tbody>
</table>


正如您可能预料的那样，这些更改对监控服务的安装和管理方式具有重要影响。例如，由于监控服务器角色不再存在，因此已将监控服务器节点从 Lync Server 拓扑生成器中删除；反之，这意味着，您不再使用拓扑生成器的“新建监控服务器”向导来向拓扑中添加新监控服务器。（该向导不再存在。）您通常通过完成以下两个步骤在拓扑中实现监控服务：

1.  在设置新 Lync Server 池的同时启用监控。（在 Lync Server 2013 中，按池启用或禁用监控。）请注意，可为池启用监控，而不实际收集监控数据，在本文档的“配置呼叫详细记录和用户体验质量设置”部分中对此过程进行了说明。

2.  使监控存储（即，监控数据库）与新池关联。请注意，一个监控存储可与多个池关联。根据驻留在注册器池中的用户数，这意味着，您不必为每个池设置单独的监控数据库。一个监控存储可供多个池使用。

尽管在创建新池的同时启用监控通常更为简单，但也可以创建新池并禁用监控。如果执行此操作，以后可以使用拓扑生成器启用该服务：拓扑生成器提供为池启用或禁用监控或将池与不同监控存储关联的方式。请记住，即使不再存在监控服务器角色，您仍需要创建一个或多个监控存储：用于存储监控服务所收集数据的后端数据库。可以使用 Microsoft SQL Server 2008 R2 或 Microsoft SQL Server 2012 创建这些后端数据库。

<table>
<thead>
<tr class="header">
<th><img src="images/Dn783119.note(OCS.15).gif" title="note" alt="note" />注意：</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>如果已为池启用监控，则可禁用收集监控数据的过程，而不必更改拓扑：Lync Server 命令行管理程序为您提供了禁用（以后再重新启用）呼叫详细记录 (CDR) 或用户体验质量 (QoE) 数据收集的方式。有关详细信息，请参阅本文的“配置呼叫详细记录和用户体验质量设置”部分。</td>
</tr>
</tbody>
</table>


对 Lync Server 2013 中的监控的另一项重要增强是 Lync Server 监控报告现在支持 IPv6：使用“IP 地址”字段的报告将显示 IPv4 或 IPv6 地址：1) 使用的 SQL 查询；以及 2) IPv6 地址是否存储在监控数据库中。

<table>
<thead>
<tr class="header">
<th><img src="images/Dn783119.note(OCS.15).gif" title="note" alt="note" />注意：</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>确保 SQL Server 代理服务启动类型为“自动”，并且 SQL Server 代理服务正在针对容纳监控数据库的 SQL 实例运行，以便默认监控 SQL Server 维护作业在 SQL Server 代理服务的控制下按计划运行。</td>
</tr>
</tbody>
</table>


本文档可指导您完成为 Lync Server 2013 安装和配置监控及监控报告的过程。本文提供了会对您有所帮助的分步说明：

  - 在拓扑中启用监控并使监控存储与前端池关联。

  - 安装 SQL Server Reporting Services 和 Lync Server 监控报告。监控报告是预配置的报告，其中提供不同视图，便于查看监控数据库中存储的信息。

  - 配置呼叫详细记录 (CDR) 和用户体验质量 (QoE) 数据收集。呼叫详细记录为您提供了一种方法，用于跟踪 Lync Server 功能（如 IP 语音 (VoIP) 电话呼叫、即时消息 (IM)、文件传输、音频/视频 (A/V) 会议和应用程序共享会话）的使用情况。QoE 指标跟踪在组织内进行的音频和视频呼叫的质量，包括丢失的网络数据包数、背景噪音和“抖动”量（数据包延迟的差异）等内容。

  - 从监控数据库中手动清除 CDR 和/或 QoE 记录。
