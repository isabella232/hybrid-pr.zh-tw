---
title: 在 Azure 與 Azure Stack Hub 中部署可跨雲端調整的應用程式
description: 了解如何在 Azure 與 Azure Stack Hub 中部署可跨雲端調整的應用程式。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 5ae6c4323324fa104cd0e5c7b5198492be14b8eb
ms.sourcegitcommit: 56980e3c118ca0a672974ee3835b18f6e81b6f43
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/26/2020
ms.locfileid: "88886810"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a><span data-ttu-id="d6515-103">部署應用程式，並使用 Azure 與 Azure Stack Hub 來進行跨雲端規模調整</span><span class="sxs-lookup"><span data-stu-id="d6515-103">Deploy an app that scales cross-cloud using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="d6515-104">了解如何建立可提供手動觸發程序的跨雲端解決方案，以透過流量管理員使用自動縮放功能，從 Azure Stack Hub 裝載的 Web 應用程式切換到 Azure 裝載的 Web 應用程式。</span><span class="sxs-lookup"><span data-stu-id="d6515-104">Learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app with autoscaling via traffic manager.</span></span> <span data-ttu-id="d6515-105">此流程可確保雲端公用程式在處理負載時具有彈性且可調整。</span><span class="sxs-lookup"><span data-stu-id="d6515-105">This process ensures flexible and scalable cloud utility when under load.</span></span>

<span data-ttu-id="d6515-106">透過此模式，您的租用戶可能還無法在公用雲端中執行應用程式。</span><span class="sxs-lookup"><span data-stu-id="d6515-106">With this pattern, your tenant may not be ready to run your app in the public cloud.</span></span> <span data-ttu-id="d6515-107">不過，若讓企業在內部部署環境中維持用來處理應用程式需求高峰的容量，似乎不切實際。</span><span class="sxs-lookup"><span data-stu-id="d6515-107">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="d6515-108">您的租用戶可以透過其內部部署解決方案來使用公用雲端的靈活性。</span><span class="sxs-lookup"><span data-stu-id="d6515-108">Your tenant can make use of the elasticity of the public cloud with their on-premises solution.</span></span>

<span data-ttu-id="d6515-109">在本解決方案中，您會建置環境範例，用以：</span><span class="sxs-lookup"><span data-stu-id="d6515-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="d6515-110">建立多節點 Web 應用程式。</span><span class="sxs-lookup"><span data-stu-id="d6515-110">Create a multi-node web app.</span></span>
> - <span data-ttu-id="d6515-111">設定和管理持續部署 (CD) 程序。</span><span class="sxs-lookup"><span data-stu-id="d6515-111">Configure and manage the Continuous Deployment (CD) process.</span></span>
> - <span data-ttu-id="d6515-112">將 Web 應用程式發佈至 Azure Stack Hub。</span><span class="sxs-lookup"><span data-stu-id="d6515-112">Publish the web app to Azure Stack Hub.</span></span>
> - <span data-ttu-id="d6515-113">建立發行。</span><span class="sxs-lookup"><span data-stu-id="d6515-113">Create a release.</span></span>
> - <span data-ttu-id="d6515-114">了解如何監視並追蹤您的部署。</span><span class="sxs-lookup"><span data-stu-id="d6515-114">Learn to monitor and track your deployments.</span></span>

> [!Tip]  
> <span data-ttu-id="d6515-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="d6515-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="d6515-116">Microsoft Azure Stack Hub 是 Azure 的延伸模組。</span><span class="sxs-lookup"><span data-stu-id="d6515-116">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="d6515-117">Azure Stack Hub 可將雲端運算的靈活性和創新能力導入您的內部部署環境中，並啟用獨特的混合式雲端，讓您能夠隨處建置及部署混合式應用程式。</span><span class="sxs-lookup"><span data-stu-id="d6515-117">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="d6515-118">[混合式應用程式設計考量](overview-app-design-considerations.md)一文檢閱了設計、部署和操作混合式應用程式時的軟體品質要素 (放置、延展性、可用性、復原、管理性和安全性)。</span><span class="sxs-lookup"><span data-stu-id="d6515-118">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="d6515-119">這些設計考量有助於您設計出最佳的混合式應用程式，減少生產環境可能會遇到的挑戰。</span><span class="sxs-lookup"><span data-stu-id="d6515-119">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="d6515-120">Prerequisites</span><span class="sxs-lookup"><span data-stu-id="d6515-120">Prerequisites</span></span>

