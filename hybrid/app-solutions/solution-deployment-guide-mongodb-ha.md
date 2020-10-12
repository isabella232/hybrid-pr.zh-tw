---
title: 將高度可用的 MongoDB 解決方案部署到 Azure 和 Azure Stack Hub
description: 了解如何將高度可用的 MongoDB 解決方案部署到 Azure 和 Azure Stack Hub
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: def9abaa2a7231648f11453f66119399be015a4d
ms.sourcegitcommit: 485a1f97fa1579364e2be1755cadfc5ea89db50e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2020
ms.locfileid: "91852502"
---
# <a name="deploy-a-highly-available-mongodb-solution-to-azure-and-azure-stack-hub"></a><span data-ttu-id="63d24-103">將高度可用的 MongoDB 解決方案部署到 Azure 和 Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="63d24-103">Deploy a highly available MongoDB solution to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="63d24-104">這篇文章會逐步引導您了解在跨兩個 Azure Stack Hub 環境中，透過災害復原 (DR) 網站自動部署基本的高度可用 (HA) MongoDB 叢集。</span><span class="sxs-lookup"><span data-stu-id="63d24-104">This article will step you through an automated deployment of a basic highly available (HA) MongoDB cluster with a disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="63d24-105">若要深入了解 MongoDB 和高可用性，請參閱[複本集集合成員](https://docs.mongodb.com/manual/core/replica-set-members/)。</span><span class="sxs-lookup"><span data-stu-id="63d24-105">To learn more about MongoDB and high availability, see [Replica Set Members](https://docs.mongodb.com/manual/core/replica-set-members/).</span></span>

<span data-ttu-id="63d24-106">在本解決方案中，您會建立環境範例，用以：</span><span class="sxs-lookup"><span data-stu-id="63d24-106">In this solution, you'll create a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="63d24-107">協調兩個 Azure Stack Hub 間的部署。</span><span class="sxs-lookup"><span data-stu-id="63d24-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="63d24-108">使用 Docker 透過 Azure API 設定檔將相依性問題降至最低。</span><span class="sxs-lookup"><span data-stu-id="63d24-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="63d24-109">透過災害復原網站部署基本的高度可用 MongoDB 叢集。</span><span class="sxs-lookup"><span data-stu-id="63d24-109">Deploy a basic highly available MongoDB cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="63d24-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="63d24-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="63d24-111">Microsoft Azure Stack Hub 是 Azure 的延伸模組。</span><span class="sxs-lookup"><span data-stu-id="63d24-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="63d24-112">Azure Stack Hub 可將雲端運算的靈活性和創新能力導入您的內部部署環境中，並啟用獨特的混合式雲端，讓您能夠隨處建置及部署混合式應用程式。</span><span class="sxs-lookup"><span data-stu-id="63d24-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="63d24-113">[混合式應用程式設計考量](overview-app-design-considerations.md)一文檢閱了設計、部署和操作混合式應用程式時的軟體品質要素 (放置、延展性、可用性、復原、管理性和安全性)。</span><span class="sxs-lookup"><span data-stu-id="63d24-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="63d24-114">這些設計考量有助於您設計出最佳的混合式應用程式，減少生產環境可能會遇到的挑戰。</span><span class="sxs-lookup"><span data-stu-id="63d24-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="63d24-115">適用於 MongoDB 搭配 Azure Stack Hub 的架構</span><span class="sxs-lookup"><span data-stu-id="63d24-115">Architecture for MongoDB with Azure Stack Hub</span></span>

