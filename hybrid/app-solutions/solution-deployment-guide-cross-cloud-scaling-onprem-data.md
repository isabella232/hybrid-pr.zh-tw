---
title: 使用內部部署資料部署可跨雲端調整的混合式應用程式
description: 了解如何部署使用內部部署資料的應用程式，並使用 Azure 和 Azure Stack Hub 進行跨雲端調整。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ecc42a94e2c59531b2a2e933772b0d8ce8c58609
ms.sourcegitcommit: 0d5b5336bdb969588d0b92e04393e74b8f682c3b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/22/2020
ms.locfileid: "92353473"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a><span data-ttu-id="819f6-103">使用內部部署資料部署可跨雲端調整的混合式應用程式</span><span class="sxs-lookup"><span data-stu-id="819f6-103">Deploy hybrid app with on-premises data that scales cross-cloud</span></span>

<span data-ttu-id="819f6-104">本解決方案指南說明如何部署跨 Azure 和 Azure Stack Hub，並使用單一內部部署資料來源的混合式應用程式。</span><span class="sxs-lookup"><span data-stu-id="819f6-104">This solution guide shows you how to deploy a hybrid app that spans both Azure and Azure Stack Hub and uses a single on-premises data source.</span></span>

<span data-ttu-id="819f6-105">藉由使用混合式雲端解決方案，您將可結合私人雲端在合規性方面的優勢與公用雲端的延展性。</span><span class="sxs-lookup"><span data-stu-id="819f6-105">By using a hybrid cloud solution, you can combine the compliance benefits of a private cloud with the scalability of the public cloud.</span></span> <span data-ttu-id="819f6-106">您的開發人員可以善用 Microsoft 開發人員生態系統，並將其技能運用在雲端和內部部署環境中。</span><span class="sxs-lookup"><span data-stu-id="819f6-106">Your developers can also take advantage of the Microsoft developer ecosystem and apply their skills to the cloud and on-premises environments.</span></span>

## <a name="overview-and-assumptions"></a><span data-ttu-id="819f6-107">概觀與假設</span><span class="sxs-lookup"><span data-stu-id="819f6-107">Overview and assumptions</span></span>

<span data-ttu-id="819f6-108">依照本教學課程設定工作流程，讓開發人員將相同的 Web 應用程式部署至公用雲端和私人雲端。</span><span class="sxs-lookup"><span data-stu-id="819f6-108">Follow this tutorial to set up a workflow that lets developers deploy an identical web app to a public cloud and a private cloud.</span></span> <span data-ttu-id="819f6-109">此應用程式可存取託管於私人雲端上的非網際網路可路由網路。</span><span class="sxs-lookup"><span data-stu-id="819f6-109">This app can access a non-internet routable network hosted on the private cloud.</span></span> <span data-ttu-id="819f6-110">這些 Web 應用程式會受到監視，且在流量暴增時，將會以程式修改 DNS 記錄，以將流量重新導向至公用雲端。</span><span class="sxs-lookup"><span data-stu-id="819f6-110">These web apps are monitored and when there's a spike in traffic, a program modifies the DNS records to redirect traffic to the public cloud.</span></span> <span data-ttu-id="819f6-111">當流量降至暴增之前的水準時，流量就會恢復為路由至私人雲端。</span><span class="sxs-lookup"><span data-stu-id="819f6-111">When traffic drops to the level before the spike, traffic is routed back to the private cloud.</span></span>

<span data-ttu-id="819f6-112">本教學課程涵蓋下列工作：</span><span class="sxs-lookup"><span data-stu-id="819f6-112">This tutorial covers the following tasks:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="819f6-113">部署混合式連線 SQL Server 資料庫伺服器。</span><span class="sxs-lookup"><span data-stu-id="819f6-113">Deploy a hybrid-connected SQL Server database server.</span></span>
> - <span data-ttu-id="819f6-114">將全域 Azure 中的 Web 應用程式連線至混合式網路。</span><span class="sxs-lookup"><span data-stu-id="819f6-114">Connect a web app in global Azure to a hybrid network.</span></span>
> - <span data-ttu-id="819f6-115">設定跨雲端縮放的 DNS。</span><span class="sxs-lookup"><span data-stu-id="819f6-115">Configure DNS for cross-cloud scaling.</span></span>
> - <span data-ttu-id="819f6-116">設定跨雲端縮放的 SSL 憑證。</span><span class="sxs-lookup"><span data-stu-id="819f6-116">Configure SSL certificates for cross-cloud scaling.</span></span>
> - <span data-ttu-id="819f6-117">設定及部署 Web 應用程式。</span><span class="sxs-lookup"><span data-stu-id="819f6-117">Configure and deploy the web app.</span></span>
> - <span data-ttu-id="819f6-118">建立流量管理員設定檔，並進行其跨雲端縮放的設定。</span><span class="sxs-lookup"><span data-stu-id="819f6-118">Create a Traffic Manager profile and configure it for cross-cloud scaling.</span></span>
> - <span data-ttu-id="819f6-119">針對增加的流量設定 Application Insights 監視和警示。</span><span class="sxs-lookup"><span data-stu-id="819f6-119">Set up Application Insights monitoring and alerting for increased traffic.</span></span>
> - <span data-ttu-id="819f6-120">設定全域 Azure 和 Azure Stack Hub 之間的自動流量切換。</span><span class="sxs-lookup"><span data-stu-id="819f6-120">Configure automatic traffic switching between global Azure and Azure Stack Hub.</span></span>

