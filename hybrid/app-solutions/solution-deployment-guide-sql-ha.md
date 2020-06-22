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
# <a name="deploy-a-sql-server-2016-availability-group-to-azure-and-azure-stack-hub"></a><span data-ttu-id="bb53b-103">將 SQL Server 2016 可用性群組部署至 Azure 和 Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="bb53b-103">Deploy a SQL Server 2016 availability group to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="bb53b-104">這篇文章會逐步引導您了解在跨兩個 Azure Stack Hub 環境中，透過非同步的災害復原 (DR) 網站自動部署基本的高度可用 (HA) SQL Server 2016 Enterprise 叢集。</span><span class="sxs-lookup"><span data-stu-id="bb53b-104">This article will step you through an automated deployment of a basic highly available (HA) SQL Server 2016 Enterprise cluster with an asynchronous disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="bb53b-105">若要深入了解 SQL Server 2016 和高可用性，請參閱[Always On 可用性群組：高可用性和災害復原解決方案](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016)。</span><span class="sxs-lookup"><span data-stu-id="bb53b-105">To learn more about SQL Server 2016 and high availability, see [Always On availability groups: a high-availability and disaster-recovery solution](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span></span>

<span data-ttu-id="bb53b-106">在本解決方案中，您會建置環境範例，用以：</span><span class="sxs-lookup"><span data-stu-id="bb53b-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="bb53b-107">協調兩個 Azure Stack Hub 間的部署。</span><span class="sxs-lookup"><span data-stu-id="bb53b-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="bb53b-108">使用 Docker 透過 Azure API 設定檔將相依性問題降至最低。</span><span class="sxs-lookup"><span data-stu-id="bb53b-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="bb53b-109">透過災害復原網站部署基本的高度可用 SQL Server 2016 Enterprise 叢集。</span><span class="sxs-lookup"><span data-stu-id="bb53b-109">Deploy a basic highly available SQL Server 2016 Enterprise cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="bb53b-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="bb53b-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="bb53b-111">Microsoft Azure Stack Hub 是 Azure 的延伸模組。</span><span class="sxs-lookup"><span data-stu-id="bb53b-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="bb53b-112">Azure Stack Hub 可將雲端運算的靈活性和創新能力導入您的內部部署環境中，並啟用獨特的混合式雲端，讓您能夠隨處建置及部署混合式應用程式。</span><span class="sxs-lookup"><span data-stu-id="bb53b-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="bb53b-113">[混合式應用程式設計考量](overview-app-design-considerations.md)一文檢閱了設計、部署和操作混合式應用程式時的軟體品質要素 (放置、延展性、可用性、復原、管理性和安全性)。</span><span class="sxs-lookup"><span data-stu-id="bb53b-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="bb53b-114">這些設計考量有助於您設計出最佳的混合式應用程式，減少生產環境可能會遇到的挑戰。</span><span class="sxs-lookup"><span data-stu-id="bb53b-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-sql-server-2016"></a><span data-ttu-id="bb53b-115">適用於 SQL Server 2016 的架構</span><span class="sxs-lookup"><span data-stu-id="bb53b-115">Architecture for SQL Server 2016</span></span>