![Azure Stack Hub 中的高可用性 MongoDB 架構](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="63d24-117">MongoDB 搭配 Azure Stack Hub 的必要條件</span><span class="sxs-lookup"><span data-stu-id="63d24-117">Prerequisites for MongoDB with Azure Stack Hub</span></span>

- <span data-ttu-id="63d24-118">兩個連線的 Azure Stack Hub 整合式系統 (Azure Stack Hub)。</span><span class="sxs-lookup"><span data-stu-id="63d24-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="63d24-119">此部署不適用於 Azure Stack 開發套件 (ASDK)。</span><span class="sxs-lookup"><span data-stu-id="63d24-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="63d24-120">若要深入了解 Azure Stack Hub，請參閱[什麼是 Azure Stack Hub？](https://azure.microsoft.com/products/azure-stack/hub/)</span><span class="sxs-lookup"><span data-stu-id="63d24-120">To learn more about Azure Stack Hub, see [What is Azure Stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/)</span></span>
  - <span data-ttu-id="63d24-121">每個 Azure Stack Hub 上的租用戶訂閱。</span><span class="sxs-lookup"><span data-stu-id="63d24-121">A tenant subscription on each Azure Stack Hub.</span></span> 
  - <span data-ttu-id="63d24-122">**記下每個訂閱識別碼，以及每個 Azure Stack Hub 的 Azure Resource Manager 端點。**</span><span class="sxs-lookup"><span data-stu-id="63d24-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="63d24-123">具有每個 Azure Stack Hub 上租用戶訂閱權限的 Azure Active Directory (Azure AD) 服務主體。</span><span class="sxs-lookup"><span data-stu-id="63d24-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="63d24-124">若針對不同的 Azure AD 租用戶部署了 Azure Stack Hub，建議您建立兩個服務主體。</span><span class="sxs-lookup"><span data-stu-id="63d24-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="63d24-125">若要了解如何建立 Azure Stack Hub 的服務主體，請參閱[使用應用程式身分識別來存取 Azure Stack Hub 資源](/azure-stack/user/azure-stack-create-service-principals)。</span><span class="sxs-lookup"><span data-stu-id="63d24-125">To learn how to create a service principal for Azure Stack Hub, see [Use an app identity to access Azure Stack Hub resources](/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="63d24-126">**記下每個服務主體的應用程式識別碼、用戶端密碼和租用戶名稱 (xxxxx.onmicrosoft.com)。**</span><span class="sxs-lookup"><span data-stu-id="63d24-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="63d24-127">Ubuntu 16.04 已與每個 Azure Stack Hub 的 Marketplace 進行摘要整合。</span><span class="sxs-lookup"><span data-stu-id="63d24-127">Ubuntu 16.04 syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="63d24-128">若要深入了解 Marketplace 摘要整合，請參閱[將 Marketplace 項目下載到 Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item)。</span><span class="sxs-lookup"><span data-stu-id="63d24-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
- <span data-ttu-id="63d24-129">[適用於 Windows 的 Docker](https://docs.docker.com/docker-for-windows/) 已安裝在您的本機電腦上。</span><span class="sxs-lookup"><span data-stu-id="63d24-129">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="63d24-130">取得 Docker 映像</span><span class="sxs-lookup"><span data-stu-id="63d24-130">Get the Docker image</span></span>

<span data-ttu-id="63d24-131">每個部署的 Docker 映像會排除不同 Azure PowerShell 版本間的相依性問題。</span><span class="sxs-lookup"><span data-stu-id="63d24-131">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="63d24-132">請確定適用於 Windows 的 Docker 會使用 Windows 容器。</span><span class="sxs-lookup"><span data-stu-id="63d24-132">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="63d24-133">在提升權限的命令提示字元中執行下列命令，以取得 Docker 容器與部署指令碼。</span><span class="sxs-lookup"><span data-stu-id="63d24-133">Run the following command in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a><span data-ttu-id="63d24-134">部署叢集</span><span class="sxs-lookup"><span data-stu-id="63d24-134">Deploy the clusters</span></span>

1. <span data-ttu-id="63d24-135">一旦成功提取容器映像之後，請啟動映像。</span><span class="sxs-lookup"><span data-stu-id="63d24-135">Once the container image has been successfully pulled, start the image.</span></span>

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. <span data-ttu-id="63d24-136">容器啟動之後，您會獲得容器中提升權限的 PowerShell 終端機。</span><span class="sxs-lookup"><span data-stu-id="63d24-136">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="63d24-137">變更目錄以部署指令碼。</span><span class="sxs-lookup"><span data-stu-id="63d24-137">Change directories to get to the deployment script.</span></span>

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. <span data-ttu-id="63d24-138">執行部署。</span><span class="sxs-lookup"><span data-stu-id="63d24-138">Run the deployment.</span></span> <span data-ttu-id="63d24-139">需要提供認證和資源名稱。</span><span class="sxs-lookup"><span data-stu-id="63d24-139">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="63d24-140">HA 指的是將部署 HA 叢集的 Azure Stack Hub。</span><span class="sxs-lookup"><span data-stu-id="63d24-140">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="63d24-141">DR 指的是將部署 DR 叢集的 Azure Stack Hub。</span><span class="sxs-lookup"><span data-stu-id="63d24-141">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

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

4. <span data-ttu-id="63d24-142">輸入 `Y` 以允許安裝 NuGet 提供者，如此可開始安裝 API 設定檔Profile "2018-03-01-hybrid" 模組。</span><span class="sxs-lookup"><span data-stu-id="63d24-142">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="63d24-143">首先會部署 HA 資源。</span><span class="sxs-lookup"><span data-stu-id="63d24-143">The HA resources will deploy first.</span></span> <span data-ttu-id="63d24-144">監視部署並等候部署完成。</span><span class="sxs-lookup"><span data-stu-id="63d24-144">Monitor the deployment and wait for it to finish.</span></span> <span data-ttu-id="63d24-145">收到指出 HA 部署已完成的訊息後，您可以檢查 HA Azure Stack Hub 的入口網站，以查看已部署的資源。</span><span class="sxs-lookup"><span data-stu-id="63d24-145">Once you have the message stating that the HA deployment is finished, you can check the HA Azure Stack Hub's portal to see the resources deployed.</span></span>

6. <span data-ttu-id="63d24-146">持續部署災害復原資源，並判斷您是否要在災害復原 Azure Stack Hub 上啟動 Jump Box 與叢集互動。</span><span class="sxs-lookup"><span data-stu-id="63d24-146">Continue with the deployment of DR resources and decide if you'd like to enable a jump box on the DR Azure Stack Hub to interact with the cluster.</span></span>

7. <span data-ttu-id="63d24-147">等待 DR 資源部署完成。</span><span class="sxs-lookup"><span data-stu-id="63d24-147">Wait for DR resource deployment to finish.</span></span>

8. <span data-ttu-id="63d24-148">在 DR 資源部署完成後，結束容器。</span><span class="sxs-lookup"><span data-stu-id="63d24-148">Once DR resource deployment has finished, exit the container.</span></span>

  ```powershell
  exit
  ```

## <a name="next-steps"></a><span data-ttu-id="63d24-149">後續步驟</span><span class="sxs-lookup"><span data-stu-id="63d24-149">Next steps</span></span>

- <span data-ttu-id="63d24-150">若您在災害復原 Azure Stack Hub 上啟用了 Jump Box 虛擬機器，就可以透過 SSH 連線並安裝 Mongo CLI 與 MongoDB 叢集互動。</span><span class="sxs-lookup"><span data-stu-id="63d24-150">If you enabled the jump box VM on the DR Azure Stack Hub, you can connect via SSH and interact with the MongoDB cluster by installing the mongo CLI.</span></span> <span data-ttu-id="63d24-151">若要深入了解如何與 MongoDB 互動，請參閱 [Mongo Shell](https://docs.mongodb.com/manual/mongo/)。</span><span class="sxs-lookup"><span data-stu-id="63d24-151">To learn more about interacting with MongoDB, see [The mongo Shell](https://docs.mongodb.com/manual/mongo/).</span></span>
- <span data-ttu-id="63d24-152">若要深入了解混合式雲端應用程式，請參閱[混合式雲端解決方案](/azure-stack/user/)。</span><span class="sxs-lookup"><span data-stu-id="63d24-152">To learn more about hybrid cloud apps, see [Hybrid Cloud Solutions.](/azure-stack/user/)</span></span>
- <span data-ttu-id="63d24-153">修改 [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns) 上的這份範例程式碼。</span><span class="sxs-lookup"><span data-stu-id="63d24-153">Modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>