> [!Tip]  
> <span data-ttu-id="819f6-121">![混合式支柱圖](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="819f6-121">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="819f6-122">Microsoft Azure Stack Hub 是 Azure 的延伸模組。</span><span class="sxs-lookup"><span data-stu-id="819f6-122">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="819f6-123">Azure Stack Hub 可將雲端運算的靈活性與創新能力導入您的內部部署環境中，並啟用獨特的混合式雲端，讓您能夠隨處建置及部署混合式應用程式。</span><span class="sxs-lookup"><span data-stu-id="819f6-123">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="819f6-124">[混合式應用程式設計考量](overview-app-design-considerations.md)一文檢閱了設計、部署和操作混合式應用程式時的軟體品質要素 (放置、延展性、可用性、復原、管理性和安全性)。</span><span class="sxs-lookup"><span data-stu-id="819f6-124">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="819f6-125">這些設計考量有助於您設計出最佳的混合式應用程式，減少生產環境可能會遇到的挑戰。</span><span class="sxs-lookup"><span data-stu-id="819f6-125">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

### <a name="assumptions"></a><span data-ttu-id="819f6-126">假設</span><span class="sxs-lookup"><span data-stu-id="819f6-126">Assumptions</span></span>

<span data-ttu-id="819f6-127">本教學課程假設您擁有全域 Azure 和 Azure Stack Hub 的基本知識。</span><span class="sxs-lookup"><span data-stu-id="819f6-127">This tutorial assumes that you have a basic knowledge of global Azure and Azure Stack Hub.</span></span> <span data-ttu-id="819f6-128">若要在開始本教學課程前深入了解，請檢閱下列文章：</span><span class="sxs-lookup"><span data-stu-id="819f6-128">If you want to learn more before starting the tutorial, review these articles:</span></span>

- [<span data-ttu-id="819f6-129">Azure 簡介</span><span class="sxs-lookup"><span data-stu-id="819f6-129">Introduction to Azure</span></span>](https://azure.microsoft.com/overview/what-is-azure/)
- [<span data-ttu-id="819f6-130">Azure Stack Hub 重要概念</span><span class="sxs-lookup"><span data-stu-id="819f6-130">Azure Stack Hub Key Concepts</span></span>](/azure-stack/operator/azure-stack-overview.md)

<span data-ttu-id="819f6-131">本教學課程也假設您已有 Azure 訂用帳戶。</span><span class="sxs-lookup"><span data-stu-id="819f6-131">This tutorial also assumes that you have an Azure subscription.</span></span> <span data-ttu-id="819f6-132">如果您沒有訂用帳戶，請先[建立免費帳戶](https://azure.microsoft.com/free/)，再開始操作。</span><span class="sxs-lookup"><span data-stu-id="819f6-132">If you don't have a subscription, [create a free account](https://azure.microsoft.com/free/) before you begin.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="819f6-133">Prerequisites</span><span class="sxs-lookup"><span data-stu-id="819f6-133">Prerequisites</span></span>

<span data-ttu-id="819f6-134">開始進行本解決方案之前，請確定您符合下列需求：</span><span class="sxs-lookup"><span data-stu-id="819f6-134">Before you start this solution, make sure you meet the following requirements:</span></span>

- <span data-ttu-id="819f6-135">Azure Stack 開發套件 (ASDK) 或 Azure Stack Hub 整合式系統的訂用帳戶。</span><span class="sxs-lookup"><span data-stu-id="819f6-135">An Azure Stack Development Kit (ASDK) or a subscription on an Azure Stack Hub Integrated System.</span></span> <span data-ttu-id="819f6-136">若要部署 ASDK，請依照[使用安裝程式部署 ASDK](/azure-stack/asdk/asdk-install.md) 中的指示操作。</span><span class="sxs-lookup"><span data-stu-id="819f6-136">To deploy the ASDK, follow the instructions in [Deploy the ASDK using the installer](/azure-stack/asdk/asdk-install.md).</span></span>
- <span data-ttu-id="819f6-137">您的 Azure Stack Hub 安裝應安裝下列項目：</span><span class="sxs-lookup"><span data-stu-id="819f6-137">Your Azure Stack Hub installation should have the following installed:</span></span>
  - <span data-ttu-id="819f6-138">Azure App Service。</span><span class="sxs-lookup"><span data-stu-id="819f6-138">The Azure App Service.</span></span> <span data-ttu-id="819f6-139">請使用 Azure Stack Hub 操作員，在您的環境中部署和設定 Azure App Service。</span><span class="sxs-lookup"><span data-stu-id="819f6-139">Work with your Azure Stack Hub Operator to deploy and configure the Azure App Service on your environment.</span></span> <span data-ttu-id="819f6-140">在本教學課程中，App Service 至少必須有一 (1) 個可用的專用背景工作角色。</span><span class="sxs-lookup"><span data-stu-id="819f6-140">This tutorial requires the App Service to have at least one (1) available dedicated worker role.</span></span>
  - <span data-ttu-id="819f6-141">Windows Server 2016 映像。</span><span class="sxs-lookup"><span data-stu-id="819f6-141">A Windows Server 2016 image.</span></span>
  - <span data-ttu-id="819f6-142">具有 Microsoft SQL Server 映像的 Windows Server 2016。</span><span class="sxs-lookup"><span data-stu-id="819f6-142">A Windows Server 2016 with a Microsoft SQL Server image.</span></span>
  - <span data-ttu-id="819f6-143">適當的方案和供應項目。</span><span class="sxs-lookup"><span data-stu-id="819f6-143">The appropriate plans and offers.</span></span>
  - <span data-ttu-id="819f6-144">Web 應用程式的網域名稱。</span><span class="sxs-lookup"><span data-stu-id="819f6-144">A domain name for your web app.</span></span> <span data-ttu-id="819f6-145">如果您沒有網域名稱，可以向 GoDaddy、Bluehost 和 InMotion 等網域提供者購買。</span><span class="sxs-lookup"><span data-stu-id="819f6-145">If you don't have a domain name, you can buy one from a domain provider like GoDaddy, Bluehost, and InMotion.</span></span>
- <span data-ttu-id="819f6-146">信任的憑證授權單位 (例如 LetsEncrypt) 為您的網域核發的 SSL 憑證。</span><span class="sxs-lookup"><span data-stu-id="819f6-146">An SSL certificate for your domain from a trusted certificate authority like LetsEncrypt.</span></span>
- <span data-ttu-id="819f6-147">與 SQL Server 資料庫通訊且支援 Application Insights 的 Web 應用程式。</span><span class="sxs-lookup"><span data-stu-id="819f6-147">A web app that communicates with a SQL Server database and supports Application Insights.</span></span> <span data-ttu-id="819f6-148">您可以從 GitHub 下載 [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) 範例。</span><span class="sxs-lookup"><span data-stu-id="819f6-148">You can download the [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) sample app from GitHub.</span></span>
- <span data-ttu-id="819f6-149">Azure 虛擬網路與 Azure Stack Hub 虛擬網路之間的混合式網路。</span><span class="sxs-lookup"><span data-stu-id="819f6-149">A hybrid network between an Azure virtual network and Azure Stack Hub virtual network.</span></span> <span data-ttu-id="819f6-150">如需詳細指示，請參閱[使用 Azure 和 Azure Stack Hub 設定混合式雲端連線](solution-deployment-guide-connectivity.md)。</span><span class="sxs-lookup"><span data-stu-id="819f6-150">For detailed instructions, see [Configure hybrid cloud connectivity with Azure and Azure Stack Hub](solution-deployment-guide-connectivity.md).</span></span>

- <span data-ttu-id="819f6-151">在 Azure Stack Hub 上具有私人組建代理程式的混合式持續整合/持續部署 (CI/CD) 管線。</span><span class="sxs-lookup"><span data-stu-id="819f6-151">A hybrid continuous integration/continuous deployment (CI/CD) pipeline with a private build agent on Azure Stack Hub.</span></span> <span data-ttu-id="819f6-152">如需詳細指示，請參閱[使用 Azure 和 Azure Stack Hub 應用程式設定混合式雲端身分識別](solution-deployment-guide-identity.md)。</span><span class="sxs-lookup"><span data-stu-id="819f6-152">For detailed instructions, see [Configure hybrid cloud identity with Azure and Azure Stack Hub apps](solution-deployment-guide-identity.md).</span></span>

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a><span data-ttu-id="819f6-153">部署混合式連線 SQL Server 資料庫伺服器</span><span class="sxs-lookup"><span data-stu-id="819f6-153">Deploy a hybrid-connected SQL Server database server</span></span>

1. <span data-ttu-id="819f6-154">登入 Azure Stack Hub 使用者入口網站。</span><span class="sxs-lookup"><span data-stu-id="819f6-154">Sign to the Azure Stack Hub user portal.</span></span>

2. <span data-ttu-id="819f6-155">在 [儀表板]  上選取 [Marketplace]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-155">On the **Dashboard** , select **Marketplace** .</span></span>

    ![Azure Stack Hub Marketplace](media/solution-deployment-guide-hybrid/image1.png)

3. <span data-ttu-id="819f6-157">在 [Marketplace]  中選取 [計算]  ，然後選擇 [其他]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-157">In **Marketplace** , select **Compute** , and then choose **More** .</span></span> <span data-ttu-id="819f6-158">在 [其他]  底下，選取 **免費的 SQL Server 授權：Windows Server 上的 SQL Server 2017 Developer** 映像。</span><span class="sxs-lookup"><span data-stu-id="819f6-158">Under **More** , select the **Free SQL Server License: SQL Server 2017 Developer on Windows Server** image.</span></span>

    ![在 Azure Stack Hub 使用者入口網站中選取虛擬機器映像](media/solution-deployment-guide-hybrid/image2.png)

4. <span data-ttu-id="819f6-160">在 **免費的 SQL Server 授權：Windows Server 上的 SQL Server 2017 Developer** 上，選取 [建立]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-160">On **Free SQL Server License: SQL Server 2017 Developer on Windows Server** , select **Create** .</span></span>

5. <span data-ttu-id="819f6-161">在 [基本 > 設定基本設定]  上，提供虛擬機器 (VM) 的 [名稱]  、SQL Server SA 的 [使用者名稱]  ，和 SA 的 [密碼]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-161">On **Basics > Configure basic settings** , provide a **Name** for the virtual machine (VM), a **User name** for the SQL Server SA, and a **Password** for the SA.</span></span>  <span data-ttu-id="819f6-162">從 [訂用帳戶]  下拉式清單中，選取要部署到的訂用帳戶。</span><span class="sxs-lookup"><span data-stu-id="819f6-162">From the **Subscription** drop-down list, select the subscription that you're deploying to.</span></span> <span data-ttu-id="819f6-163">對於 [資源群組]  ，使用 [選擇現有項目]  ，並將 VM 放在與 Azure Stack Hub Web 應用程式相同的資源群組中。</span><span class="sxs-lookup"><span data-stu-id="819f6-163">For **Resource group** , use **Choose existing** and put the VM in the same resource group as your Azure Stack Hub web app.</span></span>

    ![在 Azure Stack Hub 使用者入口網站中設定 VM 的基本設定](media/solution-deployment-guide-hybrid/image3.png)

6. <span data-ttu-id="819f6-165">在 [大小]  下方，選擇您的 VM 大小。</span><span class="sxs-lookup"><span data-stu-id="819f6-165">Under **Size** , pick a size for your VM.</span></span> <span data-ttu-id="819f6-166">在本教學課程中，建議您使用 A2_Standard 或 DS2_V2_Standard。</span><span class="sxs-lookup"><span data-stu-id="819f6-166">For this tutorial, we recommend A2_Standard or a DS2_V2_Standard.</span></span>

7. <span data-ttu-id="819f6-167">在 [設定 > 設定選用功能]  下方，進行下列設定：</span><span class="sxs-lookup"><span data-stu-id="819f6-167">Under **Settings > Configure optional features** , configure the following settings:</span></span>

   - <span data-ttu-id="819f6-168">**儲存體帳戶** ：如有需要，請建立新帳戶。</span><span class="sxs-lookup"><span data-stu-id="819f6-168">**Storage account** : Create a new account if you need one.</span></span>
   - <span data-ttu-id="819f6-169">**虛擬網路** ：</span><span class="sxs-lookup"><span data-stu-id="819f6-169">**Virtual network** :</span></span>

     > [!Important]  
     > <span data-ttu-id="819f6-170">請確定您的 SQL Server VM 部署在與 VPN 閘道相同的虛擬網路上。</span><span class="sxs-lookup"><span data-stu-id="819f6-170">Make sure your SQL Server VM is deployed on the same  virtual network as the VPN gateways.</span></span>

   - <span data-ttu-id="819f6-171">**公用 IP 位址** ：請使用預設設定。</span><span class="sxs-lookup"><span data-stu-id="819f6-171">**Public IP address** : Use the default settings.</span></span>
   - <span data-ttu-id="819f6-172">**網路安全性群組** ︰(NSG)。</span><span class="sxs-lookup"><span data-stu-id="819f6-172">**Network security group** : (NSG).</span></span> <span data-ttu-id="819f6-173">建立新的 NSG。</span><span class="sxs-lookup"><span data-stu-id="819f6-173">Create a new NSG.</span></span>
   - <span data-ttu-id="819f6-174">**擴充功能和監視** ：請保留預設設定值。</span><span class="sxs-lookup"><span data-stu-id="819f6-174">**Extensions and Monitoring** : Keep the default settings.</span></span>
   - <span data-ttu-id="819f6-175">**診斷儲存體帳戶** ：如有需要，請建立新帳戶。</span><span class="sxs-lookup"><span data-stu-id="819f6-175">**Diagnostics storage account** : Create a new account if you need one.</span></span>
   - <span data-ttu-id="819f6-176">選取 [確定]  以儲存您的設定。</span><span class="sxs-lookup"><span data-stu-id="819f6-176">Select **OK** to save your configuration.</span></span>

     ![在 Azure Stack Hub 使用者入口網站中設定選用 VM 功能](media/solution-deployment-guide-hybrid/image4.png)

8. <span data-ttu-id="819f6-178">在 [SQL Server 設定]  下方，進行下列設定：</span><span class="sxs-lookup"><span data-stu-id="819f6-178">Under **SQL Server settings** , configure the following settings:</span></span>

   - <span data-ttu-id="819f6-179">針對 [SQL 連線能力]  ，選取 [公用 (網際網路)]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-179">For **SQL connectivity** , select **Public (Internet)** .</span></span>
   - <span data-ttu-id="819f6-180">將 [連接埠]  保留為預設值 **1433** 。</span><span class="sxs-lookup"><span data-stu-id="819f6-180">For **Port** , keep the default, **1433** .</span></span>
   - <span data-ttu-id="819f6-181">針對 [SQL 驗證]  ，選取 [啟用]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-181">For **SQL authentication** , select **Enable** .</span></span>

     > [!Note]  
     > <span data-ttu-id="819f6-182">SQL 驗證啟用時，應會自動填入您在 [基本]  中設定的 "SQLAdmin" 資訊。</span><span class="sxs-lookup"><span data-stu-id="819f6-182">When you enable SQL authentication, it should auto-populate with the "SQLAdmin" information that you configured in **Basics** .</span></span>

   - <span data-ttu-id="819f6-183">其餘設定請保留為預設值。</span><span class="sxs-lookup"><span data-stu-id="819f6-183">For the rest of the settings, keep the defaults.</span></span> <span data-ttu-id="819f6-184">選取 [確定]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-184">Select **OK** .</span></span>

     ![在 Azure Stack Hub 使用者入口網站中設定 SQL Server 設定](media/solution-deployment-guide-hybrid/image5.png)

9. <span data-ttu-id="819f6-186">在 [摘要]  上檢閱 VM 設定，然後選取 [確定]  開始進行部署。</span><span class="sxs-lookup"><span data-stu-id="819f6-186">On **Summary** , review the VM configuration and then select **OK** to start the deployment.</span></span>

    ![Azure Stack Hub 使用者入口網站中的設定摘要](media/solution-deployment-guide-hybrid/image6.png)

10. <span data-ttu-id="819f6-188">建立新的 VM 需要一些時間。</span><span class="sxs-lookup"><span data-stu-id="819f6-188">It takes some time to create the new VM.</span></span> <span data-ttu-id="819f6-189">您可以在 [虛擬機器]  中檢視 VM 的 [狀態]。</span><span class="sxs-lookup"><span data-stu-id="819f6-189">You can view the STATUS of your VMs in **Virtual machines** .</span></span>

    ![Azure Stack Hub 使用者入口網站中的虛擬機器狀態](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a><span data-ttu-id="819f6-191">在 Azure 和 Azure Stack Hub 中建立 Web 應用程式</span><span class="sxs-lookup"><span data-stu-id="819f6-191">Create web apps in Azure and Azure Stack Hub</span></span>

<span data-ttu-id="819f6-192">Azure App Service 可簡化執行和管理 Web 應用程式的工作。</span><span class="sxs-lookup"><span data-stu-id="819f6-192">The Azure App Service simplifies running and managing a web app.</span></span> <span data-ttu-id="819f6-193">Azure Stack Hub 與 Azure 是一致的，因此 App Service 可以在這兩種環境中執行。</span><span class="sxs-lookup"><span data-stu-id="819f6-193">Because Azure Stack Hub is consistent with Azure,  the App Service can run in both environments.</span></span> <span data-ttu-id="819f6-194">您將使用 App Service 來裝載應用程式。</span><span class="sxs-lookup"><span data-stu-id="819f6-194">You'll use the App Service to host your app.</span></span>

### <a name="create-web-apps"></a><span data-ttu-id="819f6-195">建立 Web 應用程式</span><span class="sxs-lookup"><span data-stu-id="819f6-195">Create web apps</span></span>

1. <span data-ttu-id="819f6-196">依照[管理 Azure 中的 App Service 方案](/azure/app-service/app-service-plan-manage#create-an-app-service-plan)中的指示，在 Azure 中建立 Web 應用程式。</span><span class="sxs-lookup"><span data-stu-id="819f6-196">Create a web app in Azure by following the instructions in [Manage an App Service plan in Azure](/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span></span> <span data-ttu-id="819f6-197">請確實將 Web 應用程式放在與混合式網路相同的訂用帳戶和資源群組中。</span><span class="sxs-lookup"><span data-stu-id="819f6-197">Make sure you put the web app in the same subscription and resource group as your hybrid network.</span></span>

2. <span data-ttu-id="819f6-198">在 Azure Stack Hub 中重複前述步驟 (1)。</span><span class="sxs-lookup"><span data-stu-id="819f6-198">Repeat the previous step (1) in Azure Stack Hub.</span></span>

### <a name="add-route-for-azure-stack-hub"></a><span data-ttu-id="819f6-199">新增 Azure Stack Hub 的路由</span><span class="sxs-lookup"><span data-stu-id="819f6-199">Add route for Azure Stack Hub</span></span>

<span data-ttu-id="819f6-200">Azure Stack Hub 上的 App Service 必須可從公用網際網路進行路由，讓使用者能夠存取您的應用程式。</span><span class="sxs-lookup"><span data-stu-id="819f6-200">The App Service on Azure Stack Hub must be routable from the public internet to let users access your app.</span></span> <span data-ttu-id="819f6-201">如果您的 Azure Stack Hub 可從網際網路存取，請記下 Azure Stack Hub Web 應用程式的公眾對應 IP 位址或 URL。</span><span class="sxs-lookup"><span data-stu-id="819f6-201">If your Azure Stack Hub is accessible from the internet, make a note of the public-facing IP address or URL for the Azure Stack Hub web app.</span></span>

<span data-ttu-id="819f6-202">如果您使用 ASDK，您可以[設定靜態 NAT 對應](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal)，將 App Service 公開於虛擬環境外。</span><span class="sxs-lookup"><span data-stu-id="819f6-202">If you're using an ASDK, you can [configure a static NAT mapping](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) to expose App Service outside the virtual environment.</span></span>

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a><span data-ttu-id="819f6-203">將 Azure 中的 Web 應用程式連線至混合式網路</span><span class="sxs-lookup"><span data-stu-id="819f6-203">Connect a web app in Azure to a hybrid network</span></span>

<span data-ttu-id="819f6-204">若要讓 Azure 中的 Web 前端與 Azure Stack Hub 中的 SQL Server 資料庫能夠相互連線，Web 應用程式必須連線至 Azure 和 Azure Stack Hub 之間的混合式網路。</span><span class="sxs-lookup"><span data-stu-id="819f6-204">To provide connectivity between the web front end in Azure and the SQL Server database in Azure Stack Hub, the web app must be connected to the hybrid network between Azure and Azure Stack Hub.</span></span> <span data-ttu-id="819f6-205">若要啟用連線，您必須：</span><span class="sxs-lookup"><span data-stu-id="819f6-205">To enable connectivity, you'll have to:</span></span>

- <span data-ttu-id="819f6-206">設定點對站連線。</span><span class="sxs-lookup"><span data-stu-id="819f6-206">Configure point-to-site connectivity.</span></span>
- <span data-ttu-id="819f6-207">設定 Web 應用程式。</span><span class="sxs-lookup"><span data-stu-id="819f6-207">Configure the web app.</span></span>
- <span data-ttu-id="819f6-208">修改 Azure Stack Hub 中的區域網路閘道。</span><span class="sxs-lookup"><span data-stu-id="819f6-208">Modify the local network gateway in Azure Stack Hub.</span></span>

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a><span data-ttu-id="819f6-209">設定點對站連線所需的 Azure 虛擬網路</span><span class="sxs-lookup"><span data-stu-id="819f6-209">Configure the Azure virtual network for point-to-site connectivity</span></span>

<span data-ttu-id="819f6-210">在混合式網路中，Azure 端的虛擬網路閘道必須允許點對站連線，以便與 Azure App Service 整合。</span><span class="sxs-lookup"><span data-stu-id="819f6-210">The virtual network gateway in the Azure side of the hybrid network must allow point-to-site connections to integrate with Azure App Service.</span></span>

1. <span data-ttu-id="819f6-211">在 Azure 入口網站中，前往虛擬網路閘道頁面。</span><span class="sxs-lookup"><span data-stu-id="819f6-211">In the Azure portal, go to the virtual network gateway page.</span></span> <span data-ttu-id="819f6-212">在 [設定]  下方，選取 [點對站設定]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-212">Under **Settings** , select **Point-to-site configuration** .</span></span>

    ![Azure 虛擬網路閘道中的點對站選項](media/solution-deployment-guide-hybrid/image8.png)

2. <span data-ttu-id="819f6-214">選取 [立即設定]  ，進行點對站設定。</span><span class="sxs-lookup"><span data-stu-id="819f6-214">Select **Configure now** to configure point-to-site.</span></span>

    ![在 Azure 虛擬網路閘道中開始進行點對站設定](media/solution-deployment-guide-hybrid/image9.png)

3. <span data-ttu-id="819f6-216">在 [點對站]  設定頁面上，將您要使用的私人 IP 位址範圍輸入 [位址集區]  中。</span><span class="sxs-lookup"><span data-stu-id="819f6-216">On the **Point-to-site** configuration page, enter the private IP address range that you want to use in **Address pool** .</span></span>

   > [!Note]  
   > <span data-ttu-id="819f6-217">請確定您指定的範圍並未與混合式網路的全域 Azure 或 Azure Stack Hub 元件中的子網路已使用的任何位址範圍重疊。</span><span class="sxs-lookup"><span data-stu-id="819f6-217">Make sure that the range you specify doesn't overlap with any of the address ranges already used by subnets in the global Azure or Azure Stack Hub components of the hybrid network.</span></span>

   <span data-ttu-id="819f6-218">在 [通道類型]  下方，取消核取 [IKEv2 VPN]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-218">Under **Tunnel Type** , uncheck the **IKEv2 VPN** .</span></span> <span data-ttu-id="819f6-219">選取 [儲存]  以完成點對站設定。</span><span class="sxs-lookup"><span data-stu-id="819f6-219">Select **Save** to finish configuring point-to-site.</span></span>

   ![Azure 虛擬網路閘道中的點對站設定](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a><span data-ttu-id="819f6-221">整合 Azure App Service 應用程式與混合式網路</span><span class="sxs-lookup"><span data-stu-id="819f6-221">Integrate the Azure App Service app with the hybrid network</span></span>

1. <span data-ttu-id="819f6-222">若要將應用程式連線至 Azure VNet，請依照[ VNet 整合的必要閘道](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration)中的指示操作。</span><span class="sxs-lookup"><span data-stu-id="819f6-222">To connect the app to the Azure VNet, follow the instructions in [Gateway required VNet integration](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).</span></span>

2. <span data-ttu-id="819f6-223">找出裝載 Web 應用程式的 App Service 方案，移至其 [設定]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-223">Go to **Settings** for the App Service plan hosting the web app.</span></span> <span data-ttu-id="819f6-224">在 [設定]  中，選取 [網路]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-224">In **Settings** , select **Networking** .</span></span>

    ![設定 App Service 方案的網路功能](media/solution-deployment-guide-hybrid/image11.png)

3. <span data-ttu-id="819f6-226">在 [VNET 整合]  中，選取 [按一下這裡進行管理]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-226">In **VNET Integration** , select **Click here to manage** .</span></span>

    ![管理 App Service 方案的 VNET 整合](media/solution-deployment-guide-hybrid/image12.png)

4. <span data-ttu-id="819f6-228">選取您要設定的 VNET。</span><span class="sxs-lookup"><span data-stu-id="819f6-228">Select the VNET that you want to configure.</span></span> <span data-ttu-id="819f6-229">在 [路由至 VNET 的 IP 位址]  下方，輸入 Azure VNet、Azure Stack Hub VNet 和點對站位址空間的 IP 位址範圍。</span><span class="sxs-lookup"><span data-stu-id="819f6-229">Under **IP ADDRESSES ROUTED TO VNET** , enter the IP address range for the Azure VNet, the Azure Stack Hub VNet, and the point-to-site address spaces.</span></span> <span data-ttu-id="819f6-230">選取 [儲存]  以驗證並儲存這些設定。</span><span class="sxs-lookup"><span data-stu-id="819f6-230">Select **Save** to validate and save these settings.</span></span>

    ![要在虛擬網路整合中路由的 IP 位址範圍](media/solution-deployment-guide-hybrid/image13.png)

<span data-ttu-id="819f6-232">若要深入了解 how App Service 與 Azure VNet 的整合方式，請參閱[將您的應用程式與 Azure 虛擬網路整合](/azure/app-service/web-sites-integrate-with-vnet)。</span><span class="sxs-lookup"><span data-stu-id="819f6-232">To learn more about how App Service integrates with Azure VNets, see [Integrate your app with an Azure Virtual Network](/azure/app-service/web-sites-integrate-with-vnet).</span></span>

### <a name="configure-the-azure-stack-hub-virtual-network"></a><span data-ttu-id="819f6-233">設定 Azure Stack Hub 虛擬網路</span><span class="sxs-lookup"><span data-stu-id="819f6-233">Configure the Azure Stack Hub virtual network</span></span>

<span data-ttu-id="819f6-234">必須設定 Azure Stack Hub 虛擬網路中的區域網路閘道，以路由來自 App Service 點對站位址範圍的流量。</span><span class="sxs-lookup"><span data-stu-id="819f6-234">The local network gateway in the Azure Stack Hub virtual network needs to be configured to route traffic from the App Service point-to-site address range.</span></span>

1. <span data-ttu-id="819f6-235">在 Azure Stack Hub 入口網站中，移至 [區域網路閘道]。</span><span class="sxs-lookup"><span data-stu-id="819f6-235">In the Azure Stack Hub portal, go to **Local network gateway** .</span></span> <span data-ttu-id="819f6-236">在 [設定]  下方，選取 [設定]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-236">Under **Settings** , select **Configuration** .</span></span>

    ![Azure Stack Hub 區域網路閘道中的閘道設定選項](media/solution-deployment-guide-hybrid/image14.png)

2. <span data-ttu-id="819f6-238">在 [位址空間]  中，輸入 Azure 虛擬網路閘道的點對站位址範圍。</span><span class="sxs-lookup"><span data-stu-id="819f6-238">In **Address space** , enter the point-to-site address range for the virtual network gateway in Azure.</span></span>

    ![Azure Stack Hub 區域網路閘道中的點對站位址空間](media/solution-deployment-guide-hybrid/image15.png)

3. <span data-ttu-id="819f6-240">選取 [儲存]  以驗證並儲存此設定。</span><span class="sxs-lookup"><span data-stu-id="819f6-240">Select **Save** to validate and save the configuration.</span></span>

## <a name="configure-dns-for-cross-cloud-scaling"></a><span data-ttu-id="819f6-241">設定跨雲端縮放的 DNS</span><span class="sxs-lookup"><span data-stu-id="819f6-241">Configure DNS for cross-cloud scaling</span></span>

<span data-ttu-id="819f6-242">藉由適當設定跨雲端應用程式的 DNS，使用者將可存取您 Web 應用程式的全域 Azure 和 Azure Stack Hub 執行個體。</span><span class="sxs-lookup"><span data-stu-id="819f6-242">By properly configuring DNS for cross-cloud apps, users can access the global Azure and Azure Stack Hub instances of your web app.</span></span> <span data-ttu-id="819f6-243">本教學課程中的 DNS 設定也可讓 Azure 流量管理員在負載增加或減少時路由流量。</span><span class="sxs-lookup"><span data-stu-id="819f6-243">The DNS configuration for this tutorial also lets Azure Traffic Manager route traffic when the load increases or decreases.</span></span>

<span data-ttu-id="819f6-244">本教學課程會使用 Azure DNS 管理 DNS，因為 App Service 網域無法運作。</span><span class="sxs-lookup"><span data-stu-id="819f6-244">This tutorial uses Azure DNS to manage the DNS because App Service domains won't work.</span></span>

### <a name="create-subdomains"></a><span data-ttu-id="819f6-245">建立子網域</span><span class="sxs-lookup"><span data-stu-id="819f6-245">Create subdomains</span></span>

<span data-ttu-id="819f6-246">由於流量管理員需倚賴 DNS CNAME，因此子網域必須能夠正確地將流量路由至端點。</span><span class="sxs-lookup"><span data-stu-id="819f6-246">Because Traffic Manager relies on DNS CNAMEs, a subdomain is needed to properly route traffic to endpoints.</span></span> <span data-ttu-id="819f6-247">如需與 DNS 記錄和網域之間的對應有關的詳細資訊，請參閱[使用流量管理員對應網域](/azure/app-service/web-sites-traffic-manager-custom-domain-name)。</span><span class="sxs-lookup"><span data-stu-id="819f6-247">For more information about DNS records and domain mapping, see [map domains with Traffic Manager](/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span></span>

<span data-ttu-id="819f6-248">針對 Azure 端點，您必須建立可讓使用者用來存取 Web 應用程式的子網域。</span><span class="sxs-lookup"><span data-stu-id="819f6-248">For the Azure endpoint, you'll create a subdomain that users can use to access your web app.</span></span> <span data-ttu-id="819f6-249">在本教學課程中可以使用 **app.northwind.com** ，但您應根據自己的網域自訂此值。</span><span class="sxs-lookup"><span data-stu-id="819f6-249">For this tutorial, can use **app.northwind.com** , but you should customize this value based on your own domain.</span></span>

<span data-ttu-id="819f6-250">您也必須以 Azure Stack Hub 端點的 A 記錄建立子網域。</span><span class="sxs-lookup"><span data-stu-id="819f6-250">You'll also need to create a subdomain with an A record for the Azure Stack Hub endpoint.</span></span> <span data-ttu-id="819f6-251">您可以使用 **azurestack.northwind.com** 。</span><span class="sxs-lookup"><span data-stu-id="819f6-251">You can use **azurestack.northwind.com** .</span></span>

### <a name="configure-a-custom-domain-in-azure"></a><span data-ttu-id="819f6-252">在 Azure 中設定自訂網域</span><span class="sxs-lookup"><span data-stu-id="819f6-252">Configure a custom domain in Azure</span></span>

1. <span data-ttu-id="819f6-253">藉由 [將 CNAME 對應至 Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record)，以將 **app.northwind.com** 主機名稱新增至 Azure Web 應用程式。</span><span class="sxs-lookup"><span data-stu-id="819f6-253">Add the **app.northwind.com** hostname to the Azure web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span>

### <a name="configure-custom-domains-in-azure-stack-hub"></a><span data-ttu-id="819f6-254">在 Azure Stack Hub 中設定自訂網域</span><span class="sxs-lookup"><span data-stu-id="819f6-254">Configure custom domains in Azure Stack Hub</span></span>

1. <span data-ttu-id="819f6-255">藉由 [將 A 記錄對應至 Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record)，以將 **azurestack.northwind.com** 主機名稱新增至 Azure Stack Hub Web 應用程式。</span><span class="sxs-lookup"><span data-stu-id="819f6-255">Add the **azurestack.northwind.com** hostname to the Azure Stack Hub web app by [mapping an A record to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span></span> <span data-ttu-id="819f6-256">對於 App Service 應用程式，請使用網際網路可路由的 IP 位址。</span><span class="sxs-lookup"><span data-stu-id="819f6-256">Use the internet-routable IP address for the App Service app.</span></span>

2. <span data-ttu-id="819f6-257">藉由 [將 CNAME 對應至 Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record)，以將 **app.northwind.com** 主機名稱新增至 Azure Stack Hub Web 應用程式。</span><span class="sxs-lookup"><span data-stu-id="819f6-257">Add the **app.northwind.com** hostname to the Azure Stack Hub web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span> <span data-ttu-id="819f6-258">請使用您在先前的步驟 (1) 中設定的主機名稱作為 CNAME 的目標。</span><span class="sxs-lookup"><span data-stu-id="819f6-258">Use the hostname you configured in the previous step (1) as the target for the CNAME.</span></span>

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a><span data-ttu-id="819f6-259">設定跨雲端縮放的 SSL 憑證</span><span class="sxs-lookup"><span data-stu-id="819f6-259">Configure SSL certificates for cross-cloud scaling</span></span>

<span data-ttu-id="819f6-260">請務必確定 Web 應用程式所收集的敏感性資料在傳輸至 SQL 資料庫及儲存在該處時都受到良好的保護。</span><span class="sxs-lookup"><span data-stu-id="819f6-260">It's important to ensure sensitive data collected by your web app is secure in transit to and when stored on the SQL database.</span></span>

<span data-ttu-id="819f6-261">您將設定 Azure 和 Azure Stack Hub Web 應用程式，以對所有傳入的流量使用 SSL 憑證。</span><span class="sxs-lookup"><span data-stu-id="819f6-261">You'll configure your Azure and Azure Stack Hub web apps to use SSL certificates for all incoming traffic.</span></span>

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a><span data-ttu-id="819f6-262">將 SSL 新增至 Azure 和 Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="819f6-262">Add SSL to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="819f6-263">若要將 SSL 新增至 Azure：</span><span class="sxs-lookup"><span data-stu-id="819f6-263">To add SSL to Azure:</span></span>

1. <span data-ttu-id="819f6-264">確定您取得的 SSL 憑證適用於您所建立的子網域。</span><span class="sxs-lookup"><span data-stu-id="819f6-264">Make sure that the SSL certificate you get is valid for the subdomain you created.</span></span> <span data-ttu-id="819f6-265">(您也可以使用萬用字元憑證)。</span><span class="sxs-lookup"><span data-stu-id="819f6-265">(It's okay to use wildcard certificates.)</span></span>

2. <span data-ttu-id="819f6-266">在 Azure 入口網站中，依照 [將現有的自訂 SSL 憑證繫結至 Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) 一文的 **準備您的 Web 應用程式** 和 **繫結 SSL 憑證** 小節所提供的指示操作。</span><span class="sxs-lookup"><span data-stu-id="819f6-266">In the Azure portal, follow the instructions in the **Prepare your web app** and **Bind your SSL certificate** sections of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span> <span data-ttu-id="819f6-267">請選取 [以 SNI 為基礎的 SSL]  作為 [SSL 類型]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-267">Select **SNI-based SSL** as the **SSL Type** .</span></span>

3. <span data-ttu-id="819f6-268">將所有流量重新都導向至 HTTPS 連接埠。</span><span class="sxs-lookup"><span data-stu-id="819f6-268">Redirect all traffic to the HTTPS port.</span></span> <span data-ttu-id="819f6-269">請依照 [將現有的自訂 SSL 憑證繫結至 Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) 一文的 **強制執行 HTTPS** 小節所提供的指示操作。</span><span class="sxs-lookup"><span data-stu-id="819f6-269">Follow the instructions in the   **Enforce HTTPS** section of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span>

<span data-ttu-id="819f6-270">若要將 SSL 新增至 Azure Stack Hub：</span><span class="sxs-lookup"><span data-stu-id="819f6-270">To add SSL to Azure Stack Hub:</span></span>

1. <span data-ttu-id="819f6-271">使用 Azure Stack Hub 入口網站，重複您用於 Azure 的步驟 1-3。</span><span class="sxs-lookup"><span data-stu-id="819f6-271">Repeat steps 1-3 that you used for Azure, using the Azure Stack Hub portal.</span></span>

## <a name="configure-and-deploy-the-web-app"></a><span data-ttu-id="819f6-272">設定及部署 Web 應用程式</span><span class="sxs-lookup"><span data-stu-id="819f6-272">Configure and deploy the web app</span></span>

<span data-ttu-id="819f6-273">您將設定應用程式程式碼，以將遙測資料報告至正確的 Application Insights 執行個體，並使用正確的連接字串來設定 Web 應用程式。</span><span class="sxs-lookup"><span data-stu-id="819f6-273">You'll configure the app code to report telemetry to the correct Application Insights instance and configure the web apps with the right connection strings.</span></span> <span data-ttu-id="819f6-274">若要深入了解 Application Insights，請參閱[什麼是 Application Insights？](/azure/application-insights/app-insights-overview)。</span><span class="sxs-lookup"><span data-stu-id="819f6-274">To learn more about Application Insights, see [What is Application Insights?](/azure/application-insights/app-insights-overview)</span></span>

### <a name="add-application-insights"></a><span data-ttu-id="819f6-275">新增 Application Insights</span><span class="sxs-lookup"><span data-stu-id="819f6-275">Add Application Insights</span></span>

1. <span data-ttu-id="819f6-276">在 Microsoft Visual Studio 中開啟您的 Web 應用程式。</span><span class="sxs-lookup"><span data-stu-id="819f6-276">Open your web app in Microsoft Visual Studio.</span></span>

2. <span data-ttu-id="819f6-277">在您的專案中[新增 Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications)，以傳輸 Application Insights 在 Web 流量增加或減少時用來建立警示的遙測資料。</span><span class="sxs-lookup"><span data-stu-id="819f6-277">[Add Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) to your project to transmit the telemetry that Application Insights uses to create alerts when web traffic increases or decreases.</span></span>

### <a name="configure-dynamic-connection-strings"></a><span data-ttu-id="819f6-278">設定動態連接字串</span><span class="sxs-lookup"><span data-stu-id="819f6-278">Configure dynamic connection strings</span></span>

<span data-ttu-id="819f6-279">Web 應用程式的每個執行個體會使用不同的方法連線至 SQL 資料庫。</span><span class="sxs-lookup"><span data-stu-id="819f6-279">Each instance of the web app will use a different method to connect to the SQL database.</span></span> <span data-ttu-id="819f6-280">在 Azure 中的應用程式會使用 SQL Server VM 的私人 IP 位址，而 Azure Stack Hub 中的應用程式則會使用 SQL Server VM 的公用 IP 位址。</span><span class="sxs-lookup"><span data-stu-id="819f6-280">The app in Azure uses the private IP address of the SQL Server VM and the app in Azure Stack Hub uses the public IP address of the SQL Server VM.</span></span>

> [!Note]  
> <span data-ttu-id="819f6-281">在 Azure Stack Hub 整合式系統中，公用 IP 位址應該無法使用網際網路路由。</span><span class="sxs-lookup"><span data-stu-id="819f6-281">On an Azure Stack Hub integrated system, the public IP address shouldn't be internet-routable.</span></span> <span data-ttu-id="819f6-282">在 ASDK 上，公用 IP 位址無法在 ASDK 以外路由。</span><span class="sxs-lookup"><span data-stu-id="819f6-282">On an ASDK, the public IP address isn't routable outside the ASDK.</span></span>

<span data-ttu-id="819f6-283">您可以使用 App Service 環境變數將不同的連接字串傳至應用程式的每個執行個體。</span><span class="sxs-lookup"><span data-stu-id="819f6-283">You can use App Service environment variables to pass a different connection string to each instance of the app.</span></span>

1. <span data-ttu-id="819f6-284">在 Visual Studio 中開啟應用程式。</span><span class="sxs-lookup"><span data-stu-id="819f6-284">Open the app in Visual Studio.</span></span>

2. <span data-ttu-id="819f6-285">開啟 Startup.cs 並且尋找下列程式碼區塊：</span><span class="sxs-lookup"><span data-stu-id="819f6-285">Open Startup.cs and find the following code block:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. <span data-ttu-id="819f6-286">將上述程式碼區塊取代為下列程式碼，以使用 *appsettings.json* 檔案中定義的連接字串：</span><span class="sxs-lookup"><span data-stu-id="819f6-286">Replace the previous code block with the following code, which uses a connection string defined in the *appsettings.json* file:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a><span data-ttu-id="819f6-287">設定 App Service 應用程式設定</span><span class="sxs-lookup"><span data-stu-id="819f6-287">Configure App Service app settings</span></span>

1. <span data-ttu-id="819f6-288">建立適用於 Azure 和 Azure Stack Hub 的連接字串。</span><span class="sxs-lookup"><span data-stu-id="819f6-288">Create connection strings for Azure and Azure Stack Hub.</span></span> <span data-ttu-id="819f6-289">這兩個字串除了所使用的 IP 位址外，其餘部分應相同。</span><span class="sxs-lookup"><span data-stu-id="819f6-289">The strings should be the same, except for the IP addresses that are used.</span></span>

2. <span data-ttu-id="819f6-290">在 Azure 和 Azure Stack Hub 中，以 `SQLCONNSTR\_` 作為名稱中的前置詞，為 Web 應用程式新增適當的連接字串[作為應用程式設定](/azure/app-service/web-sites-configure)。</span><span class="sxs-lookup"><span data-stu-id="819f6-290">In Azure and Azure Stack Hub, add the appropriate connection string [as an app setting](/azure/app-service/web-sites-configure) in the web app, using `SQLCONNSTR\_` as a prefix in the name.</span></span>

3. <span data-ttu-id="819f6-291">**儲存** Web 應用程式設定，並重新啟動應用程式。</span><span class="sxs-lookup"><span data-stu-id="819f6-291">**Save** the web app settings and restart the app.</span></span>

## <a name="enable-automatic-scaling-in-global-azure"></a><span data-ttu-id="819f6-292">在全域 Azure 中啟用自動調整</span><span class="sxs-lookup"><span data-stu-id="819f6-292">Enable automatic scaling in global Azure</span></span>

<span data-ttu-id="819f6-293">當您在 App Service 環境中建立 Web 應用程式時，它最初會有一個執行個體。</span><span class="sxs-lookup"><span data-stu-id="819f6-293">When you create your web app in an App Service environment, it starts with one instance.</span></span> <span data-ttu-id="819f6-294">您可以自動擴增以新增執行個體，為應用程式提供更多計算資源。</span><span class="sxs-lookup"><span data-stu-id="819f6-294">You can automatically scale out to add instances to provide more compute resources for your app.</span></span> <span data-ttu-id="819f6-295">同樣地，您也可以自動縮減並減少應用程式所需的執行個體數目。</span><span class="sxs-lookup"><span data-stu-id="819f6-295">Similarly, you can automatically scale in and reduce the number of instances your app needs.</span></span>

> [!Note]  
> <span data-ttu-id="819f6-296">您必須以 App Service 方案來設定擴增和縮減。</span><span class="sxs-lookup"><span data-stu-id="819f6-296">You need to have an App Service plan to configure scale out and scale in.</span></span> <span data-ttu-id="819f6-297">如果您沒有方案，請先建立方案，再開始執行後續步驟。</span><span class="sxs-lookup"><span data-stu-id="819f6-297">If you don't have a plan, create one before starting the next steps.</span></span>

### <a name="enable-automatic-scale-out"></a><span data-ttu-id="819f6-298">啟用自動相應放大</span><span class="sxs-lookup"><span data-stu-id="819f6-298">Enable automatic scale-out</span></span>

1. <span data-ttu-id="819f6-299">在 Azure 入口網站中，找出要擴增的網站所使用的 App Service 方案，然後選取 [擴增 (App Service 方案)]。</span><span class="sxs-lookup"><span data-stu-id="819f6-299">In the Azure portal, find the App Service plan for the sites you want to scale out, and then select **Scale-out (App Service plan)** .</span></span>

    ![擴增 Azure App Service](media/solution-deployment-guide-hybrid/image16.png)

2. <span data-ttu-id="819f6-301">選取 [啟用自動調整]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-301">Select **Enable autoscale** .</span></span>

    ![在 Azure App Service 中啟用自動調整](media/solution-deployment-guide-hybrid/image17.png)

3. <span data-ttu-id="819f6-303">輸入 [自動調整設定名稱]  的名稱。</span><span class="sxs-lookup"><span data-stu-id="819f6-303">Enter a name for **Autoscale Setting Name** .</span></span> <span data-ttu-id="819f6-304">選取 [依據計量調整規模]  ，作為 **預設** 自動調整規則。</span><span class="sxs-lookup"><span data-stu-id="819f6-304">For the **Default** auto scale rule, select **Scale based on a metric** .</span></span> <span data-ttu-id="819f6-305">將 [執行個體限制]  設為 **最小值：1** 、 **最大值：10** 和 **預設值：1** 。</span><span class="sxs-lookup"><span data-stu-id="819f6-305">Set the **Instance limits** to **Minimum: 1** , **Maximum: 10** , and **Default: 1** .</span></span>

    ![在 Azure App Service 中設定自動調整](media/solution-deployment-guide-hybrid/image18.png)

4. <span data-ttu-id="819f6-307">選取 [+新增規則]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-307">Select **+Add a rule** .</span></span>

5. <span data-ttu-id="819f6-308">在 [計量來源]  中，選取 [目前的資源]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-308">In **Metric Source** , select **Current Resource** .</span></span> <span data-ttu-id="819f6-309">請為規則使用下列準則和動作。</span><span class="sxs-lookup"><span data-stu-id="819f6-309">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="819f6-310">準則</span><span class="sxs-lookup"><span data-stu-id="819f6-310">Criteria</span></span>

1. <span data-ttu-id="819f6-311">在 [時間彙總]  下方，選取 [平均]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-311">Under **Time Aggregation,** select **Average** .</span></span>

2. <span data-ttu-id="819f6-312">在 [計量名稱]  下方，選取 [CPU 百分比]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-312">Under **Metric Name** , select **CPU Percentage** .</span></span>

3. <span data-ttu-id="819f6-313">在 [運算子]  下方，選取 [大於]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-313">Under **Operator** , select **Greater than** .</span></span>

   - <span data-ttu-id="819f6-314">將 [閾值]  設為 **50** 。</span><span class="sxs-lookup"><span data-stu-id="819f6-314">Set the **Threshold** to **50** .</span></span>
   - <span data-ttu-id="819f6-315">將 [持續時間]  設為 **10** 。</span><span class="sxs-lookup"><span data-stu-id="819f6-315">Set the **Duration** to **10** .</span></span>

#### <a name="action"></a><span data-ttu-id="819f6-316">動作</span><span class="sxs-lookup"><span data-stu-id="819f6-316">Action</span></span>

1. <span data-ttu-id="819f6-317">在 [作業]  下方，選取 [將計數增加]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-317">Under **Operation** , select **Increase Count by** .</span></span>

2. <span data-ttu-id="819f6-318">將 [執行個體計數]  設為 **2** 。</span><span class="sxs-lookup"><span data-stu-id="819f6-318">Set the **Instance Count** to **2** .</span></span>

3. <span data-ttu-id="819f6-319">將 [緩和時間]  設為 **5** 。</span><span class="sxs-lookup"><span data-stu-id="819f6-319">Set the **Cool down** to **5** .</span></span>

4. <span data-ttu-id="819f6-320">選取 [新增]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-320">Select **Add** .</span></span>

5. <span data-ttu-id="819f6-321">選取 [+新增規則]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-321">Select the **+ Add a rule** .</span></span>

6. <span data-ttu-id="819f6-322">在 [計量來源]  中，選取 [目前的資源]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-322">In **Metric Source** , select **Current Resource.**</span></span>

   > [!Note]  
   > <span data-ttu-id="819f6-323">目前的資源會包含您 App Service 方案的名稱/GUID，而 [資源類型]  和 [資源]  下拉式清單將無法使用。</span><span class="sxs-lookup"><span data-stu-id="819f6-323">The current resource will contain your App Service plan's name/GUID and the **Resource Type** and **Resource** drop-down lists will be unavailable.</span></span>

### <a name="enable-automatic-scale-in"></a><span data-ttu-id="819f6-324">啟用自動縮減</span><span class="sxs-lookup"><span data-stu-id="819f6-324">Enable automatic scale in</span></span>

<span data-ttu-id="819f6-325">當流量減少時，Azure Web 應用程式可以自動減少作用中的執行個體數目，以降低成本。</span><span class="sxs-lookup"><span data-stu-id="819f6-325">When traffic decreases, the Azure web app can automatically reduce the number of active instances to reduce costs.</span></span> <span data-ttu-id="819f6-326">此動作的主動性低於相應放大，且可盡量降低應用程式使用者所受到的影響。</span><span class="sxs-lookup"><span data-stu-id="819f6-326">This action is less aggressive than scale-out and minimizes the impact on app users.</span></span>

1. <span data-ttu-id="819f6-327">移至 **預設** 擴增條件，然後選取 [+ 新增規則]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-327">Go to the **Default** scale out condition, then select **+ Add a rule** .</span></span> <span data-ttu-id="819f6-328">請為規則使用下列準則和動作。</span><span class="sxs-lookup"><span data-stu-id="819f6-328">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="819f6-329">準則</span><span class="sxs-lookup"><span data-stu-id="819f6-329">Criteria</span></span>

1. <span data-ttu-id="819f6-330">在 [時間彙總]  下方，選取 [平均]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-330">Under **Time Aggregation,** select **Average** .</span></span>

2. <span data-ttu-id="819f6-331">在 [計量名稱]  下方，選取 [CPU 百分比]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-331">Under **Metric Name** , select **CPU Percentage** .</span></span>

3. <span data-ttu-id="819f6-332">在 [運算子]  下方，選取 [小於]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-332">Under **Operator** , select **Less than** .</span></span>

   - <span data-ttu-id="819f6-333">將 [閾值]  設為 **30** 。</span><span class="sxs-lookup"><span data-stu-id="819f6-333">Set the **Threshold** to **30** .</span></span>
   - <span data-ttu-id="819f6-334">將 [持續時間]  設為 **10** 。</span><span class="sxs-lookup"><span data-stu-id="819f6-334">Set the **Duration** to **10** .</span></span>

#### <a name="action"></a><span data-ttu-id="819f6-335">動作</span><span class="sxs-lookup"><span data-stu-id="819f6-335">Action</span></span>

1. <span data-ttu-id="819f6-336">在 [作業]  下方，選取 [將計數減少]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-336">Under **Operation** , select **Decrease Count by** .</span></span>

   - <span data-ttu-id="819f6-337">將 [執行個體計數]  設為 **1** 。</span><span class="sxs-lookup"><span data-stu-id="819f6-337">Set the **Instance Count** to **1** .</span></span>
   - <span data-ttu-id="819f6-338">將 [緩和時間]  設為 **5** 。</span><span class="sxs-lookup"><span data-stu-id="819f6-338">Set the **Cool down** to **5** .</span></span>

2. <span data-ttu-id="819f6-339">選取 [新增]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-339">Select **Add** .</span></span>

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a><span data-ttu-id="819f6-340">建立流量管理員設定檔並設定跨雲端縮放</span><span class="sxs-lookup"><span data-stu-id="819f6-340">Create a Traffic Manager profile and configure cross-cloud scaling</span></span>

<span data-ttu-id="819f6-341">使用 Azure 入口網站中建立流量管理員設定檔，然後設定端點以啟用跨雲端縮放。</span><span class="sxs-lookup"><span data-stu-id="819f6-341">Create a Traffic Manager profile using the Azure portal, then configure endpoints to enable cross-cloud scaling.</span></span>

### <a name="create-traffic-manager-profile"></a><span data-ttu-id="819f6-342">建立流量管理員設定檔</span><span class="sxs-lookup"><span data-stu-id="819f6-342">Create Traffic Manager profile</span></span>

1. <span data-ttu-id="819f6-343">選取 [建立資源]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-343">Select **Create a resource** .</span></span>
2. <span data-ttu-id="819f6-344">選取 [網路功能]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-344">Select **Networking** .</span></span>
3. <span data-ttu-id="819f6-345">選取 [流量管理員設定檔]  並進行下列設定：</span><span class="sxs-lookup"><span data-stu-id="819f6-345">Select **Traffic Manager profile** and configure the following settings:</span></span>

   - <span data-ttu-id="819f6-346">在 [名稱]  中，輸入設定檔的名稱。</span><span class="sxs-lookup"><span data-stu-id="819f6-346">In **Name** , enter a name for your profile.</span></span> <span data-ttu-id="819f6-347">此名稱在 trafficmanager.net 區域內 **必須** 是唯一的，並且用來建立新的 DNS 名稱 (例如 northwindstore.trafficmanager.net)。</span><span class="sxs-lookup"><span data-stu-id="819f6-347">This name **must** be unique in the trafficmanager.net zone and is used to create a new DNS name (for example, northwindstore.trafficmanager.net).</span></span>
   - <span data-ttu-id="819f6-348">針對 [路由方法]  ，選取 [加權]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-348">For **Routing method** , select the **Weighted** .</span></span>
   - <span data-ttu-id="819f6-349">針對 [訂用帳戶]  ，選取您要用來建立此設定檔的訂用帳戶。</span><span class="sxs-lookup"><span data-stu-id="819f6-349">For **Subscription** , select the subscription you want to create  this profile in.</span></span>
   - <span data-ttu-id="819f6-350">在 [資源群組]  中，為此設定檔建立新的資源群組。</span><span class="sxs-lookup"><span data-stu-id="819f6-350">In **Resource Group** , create a new resource group for this profile.</span></span>
   - <span data-ttu-id="819f6-351">在 [資源群組位置]  中，選取資源群組的位置。</span><span class="sxs-lookup"><span data-stu-id="819f6-351">In **Resource group location** , select the location of the resource group.</span></span> <span data-ttu-id="819f6-352">這項設定是指資源群組的位置，完全不影響將部署到全球的流量管理員設定檔。</span><span class="sxs-lookup"><span data-stu-id="819f6-352">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile that's deployed globally.</span></span>

4. <span data-ttu-id="819f6-353">選取 [建立]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-353">Select **Create** .</span></span>

    ![建立流量管理員設定檔](media/solution-deployment-guide-hybrid/image19.png)

   <span data-ttu-id="819f6-355">流量管理員設定檔在完成全域部署後，會顯示在其建立後所屬資源群組的資源清單中。</span><span class="sxs-lookup"><span data-stu-id="819f6-355">When the global deployment of your Traffic Manager profile is complete, it's shown in the list of resources for the resource group you created it under.</span></span>

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="819f6-356">新增流量管理員端點</span><span class="sxs-lookup"><span data-stu-id="819f6-356">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="819f6-357">搜尋您所建立的流量管理員設定檔。</span><span class="sxs-lookup"><span data-stu-id="819f6-357">Search for the Traffic Manager profile you created.</span></span> <span data-ttu-id="819f6-358">如果您瀏覽至設定檔的資源群組，請選取設定檔。</span><span class="sxs-lookup"><span data-stu-id="819f6-358">If you navigated to the resource group for the profile, select the profile.</span></span>

2. <span data-ttu-id="819f6-359">在 [流量管理員設定檔]  的 [設定]  下方，選取 [端點]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-359">In **Traffic Manager profile** , under **SETTINGS** , select **Endpoints** .</span></span>

3. <span data-ttu-id="819f6-360">選取 [新增]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-360">Select **Add** .</span></span>

4. <span data-ttu-id="819f6-361">在 [新增端點]  中，使用 Azure Stack Hub 的下列設定：</span><span class="sxs-lookup"><span data-stu-id="819f6-361">In **Add endpoint** , use the following settings for Azure Stack Hub:</span></span>

   - <span data-ttu-id="819f6-362">針對 [類型]  ，選取 [外部端點]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-362">For **Type** , select **External endpoint** .</span></span>
   - <span data-ttu-id="819f6-363">輸入端點的 **名稱** 。</span><span class="sxs-lookup"><span data-stu-id="819f6-363">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="819f6-364">針對 [完整網域名稱 (FQDN) 或 IP]  ，輸入 Azure Stack Hub Web 應用程式的外部 URL。</span><span class="sxs-lookup"><span data-stu-id="819f6-364">For **Fully qualified domain name (FQDN) or IP** , enter the external URL for your Azure Stack Hub web app.</span></span>
   - <span data-ttu-id="819f6-365">將 [權數]  保留為預設值 **1** 。</span><span class="sxs-lookup"><span data-stu-id="819f6-365">For **Weight** , keep the default, **1** .</span></span> <span data-ttu-id="819f6-366">此權數會使所有流量都傳送至此端點 (如果其狀況良好)。</span><span class="sxs-lookup"><span data-stu-id="819f6-366">This weight results in all traffic going to this endpoint if it's healthy.</span></span>
   - <span data-ttu-id="819f6-367">將 [新增為已停用]  保持為未核取。</span><span class="sxs-lookup"><span data-stu-id="819f6-367">Leave **Add as disabled** unchecked.</span></span>

5. <span data-ttu-id="819f6-368">選取 [確定]  以儲存 Azure Stack Hub 端點。</span><span class="sxs-lookup"><span data-stu-id="819f6-368">Select **OK** to save the Azure Stack Hub endpoint.</span></span>

<span data-ttu-id="819f6-369">接下來您將設定 Azure 端點。</span><span class="sxs-lookup"><span data-stu-id="819f6-369">You'll configure the Azure endpoint next.</span></span>

1. <span data-ttu-id="819f6-370">在 [流量管理員設定檔]  上，選取 [端點]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-370">On **Traffic Manager profile** , select **Endpoints** .</span></span>
2. <span data-ttu-id="819f6-371">選取 [+新增]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-371">Select **+Add** .</span></span>
3. <span data-ttu-id="819f6-372">在 [新增端點]  上，使用 Azure 的下列設定：</span><span class="sxs-lookup"><span data-stu-id="819f6-372">On **Add endpoint** , use the following settings for Azure:</span></span>

   - <span data-ttu-id="819f6-373">針對 [類型]  ，選取 [Azure 端點]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-373">For **Type** , select **Azure endpoint** .</span></span>
   - <span data-ttu-id="819f6-374">輸入端點的 **名稱** 。</span><span class="sxs-lookup"><span data-stu-id="819f6-374">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="819f6-375">針對 [目標資源類型]  ，選取 [App Service]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-375">For **Target resource type** , select **App Service** .</span></span>
   - <span data-ttu-id="819f6-376">針對 [目標資源]  選取 [選擇 App Service]  ，以顯示相同訂用帳戶中的 Web Apps 清單。</span><span class="sxs-lookup"><span data-stu-id="819f6-376">For **Target resource** , select **Choose an app service** to see a list of Web Apps in the same subscription.</span></span>
   - <span data-ttu-id="819f6-377">在 [資源]  中，挑選您想要新增為第一個端點的應用程式服務。</span><span class="sxs-lookup"><span data-stu-id="819f6-377">In **Resource** , pick the App service that you want to add as the first endpoint.</span></span>
   - <span data-ttu-id="819f6-378">針對 [權數]  ，選取 **2** 。</span><span class="sxs-lookup"><span data-stu-id="819f6-378">For **Weight** , select **2** .</span></span> <span data-ttu-id="819f6-379">此設定會使所有的流量在主要端點狀況不良時均傳送至此端點；或者，您可以使用經觸發會重新導向流量的規則/警示。</span><span class="sxs-lookup"><span data-stu-id="819f6-379">This setting results in all traffic going to this endpoint if the primary endpoint is unhealthy, or if you have a rule/alert that redirects traffic when triggered.</span></span>
   - <span data-ttu-id="819f6-380">將 [新增為已停用]  保持為未核取。</span><span class="sxs-lookup"><span data-stu-id="819f6-380">Leave **Add as disabled** unchecked.</span></span>

4. <span data-ttu-id="819f6-381">選取 [確定]  以儲存 Azure 端點。</span><span class="sxs-lookup"><span data-stu-id="819f6-381">Select **OK** to save the Azure endpoint.</span></span>

<span data-ttu-id="819f6-382">兩個端點都設定之後，將會在您選取 [端點]  時列於 [流量管理員設定檔]  中。</span><span class="sxs-lookup"><span data-stu-id="819f6-382">After both endpoints are configured, they're listed in **Traffic Manager profile** when you select **Endpoints** .</span></span> <span data-ttu-id="819f6-383">下列螢幕擷取畫面中的範例顯示兩個端點及其各自的狀態和設定資訊。</span><span class="sxs-lookup"><span data-stu-id="819f6-383">The example in the following screen capture shows two endpoints, with status and configuration information for each one.</span></span>

![流量管理員設定檔中的端點](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting-in-azure"></a><span data-ttu-id="819f6-385">在 Azure 中設定 Application Insights 監視和警示</span><span class="sxs-lookup"><span data-stu-id="819f6-385">Set up Application Insights monitoring and alerting in Azure</span></span>

<span data-ttu-id="819f6-386">Azure Application Insights 可讓您根據自己設定的條件來監視應用程式及傳送警示。</span><span class="sxs-lookup"><span data-stu-id="819f6-386">Azure Application Insights lets you monitor your app and send alerts based on conditions you configure.</span></span> <span data-ttu-id="819f6-387">其範例包括：應用程式無法使用、發生失敗狀況，或顯示效能問題。</span><span class="sxs-lookup"><span data-stu-id="819f6-387">Some examples are: the app is unavailable, is experiencing failures, or is showing performance issues.</span></span>

<span data-ttu-id="819f6-388">您將使用 Azure Application Insights 計量來建立警示。</span><span class="sxs-lookup"><span data-stu-id="819f6-388">You'll use Azure Application Insights metrics to create alerts.</span></span> <span data-ttu-id="819f6-389">當這些警示觸發時，Web 應用程式執行個體將會自動從 Azure Stack Hub 切換至 Azure 以擴增，然後再切換回 Azure Stack Hub 以縮減。</span><span class="sxs-lookup"><span data-stu-id="819f6-389">When these alerts trigger, your web app's instance will automatically switch from Azure Stack Hub to Azure to scale out, and then back to Azure Stack Hub to scale in.</span></span>

### <a name="create-an-alert-from-metrics"></a><span data-ttu-id="819f6-390">建立以計量為依據的警示</span><span class="sxs-lookup"><span data-stu-id="819f6-390">Create an alert from metrics</span></span>

<span data-ttu-id="819f6-391">在 Azure 入口網站中，請移至本教學課程的資源群組，然後選取 Application Insights 執行個體以開啟 **Application Insights** 。</span><span class="sxs-lookup"><span data-stu-id="819f6-391">In the Azure portal, go to the resource group for this tutorial, and select the Application Insights instance to open **Application Insights** .</span></span>

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

<span data-ttu-id="819f6-393">您將使用下列檢視建立擴增警示和縮減警示。</span><span class="sxs-lookup"><span data-stu-id="819f6-393">You'll use this view to create a scale-out alert and a scale-in alert.</span></span>

### <a name="create-the-scale-out-alert"></a><span data-ttu-id="819f6-394">建立相應放大警示</span><span class="sxs-lookup"><span data-stu-id="819f6-394">Create the scale-out alert</span></span>

1. <span data-ttu-id="819f6-395">在 [設定]  下方，選取 [警示 (傳統)]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-395">Under **CONFIGURE** , select **Alerts (classic)** .</span></span>
2. <span data-ttu-id="819f6-396">選取 [新增計量警示 (傳統)]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-396">Select **Add metric alert (classic)** .</span></span>
3. <span data-ttu-id="819f6-397">在 [新增規則]  中，進行下列設定：</span><span class="sxs-lookup"><span data-stu-id="819f6-397">In **Add rule** , configure the following settings:</span></span>

   - <span data-ttu-id="819f6-398">針對 [名稱]  ，輸入 [高載至 Azure 雲端]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-398">For **Name** , enter **Burst into Azure Cloud** .</span></span>
   - <span data-ttu-id="819f6-399">[描述]  是選擇性的。</span><span class="sxs-lookup"><span data-stu-id="819f6-399">A **Description** is optional.</span></span>
   - <span data-ttu-id="819f6-400">在 [來源]   > 、[警示標的]  下方，選取 [計量]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-400">Under **Source** > **Alert on** , select **Metrics** .</span></span>
   - <span data-ttu-id="819f6-401">在 [準則]  下方選取您的訂用帳戶、流量管理員設定檔的資源群組，和資源的流量管理員設定檔名稱。</span><span class="sxs-lookup"><span data-stu-id="819f6-401">Under **Criteria** , select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="819f6-402">針對 [計量]  ，選取 [要求率]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-402">For **Metric** , select **Request Rate** .</span></span>
5. <span data-ttu-id="819f6-403">針對 [條件]  ，選取 [大於]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-403">For **Condition** , select **Greater than** .</span></span>
6. <span data-ttu-id="819f6-404">輸入 **2** 作為 [閾值]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-404">For **Threshold** , enter **2** .</span></span>
7. <span data-ttu-id="819f6-405">針對 [期間]  ，選取 [過去 5 分鐘內]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-405">For **Period** , select **Over the last 5 minutes** .</span></span>
8. <span data-ttu-id="819f6-406">在 [通知方式]  下方：</span><span class="sxs-lookup"><span data-stu-id="819f6-406">Under **Notify via** :</span></span>
   - <span data-ttu-id="819f6-407">核取 [電子郵件擁有者、參與者和讀取者]  的核取方塊。</span><span class="sxs-lookup"><span data-stu-id="819f6-407">Check the checkbox for **Email owners, contributors, and readers** .</span></span>
   - <span data-ttu-id="819f6-408">輸入 [其他系統管理員電子郵件]  的電子郵件地址。</span><span class="sxs-lookup"><span data-stu-id="819f6-408">Enter your email address for **Additional administrator email(s)** .</span></span>

9. <span data-ttu-id="819f6-409">在功能表列上，選取 [儲存]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-409">On the menu bar, select **Save** .</span></span>

### <a name="create-the-scale-in-alert"></a><span data-ttu-id="819f6-410">建立縮減警示</span><span class="sxs-lookup"><span data-stu-id="819f6-410">Create the scale-in alert</span></span>

1. <span data-ttu-id="819f6-411">在 [設定]  下方，選取 [警示 (傳統)]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-411">Under **CONFIGURE** , select **Alerts (classic)** .</span></span>
2. <span data-ttu-id="819f6-412">選取 [新增計量警示 (傳統)]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-412">Select **Add metric alert (classic)** .</span></span>
3. <span data-ttu-id="819f6-413">在 [新增規則]  中，進行下列設定：</span><span class="sxs-lookup"><span data-stu-id="819f6-413">In **Add rule** , configure the following settings:</span></span>

   - <span data-ttu-id="819f6-414">針對 [名稱]  ，輸入 [調整回 Azure Stack Hub]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-414">For **Name** , enter **Scale back into Azure Stack Hub** .</span></span>
   - <span data-ttu-id="819f6-415">[描述]  是選擇性的。</span><span class="sxs-lookup"><span data-stu-id="819f6-415">A **Description** is optional.</span></span>
   - <span data-ttu-id="819f6-416">在 [來源]   > 、[警示標的]  下方，選取 [計量]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-416">Under **Source** > **Alert on** , select **Metrics** .</span></span>
   - <span data-ttu-id="819f6-417">在 [準則]  下方選取您的訂用帳戶、流量管理員設定檔的資源群組，和資源的流量管理員設定檔名稱。</span><span class="sxs-lookup"><span data-stu-id="819f6-417">Under **Criteria** , select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="819f6-418">針對 [計量]  ，選取 [要求率]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-418">For **Metric** , select **Request Rate** .</span></span>
5. <span data-ttu-id="819f6-419">針對 [條件]  ，選取 [小於]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-419">For **Condition** , select **Less than** .</span></span>
6. <span data-ttu-id="819f6-420">輸入 **2** 作為 [閾值]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-420">For **Threshold** , enter **2** .</span></span>
7. <span data-ttu-id="819f6-421">針對 [期間]  ，選取 [過去 5 分鐘內]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-421">For **Period** , select **Over the last 5 minutes** .</span></span>
8. <span data-ttu-id="819f6-422">在 [通知方式]  下方：</span><span class="sxs-lookup"><span data-stu-id="819f6-422">Under **Notify via** :</span></span>
   - <span data-ttu-id="819f6-423">核取 [電子郵件擁有者、參與者和讀取者]  的核取方塊。</span><span class="sxs-lookup"><span data-stu-id="819f6-423">Check the checkbox for **Email owners, contributors, and readers** .</span></span>
   - <span data-ttu-id="819f6-424">輸入 [其他系統管理員電子郵件]  的電子郵件地址。</span><span class="sxs-lookup"><span data-stu-id="819f6-424">Enter your email address for **Additional administrator email(s)** .</span></span>

9. <span data-ttu-id="819f6-425">在功能表列上，選取 [儲存]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-425">On the menu bar, select **Save** .</span></span>

<span data-ttu-id="819f6-426">下列螢幕擷取畫面顯示擴增和縮減的警示。</span><span class="sxs-lookup"><span data-stu-id="819f6-426">The following screenshot shows the alerts for scale-out and scale-in.</span></span>

   ![Application Insights 警示 (傳統)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a><span data-ttu-id="819f6-428">在 Azure 與 Azure Stack Hub 之間重新導向流量</span><span class="sxs-lookup"><span data-stu-id="819f6-428">Redirect traffic between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="819f6-429">您可以為 Azure 與 Azure Stack Hub 之間的 Web 應用程式流量設定手動或自動切換。</span><span class="sxs-lookup"><span data-stu-id="819f6-429">You can configure manual or automatic switching of your web app traffic between Azure and Azure Stack Hub.</span></span>

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="819f6-430">設定 Azure 與 Azure Stack Hub 之間的手動切換</span><span class="sxs-lookup"><span data-stu-id="819f6-430">Configure manual switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="819f6-431">當網站達到您所設定的閾值時，您將收到警示。</span><span class="sxs-lookup"><span data-stu-id="819f6-431">When your web site reaches the thresholds that you configure, you'll receive an alert.</span></span> <span data-ttu-id="819f6-432">請使用下列步驟以手動方式將流量重新導向至 Azure。</span><span class="sxs-lookup"><span data-stu-id="819f6-432">Use the following steps to manually redirect traffic to Azure.</span></span>

1. <span data-ttu-id="819f6-433">在 Azure 入口網站中，選取您的流量管理員設定檔。</span><span class="sxs-lookup"><span data-stu-id="819f6-433">In the Azure portal, select your Traffic Manager profile.</span></span>

    ![Azure 入口網站中的流量管理員端點](media/solution-deployment-guide-hybrid/image20.png)

2. <span data-ttu-id="819f6-435">選取 [端點]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-435">Select **Endpoints** .</span></span>
3. <span data-ttu-id="819f6-436">選取 [Azure 端點]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-436">Select the **Azure endpoint** .</span></span>
4. <span data-ttu-id="819f6-437">在 [狀態]  下方選取 [已啟用]  ，然後選取 [儲存]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-437">Under **Status** , select **Enabled** , and then select **Save** .</span></span>

    ![在 Azure 入口網站中啟用 Azure 端點](media/solution-deployment-guide-hybrid/image23.png)

5. <span data-ttu-id="819f6-439">在流量管理員設定檔的 [端點]  上，選取 [外部端點]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-439">On **Endpoints** for the Traffic Manager profile, select **External endpoint** .</span></span>
6. <span data-ttu-id="819f6-440">在 [狀態]  下方選取 [已停用]  ，然後選取 [儲存]  。</span><span class="sxs-lookup"><span data-stu-id="819f6-440">Under **Status** , select **Disabled** , and then select **Save** .</span></span>

    ![在 Azure 入口網站中停用 Azure Stack Hub 端點](media/solution-deployment-guide-hybrid/image24.png)

<span data-ttu-id="819f6-442">設定端點之後，應用程式流量將會傳送至您的 Azure 相應放大 Web 應用程式，而不是 Azure Stack Hub Web 應用程式。</span><span class="sxs-lookup"><span data-stu-id="819f6-442">After the endpoints are configured, app traffic goes to your Azure scale-out web app instead of the Azure Stack Hub web app.</span></span>

 ![Azure Web 應用程式流量中已變更的端點](media/solution-deployment-guide-hybrid/image25.png)

<span data-ttu-id="819f6-444">若要將流量反轉回 Azure Stack Hub，請使用先前的步驟執行下列動作：</span><span class="sxs-lookup"><span data-stu-id="819f6-444">To reverse the flow back to Azure Stack Hub, use the previous steps to:</span></span>

- <span data-ttu-id="819f6-445">啟用 Azure Stack Hub 端點。</span><span class="sxs-lookup"><span data-stu-id="819f6-445">Enable the Azure Stack Hub endpoint.</span></span>
- <span data-ttu-id="819f6-446">停用 Azure 端點。</span><span class="sxs-lookup"><span data-stu-id="819f6-446">Disable the Azure endpoint.</span></span>

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="819f6-447">設定 Azure 與 Azure Stack Hub 之間的自動切換</span><span class="sxs-lookup"><span data-stu-id="819f6-447">Configure automatic switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="819f6-448">如果您的應用程式執行於 Azure Functions 所提供的[無伺服器](https://azure.microsoft.com/overview/serverless-computing/)環境中，您也可以使用 Application Insights 監視。</span><span class="sxs-lookup"><span data-stu-id="819f6-448">You can also use Application Insights monitoring if your app runs in a [serverless](https://azure.microsoft.com/overview/serverless-computing/) environment provided by Azure Functions.</span></span>

<span data-ttu-id="819f6-449">在此案例中，您可以設定 Application Insights 以使用 Webhook 呼叫函式應用程式。</span><span class="sxs-lookup"><span data-stu-id="819f6-449">In this scenario, you can configure Application Insights to use a webhook that calls a function app.</span></span> <span data-ttu-id="819f6-450">此應用程式會自動啟用或停用端點，以回應警示。</span><span class="sxs-lookup"><span data-stu-id="819f6-450">This app automatically enables or disables an endpoint in response to an alert.</span></span>

<span data-ttu-id="819f6-451">若要設定自動流量切換，請依據下列步驟操作。</span><span class="sxs-lookup"><span data-stu-id="819f6-451">Use the following steps as a guide to configure automatic traffic switching.</span></span>

1. <span data-ttu-id="819f6-452">建立 Azure 函式應用程式。</span><span class="sxs-lookup"><span data-stu-id="819f6-452">Create an Azure Function app.</span></span>
2. <span data-ttu-id="819f6-453">建立由 HTTP 觸發的函式。</span><span class="sxs-lookup"><span data-stu-id="819f6-453">Create an HTTP-triggered function.</span></span>
3. <span data-ttu-id="819f6-454">匯入資源管理員、Web 應用程式和流量管理員的 Azure SDK。</span><span class="sxs-lookup"><span data-stu-id="819f6-454">Import the Azure SDKs for Resource Manager, Web Apps, and Traffic Manager.</span></span>
4. <span data-ttu-id="819f6-455">開發下列作業的程式碼：</span><span class="sxs-lookup"><span data-stu-id="819f6-455">Develop code to:</span></span>

   - <span data-ttu-id="819f6-456">向您的 Azure 訂用帳戶驗證。</span><span class="sxs-lookup"><span data-stu-id="819f6-456">Authenticate to your Azure subscription.</span></span>
   - <span data-ttu-id="819f6-457">使用參數切換流量管理員端點，以將流量導向至 Azure 或 Azure Stack Hub。</span><span class="sxs-lookup"><span data-stu-id="819f6-457">Use a parameter that toggles the Traffic Manager endpoints to direct traffic to Azure or Azure Stack Hub.</span></span>

5. <span data-ttu-id="819f6-458">儲存您的程式碼，並將含有適當參數的函式應用程式 URL 新增至 Application Insights 警示規則設定的 [Webhook]  區段。</span><span class="sxs-lookup"><span data-stu-id="819f6-458">Save your code and add the function app's URL with the appropriate parameters to the **Webhook** section of the Application Insights alert rule settings.</span></span>
6. <span data-ttu-id="819f6-459">Application Insights 警示引發時，流量會自動重新導向。</span><span class="sxs-lookup"><span data-stu-id="819f6-459">Traffic is automatically redirected when an Application Insights alert fires.</span></span>

## <a name="next-steps"></a><span data-ttu-id="819f6-460">後續步驟</span><span class="sxs-lookup"><span data-stu-id="819f6-460">Next steps</span></span>

- <span data-ttu-id="819f6-461">若要深入了解 Azure 雲端模式，請參閱[雲端設計模式](/azure/architecture/patterns)。</span><span class="sxs-lookup"><span data-stu-id="819f6-461">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