![SQL Server 2016 SQL HA Azure Stack Hub](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a><span data-ttu-id="bb53b-117">SQL Server 2016 的必要條件</span><span class="sxs-lookup"><span data-stu-id="bb53b-117">Prerequisites for SQL Server 2016</span></span>

- <span data-ttu-id="bb53b-118">兩個連線的 Azure Stack Hub 整合式系統 (Azure Stack Hub)。</span><span class="sxs-lookup"><span data-stu-id="bb53b-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="bb53b-119">此部署不適用於 Azure Stack 開發套件 (ASDK)。</span><span class="sxs-lookup"><span data-stu-id="bb53b-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="bb53b-120">若要深入了解 Azure Stack Hub，請參閱 [Azure Stack 概觀](https://azure.microsoft.com/overview/azure-stack/)。</span><span class="sxs-lookup"><span data-stu-id="bb53b-120">To learn more about Azure Stack Hub, see the [Azure Stack overview](https://azure.microsoft.com/overview/azure-stack/).</span></span>
- <span data-ttu-id="bb53b-121">每個 Azure Stack Hub 上的租用戶訂閱。</span><span class="sxs-lookup"><span data-stu-id="bb53b-121">A tenant subscription on each Azure Stack Hub.</span></span>
  - <span data-ttu-id="bb53b-122">**記下每個訂閱識別碼，以及每個 Azure Stack Hub 的 Azure Resource Manager 端點。**</span><span class="sxs-lookup"><span data-stu-id="bb53b-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="bb53b-123">具有每個 Azure Stack Hub 上租用戶訂閱權限的 Azure Active Directory (Azure AD) 服務主體。</span><span class="sxs-lookup"><span data-stu-id="bb53b-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="bb53b-124">若針對不同的 Azure AD 租用戶部署了 Azure Stack Hub，建議您建立兩個服務主體。</span><span class="sxs-lookup"><span data-stu-id="bb53b-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="bb53b-125">若要了解如何建立 Azure Stack Hub 的服務主體，請參閱[建立服務主體以使應用程式可存取 Azure Stack Hub 資源](https://docs.microsoft.com/azure-stack/user/azure-stack-create-service-principals)。</span><span class="sxs-lookup"><span data-stu-id="bb53b-125">To learn how to create a service principal for Azure Stack Hub, see [Create service principals to give apps access to Azure Stack Hub resources](https://docs.microsoft.com/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="bb53b-126">**記下每個服務主體的應用程式識別碼、用戶端密碼和租用戶名稱 (xxxxx.onmicrosoft.com)。**</span><span class="sxs-lookup"><span data-stu-id="bb53b-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="bb53b-127">SQL Server 2016 Enterprise 已與每個 Azure Stack Hub 的 Marketplace 進行摘要整合。</span><span class="sxs-lookup"><span data-stu-id="bb53b-127">SQL Server 2016 Enterprise syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="bb53b-128">若要深入了解 Marketplace 摘要整合，請參閱[將 Marketplace 項目下載到 Azure Stack Hub](https://docs.microsoft.com/azure-stack/operator/azure-stack-download-azure-marketplace-item)。</span><span class="sxs-lookup"><span data-stu-id="bb53b-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](https://docs.microsoft.com/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
    <span data-ttu-id="bb53b-129">**請確保貴組織具備適當的 SQL 授權。**</span><span class="sxs-lookup"><span data-stu-id="bb53b-129">**Make sure that your organization has the appropriate SQL licenses.**</span></span>
- <span data-ttu-id="bb53b-130">[適用於 Windows 的 Docker](https://docs.docker.com/docker-for-windows/) 已安裝在您的本機電腦上。</span><span class="sxs-lookup"><span data-stu-id="bb53b-130">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="bb53b-131">取得 Docker 映像</span><span class="sxs-lookup"><span data-stu-id="bb53b-131">Get the Docker image</span></span>

<span data-ttu-id="bb53b-132">每個部署的 Docker 映像會排除不同 Azure PowerShell 版本間的相依性問題。</span><span class="sxs-lookup"><span data-stu-id="bb53b-132">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="bb53b-133">請確定適用於 Windows 的 Docker 會使用 Windows 容器。</span><span class="sxs-lookup"><span data-stu-id="bb53b-133">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="bb53b-134">在提升權限的命令提示字元中執行下列指令碼，以取得 Docker 容器與部署指令碼。</span><span class="sxs-lookup"><span data-stu-id="bb53b-134">Run the following script in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a><span data-ttu-id="bb53b-135">部署可用性群組</span><span class="sxs-lookup"><span data-stu-id="bb53b-135">Deploy the availability group</span></span>

1. <span data-ttu-id="bb53b-136">一旦成功提取容器映像之後，請啟動映像。</span><span class="sxs-lookup"><span data-stu-id="bb53b-136">Once the container image has been successfully pulled, start the image.</span></span>

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. <span data-ttu-id="bb53b-137">容器啟動之後，您會獲得容器中提升權限的 PowerShell 終端機。</span><span class="sxs-lookup"><span data-stu-id="bb53b-137">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="bb53b-138">變更目錄以部署指令碼。</span><span class="sxs-lookup"><span data-stu-id="bb53b-138">Change directories to get to the deployment script.</span></span>

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. <span data-ttu-id="bb53b-139">執行部署。</span><span class="sxs-lookup"><span data-stu-id="bb53b-139">Run the deployment.</span></span> <span data-ttu-id="bb53b-140">需要提供認證和資源名稱。</span><span class="sxs-lookup"><span data-stu-id="bb53b-140">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="bb53b-141">HA 指的是將部署 HA 叢集的 Azure Stack Hub。</span><span class="sxs-lookup"><span data-stu-id="bb53b-141">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="bb53b-142">DR 指的是將部署 DR 叢集的 Azure Stack Hub。</span><span class="sxs-lookup"><span data-stu-id="bb53b-142">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

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

4. <span data-ttu-id="bb53b-143">輸入 `Y` 以允許安裝 NuGet 提供者，如此可開始安裝 API 設定檔Profile "2018-03-01-hybrid" 模組。</span><span class="sxs-lookup"><span data-stu-id="bb53b-143">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="bb53b-144">等待資源部署完成。</span><span class="sxs-lookup"><span data-stu-id="bb53b-144">Wait for resource deployment to complete.</span></span>

6. <span data-ttu-id="bb53b-145">一旦 DR 資源部署完成，結束容器。</span><span class="sxs-lookup"><span data-stu-id="bb53b-145">Once DR resource deployment has completed, exit the container.</span></span>

      ```powershell
      exit
      ```

7. <span data-ttu-id="bb53b-146">檢視每個 Azure Stack Hub 入口網站中的資源，確定部署完成。</span><span class="sxs-lookup"><span data-stu-id="bb53b-146">Inspect the deployment by viewing the resources in each Azure Stack Hub's portal.</span></span> <span data-ttu-id="bb53b-147">連線至 HA 環境中其中一個 SQL 執行個體，並透過 SQL Server Management Studio (SSMS) 檢查可用性群組。</span><span class="sxs-lookup"><span data-stu-id="bb53b-147">Connect to one of the SQL instances on the HA environment and inspect the Availability Group through SQL Server Management Studio (SSMS).</span></span>

    ![SQL Server 2016 SQL HA](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a><span data-ttu-id="bb53b-149">後續步驟</span><span class="sxs-lookup"><span data-stu-id="bb53b-149">Next steps</span></span>

- <span data-ttu-id="bb53b-150">使用 SQL Server Management Studio 手動容錯移轉至叢集。</span><span class="sxs-lookup"><span data-stu-id="bb53b-150">Use SQL Server Management Studio to manually fail over the cluster.</span></span> <span data-ttu-id="bb53b-151">請參閱[執行 Always On 可用性群組的強制手動容錯移轉 (SQL Server)](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span><span class="sxs-lookup"><span data-stu-id="bb53b-151">See [Perform a Forced Manual Failover of an Always On Availability Group (SQL Server)](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span></span>
- <span data-ttu-id="bb53b-152">深入了解混合式雲端應用程式。</span><span class="sxs-lookup"><span data-stu-id="bb53b-152">Learn more about hybrid cloud apps.</span></span> <span data-ttu-id="bb53b-153">請參閱[混合式雲端解決方案](https://aka.ms/azsdevtutorials)。</span><span class="sxs-lookup"><span data-stu-id="bb53b-153">See [Hybrid Cloud Solutions.](https://aka.ms/azsdevtutorials)</span></span>
- <span data-ttu-id="bb53b-154">使用您自己的資料，或修改 [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns) 上的這份範例程式碼。</span><span class="sxs-lookup"><span data-stu-id="bb53b-154">Use your own data or modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>