- <span data-ttu-id="d6515-121">Azure 訂用帳戶。</span><span class="sxs-lookup"><span data-stu-id="d6515-121">Azure subscription.</span></span> <span data-ttu-id="d6515-122">如有需要，請在開始之前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。</span><span class="sxs-lookup"><span data-stu-id="d6515-122">If needed, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before beginning.</span></span>
- <span data-ttu-id="d6515-123">Azure Stack Hub 整合式系統，或 Azure Stack 開發套件 (ASDK) 的部署。</span><span class="sxs-lookup"><span data-stu-id="d6515-123">An Azure Stack Hub integrated system or deployment of Azure Stack Development Kit (ASDK).</span></span>
  - <span data-ttu-id="d6515-124">如需如何安裝 Azure Stack Hub 的指示，請參閱[安裝 ASDK](/azure-stack/asdk/asdk-install.md)。</span><span class="sxs-lookup"><span data-stu-id="d6515-124">For instructions on installing Azure Stack Hub, see [Install the ASDK](/azure-stack/asdk/asdk-install.md).</span></span>
  - <span data-ttu-id="d6515-125">如需 ASDK 部署後自動化指令碼，請移至：[https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span><span class="sxs-lookup"><span data-stu-id="d6515-125">For an ASDK post-deployment automation script, go to: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span></span>
  - <span data-ttu-id="d6515-126">此安裝可能需要幾個小時才能完成。</span><span class="sxs-lookup"><span data-stu-id="d6515-126">This installation may require a few hours to complete.</span></span>
- <span data-ttu-id="d6515-127">將 [App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS 服務部署至 Azure Stack Hub。</span><span class="sxs-lookup"><span data-stu-id="d6515-127">Deploy [App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS services to Azure Stack Hub.</span></span>
- <span data-ttu-id="d6515-128">在 Azure Stack Hub 環境內[建立方案/供應項目](/azure-stack/operator/service-plan-offer-subscription-overview.md)。</span><span class="sxs-lookup"><span data-stu-id="d6515-128">[Create plans/offers](/azure-stack/operator/service-plan-offer-subscription-overview.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="d6515-129">在 Azure Stack Hub 環境內[建立租用戶訂用帳戶](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md)。</span><span class="sxs-lookup"><span data-stu-id="d6515-129">[Create tenant subscription](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="d6515-130">建立租用戶訂用帳戶中建立 Web 應用程式。</span><span class="sxs-lookup"><span data-stu-id="d6515-130">Create a web app within the tenant subscription.</span></span> <span data-ttu-id="d6515-131">記下新的 Web 應用程式 URL，以供後續使用。</span><span class="sxs-lookup"><span data-stu-id="d6515-131">Make note of the new web app URL for later use.</span></span>
- <span data-ttu-id="d6515-132">在租用戶訂用帳戶中部署 Azure Pipelines 虛擬機器 (VM)。</span><span class="sxs-lookup"><span data-stu-id="d6515-132">Deploy Azure Pipelines virtual machine (VM) within the tenant subscription.</span></span>
- <span data-ttu-id="d6515-133">需要具有 .NET 3.5 的 Windows Server 2016 VM。</span><span class="sxs-lookup"><span data-stu-id="d6515-133">Windows Server 2016 VM with .NET 3.5 is required.</span></span> <span data-ttu-id="d6515-134">此 VM 將會建置在 Azure Stack Hub 上的租用戶訂用帳戶中，作為私人組建代理程式。</span><span class="sxs-lookup"><span data-stu-id="d6515-134">This VM will be built in the tenant subscription on Azure Stack Hub as the private build agent.</span></span>
- <span data-ttu-id="d6515-135">您可以在 Azure Stack Hub Marketplace 中取得[具有 SQL 2017 的 Windows Server 2016 VM 映像](/azure-stack/operator/azure-stack-add-vm-image.md)。</span><span class="sxs-lookup"><span data-stu-id="d6515-135">[Windows Server 2016 with SQL 2017 VM image](/azure-stack/operator/azure-stack-add-vm-image.md) is available in the Azure Stack Hub Marketplace.</span></span> <span data-ttu-id="d6515-136">如果無法取得此映像，可與 Azure Stack Hub 操作員合作，以確實地將此映像新增至環境中。</span><span class="sxs-lookup"><span data-stu-id="d6515-136">If this image isn't available, work with an Azure Stack Hub Operator to ensure it's added to the environment.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="d6515-137">問題和考量</span><span class="sxs-lookup"><span data-stu-id="d6515-137">Issues and considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="d6515-138">延展性</span><span class="sxs-lookup"><span data-stu-id="d6515-138">Scalability</span></span>

<span data-ttu-id="d6515-139">跨雲端縮放的關鍵要素是能視需求立即在公用和內部部署雲端基礎結構之間提供縮放功能，證明服務能保持一致且可靠。</span><span class="sxs-lookup"><span data-stu-id="d6515-139">The key component of cross-cloud scaling is the ability to deliver immediate and on-demand scaling between public and on-premises cloud infrastructure, providing consistent and reliable service.</span></span>

### <a name="availability"></a><span data-ttu-id="d6515-140">可用性</span><span class="sxs-lookup"><span data-stu-id="d6515-140">Availability</span></span>

<span data-ttu-id="d6515-141">確定已透過內部部署硬體設定和軟體部署，針對本機部署應用程式的高可用性進行設定。</span><span class="sxs-lookup"><span data-stu-id="d6515-141">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="d6515-142">管理能力</span><span class="sxs-lookup"><span data-stu-id="d6515-142">Manageability</span></span>

<span data-ttu-id="d6515-143">跨雲端解決方案保證能在環境間提供無縫式管理和熟悉介面。</span><span class="sxs-lookup"><span data-stu-id="d6515-143">The cross-cloud solution ensures seamless management and familiar interface between environments.</span></span> <span data-ttu-id="d6515-144">建議您使用 PowerShell 進行跨平台管理。</span><span class="sxs-lookup"><span data-stu-id="d6515-144">PowerShell is recommended for cross-platform management.</span></span>

## <a name="cross-cloud-scaling"></a><span data-ttu-id="d6515-145">跨雲端縮放</span><span class="sxs-lookup"><span data-stu-id="d6515-145">Cross-cloud scaling</span></span>

### <a name="get-a-custom-domain-and-configure-dns"></a><span data-ttu-id="d6515-146">取得自訂網域並設定 DNS</span><span class="sxs-lookup"><span data-stu-id="d6515-146">Get a custom domain and configure DNS</span></span>

<span data-ttu-id="d6515-147">更新網域的 DNS 區域檔案。</span><span class="sxs-lookup"><span data-stu-id="d6515-147">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="d6515-148">Azure AD 會驗證自訂網域名稱的擁有權。</span><span class="sxs-lookup"><span data-stu-id="d6515-148">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="d6515-149">對於 Azure 中的 Azure/Microsoft 365/外部 DNS 記錄使用 [Azure DNS](/azure/dns/dns-getstarted-portal)，或在[不同的 DNS 註冊機構](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider)新增 DNS 項目。</span><span class="sxs-lookup"><span data-stu-id="d6515-149">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

1. <span data-ttu-id="d6515-150">向公用註冊機構註冊自訂網域。</span><span class="sxs-lookup"><span data-stu-id="d6515-150">Register a custom domain with a public registrar.</span></span>
2. <span data-ttu-id="d6515-151">登入網域的網域名稱註冊機構。</span><span class="sxs-lookup"><span data-stu-id="d6515-151">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="d6515-152">已核准的系統管理員可能需要進行 DNS 更新。</span><span class="sxs-lookup"><span data-stu-id="d6515-152">An approved admin may be required to make DNS updates.</span></span>
3. <span data-ttu-id="d6515-153">透過新增 Azure AD 提供的 DNS 項目來更新網域的 DNS 區域檔案。</span><span class="sxs-lookup"><span data-stu-id="d6515-153">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="d6515-154">(DNS 項目不會影響電子郵件路由或 Web 裝載行為。)</span><span class="sxs-lookup"><span data-stu-id="d6515-154">(The DNS entry won't affect email routing or web hosting behaviors.)</span></span>

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a><span data-ttu-id="d6515-155">在 Azure Stack Hub 中建立預設的多節點 Web 應用程式</span><span class="sxs-lookup"><span data-stu-id="d6515-155">Create a default multi-node web app in Azure Stack Hub</span></span>

<span data-ttu-id="d6515-156">設定混合式持續整合與持續部署 (CI/CD)，以將 Web 應用程式部署至 Azure 與 Azure Stack Hub，並自動將變更推送至這兩個雲端。</span><span class="sxs-lookup"><span data-stu-id="d6515-156">Set up hybrid continuous integration and continuous deployment (CI/CD) to deploy web apps to Azure and Azure Stack Hub and to autopush changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="d6515-157">需要適當映像摘要整合執行的 Azure Stack Hub (Windows Server 和 SQL) 及 App Service 部署。</span><span class="sxs-lookup"><span data-stu-id="d6515-157">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="d6515-158">如需詳細資訊，請參閱 App Service 文件[部署 App Service on Azure Stack Hub 的必要條件](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md)。</span><span class="sxs-lookup"><span data-stu-id="d6515-158">For more information, review the App Service documentation [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

### <a name="add-code-to-azure-repos"></a><span data-ttu-id="d6515-159">將程式碼新增至 Azure Repos</span><span class="sxs-lookup"><span data-stu-id="d6515-159">Add Code to Azure Repos</span></span>

<span data-ttu-id="d6515-160">Azure Repos</span><span class="sxs-lookup"><span data-stu-id="d6515-160">Azure Repos</span></span>

1. <span data-ttu-id="d6515-161">使用在 Azure Repos 上具有專案建立權限的帳戶登入 Azure Repos。</span><span class="sxs-lookup"><span data-stu-id="d6515-161">Sign in to Azure Repos with an account that has project creation rights on Azure Repos.</span></span>

    <span data-ttu-id="d6515-162">混合式 CI/CD 可同時套用至應用程式程式碼和基礎結構程式碼。</span><span class="sxs-lookup"><span data-stu-id="d6515-162">Hybrid CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="d6515-163">使用 [Azure Resource Manager 範本](https://azure.microsoft.com/resources/templates/)進行私用與託管的雲端開發。</span><span class="sxs-lookup"><span data-stu-id="d6515-163">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![連線到 Azure Repos 專案](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. <span data-ttu-id="d6515-165">建立並開啟預設 Web 應用程式以**複製存放庫**。</span><span class="sxs-lookup"><span data-stu-id="d6515-165">**Clone the repository** by creating and opening the default web app.</span></span>

    ![在 Azure Web 應用程式中複製存放庫](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="d6515-167">為這兩個雲端中的應用程式服務建立獨立的 Web 應用程式部署</span><span class="sxs-lookup"><span data-stu-id="d6515-167">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="d6515-168">編輯 **WebApplication.csproj** 檔案。</span><span class="sxs-lookup"><span data-stu-id="d6515-168">Edit the **WebApplication.csproj** file.</span></span> <span data-ttu-id="d6515-169">選取 `Runtimeidentifier` 並新增 `win10-x64`。</span><span class="sxs-lookup"><span data-stu-id="d6515-169">Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="d6515-170">(請參閱[獨立式部署](/dotnet/core/deploying/deploy-with-vs#simpleSelf)文件。)</span><span class="sxs-lookup"><span data-stu-id="d6515-170">(See [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![編輯 Web 應用程式專案檔](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. <span data-ttu-id="d6515-172">使用 Team Explorer 將程式碼簽入 Azure Repos 中。</span><span class="sxs-lookup"><span data-stu-id="d6515-172">Check in the code to Azure Repos using Team Explorer.</span></span>

3. <span data-ttu-id="d6515-173">確認應用程式程式碼已簽入 Azure Repos 中。</span><span class="sxs-lookup"><span data-stu-id="d6515-173">Confirm that the app code has been checked into Azure Repos.</span></span>

## <a name="create-the-build-definition"></a><span data-ttu-id="d6515-174">建立組建定義</span><span class="sxs-lookup"><span data-stu-id="d6515-174">Create the build definition</span></span>

1. <span data-ttu-id="d6515-175">登入 Azure Pipelines 以確認能夠建立組建定義。</span><span class="sxs-lookup"><span data-stu-id="d6515-175">Sign in to Azure Pipelines to confirm the ability to create build definitions.</span></span>

2. <span data-ttu-id="d6515-176">新增 **-r win10-x64** 程式碼。</span><span class="sxs-lookup"><span data-stu-id="d6515-176">Add **-r win10-x64** code.</span></span> <span data-ttu-id="d6515-177">新增的部分為觸發 .NNE Core 獨立部署所需的程式碼。</span><span class="sxs-lookup"><span data-stu-id="d6515-177">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![將程式碼加入 Web 應用程式](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. <span data-ttu-id="d6515-179">執行組建。</span><span class="sxs-lookup"><span data-stu-id="d6515-179">Run the build.</span></span> <span data-ttu-id="d6515-180">[獨立式部署組建](/dotnet/core/deploying/deploy-with-vs#simpleSelf)程序將會發佈可在 Azure 與 Azure Stack Hub 上執行的成品。</span><span class="sxs-lookup"><span data-stu-id="d6515-180">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that run on Azure and Azure Stack Hub.</span></span>

## <a name="use-an-azure-hosted-agent"></a><span data-ttu-id="d6515-181">使用 Azure 託管的代理程式</span><span class="sxs-lookup"><span data-stu-id="d6515-181">Use an Azure hosted agent</span></span>

<span data-ttu-id="d6515-182">在 Azure Pipelines 中使用託管的組建代理程式，是建置及部署 Web 應用程式的便利選項。</span><span class="sxs-lookup"><span data-stu-id="d6515-182">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="d6515-183">Microsoft Azure 會自動完成代理程式的維護和升級，以支援持續而不間斷的開發週期。</span><span class="sxs-lookup"><span data-stu-id="d6515-183">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="d6515-184">管理和設定 CD 程序</span><span class="sxs-lookup"><span data-stu-id="d6515-184">Manage and configure the CD process</span></span>

<span data-ttu-id="d6515-185">Azure Pipelines 與 Azure DevOps Services 提供具有高度設定和管理能力的管線，可用於對多個環境 (例如開發、暫存、QA 和生產環境) 的發行；包括特定階段需要核准。</span><span class="sxs-lookup"><span data-stu-id="d6515-185">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="d6515-186">建立發行定義</span><span class="sxs-lookup"><span data-stu-id="d6515-186">Create release definition</span></span>

1. <span data-ttu-id="d6515-187">在 Azure DevOps Services 的 [建置及發行]  區段的 [發行]  索引標籤下選取 [加號]  按鈕，以新增發行。</span><span class="sxs-lookup"><span data-stu-id="d6515-187">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![建立發行定義](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. <span data-ttu-id="d6515-189">套用 Azure App Service 部署範本。</span><span class="sxs-lookup"><span data-stu-id="d6515-189">Apply the Azure App Service Deployment template.</span></span>

   ![套用 Azure App Service 部署範本](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. <span data-ttu-id="d6515-191">在 [新增成品]  下，為 Azure 雲端建置應用程式新增成品。</span><span class="sxs-lookup"><span data-stu-id="d6515-191">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![將成品新增至 Azure 雲端組建](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. <span data-ttu-id="d6515-193">在 [管線] 索引標籤下選取環境的 [階段]、[工作]  連結，並設定 Azure 雲端環境值。</span><span class="sxs-lookup"><span data-stu-id="d6515-193">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![設定 Azure 雲端環境值](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. <span data-ttu-id="d6515-195">設定 [環境名稱]  ，並選取 Azure 雲端端點的 **Azure訂用帳戶**。</span><span class="sxs-lookup"><span data-stu-id="d6515-195">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![選取 Azure 訂用帳戶作為 Azure 雲端端點](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. <span data-ttu-id="d6515-197">在 [應用程式服務名稱]  下設定所需的 Azure 應用程式服務名稱。</span><span class="sxs-lookup"><span data-stu-id="d6515-197">Under **App service name**, set the required Azure app service name.</span></span>

      ![設定 Azure 應用程式服務名稱](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. <span data-ttu-id="d6515-199">在 Azure 雲端託管環境的**代理程式佇列**下輸入 "Hosted VS2017"。</span><span class="sxs-lookup"><span data-stu-id="d6515-199">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![為 Azure 雲端託管環境設定代理程式佇列](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. <span data-ttu-id="d6515-201">在 [部署 Azure App Service] 功能表中，為環境選取有效的**套件或資料夾**。</span><span class="sxs-lookup"><span data-stu-id="d6515-201">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="d6515-202">對**資料夾位置**選取 [確定]  。</span><span class="sxs-lookup"><span data-stu-id="d6515-202">Select **OK** to **folder location**.</span></span>
  
      ![選取適用於 Azure App Service 環境的套件或資料夾](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![選取適用於 Azure App Service 環境的套件或資料夾](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. <span data-ttu-id="d6515-205">儲存所有變更，並返回**發行管線**。</span><span class="sxs-lookup"><span data-stu-id="d6515-205">Save all changes and go back to **release pipeline**.</span></span>

    ![在發行管線中儲存變更](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. <span data-ttu-id="d6515-207">選取 Azure Stack Hub 應用程式的組建，以新增成品。</span><span class="sxs-lookup"><span data-stu-id="d6515-207">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![為 Azure Stack Hub 應用程式新增成品](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. <span data-ttu-id="d6515-209">套用 Azure App Service 部署，以便再新增一個環境。</span><span class="sxs-lookup"><span data-stu-id="d6515-209">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![將環境新增至 Azure App Service 部署](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. <span data-ttu-id="d6515-211">將新環境命名為 "Azure Stack"。</span><span class="sxs-lookup"><span data-stu-id="d6515-211">Name the new environment "Azure Stack".</span></span>

    ![為 Azure App Service 部署中的環境命名](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. <span data-ttu-id="d6515-213">在 [工作]  索引標籤下找出 Azure Stack 環境。</span><span class="sxs-lookup"><span data-stu-id="d6515-213">Find the Azure Stack environment under **Task** tab.</span></span>

    ![Azure Stack 環境](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. <span data-ttu-id="d6515-215">選取 Azure Stack 端點的訂用帳戶。</span><span class="sxs-lookup"><span data-stu-id="d6515-215">Select the subscription for the Azure Stack endpoint.</span></span>

    ![選取 Azure Stack 端點的訂用帳戶](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. <span data-ttu-id="d6515-217">將 Azure Stack Web 應用程式名稱設定為 App Service 名稱。</span><span class="sxs-lookup"><span data-stu-id="d6515-217">Set the Azure Stack web app name as the App service name.</span></span>
    <span data-ttu-id="d6515-218">![設定 Azure Stack Web 應用程式名稱](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span><span class="sxs-lookup"><span data-stu-id="d6515-218">![Set Azure Stack web app name](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span></span>

16. <span data-ttu-id="d6515-219">選取 [Azure Stack 代理程式]。</span><span class="sxs-lookup"><span data-stu-id="d6515-219">Select the Azure Stack agent.</span></span>

    ![選取 Azure Stack 代理程式](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. <span data-ttu-id="d6515-221">在 [部署 Azure App Service] 區段下，為環境選取有效的**套件或資料夾**。</span><span class="sxs-lookup"><span data-stu-id="d6515-221">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="d6515-222">對資料夾位置選取 [確定]  。</span><span class="sxs-lookup"><span data-stu-id="d6515-222">Select **OK** to folder location.</span></span>

    ![為 Azure App Service 部署選取資料夾](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![為 Azure App Service 部署選取資料夾](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. <span data-ttu-id="d6515-225">在 [變數] 索引標籤下新增名為 `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` 的變數，並將其值設定為 **true**，範圍設定為 Azure Stack。</span><span class="sxs-lookup"><span data-stu-id="d6515-225">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack.</span></span>

    ![將變數新增至 Azure 應用程式部署](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. <span data-ttu-id="d6515-227">選取兩個成品中的 [持續部署觸發程序]  圖示，並啟用**持續**部署觸發程序。</span><span class="sxs-lookup"><span data-stu-id="d6515-227">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![選取持續部署觸發程序](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. <span data-ttu-id="d6515-229">選取 Azure Stack 環境中的 [預先部署]  條件圖示，並將觸發程序設定為 [發行之後]  。</span><span class="sxs-lookup"><span data-stu-id="d6515-229">Select the **Pre-deployment** conditions icon in the Azure Stack environment and set the trigger to **After release.**</span></span>

    ![選取部署前的條件](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. <span data-ttu-id="d6515-231">儲存所有變更。</span><span class="sxs-lookup"><span data-stu-id="d6515-231">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="d6515-232">工作的某些設定可能已在從範本建立發行定義時自動定義為[環境變數](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables)。</span><span class="sxs-lookup"><span data-stu-id="d6515-232">Some settings for the tasks may have been automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="d6515-233">這些設定無法在工作設定中修改；而是必須選取父環境項目才能編輯這些設定。</span><span class="sxs-lookup"><span data-stu-id="d6515-233">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a><span data-ttu-id="d6515-234">透過 Visual Studio 發佈至 Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="d6515-234">Publish to Azure Stack Hub via Visual Studio</span></span>

<span data-ttu-id="d6515-235">藉由建立端點，Azure DevOps Services 組建可以將 Azure 服務應用程式部署到 Azure Stack Hub。</span><span class="sxs-lookup"><span data-stu-id="d6515-235">By creating endpoints, an Azure DevOps Services build can deploy Azure Service apps to Azure Stack Hub.</span></span> <span data-ttu-id="d6515-236">Azure Pipelines 會連線至組建代理程式，後者再連線至 Azure Stack Hub。</span><span class="sxs-lookup"><span data-stu-id="d6515-236">Azure Pipelines connects to the build agent, which connects to Azure Stack Hub.</span></span>

1. <span data-ttu-id="d6515-237">登入 Azure DevOps Services，並移至 [應用程式設定] 頁面。</span><span class="sxs-lookup"><span data-stu-id="d6515-237">Sign in to Azure DevOps Services and go to the app settings page.</span></span>

2. <span data-ttu-id="d6515-238">在 [設定]  上，選取 [安全性]  。</span><span class="sxs-lookup"><span data-stu-id="d6515-238">On **Settings**, select **Security**.</span></span>

3. <span data-ttu-id="d6515-239">在 [VSTS 群組]  中，選取 [端點建立者]  。</span><span class="sxs-lookup"><span data-stu-id="d6515-239">In **VSTS Groups**, select **Endpoint Creators**.</span></span>

4. <span data-ttu-id="d6515-240">在 [成員]  索引標籤上，選取 [新增]  。</span><span class="sxs-lookup"><span data-stu-id="d6515-240">On the **Members** tab, select **Add**.</span></span>

5. <span data-ttu-id="d6515-241">在 [新增使用者和群組]  中，輸入使用者名稱，然後從使用者清單中選取該使用者。</span><span class="sxs-lookup"><span data-stu-id="d6515-241">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

6. <span data-ttu-id="d6515-242">選取 [儲存變更]  。</span><span class="sxs-lookup"><span data-stu-id="d6515-242">Select **Save changes**.</span></span>

7. <span data-ttu-id="d6515-243">在 [VSTS 群組]  清單中，選取 [端點管理員]  。</span><span class="sxs-lookup"><span data-stu-id="d6515-243">In the **VSTS Groups** list, select **Endpoint Administrators**.</span></span>

8. <span data-ttu-id="d6515-244">在 [成員]  索引標籤上，選取 [新增]  。</span><span class="sxs-lookup"><span data-stu-id="d6515-244">On the **Members** tab, select **Add**.</span></span>

9. <span data-ttu-id="d6515-245">在 [新增使用者和群組]  中，輸入使用者名稱，然後從使用者清單中選取該使用者。</span><span class="sxs-lookup"><span data-stu-id="d6515-245">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

10. <span data-ttu-id="d6515-246">選取 [儲存變更]  。</span><span class="sxs-lookup"><span data-stu-id="d6515-246">Select **Save changes**.</span></span>

<span data-ttu-id="d6515-247">既然端點資訊已存在，所以 Azure Pipelines 對 Azure Stack Hub 的連線已可供使用。</span><span class="sxs-lookup"><span data-stu-id="d6515-247">Now that the endpoint information exists, the Azure Pipelines to Azure Stack Hub connection is ready to use.</span></span> <span data-ttu-id="d6515-248">Azure Stack Hub 中的組建代理程式會取得來自 Azure Pipelines 的指示，然後代理程式會傳達與 Azure Stack Hub 進行通訊所需的端點資訊。</span><span class="sxs-lookup"><span data-stu-id="d6515-248">The build agent in Azure Stack Hub gets instructions from Azure Pipelines and then the agent conveys endpoint information for communication with Azure Stack Hub.</span></span>

## <a name="develop-the-app-build"></a><span data-ttu-id="d6515-249">開發應用程式組建</span><span class="sxs-lookup"><span data-stu-id="d6515-249">Develop the app build</span></span>

> [!Note]  
> <span data-ttu-id="d6515-250">需要適當映像摘要整合執行的 Azure Stack Hub (Windows Server 和 SQL) 及 App Service 部署。</span><span class="sxs-lookup"><span data-stu-id="d6515-250">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="d6515-251">如需詳細資訊，請參閱[部署 Azure Stack Hub 上的 App Service 必要條件](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md)。</span><span class="sxs-lookup"><span data-stu-id="d6515-251">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

<span data-ttu-id="d6515-252">請使用 [Azure Resource Manager 範本](https://azure.microsoft.com/resources/templates/) (例如來自 Azure Repos 的 Web 應用程式程式碼) 以部署至這兩個雲端。</span><span class="sxs-lookup"><span data-stu-id="d6515-252">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) like web app code from Azure Repos to deploy to both clouds.</span></span>

### <a name="add-code-to-an-azure-repos-project"></a><span data-ttu-id="d6515-253">將程式碼新增至 Azure Repos 專案</span><span class="sxs-lookup"><span data-stu-id="d6515-253">Add code to an Azure Repos project</span></span>

1. <span data-ttu-id="d6515-254">使用在 Azure Stack Hub 上具有專案建立權限的帳戶登入 Azure Repos。</span><span class="sxs-lookup"><span data-stu-id="d6515-254">Sign in to Azure Repos with an account that has project creation rights on Azure Stack Hub.</span></span>

2. <span data-ttu-id="d6515-255">建立並開啟預設 Web 應用程式以**複製存放庫**。</span><span class="sxs-lookup"><span data-stu-id="d6515-255">**Clone the repository** by creating and opening the default web app.</span></span>

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="d6515-256">為這兩個雲端中的應用程式服務建立獨立的 Web 應用程式部署</span><span class="sxs-lookup"><span data-stu-id="d6515-256">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="d6515-257">編輯 **WebApplication.csproj**檔案：選取 `Runtimeidentifier`，然後新增 `win10-x64`。</span><span class="sxs-lookup"><span data-stu-id="d6515-257">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and then add `win10-x64`.</span></span> <span data-ttu-id="d6515-258">如需詳細資訊，請參閱[獨立式部署](/dotnet/core/deploying/deploy-with-vs#simpleSelf)文件。</span><span class="sxs-lookup"><span data-stu-id="d6515-258">For more information, see [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.</span></span>

2. <span data-ttu-id="d6515-259">使用 Team Explorer 將程式碼簽入 Azure Repos 中。</span><span class="sxs-lookup"><span data-stu-id="d6515-259">Use Team Explorer to check the code into Azure Repos.</span></span>

3. <span data-ttu-id="d6515-260">確認應用程式程式碼已簽入 Azure Repos 中。</span><span class="sxs-lookup"><span data-stu-id="d6515-260">Confirm that the app code was checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="d6515-261">建立組建定義</span><span class="sxs-lookup"><span data-stu-id="d6515-261">Create the build definition</span></span>

1. <span data-ttu-id="d6515-262">使用可建立組建定義的帳戶登入 Azure Pipelines。</span><span class="sxs-lookup"><span data-stu-id="d6515-262">Sign in to Azure Pipelines with an account that can create a build definition.</span></span>

2. <span data-ttu-id="d6515-263">移至專案的 [建置 Web 應用程式]  頁面。</span><span class="sxs-lookup"><span data-stu-id="d6515-263">Go to the **Build Web Application** page for the project.</span></span>

3. <span data-ttu-id="d6515-264">在 [引數]  中，新增 **-r win10-x64** 程式碼。</span><span class="sxs-lookup"><span data-stu-id="d6515-264">In **Arguments**, add **-r win10-x64** code.</span></span> <span data-ttu-id="d6515-265">想要透過 .NET Core 觸發獨立式部署就必須新增此項目。</span><span class="sxs-lookup"><span data-stu-id="d6515-265">This addition is required to trigger a self-contained deployment with .NET Core.</span></span>

4. <span data-ttu-id="d6515-266">執行組建。</span><span class="sxs-lookup"><span data-stu-id="d6515-266">Run the build.</span></span> <span data-ttu-id="d6515-267">[獨立式部署組建](/dotnet/core/deploying/deploy-with-vs#simpleSelf)程序將會發佈可在 Azure 與 Azure Stack Hub 上執行的成品。</span><span class="sxs-lookup"><span data-stu-id="d6515-267">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="use-an-azure-hosted-build-agent"></a><span data-ttu-id="d6515-268">使用 Azure 託管的組建代理程式</span><span class="sxs-lookup"><span data-stu-id="d6515-268">Use an Azure hosted build agent</span></span>

<span data-ttu-id="d6515-269">在 Azure Pipelines 中使用託管的組建代理程式，是建置及部署 Web 應用程式的便利選項。</span><span class="sxs-lookup"><span data-stu-id="d6515-269">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="d6515-270">Microsoft Azure 會自動完成代理程式的維護和升級，以支援持續而不間斷的開發週期。</span><span class="sxs-lookup"><span data-stu-id="d6515-270">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="configure-the-continuous-deployment-cd-process"></a><span data-ttu-id="d6515-271">設定持續部署 (CD) 程序</span><span class="sxs-lookup"><span data-stu-id="d6515-271">Configure the continuous deployment (CD) process</span></span>

<span data-ttu-id="d6515-272">Azure Pipelines 與 Azure DevOps Services 提供具有高度設定和管理能力的管線，可用於對多個環境 (例如開發、暫存、品質保證 (QA) 和生產環境) 的發行。</span><span class="sxs-lookup"><span data-stu-id="d6515-272">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, quality assurance (QA), and production.</span></span> <span data-ttu-id="d6515-273">此程序中可以包括在應用程式生命週期的特定階段要求核准。</span><span class="sxs-lookup"><span data-stu-id="d6515-273">This process can include requiring approvals at specific stages of the app life cycle.</span></span>

#### <a name="create-release-definition"></a><span data-ttu-id="d6515-274">建立發行定義</span><span class="sxs-lookup"><span data-stu-id="d6515-274">Create release definition</span></span>

<span data-ttu-id="d6515-275">建立發行定義是應用程式建置程序的最後一個步驟。</span><span class="sxs-lookup"><span data-stu-id="d6515-275">Creating a release definition is the final step in the app build process.</span></span> <span data-ttu-id="d6515-276">這個發行定義可用來建立發行並部署組建。</span><span class="sxs-lookup"><span data-stu-id="d6515-276">This release definition is used to create a release and deploy a build.</span></span>

1. <span data-ttu-id="d6515-277">登入 Azure Pipelines，並移至專案的 [建置及發行]  。</span><span class="sxs-lookup"><span data-stu-id="d6515-277">Sign in to Azure Pipelines and go to **Build and Release** for the project.</span></span>

2. <span data-ttu-id="d6515-278">在 [發行]  索引標籤上，選取 [ + ]  ，然後挑選 [建立發行定義]  。</span><span class="sxs-lookup"><span data-stu-id="d6515-278">On the **Releases** tab, select **[ + ]** and then pick **Create release definition**.</span></span>

3. <span data-ttu-id="d6515-279">在 [選取範本]  中，選擇 [Azure App Service 部署]  ，然後選取 [套用]  。</span><span class="sxs-lookup"><span data-stu-id="d6515-279">On **Select a Template**, choose **Azure App Service Deployment**, and then select **Apply**.</span></span>

4. <span data-ttu-id="d6515-280">在 [新增成品]  上，從 [來源 /(組建定義/)]  中選取 [Azure 雲端] 組建應用程式。</span><span class="sxs-lookup"><span data-stu-id="d6515-280">On **Add artifact**, from the **Source (Build definition)**, select the Azure Cloud build app.</span></span>

5. <span data-ttu-id="d6515-281">在 [管線]  索引標籤上，選取 [檢視環境工作]  的 [1 階段 1 個工作]   連結。</span><span class="sxs-lookup"><span data-stu-id="d6515-281">On the **Pipeline** tab, select the **1 Phase**, **1 Task** link to **View environment tasks**.</span></span>

6. <span data-ttu-id="d6515-282">在 [工作]  索引標籤上，輸入「Azure」作為 [環境名稱]  ，然後從 [Azure 訂用帳戶]  清單選取 [AzureCloud Traders-Web EP]。</span><span class="sxs-lookup"><span data-stu-id="d6515-282">On the **Tasks** tab, enter Azure as the **Environment name** and select the AzureCloud Traders-Web EP from the **Azure subscription** list.</span></span>

7. <span data-ttu-id="d6515-283">輸入 **Azure App Service 名稱**，也就是下一個螢幕擷取畫面中的 `northwindtraders`。</span><span class="sxs-lookup"><span data-stu-id="d6515-283">Enter the **Azure app service name**, which is `northwindtraders` in the next screen capture.</span></span>

8. <span data-ttu-id="d6515-284">在 [代理程式階段] 中，從 [代理程式佇列]  清單中選取 [Hosted VS2017]  。</span><span class="sxs-lookup"><span data-stu-id="d6515-284">For the Agent phase, select **Hosted VS2017** from the **Agent queue** list.</span></span>

9. <span data-ttu-id="d6515-285">在 [部署 Azure App Service]  中，為環境選取有效的**套件或資料夾**。</span><span class="sxs-lookup"><span data-stu-id="d6515-285">In **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span>

10. <span data-ttu-id="d6515-286">在 [選取檔案或資料夾]  中，對 [位置]  選取 [確定]  。</span><span class="sxs-lookup"><span data-stu-id="d6515-286">In **Select File or Folder**, select **OK** to **Location**.</span></span>

11. <span data-ttu-id="d6515-287">儲存所有變更，並返回 [管線]  。</span><span class="sxs-lookup"><span data-stu-id="d6515-287">Save all changes and go back to **Pipeline**.</span></span>

12. <span data-ttu-id="d6515-288">在 [管線]  索引標籤上，選取 [新增成品]  ，然後從 [來源 (組建定義)]  清單中選擇 [NorthwindCloud Traders-Vessel]  。</span><span class="sxs-lookup"><span data-stu-id="d6515-288">On the **Pipeline** tab, select **Add artifact**, and choose the **NorthwindCloud Traders-Vessel** from the **Source (Build Definition)** list.</span></span>

13. <span data-ttu-id="d6515-289">在 [選取範本]  上，新增另一個環境。</span><span class="sxs-lookup"><span data-stu-id="d6515-289">On **Select a Template**, add another environment.</span></span> <span data-ttu-id="d6515-290">挑選 [Azure App Service 部署]  ，然後選取 [套用]  。</span><span class="sxs-lookup"><span data-stu-id="d6515-290">Pick **Azure App Service Deployment** and then select **Apply**.</span></span>

14. <span data-ttu-id="d6515-291">輸入 `Azure Stack Hub` 作為 [環境名稱]  。</span><span class="sxs-lookup"><span data-stu-id="d6515-291">Enter `Azure Stack Hub` as the **Environment name**.</span></span>

15. <span data-ttu-id="d6515-292">在 [工作]  索引標籤上，尋找並選取 Azure Stack Hub。</span><span class="sxs-lookup"><span data-stu-id="d6515-292">On the **Tasks** tab, find and select Azure Stack Hub.</span></span>

16. <span data-ttu-id="d6515-293">從 [Azure 訂用帳戶]  清單中，選取 [AzureStack Traders-Vessel EP]  作為 Azure Stack Hub 端點。</span><span class="sxs-lookup"><span data-stu-id="d6515-293">From the **Azure subscription** list, select **AzureStack Traders-Vessel EP** for the Azure Stack Hub endpoint.</span></span>

17. <span data-ttu-id="d6515-294">輸入 Azure Stack Hub Web 應用程式名作為 **App Service 名稱**。</span><span class="sxs-lookup"><span data-stu-id="d6515-294">Enter the Azure Stack Hub web app name as the **App service name**.</span></span>

18. <span data-ttu-id="d6515-295">在 [代理程式選擇]  底下，從 [代理程式佇列]  清單中挑選 **AzureStack -b Douglas Fir**。</span><span class="sxs-lookup"><span data-stu-id="d6515-295">Under **Agent selection**, pick **AzureStack -b Douglas Fir** from the **Agent queue** list.</span></span>

19. <span data-ttu-id="d6515-296">在 [部署 Azure App Service]  中，為環境選取有效的**套件或資料夾**。</span><span class="sxs-lookup"><span data-stu-id="d6515-296">For **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span> <span data-ttu-id="d6515-297">在 [選取檔案或資料夾]  中，對 [位置]  資料夾選取 [確定]  。</span><span class="sxs-lookup"><span data-stu-id="d6515-297">On **Select File Or Folder**, select **OK** for the folder **Location**.</span></span>

20. <span data-ttu-id="d6515-298">在 [變數]  索引標籤上，尋找名為 `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` 的變數。</span><span class="sxs-lookup"><span data-stu-id="d6515-298">On the **Variable** tab, find the variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`.</span></span> <span data-ttu-id="d6515-299">將變數值設定為 **true**，並將其範圍設定為 [Azure Stack Hub]  。</span><span class="sxs-lookup"><span data-stu-id="d6515-299">Set the variable value to **true**, and set its scope to **Azure Stack Hub**.</span></span>

21. <span data-ttu-id="d6515-300">在 [管線]  索引標籤上，選取 NorthwindCloud Traders-Web 成品的 [持續部署觸發程序]  圖示，並將 [持續部署觸發程序]  設定為 [啟用]  。</span><span class="sxs-lookup"><span data-stu-id="d6515-300">On the **Pipeline** tab, select the **Continuous deployment trigger** icon for the NorthwindCloud Traders-Web artifact and set the **Continuous deployment trigger** to **Enabled**.</span></span> <span data-ttu-id="d6515-301">針對 **NorthwindCloud Traders-Vessel** 成品執行相同的動作。</span><span class="sxs-lookup"><span data-stu-id="d6515-301">Do the same thing for the **NorthwindCloud Traders-Vessel** artifact.</span></span>

22. <span data-ttu-id="d6515-302">針對 Azure Stack Hub 環境，選取 [部署前的條件]  圖示，將觸發程序設定為 [發行之後]  。</span><span class="sxs-lookup"><span data-stu-id="d6515-302">For the Azure Stack Hub environment, select the **Pre-deployment conditions** icon set the trigger to **After release**.</span></span>

23. <span data-ttu-id="d6515-303">儲存所有變更。</span><span class="sxs-lookup"><span data-stu-id="d6515-303">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="d6515-304">發行工作的某些設定已在從範本建立發行定義時自動定義為[環境變數](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables)。</span><span class="sxs-lookup"><span data-stu-id="d6515-304">Some settings for release tasks are automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="d6515-305">這些設定無法在工作設定中修改，但可在父環境項目中修改。</span><span class="sxs-lookup"><span data-stu-id="d6515-305">These settings can't be modified in the task settings but can be modified in the parent environment items.</span></span>

## <a name="create-a-release"></a><span data-ttu-id="d6515-306">建立發行</span><span class="sxs-lookup"><span data-stu-id="d6515-306">Create a release</span></span>

1. <span data-ttu-id="d6515-307">在 [管線]  索引標籤上，開啟 [發行]  清單，然後選取 [建立發行]  。</span><span class="sxs-lookup"><span data-stu-id="d6515-307">On the **Pipeline** tab, open the **Release** list and select **Create release**.</span></span>

2. <span data-ttu-id="d6515-308">輸入發行的說明，確認已選取正確的成品，然後選取 [建立]  。</span><span class="sxs-lookup"><span data-stu-id="d6515-308">Enter a description for the release, check to see that the correct artifacts are selected, and then select **Create**.</span></span> <span data-ttu-id="d6515-309">幾分鐘之後將會出現一個橫幅，指出新的發行已建立，且發行名稱會顯示為連結。</span><span class="sxs-lookup"><span data-stu-id="d6515-309">After a few moments, a banner appears indicating that the new release was created and the release name is displayed as a link.</span></span> <span data-ttu-id="d6515-310">選取連結以查看 [發行摘要] 頁面。</span><span class="sxs-lookup"><span data-stu-id="d6515-310">Select the link to see the release summary page.</span></span>

3. <span data-ttu-id="d6515-311">[發行摘要] 頁面會顯示有關發行的詳細資料。</span><span class="sxs-lookup"><span data-stu-id="d6515-311">The release summary page shows details about the release.</span></span> <span data-ttu-id="d6515-312">在下列 "Release-2" 螢幕擷取畫面中，[環境]  區段顯示 Azure 的 [部署狀態]  為 [進行中]，Azure Stack Hub 的狀態為 [成功]。</span><span class="sxs-lookup"><span data-stu-id="d6515-312">In the following screen capture for "Release-2", the **Environments** section shows the **Deployment status** for Azure as "IN PROGRESS", and the status for Azure Stack Hub is "SUCCEEDED".</span></span> <span data-ttu-id="d6515-313">當 Azure 環境的部署狀態變更為 [成功] 時，便會出現橫幅指出發行已可供核准。</span><span class="sxs-lookup"><span data-stu-id="d6515-313">When the deployment status for the Azure environment changes to "SUCCEEDED", a banner appears indicating that the release is ready for approval.</span></span> <span data-ttu-id="d6515-314">對部署若擱置或失敗，將會顯示藍色的 **(i)** 資訊圖示。</span><span class="sxs-lookup"><span data-stu-id="d6515-314">When a deployment is pending or has failed, a blue **(i)** information icon is shown.</span></span> <span data-ttu-id="d6515-315">將滑鼠暫留在圖示上，即可查看快顯，其中會包含延遲或失敗的原因。</span><span class="sxs-lookup"><span data-stu-id="d6515-315">Hover over the icon to see a pop-up that contains the reason for delay or failure.</span></span>

4. <span data-ttu-id="d6515-316">其他檢視 (例如發行清單) 也會顯示指出核准擱置中的圖示。</span><span class="sxs-lookup"><span data-stu-id="d6515-316">Other views, like the list of releases, also display an icon that indicates approval is pending.</span></span> <span data-ttu-id="d6515-317">這個圖示的快顯會顯示環境名稱以及更多與部署相關的詳細資料。</span><span class="sxs-lookup"><span data-stu-id="d6515-317">The pop-up for this icon shows the environment name and more details related to the deployment.</span></span> <span data-ttu-id="d6515-318">管理員可輕鬆查看發行的整體進度，以及查看哪些版本正在等待核准。</span><span class="sxs-lookup"><span data-stu-id="d6515-318">It's easy for an admin see the overall progress of releases and see which releases are waiting for approval.</span></span>

## <a name="monitor-and-track-deployments"></a><span data-ttu-id="d6515-319">監視和追蹤部署</span><span class="sxs-lookup"><span data-stu-id="d6515-319">Monitor and track deployments</span></span>

1. <span data-ttu-id="d6515-320">在 [Release-2]  摘要頁面上，選取 [記錄]  。</span><span class="sxs-lookup"><span data-stu-id="d6515-320">On the **Release-2** summary page, select **Logs**.</span></span> <span data-ttu-id="d6515-321">在部署期間，此頁面會顯示來自代理程式的即時記錄。</span><span class="sxs-lookup"><span data-stu-id="d6515-321">During a deployment, this page shows the live log from the agent.</span></span> <span data-ttu-id="d6515-322">左窗格會顯示每個環境部署中每個作業的狀態。</span><span class="sxs-lookup"><span data-stu-id="d6515-322">The left pane shows the status of each operation in the deployment for each environment.</span></span>

2. <span data-ttu-id="d6515-323">選取部署前或部署後核准 [動作]  資料行中的人形圖示，以查看哪些人已核准 (或拒絕) 部署，以及該人員提供的訊息。</span><span class="sxs-lookup"><span data-stu-id="d6515-323">Select the person icon in the **Action** column for a pre-deployment or post-deployment approval to see who approved (or rejected) the deployment and the message they provided.</span></span>

3. <span data-ttu-id="d6515-324">部署完成後，整個記錄檔會顯示在右窗格中。</span><span class="sxs-lookup"><span data-stu-id="d6515-324">After the deployment finishes, the entire log file is displayed in the right pane.</span></span> <span data-ttu-id="d6515-325">在左窗格中選取任何 [步驟]  ，以查看單一步驟的記錄檔，例如「初始化作業」  。</span><span class="sxs-lookup"><span data-stu-id="d6515-325">Select any **Step** in the left pane to see the log file for a single step, like **Initialize Job**.</span></span> <span data-ttu-id="d6515-326">能夠查看個別記錄，可讓您輕鬆地針對整體部署的各個部分進行追蹤和偵錯。</span><span class="sxs-lookup"><span data-stu-id="d6515-326">The ability to see individual logs makes it easier to trace and debug parts of the overall deployment.</span></span> <span data-ttu-id="d6515-327">[儲存]  步驟的記錄，或是 [將所有記錄下載為 zip]  。</span><span class="sxs-lookup"><span data-stu-id="d6515-327">**Save** the log file for a step or **Download all logs as zip**.</span></span>

4. <span data-ttu-id="d6515-328">開啟 [摘要]  索引標籤可查看發行的一般資訊。</span><span class="sxs-lookup"><span data-stu-id="d6515-328">Open the **Summary** tab to see general information about the release.</span></span> <span data-ttu-id="d6515-329">此檢視會顯示組建的詳細資料、組件所部署到的環境、部署狀態，以及其他關於發行的資訊。</span><span class="sxs-lookup"><span data-stu-id="d6515-329">This view shows details about the build, the environments it was deployed to, deployment status, and other information about the release.</span></span>

5. <span data-ttu-id="d6515-330">選取環境連結 (**Azure** 或 **Azure Stack Hub**) 以查看特定環境現有與擱置中部署的資訊。</span><span class="sxs-lookup"><span data-stu-id="d6515-330">Select an environment link (**Azure** or **Azure Stack Hub**) to see information about existing and pending deployments to a specific environment.</span></span> <span data-ttu-id="d6515-331">使用這些檢視可快速確認同一個組建已部署至兩個環境。</span><span class="sxs-lookup"><span data-stu-id="d6515-331">Use these views as a quick way to check that the same build was deployed to both environments.</span></span>

6. <span data-ttu-id="d6515-332">在瀏覽器中開啟**已部署的生產應用程式**。</span><span class="sxs-lookup"><span data-stu-id="d6515-332">Open the **deployed production app** in a browser.</span></span> <span data-ttu-id="d6515-333">例如，針對 Azure App Service 網站，請開啟 URL `https://[your-app-name\].azurewebsites.net`。</span><span class="sxs-lookup"><span data-stu-id="d6515-333">For example, for the Azure App Services website, open the URL `https://[your-app-name\].azurewebsites.net`.</span></span>

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a><span data-ttu-id="d6515-334">Azure 與 Azure Stack Hub 的整合提供可調整的跨雲端解決方案</span><span class="sxs-lookup"><span data-stu-id="d6515-334">Integration of Azure and Azure Stack Hub provides a scalable cross-cloud solution</span></span>

<span data-ttu-id="d6515-335">具彈性且完善的多雲端服務可提供資料安全性、備份和備援、一致且快速的可用性、可縮放的儲存體和散發，以及異地相容的路由。</span><span class="sxs-lookup"><span data-stu-id="d6515-335">A flexible and robust multi-cloud service provides data security, back up and redundancy, consistent and rapid availability, scalable storage and distribution, and geo-compliant routing.</span></span> <span data-ttu-id="d6515-336">此手動觸發程序可確實地在託管的 Web 應用程式之間提供可靠且有效率的負載切換，確保重要資料的立即可用性。</span><span class="sxs-lookup"><span data-stu-id="d6515-336">This manually triggered process ensures reliable and efficient load switching between hosted web apps and immediate availability of crucial data.</span></span>

## <a name="next-steps"></a><span data-ttu-id="d6515-337">後續步驟</span><span class="sxs-lookup"><span data-stu-id="d6515-337">Next steps</span></span>

- <span data-ttu-id="d6515-338">若要深入了解 Azure 雲端模式，請參閱[雲端設計模式](/azure/architecture/patterns)。</span><span class="sxs-lookup"><span data-stu-id="d6515-338">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
