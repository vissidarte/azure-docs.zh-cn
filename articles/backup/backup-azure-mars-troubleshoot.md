---
title: 对 Azure 备份代理进行故障排除 | Microsoft Docs
description: 排除与 Azure 备份代理的安装和注册相关的问题
services: backup
documentationcenter: ''
author: saurabhsensharma
manager: shreeshd
editor: ''
ms.assetid: 778c6ccf-3e57-4103-a022-367cc60c411a
ms.service: backup
ms.workload: storage-backup-recovery
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/4/2017
ms.author: saurse;markgal;
ms.openlocfilehash: f7f4ac328c4e35f52bcc9708faf96d06189dd9ac
ms.sourcegitcommit: a36a1ae91968de3fd68ff2f0c1697effbb210ba8
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/17/2018
---
# <a name="troubleshoot-azure-backup-agent-configuration-and-registration-issues"></a>排除 Azure 备份代理配置和注册问题
## <a name="recommended-steps"></a>建议的步骤
请参考下表中建议的操作来解决在 Azure 备份代理的安装和注册过程中可能会遇到的错误。

[!INCLUDE [backup-upgrade-mars-agent.md](../../includes/backup-upgrade-mars-agent.md)]

## <a name="invalid-vault-credentials-provided-the-file-is-either-corrupted-or-does-not-have-the-latest-credentials-associated-with-recovery-service"></a>提供的保管库凭据无效。 该文件已损坏，或者没有与恢复服务关联的最新凭据。
| 错误详细信息 | 可能的原因 | 建议的操作 |
| ---     | ---     | ---    |
| **错误** </br> *提供的保管库凭据无效。该文件已损坏，或者没有与恢复服务关联的最新凭据。(ID: 34513)* | <ul><li> 保管库凭据无效（也就是说，它们是在注册之前的 48 小时前下载的）。<li>Azure 备份代理无法将某个临时文件下载到 Windows 的 Temp 文件夹。 <li>保管库凭据位于某个网络位置。 <li>TLS 1.0 已禁用<li> 所配置的某个代理服务器正在阻止连接。 <br> |  <ul><li>下载新的保管库凭据。<li>转到“Internet 选项” > “安全” > “Internet”。 接下来，选择“自定义级别”，并滚动直至看到“文件下载”部分。 然后选择“启用”。<li>可能还需要将站点添加到[受信任的站点](https://docs.microsoft.com/azure/backup/backup-try-azure-backup-in-10-mins#network-and-connectivity-requirements)。<li>更改设置以使用代理服务器。 然后提供代理服务器详细信息。 <li> 使日期和时间与你的计算机匹配。<li>转到 C:/Windows/Temp，检查是否存在超过 60,000 或 65,000 个扩展名为 .tmp 的文件。 如果存在，请删除这些文件。<li>可以在服务器上运行 SDP 包进行测试。 如果收到的错误指出不允许下载文件，则很可能是因为 C:/Windows/Temp 目录中存在大量文件。<li>确保已安装了 .NET Framework 4.6.2。 <li>如果由于 PCI 符合性而禁用了 TLS 1.0，请参阅此[故障排除页面](https://support.microsoft.com/help/4022913)。 <li>如果服务器中安装了防病毒软件，请从防病毒软件扫描中排除以下文件： <ul><li>CBengine.exe<li>CSC.exe，这与 .NET Framework 相关。 服务器上安装的每个 .NET 版本都有一个 CSC.exe。 请排除受影响服务器上的所有 .NET Framework 版本绑定的 CSC.exe 文件。 <li>Scratch 文件夹或缓存位置。 <br>*scratch 文件夹的默认位置或缓存位置路径为 C:\Program Files\Microsoft Azure Recovery Services Agent\Scratch*。

## <a name="the-microsoft-azure-recovery-service-agent-was-unable-to-connect-to-microsoft-azure-backup"></a>Microsoft Azure 恢复服务代理无法连接到 Microsoft Azure 备份

| 错误详细信息 | 可能的原因 | 建议的操作 |
| ---     | ---     | ---    |
| **错误** </br><ol><li>Microsoft Azure 恢复服务代理无法连接到 Microsoft Azure 备份。(ID: 100050) 请检查网络设置，并确保能够连接到 Internet<li>(407) 需要代理身份验证 |代理正在阻止连接。 |  <ul><li>转到“Internet 选项” > “安全” > “Internet”。 然后选择“自定义级别”，并滚动直至看到“文件下载”部分。 选择“启用”。<li>可能还需要将站点添加到[受信任的站点](https://docs.microsoft.com/azure/backup/backup-try-azure-backup-in-10-mins#network-and-connectivity-requirements)。<li>更改设置以使用代理服务器。 然后提供代理服务器详细信息。 <li>如果服务器中安装了防病毒软件，请从防病毒软件扫描中排除以下文件。 <ul><li>CBEngine.exe（而非 dpmra.exe）。<li>CSC.exe（与 .NET Framework 相关）。 服务器上安装的每个 .NET 版本都有一个 CSC.exe。 请排除受影响服务器上的所有 .NET Framework 版本绑定的 CSC.exe 文件。 <li>Scratch 文件夹或缓存位置。 <br>*scratch 文件夹的默认位置或缓存位置路径为 C:\Program Files\Microsoft Azure Recovery Services Agent\Scratch*。

## <a name="failed-to-set-the-encryption-key-for-secure-backups"></a>未能设置安全备份的加密密钥

| 错误详细信息 | 可能的原因 | 建议的操作 |
| ---     | ---     | ---    |      
| **错误** </br>*未能设置安全备份的加密密钥。由于内部服务错误“输入无效错误”，当前操作失败。请稍后重试操作。如果该问题仍然存在，请联系 Microsoft 支持部门*。 |服务器已注册到另一个保管库。| 从该保管库中取消注册服务器，然后再次注册。

## <a name="the-activation-did-not-complete-successfully-the-current-operation-failed-due-to-an-internal-service-error-0x1fc07"></a>激活未成功完成。 由于内部服务错误 [0x1FC07]，当前操作失败

| 错误详细信息 | 可能的原因 | 建议的操作 |
| ---     | ---     | ---    |          
| **错误** </br><ol><li>*激活未成功完成。由于内部服务错误 [0x1FC07]，当前操作失败。请稍后重试操作。如果该问题仍然存在，请联系 Microsoft 支持部门* <li>错误 34506。*未在此计算机上正确配置存储的加密密码*。 | <li> Scratch 文件夹位于空间不足的卷上。 <li> 已错误地将 Scratch 文件夹移到另一位置。 <li> 缺少 OnlineBackup.KEK 文件。 | <li>将 scratch 文件夹或缓存位置移到可用空间相当于备份数据总大小 5-10% 的卷。 若要正确地移动缓存位置，请参阅[有关 Azure 备份代理的问题](https://docs.microsoft.com/azure/backup/backup-azure-file-folder-backup-faq#backup)中的步骤。<li> 确保 OnlineBackup.KEK 文件存在。 <br>*scratch 文件夹的默认位置或缓存位置路径为 C:\Program Files\Microsoft Azure Recovery Services Agent\Scratch*。

## <a name="need-help-contact-support"></a>需要帮助？ 联系支持人员
如果仍需帮助，请[联系支持人员](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以快速解决问题。

## <a name="next-steps"></a>后续步骤
* 详细了解[如何使用 Azure 备份代理备份 Windows Server](tutorial-backup-windows-server-to-azure.md)。
* 如果需要还原备份，请参阅[将文件还原到 Windows 计算机](backup-azure-restore-windows-server.md)一文。
