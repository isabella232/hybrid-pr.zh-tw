---
title: 將高度可用的 MongoDB 解決方案部署到 Azure 和 Azure Stack Hub
description: 了解如何將高度可用的 MongoDB 解決方案部署到 Azure 和 Azure Stack Hub
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: f6064aaa1087a3c0cfc26e09371e81752c777edb
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477264"
---
# <a name="deploy-a-highly-available-mongodb-solution-to-azure-and-azure-stack-hub"></a>將高度可用的 MongoDB 解決方案部署到 Azure 和 Azure Stack Hub

這篇文章會逐步引導您了解在跨兩個 Azure Stack Hub 環境中，透過災害復原 (DR) 網站自動部署基本的高度可用 (HA) MongoDB 叢集。 若要深入了解 MongoDB 和高可用性，請參閱[複本集集合成員](https://docs.mongodb.com/manual/core/replica-set-members/)。

在本解決方案中，您會建立環境範例，用以：

> [!div class="checklist"]
> - 協調兩個 Azure Stack Hub 間的部署。
> - 使用 Docker 透過 Azure API 設定檔將相依性問題降至最低。
> - 透過災害復原網站部署基本的高度可用 MongoDB 叢集。

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub 是 Azure 的延伸模組。 Azure Stack Hub 可將雲端運算的靈活性和創新能力導入您的內部部署環境中，並啟用獨特的混合式雲端，讓您能夠隨處建置及部署混合式應用程式。  
> 
> [混合式應用程式設計考量](overview-app-design-considerations.md)一文檢閱了設計、部署和操作混合式應用程式時的軟體品質要素 (放置、延展性、可用性、復原、管理性和安全性)。 這些設計考量有助於您設計出最佳的混合式應用程式，減少生產環境可能會遇到的挑戰。

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a>適用於 MongoDB 搭配 Azure Stack Hub 的架構

![Azure Stack Hub 中的高可用性 MongoDB 架構](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a>MongoDB 搭配 Azure Stack Hub 的必要條件

- 兩個連線的 Azure Stack Hub 整合式系統 (Azure Stack Hub)。 此部署不適用於 Azure Stack 開發套件 (ASDK)。 若要深入了解 Azure Stack Hub，請參閱[什麼是 Azure Stack Hub？](https://azure.microsoft.com/products/azure-stack/hub/)
  - 每個 Azure Stack Hub 上的租用戶訂閱。 
  - **記下每個訂閱識別碼，以及每個 Azure Stack Hub 的 Azure Resource Manager 端點。**
- 具有每個 Azure Stack Hub 上租用戶訂閱權限的 Azure Active Directory (Azure AD) 服務主體。 若針對不同的 Azure AD 租用戶部署了 Azure Stack Hub，建議您建立兩個服務主體。 若要了解如何建立 Azure Stack Hub 的服務主體，請參閱[使用應用程式身分識別來存取 Azure Stack Hub 資源](/azure-stack/user/azure-stack-create-service-principals)。
  - **記下每個服務主體的應用程式識別碼、用戶端密碼和租用戶名稱 (xxxxx.onmicrosoft.com)。**
- Ubuntu 16.04 已與每個 Azure Stack Hub 的 Marketplace 進行摘要整合。 若要深入了解 Marketplace 摘要整合，請參閱[將 Marketplace 項目下載到 Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item)。
- [適用於 Windows 的 Docker](https://docs.docker.com/docker-for-windows/) 已安裝在您的本機電腦上。

## <a name="get-the-docker-image"></a>取得 Docker 映像

每個部署的 Docker 映像會排除不同 Azure PowerShell 版本間的相依性問題。

1. 請確定適用於 Windows 的 Docker 會使用 Windows 容器。
2. 在提升權限的命令提示字元中執行下列命令，以取得 Docker 容器與部署指令碼。

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a>部署叢集

1. 一旦成功提取容器映像之後，請啟動映像。

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. 容器啟動之後，您會獲得容器中提升權限的 PowerShell 終端機。 變更目錄以部署指令碼。

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. 執行部署。 需要提供認證和資源名稱。 HA 指的是將部署 HA 叢集的 Azure Stack Hub。 DR 指的是將部署 DR 叢集的 Azure Stack Hub。

    ```powershell
    .\Deploy-AzureResourceGroup.ps1 `
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

5. 首先會部署 HA 資源。 監視部署並等候部署完成。 收到指出 HA 部署已完成的訊息後，您可以檢查 HA Azure Stack Hub 的入口網站，以查看已部署的資源。

6. 持續部署災害復原資源，並判斷您是否要在災害復原 Azure Stack Hub 上啟動 Jump Box 與叢集互動。

7. 等待 DR 資源部署完成。

8. 在 DR 資源部署完成後，結束容器。

  ```powershell
  exit
  ```

## <a name="next-steps"></a>後續步驟

- 若您在災害復原 Azure Stack Hub 上啟用了 Jump Box 虛擬機器，就可以透過 SSH 連線並安裝 Mongo CLI 與 MongoDB 叢集互動。 若要深入了解如何與 MongoDB 互動，請參閱 [Mongo Shell](https://docs.mongodb.com/manual/mongo/)。
- 若要深入了解混合式雲端應用程式，請參閱[混合式雲端解決方案](https://aka.ms/azsdevtutorials)。
- 修改 [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns) 上的這份範例程式碼。
