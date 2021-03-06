﻿---
title: Web 会议的部署清单
TOCTitle: Web 会议的部署清单
ms:assetid: 9908ebe0-e5d3-4920-b9b1-85021f7e69e9
ms:mtpsurl: https://technet.microsoft.com/zh-cn/library/JJ205104(v=OCS.15)
ms:contentKeyID: 49313686
ms.date: 05/19/2016
mtps_version: v=OCS.15
ms.translationtype: HT
---

# Web 会议的部署清单

 

_**上一次修改主题：** 2015-03-09_

与其他 Lync Server 2013 组件的部署一样，Web 会议的部署也需要您使用拓扑生成器创建并发布融入了会议的拓扑。

## 部署顺序

可以在部署初始拓扑的同时部署会议，或在部署至少一个前端池或 Standard Edition Server 之后部署会议。

## 会议部署过程

下表提供在现有拓扑中部署会议所需步骤的概述。


<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 25%" />
<col style="width: 25%" />
<col style="width: 25%" />
</colgroup>
<thead>
<tr class="header">
<th>阶段</th>
<th>步骤</th>
<th>角色和组成员身份</th>
<th>文档</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><strong>安装必备软硬件</strong></p></td>
<td><p>会议在前端池中的前端服务器上和 Standard Edition 服务器上运行。除了需要安装这些服务器外，没有额外的软硬件要求。</p>
<div>

> [!NOTE]  
> Lync Server 2013 使用 Office Web Apps 和 Office Web Apps Server 处理 PowerPoint 演示文稿的共享和呈现。有关安装和配置 Office Web Apps Server 的信息，请参阅<a href="lync-server-2013-enabling-office-web-apps-server-and-lync-server-2013.md">配置与 Office Web Apps Server 和 Lync Server 2013 的集成</a>。


</div></td>
<td><p>属于本地 Administrators 组成员的域用户</p></td>
<td><p>可支持性文档中的<a href="lync-server-2013-supported-hardware.md">支持的适用于 Lync Server 2013 的硬件</a></p>
<p>可支持性文档中的<a href="lync-server-2013-server-software-and-infrastructure-support.md">Lync Server 2013 中的服务器软件和基础结构支持</a></p>
<p>规划文档中的<a href="lync-server-2013-determining-your-system-requirements.md">确定 Lync Server 2013 的系统要求</a>。</p>
<p>规划文档中的<a href="lync-server-2013-technical-requirements-for-archiving.md">Lync Server 2013 中的存档技术要求</a>。</p>
<p></p></td>
</tr>
<tr class="even">
<td><p><strong>创建相应的内部拓扑以支持会议</strong></p></td>
<td><p>运行拓扑生成器以向拓扑中添加会议，然后发布该拓扑。</p></td>
<td><p>若要定义拓扑，需具有本地 Users 组成员的帐户</p>
<p>若要发布拓扑，需具有 Domain Admins 组和 RTCUniversalServerAdmins 组成员的帐户，该帐户还需在要用于 Lync Server 2013 文件存储的文件共享上具有完全控制权限（即读取、写入和修改），以使拓扑生成器可以配置所需的 DACL</p></td>
<td><p>部署文档中的<a href="lync-server-2013-define-and-configure-a-topology-in-topology-builder.md">在 Lync Server 2013 拓扑生成器中定义和配置拓扑</a>。</p></td>
</tr>
<tr class="odd">
<td><p><strong>配置会议策略和支持</strong></p></td>
<td><p>使用 Lync Server 2013 控制面板或 Lync Server 命令行管理程序配置会议设置。</p></td>
<td><p>RTCUniversalServerAdmins 组（仅限于 Windows PowerShell）或向 [] 或 CSAdministrator 角色分配用户</p></td>
<td><p>操作文档中的<a href="lync-server-2013-conferencing-policies.md">会议策略</a>。</p></td>
</tr>
</tbody>
</table>


Lync Server 2013 现在包括 **MaxUploadFileSizeMb** 设置，此设置限制会议期间可上载的文件的大小。此设置的默认值是 500 MB。您可以使用 **Set-CsConferencingConfiguration** cmdlet 调整 **MaxUploadFileSizeMb**。

**MaxUploadFileSizeMb** 未限制 Lync Web App 的文件上载设置。Lync Web App 的文件大小上载限制设置为约 30MB，并由 IIS web.config 文件 /DataCollabWeb/Int\[Ext\]/Handler/web.config 控制。若要配置 Lync Web App 的文件大小上载限制，请更新 web.config 文件中的 `maxRequestLength` 和 `maxAllowedContentLength`，如下所示。

    <system.web>
        <!-- 
            Since this handler is used to upload files to DMCU the request size (in kilobytes) 
            has to fit max allowed file size uploaded by LWA client.
            The timeout has to reflect the min client bandwidth. Timeout of 600 secs 
            and 512 Kbits of *client* bandwidth would result into aproximately 30 Mbytes 
            for LWA upload size limit.
        -->
          <httpRuntime maxRequestLength="500000" executionTimeout="600" />
    
    
    
                    <security>
                    <requestFiltering>
                               <requestLimits maxAllowedContentLength="524288000" />
                    </requestFiltering>
                    </security>

必须更新每个前端服务器的 web.config 文件。

