---
title: 將 SQL Server 2016 可用性群組部署至 Azure 和 Azure Stack Hub
description: 了解如何將 SQL Server 2016 可用性群組部署至 Azure 和 Azure Stack Hub。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ff6d5b9667e63a6b8d232b6dd93db2d8b12fd46d
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910098"
---
# <a name="deploy-a-sql-server-2016-availability-group-to-azure-and-azure-stack-hub"></a>將 SQL Server 2016 可用性群組部署至 Azure 和 Azure Stack Hub

這篇文章會逐步引導您了解在跨兩個 Azure Stack Hub 環境中，透過非同步的災害復原 (DR) 網站自動部署基本的高度可用 (HA) SQL Server 2016 Enterprise 叢集。 若要深入了解 SQL Server 2016 和高可用性，請參閱[Always On 可用性群組：高可用性和災害復原解決方案](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016)。

在本解決方案中，您會建置環境範例，用以：

> [!div class="checklist"]
> - 協調兩個 Azure Stack Hub 間的部署。
> - 使用 Docker 透過 Azure API 設定檔將相依性問題降至最低。
> - 透過災害復原網站部署基本的高度可用 SQL Server 2016 Enterprise 叢集。

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub 是 Azure 的延伸模組。 Azure Stack Hub 可將雲端運算的靈活性和創新能力導入您的內部部署環境中，並啟用獨特的混合式雲端，讓您能夠隨處建置及部署混合式應用程式。  
> 
> [混合式應用程式設計考量](overview-app-design-considerations.md)一文檢閱了設計、部署和操作混合式應用程式時的軟體品質要素 (放置、延展性、可用性、復原、管理性和安全性)。 這些設計考量有助於您設計出最佳的混合式應用程式，減少生產環境可能會遇到的挑戰。

## <a name="architecture-for-sql-server-2016"></a>適用於 SQL Server 2016 的架構

![SQL Server 2016 SQL HA Azure Stack Hub](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a>SQL Server 2016 的必要條件

- 兩個連線的 Azure Stack Hub 整合式系統 (Azure Stack Hub)。 此部署不適用於 Azure Stack 開發套件 (ASDK)。 若要深入了解 Azure Stack Hub，請參閱 [Azure Stack 概觀](https://azure.microsoft.com/overview/azure-stack/)。
- 每個 Azure Stack Hub 上的租用戶訂閱。
  - **記下每個訂閱識別碼，以及每個 Azure Stack Hub 的 Azure Resource Manager 端點。**
- 具有每個 Azure Stack Hub 上租用戶訂閱權限的 Azure Active Directory (Azure AD) 服務主體。 若針對不同的 Azure AD 租用戶部署了 Azure Stack Hub，建議您建立兩個服務主體。 若要了解如何建立 Azure Stack Hub 的服務主體，請參閱[建立服務主體以使應用程式可存取 Azure Stack Hub 資源](https://docs.microsoft.com/azure-stack/user/azure-stack-create-service-principals)。
  - **記下每個服務主體的應用程式識別碼、用戶端密碼和租用戶名稱 (xxxxx.onmicrosoft.com)。**
- SQL Server 2016 Enterprise 已與每個 Azure Stack Hub 的 Marketplace 進行摘要整合。 若要深入了解 Marketplace 摘要整合，請參閱[將 Marketplace 項目下載到 Azure Stack Hub](https://docs.microsoft.com/azure-stack/operator/azure-stack-download-azure-marketplace-item)。
    **請確保貴組織具備適當的 SQL 授權。**
- [適用於 Windows 的 Docker](https://docs.docker.com/docker-for-windows/) 已安裝在您的本機電腦上。

## <a name="get-the-docker-image"></a>取得 Docker 映像

每個部署的 Docker 映像會排除不同 Azure PowerShell 版本間的相依性問題。

1. 請確定適用於 Windows 的 Docker 會使用 Windows 容器。
2. 在提升權限的命令提示字元中執行下列指令碼，以取得 Docker 容器與部署指令碼。

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a>部署可用性群組

1. 一旦成功提取容器映像之後，請啟動映像。

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. 容器啟動之後，您會獲得容器中提升權限的 PowerShell 終端機。 變更目錄以部署指令碼。

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. 執行部署。 需要提供認證和資源名稱。 HA 指的是將部署 HA 叢集的 Azure Stack Hub。 DR 指的是將部署 DR 叢集的 Azure Stack Hub。

      ```powershell
      > .\Deploy-AzureResourceGroup.ps1 `
      -AzureStackApplicationId_HA "applicationIDforHAServicePrincipal" `
      -AzureStackApplicationSercet_HA "clientSecretforHAServicePrincipal" `
      -AADTenantName_HA "hatenantname.onmicrosoft.com" `
      -AzureStackResourceGroup_HA "haresourcegroupname" `
      -AzureStackArmEndpoint_HA "https://management.haazurestack.com" `
      -AzureStackSubscriptionId_HA "haSubscriptionId" `
      -AzureStackApplicationId_DR "applicationIDforDRServicePrincipal" `
      -AzureStackApplicationSercet_DR "ClientSecretforDRServicePrincipal" `
      -AADTenantName_DR "drtenantname.onmicrosoft.com" `
      -AzureStackResourceGroup_DR "drresourcegroupname" `
      -AzureStackArmEndpoint_DR "https://management.drazurestack.com" `
      -AzureStackSubscriptionId_DR "drSubscriptionId"
      ```

4. 輸入 `Y` 以允許安裝 NuGet 提供者，如此可開始安裝 API 設定檔Profile "2018-03-01-hybrid" 模組。

5. 等待資源部署完成。

6. 一旦 DR 資源部署完成，結束容器。

      ```powershell
      exit
      ```

7. 檢視每個 Azure Stack Hub 入口網站中的資源，確定部署完成。 連線至 HA 環境中其中一個 SQL 執行個體，並透過 SQL Server Management Studio (SSMS) 檢查可用性群組。

    ![SQL Server 2016 SQL HA](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a>後續步驟

- 使用 SQL Server Management Studio 手動容錯移轉至叢集。 請參閱[執行 Always On 可用性群組的強制手動容錯移轉 (SQL Server)](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)
- 深入了解混合式雲端應用程式。 請參閱[混合式雲端解決方案](https://aka.ms/azsdevtutorials)。
- 使用您自己的資料，或修改 [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns) 上的這份範例程式碼。
