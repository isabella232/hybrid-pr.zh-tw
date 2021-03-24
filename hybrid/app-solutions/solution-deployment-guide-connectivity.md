---
title: 在 Azure 與 Azure Stack Hub 中設定混合式雲端連線
description: 了解如何使用 Azure 與 Azure Stack Hub 設定混合式雲端連線。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 4480f51b03082f2a0cbb7f2f213e05b7bf488646
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895368"
---
# <a name="configure-hybrid-cloud-connectivity-using-azure-and-azure-stack-hub"></a><span data-ttu-id="d6f3a-103">使用 Azure 與 Azure Stack Hub 設定混合式雲端連線</span><span class="sxs-lookup"><span data-stu-id="d6f3a-103">Configure hybrid cloud connectivity using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="d6f3a-104">您可以在全域 Azure 與 Azure Stack Hub 中使用混合式連線模式安全地存取資源。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-104">You can access resources with security in global Azure and Azure Stack Hub using the hybrid connectivity pattern.</span></span>

<span data-ttu-id="d6f3a-105">在本解決方案中，您會建置環境範例，用以：</span><span class="sxs-lookup"><span data-stu-id="d6f3a-105">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="d6f3a-106">使內部部署資料符合隱私權或法規需求，同時能保有全域 Azure 資源的存取權。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-106">Keep data on-premises to meet privacy or regulatory requirements but keep access to global Azure resources.</span></span>
> - <span data-ttu-id="d6f3a-107">在全域 Azure 中使用雲端調整應用程式部署和資源的同時，維護舊版系統。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-107">Maintain a legacy system while using cloud-scaled app deployment and resources in global Azure.</span></span>

> [!Tip]  
> <span data-ttu-id="d6f3a-108">![混合式支柱圖](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="d6f3a-108">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="d6f3a-109">Microsoft Azure Stack Hub 是 Azure 的延伸模組。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-109">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="d6f3a-110">Azure Stack Hub 可將雲端運算的靈活性與創新能力導入您的內部部署環境中，並啟用獨特的混合式雲端，讓您能夠隨處建置及部署混合式應用程式。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-110">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="d6f3a-111">[混合式應用程式設計考量](overview-app-design-considerations.md)一文檢閱了設計、部署和操作混合式應用程式時的軟體品質要素 (放置、延展性、可用性、復原、管理性和安全性)。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-111">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="d6f3a-112">這些設計考量有助於您設計出最佳的混合式應用程式，減少生產環境可能會遇到的挑戰。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-112">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="d6f3a-113">Prerequisites</span><span class="sxs-lookup"><span data-stu-id="d6f3a-113">Prerequisites</span></span>

<span data-ttu-id="d6f3a-114">建置混合式連線部署時需要幾項元件。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-114">A few components are required to build a hybrid connectivity deployment.</span></span> <span data-ttu-id="d6f3a-115">其中有些元件可能需要一些時間來準備，請妥善規劃。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-115">Some of these components take time to prepare, so plan accordingly.</span></span>

### <a name="azure"></a><span data-ttu-id="d6f3a-116">Azure</span><span class="sxs-lookup"><span data-stu-id="d6f3a-116">Azure</span></span>

- <span data-ttu-id="d6f3a-117">如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-117">If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.</span></span>
- <span data-ttu-id="d6f3a-118">在 Azure 中建立 [Web 應用程式](/aspnet/core/tutorials/publish-to-azure-webapp-using-vs)。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-118">Create a [web app](/aspnet/core/tutorials/publish-to-azure-webapp-using-vs) in Azure.</span></span> <span data-ttu-id="d6f3a-119">請記下 Web 應用程式 URL，您在解決方案中將會需要用到這項資訊。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-119">Make note of the web app URL because you'll need it in the solution.</span></span>

### <a name="azure-stack-hub"></a><span data-ttu-id="d6f3a-120">Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="d6f3a-120">Azure Stack Hub</span></span>

<span data-ttu-id="d6f3a-121">Azure OEM/硬體合作夥伴可部署生產 Azure Stack Hub，而所有使用者均可部署 Azure Stack 開發套件 (ASDK)。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-121">An Azure OEM/hardware partner can deploy a production Azure Stack Hub, and all users can deploy an Azure Stack Development Kit (ASDK).</span></span>

- <span data-ttu-id="d6f3a-122">使用生產 Azure Stack Hub 或部署 ASDK。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-122">Use your production Azure Stack Hub or deploy the ASDK.</span></span>
   >[!Note]
   ><span data-ttu-id="d6f3a-123">部署 ASDK 可能需要 7 小時，因此請加以規劃。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-123">Deploying the ASDK can take up to 7 hours, so plan accordingly.</span></span>

- <span data-ttu-id="d6f3a-124">將 [App Service](/azure-stack/operator/azure-stack-app-service-deploy) PaaS 服務部署至 Azure Stack Hub。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-124">Deploy [App Service](/azure-stack/operator/azure-stack-app-service-deploy) PaaS services to Azure Stack Hub.</span></span>
- <span data-ttu-id="d6f3a-125">在 Azure Stack Hub 環境中[建立方案和供應項目](/azure-stack/operator/service-plan-offer-subscription-overview)。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-125">[Create plans and offers](/azure-stack/operator/service-plan-offer-subscription-overview) in the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="d6f3a-126">在 Azure Stack Hub 環境內[建立租用戶訂用帳戶](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm)。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-126">[Create tenant subscription](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm) within the Azure Stack Hub environment.</span></span>

### <a name="azure-stack-hub-components"></a><span data-ttu-id="d6f3a-127">Azure Stack Hub 元件</span><span class="sxs-lookup"><span data-stu-id="d6f3a-127">Azure Stack Hub components</span></span>

<span data-ttu-id="d6f3a-128">Azure Stack Hub 操作員必須部署 App Service、建立方案與供應項目、建立租用戶訂用帳戶，以及新增 Windows Server 2016 映像。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-128">An Azure Stack Hub operator must deploy the App Service, create plans and offers, create a tenant subscription, and add the Windows Server 2016 image.</span></span> <span data-ttu-id="d6f3a-129">如果您已擁有這些元件，請在開始本解決方案之前，先確定它們符合需求。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-129">If you already have these components, make sure they meet the requirements before you start this  solution.</span></span>

<span data-ttu-id="d6f3a-130">此解決方案範例假設您有 Azure 與 Azure Stack Hub 的一些基本知識。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-130">This solution example assumes that you have some basic knowledge of Azure and Azure Stack Hub.</span></span> <span data-ttu-id="d6f3a-131">若要在開始此解決方案前深入了解，請閱讀下列文章：</span><span class="sxs-lookup"><span data-stu-id="d6f3a-131">To learn more before starting the solution, read the following articles:</span></span>

- [<span data-ttu-id="d6f3a-132">Azure 簡介</span><span class="sxs-lookup"><span data-stu-id="d6f3a-132">Introduction to Azure</span></span>](https://azure.microsoft.com/overview/what-is-azure/)
- [<span data-ttu-id="d6f3a-133">Azure Stack Hub 重要概念</span><span class="sxs-lookup"><span data-stu-id="d6f3a-133">Azure Stack Hub Key Concepts</span></span>](/azure-stack/operator/azure-stack-overview)

### <a name="before-you-begin"></a><span data-ttu-id="d6f3a-134">開始之前</span><span class="sxs-lookup"><span data-stu-id="d6f3a-134">Before you begin</span></span>

<span data-ttu-id="d6f3a-135">先確認您符合下列準則，再開始設定混合式雲端連線：</span><span class="sxs-lookup"><span data-stu-id="d6f3a-135">Verify that you meet the following criteria before you start configuring hybrid cloud connectivity:</span></span>

- <span data-ttu-id="d6f3a-136">您需要 VPN 裝置對外開放的公用 IPv4 位址。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-136">You need an externally facing public IPv4 address for your VPN device.</span></span> <span data-ttu-id="d6f3a-137">此 IP 位址不能位於 NAT (網路位址轉譯) 後方。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-137">This IP address can't be located behind a NAT (Network Address Translation).</span></span>
- <span data-ttu-id="d6f3a-138">所有資源皆部署於相同的區域/位置。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-138">All resources are deployed in the same region/location.</span></span>

#### <a name="solution-example-values"></a><span data-ttu-id="d6f3a-139">解決方案範例值</span><span class="sxs-lookup"><span data-stu-id="d6f3a-139">Solution example values</span></span>

<span data-ttu-id="d6f3a-140">此解決方案中的範例使用下列值。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-140">The examples in this solution use the following values.</span></span> <span data-ttu-id="d6f3a-141">您可以使用這些值來建立測試環境，或參考這些值，進一步了解範例。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-141">You can use these values to create a test environment or refer to them for a better understanding of the examples.</span></span> <span data-ttu-id="d6f3a-142">如需特定 VPN 閘道設定的詳細資訊，請參閱[關於 VPN 閘道設定](/azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings)。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-142">For more information about VPN gateway settings, see [About VPN Gateway Settings](/azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings).</span></span>

<span data-ttu-id="d6f3a-143">連線規格：</span><span class="sxs-lookup"><span data-stu-id="d6f3a-143">Connection specifications:</span></span>

- <span data-ttu-id="d6f3a-144">**VPN 類型**：路由式</span><span class="sxs-lookup"><span data-stu-id="d6f3a-144">**VPN type**: route-based</span></span>
- <span data-ttu-id="d6f3a-145">**連線類型**︰站對站 (IPsec)</span><span class="sxs-lookup"><span data-stu-id="d6f3a-145">**Connection type**: site-to-site (IPsec)</span></span>
- <span data-ttu-id="d6f3a-146">**閘道類型**：VPN</span><span class="sxs-lookup"><span data-stu-id="d6f3a-146">**Gateway type**: VPN</span></span>
- <span data-ttu-id="d6f3a-147">**Azure 連線名稱**：Azure-Gateway-AzureStack-S2SGateway (入口網站會自動填入此值)</span><span class="sxs-lookup"><span data-stu-id="d6f3a-147">**Azure connection name**: Azure-Gateway-AzureStack-S2SGateway (the portal will autofill this value)</span></span>
- <span data-ttu-id="d6f3a-148">**Azure Stack Hub 連線名稱**：AzureStack-Gateway-Azure-S2SGateway (入口網站會自動填入此值)</span><span class="sxs-lookup"><span data-stu-id="d6f3a-148">**Azure Stack Hub connection name**: AzureStack-Gateway-Azure-S2SGateway (the portal will autofill this value)</span></span>
- <span data-ttu-id="d6f3a-149">**共用金鑰**：任何與 VPN 硬體相容、在連線兩端皆具有相符值的金鑰</span><span class="sxs-lookup"><span data-stu-id="d6f3a-149">**Shared key**: any compatible with VPN hardware, with matching values on both sides of connection</span></span>
- <span data-ttu-id="d6f3a-150">**訂用帳戶**：任何慣用的訂用帳戶</span><span class="sxs-lookup"><span data-stu-id="d6f3a-150">**Subscription**: any preferred subscription</span></span>
- <span data-ttu-id="d6f3a-151">**資源群組**：Test-Infra</span><span class="sxs-lookup"><span data-stu-id="d6f3a-151">**Resource group**: Test-Infra</span></span>

<span data-ttu-id="d6f3a-152">網路和子網路 IP 位址：</span><span class="sxs-lookup"><span data-stu-id="d6f3a-152">Network and subnet IP addresses:</span></span>

| <span data-ttu-id="d6f3a-153">Azure/Azure Stack Hub 連線</span><span class="sxs-lookup"><span data-stu-id="d6f3a-153">Azure/Azure Stack Hub Connection</span></span> | <span data-ttu-id="d6f3a-154">名稱</span><span class="sxs-lookup"><span data-stu-id="d6f3a-154">Name</span></span> | <span data-ttu-id="d6f3a-155">子網路</span><span class="sxs-lookup"><span data-stu-id="d6f3a-155">Subnet</span></span> | <span data-ttu-id="d6f3a-156">IP 位址</span><span class="sxs-lookup"><span data-stu-id="d6f3a-156">IP Address</span></span> |
|---|---|---|---|
| <span data-ttu-id="d6f3a-157">Azure vNet</span><span class="sxs-lookup"><span data-stu-id="d6f3a-157">Azure vNet</span></span> | <span data-ttu-id="d6f3a-158">ApplicationvNet</span><span class="sxs-lookup"><span data-stu-id="d6f3a-158">ApplicationvNet</span></span><br><span data-ttu-id="d6f3a-159">10.100.102.9/23</span><span class="sxs-lookup"><span data-stu-id="d6f3a-159">10.100.102.9/23</span></span> | <span data-ttu-id="d6f3a-160">ApplicationSubnet</span><span class="sxs-lookup"><span data-stu-id="d6f3a-160">ApplicationSubnet</span></span><br><span data-ttu-id="d6f3a-161">10.100.102.0/24</span><span class="sxs-lookup"><span data-stu-id="d6f3a-161">10.100.102.0/24</span></span> |  |
|  |  | <span data-ttu-id="d6f3a-162">GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="d6f3a-162">GatewaySubnet</span></span><br><span data-ttu-id="d6f3a-163">10.100.103.0/24</span><span class="sxs-lookup"><span data-stu-id="d6f3a-163">10.100.103.0/24</span></span> |  |
| <span data-ttu-id="d6f3a-164">Azure Stack Hub vNet</span><span class="sxs-lookup"><span data-stu-id="d6f3a-164">Azure Stack Hub vNet</span></span> | <span data-ttu-id="d6f3a-165">ApplicationvNet</span><span class="sxs-lookup"><span data-stu-id="d6f3a-165">ApplicationvNet</span></span><br><span data-ttu-id="d6f3a-166">10.100.100.0/23</span><span class="sxs-lookup"><span data-stu-id="d6f3a-166">10.100.100.0/23</span></span> | <span data-ttu-id="d6f3a-167">ApplicationSubnet</span><span class="sxs-lookup"><span data-stu-id="d6f3a-167">ApplicationSubnet</span></span> <br><span data-ttu-id="d6f3a-168">10.100.100.0/24</span><span class="sxs-lookup"><span data-stu-id="d6f3a-168">10.100.100.0/24</span></span> |  |
|  |  | <span data-ttu-id="d6f3a-169">GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="d6f3a-169">GatewaySubnet</span></span> <br><span data-ttu-id="d6f3a-170">10.100101.0/24</span><span class="sxs-lookup"><span data-stu-id="d6f3a-170">10.100101.0/24</span></span> |  |
| <span data-ttu-id="d6f3a-171">Azure 虛擬網路閘道</span><span class="sxs-lookup"><span data-stu-id="d6f3a-171">Azure Virtual Network Gateway</span></span> | <span data-ttu-id="d6f3a-172">Azure-Gateway</span><span class="sxs-lookup"><span data-stu-id="d6f3a-172">Azure-Gateway</span></span> |  |  |
| <span data-ttu-id="d6f3a-173">Azure Stack Hub 虛擬網路閘道</span><span class="sxs-lookup"><span data-stu-id="d6f3a-173">Azure Stack Hub Virtual Network Gateway</span></span> | <span data-ttu-id="d6f3a-174">AzureStack-Gateway</span><span class="sxs-lookup"><span data-stu-id="d6f3a-174">AzureStack-Gateway</span></span> |  |  |
| <span data-ttu-id="d6f3a-175">Azure 公用 IP</span><span class="sxs-lookup"><span data-stu-id="d6f3a-175">Azure Public IP</span></span> | <span data-ttu-id="d6f3a-176">Azure-GatewayPublicIP</span><span class="sxs-lookup"><span data-stu-id="d6f3a-176">Azure-GatewayPublicIP</span></span> |  | <span data-ttu-id="d6f3a-177">在建立時決定</span><span class="sxs-lookup"><span data-stu-id="d6f3a-177">Determined at creation</span></span> |
| <span data-ttu-id="d6f3a-178">Azure Stack Hub 公用 IP</span><span class="sxs-lookup"><span data-stu-id="d6f3a-178">Azure Stack Hub Public IP</span></span> | <span data-ttu-id="d6f3a-179">AzureStack-GatewayPublicIP</span><span class="sxs-lookup"><span data-stu-id="d6f3a-179">AzureStack-GatewayPublicIP</span></span> |  | <span data-ttu-id="d6f3a-180">在建立時決定</span><span class="sxs-lookup"><span data-stu-id="d6f3a-180">Determined at creation</span></span> |
| <span data-ttu-id="d6f3a-181">Azure 區域網路閘道</span><span class="sxs-lookup"><span data-stu-id="d6f3a-181">Azure Local Network Gateway</span></span> | <span data-ttu-id="d6f3a-182">AzureStack-S2SGateway</span><span class="sxs-lookup"><span data-stu-id="d6f3a-182">AzureStack-S2SGateway</span></span><br>   <span data-ttu-id="d6f3a-183">10.100.100.0/23</span><span class="sxs-lookup"><span data-stu-id="d6f3a-183">10.100.100.0/23</span></span> |  | <span data-ttu-id="d6f3a-184">Azure Stack Hub 公用 IP 值</span><span class="sxs-lookup"><span data-stu-id="d6f3a-184">Azure Stack Hub Public IP Value</span></span> |
| <span data-ttu-id="d6f3a-185">Azure Stack Hub 區域網路閘道</span><span class="sxs-lookup"><span data-stu-id="d6f3a-185">Azure Stack Hub Local Network Gateway</span></span> | <span data-ttu-id="d6f3a-186">Azure-S2SGateway</span><span class="sxs-lookup"><span data-stu-id="d6f3a-186">Azure-S2SGateway</span></span><br><span data-ttu-id="d6f3a-187">10.100.102.0/23</span><span class="sxs-lookup"><span data-stu-id="d6f3a-187">10.100.102.0/23</span></span> |  | <span data-ttu-id="d6f3a-188">Azure 公用 IP 值</span><span class="sxs-lookup"><span data-stu-id="d6f3a-188">Azure Public IP Value</span></span> |

## <a name="create-a-virtual-network-in-global-azure-and-azure-stack-hub"></a><span data-ttu-id="d6f3a-189">在全域 Azure 與 Azure Stack Hub 中建立虛擬網路</span><span class="sxs-lookup"><span data-stu-id="d6f3a-189">Create a virtual network in global Azure and Azure Stack Hub</span></span>

<span data-ttu-id="d6f3a-190">透過 Azure 入口網站，使用下列步驟來建立虛擬網路。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-190">Use the following steps to create a virtual network by using the portal.</span></span> <span data-ttu-id="d6f3a-191">如果您使用本文作為解決方案，則可使用[範例值](/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal#values)。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-191">You can use these [example values](/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal#values) if you're using this article as only a  solution.</span></span> <span data-ttu-id="d6f3a-192">如果使用本文來設定生產環境，請以您自己的值取代範例設定。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-192">If you're using this article to configure a production environment, replace the example settings with  your own values.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="d6f3a-193">您必須確定在 Azure 或 Azure Stack Hub vNet 位址空間中沒有任何重疊的 IP 位址。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-193">You must ensure that there isn't an overlap of IP addresses in Azure or Azure Stack Hub vNet address spaces.</span></span>

<span data-ttu-id="d6f3a-194">若要在 Azure 中建立 vNet：</span><span class="sxs-lookup"><span data-stu-id="d6f3a-194">To create a vNet in Azure:</span></span>

1. <span data-ttu-id="d6f3a-195">使用瀏覽器連線至 [Azure 入口網站](https://portal.azure.com/) ，並使用您的 Azure 帳戶登入。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-195">Use your browser to connect to the [Azure portal](https://portal.azure.com/) and sign in with your Azure account.</span></span>
2. <span data-ttu-id="d6f3a-196">選取 [建立資源]  。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-196">Select **Create a resource**.</span></span> <span data-ttu-id="d6f3a-197">在 [搜尋 Marketplace]  欄位中，輸入「虛擬網路」。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-197">In the **Search the marketplace** field, enter 'virtual network'.</span></span> <span data-ttu-id="d6f3a-198">從結果中選取 [虛擬網路]  。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-198">Select **Virtual network** from the results.</span></span>
3. <span data-ttu-id="d6f3a-199">從 [選取部署模型]  清單，選取 [Resource Manager]  ，然後選取 [建立]  。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-199">From the **Select a deployment model** list, select **Resource Manager**, and then select **Create**.</span></span>
4. <span data-ttu-id="d6f3a-200">在 [建立虛擬網路]  上進行 VNet 設定。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-200">On **Create virtual network**, configure the VNet settings.</span></span> <span data-ttu-id="d6f3a-201">必填欄位的名稱前面會加上紅色星號。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-201">The required fields names are prefixed with a red asterisk.</span></span>  <span data-ttu-id="d6f3a-202">當您輸入有效值時，星號會變成綠色核取記號。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-202">When you enter a valid value, the asterisk changes to a green check mark.</span></span>

<span data-ttu-id="d6f3a-203">在 Azure Stack Hub 中建立 vNet：</span><span class="sxs-lookup"><span data-stu-id="d6f3a-203">To create a vNet in Azure Stack Hub:</span></span>

1. <span data-ttu-id="d6f3a-204">使用 Azure Stack Hub 的 **租用戶入口網站**，重複執行上述步驟 (1-4)。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-204">Repeat the steps above (1-4) using the Azure Stack Hub **tenant portal**.</span></span>

## <a name="add-a-gateway-subnet"></a><span data-ttu-id="d6f3a-205">新增閘道子網路</span><span class="sxs-lookup"><span data-stu-id="d6f3a-205">Add a gateway subnet</span></span>

<span data-ttu-id="d6f3a-206">將虛擬網路連線到閘道之前，您必須先為您要連線的虛擬網路建立閘道子網路。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-206">Before connecting your virtual network to a gateway, you need to create the gateway subnet for the virtual network that you want to connect to.</span></span> <span data-ttu-id="d6f3a-207">閘道服務會使用您在閘道子網路中指定的 IP 位址。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-207">The gateway services use the IP addresses you specify in the gateway subnet.</span></span>

<span data-ttu-id="d6f3a-208">在 [Azure 入口網站](https://portal.azure.com/)中，瀏覽至要建立虛擬網路閘道的 Resource Manager 虛擬網路。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-208">In the [Azure portal](https://portal.azure.com/), navigate to the Resource Manager virtual network where you want to create a virtual network gateway.</span></span>

1. <span data-ttu-id="d6f3a-209">選取 vNet 以開啟 [虛擬網路]  頁面。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-209">Select the vNet to open the **Virtual network** page.</span></span>
2. <span data-ttu-id="d6f3a-210">在 [設定]  中，選取 [子網路]  。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-210">In **SETTINGS**, select **Subnets**.</span></span>
3. <span data-ttu-id="d6f3a-211">在 [子網路]  頁面中，選取 [+閘道子網路]  以開啟 [新增子網路]  頁面。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-211">On the **Subnets** page, select **+Gateway subnet** to open the **Add subnet** page.</span></span>

    ![新增閘道子網路](media/solution-deployment-guide-connectivity/image4.png)

4. <span data-ttu-id="d6f3a-213">子網路的 [名稱]  會自動填入 'GatewaySubnet' 這個值。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-213">The **Name** for the subnet is automatically filled in with the value 'GatewaySubnet'.</span></span> <span data-ttu-id="d6f3a-214">若要讓 Azure 將此子網路視為閘道子網路，則必須要有此值。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-214">This value is required for Azure to recognize the subnet as the gateway subnet.</span></span>
5. <span data-ttu-id="d6f3a-215">變更提供的 [位址範圍]  值以符合您的設定需求，然後選取 [確定]  。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-215">Change the **Address range** values that are provided to match your configuration requirements and then select **OK**.</span></span>

## <a name="create-a-virtual-network-gateway-in-azure-and-azure-stack"></a><span data-ttu-id="d6f3a-216">在 Azure 與 Azure Stack 中建立虛擬網路閘道</span><span class="sxs-lookup"><span data-stu-id="d6f3a-216">Create a Virtual Network Gateway in Azure and Azure Stack</span></span>

<span data-ttu-id="d6f3a-217">在 Azure 中使用下列步驟來建立虛擬網路閘道。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-217">Use the following steps to create a virtual network gateway in Azure.</span></span>

1. <span data-ttu-id="d6f3a-218">在入口網站頁面的左側，選取 **+** 並且在搜尋欄位中輸入「虛擬網路閘道」。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-218">On the left side of the portal page, select **+** and enter 'virtual network gateway' in the search field.</span></span>
2. <span data-ttu-id="d6f3a-219">在 [結果]  中，選取 [虛擬網路閘道]  。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-219">In **Results**, select **Virtual network gateway**.</span></span>
3. <span data-ttu-id="d6f3a-220">在 [虛擬網路閘道]  中，選取 [建立]  以開啟 [建立虛擬網路閘道]  頁面。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-220">In **Virtual network gateway**, select **Create** to open the **Create virtual network gateway** page.</span></span>
4. <span data-ttu-id="d6f3a-221">在 [建立虛擬網路閘道]  上，使用 [教學課程範例值]  指定網路閘道的值。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-221">On **Create virtual network gateway**, specify the values for your network gateway using our **Tutorial example values**.</span></span> <span data-ttu-id="d6f3a-222">請納入下列額外值：</span><span class="sxs-lookup"><span data-stu-id="d6f3a-222">Include the following additional values:</span></span>

   - <span data-ttu-id="d6f3a-223">**SKU**：基本</span><span class="sxs-lookup"><span data-stu-id="d6f3a-223">**SKU**: basic</span></span>
   - <span data-ttu-id="d6f3a-224">**虛擬網路**：選取您先前建立的虛擬網路。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-224">**Virtual Network**: Select the virtual network you created earlier.</span></span> <span data-ttu-id="d6f3a-225">系統會自動選取您建立的閘道子網路。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-225">The gateway subnet you created is automatically selected.</span></span>
   - <span data-ttu-id="d6f3a-226">**第一個 IP 組態**：這是您閘道的公用 IP。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-226">**First IP Configuration**:  The public IP of your gateway.</span></span>
     - <span data-ttu-id="d6f3a-227">選取 [建立閘道 IP 組態]  ，您即會導向至 [選擇公用 IP 位址]  頁面。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-227">Select **Create gateway IP configuration**, which takes you to the **Choose public IP address** page.</span></span>
     - <span data-ttu-id="d6f3a-228">選取 [+新建]  ，以開啟 [建立公用 IP 位址]  頁面。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-228">Select **+Create new** to open the **Create public IP address** page.</span></span>
     - <span data-ttu-id="d6f3a-229">輸入公用 IP 位址的 [名稱]  。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-229">Enter a **Name** for your public IP address.</span></span> <span data-ttu-id="d6f3a-230">讓 SKU 保持為 [基本]  ，然後選取 [確定]  以儲存變更。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-230">Leave the SKU as **Basic**, and then select **OK** to save your changes.</span></span>

       > [!Note]
       > <span data-ttu-id="d6f3a-231">VPN 閘道目前僅支援動態公用 IP 位址配置。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-231">Currently, VPN Gateway only supports Dynamic Public IP address allocation.</span></span> <span data-ttu-id="d6f3a-232">不過，這不表示之後的 IP 位址變更已指派至您的 VPN 閘道。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-232">However, this doesn't mean that the IP address changes after it's assigned to your VPN gateway.</span></span> <span data-ttu-id="d6f3a-233">公用 IP 位址只會在刪除或重新建立閘道時變更。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-233">The only time the public IP address changes is when the gateway is deleted and re-created.</span></span> <span data-ttu-id="d6f3a-234">重新調整、重設或 VPN 閘道的其他內部維護/升級不會變更 IP 位址。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-234">Resizing, resetting, or other internal maintenance/upgrades to your VPN gateway don't change the IP address.</span></span>

5. <span data-ttu-id="d6f3a-235">確認您的閘道設定。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-235">Verify your gateway settings.</span></span>
6. <span data-ttu-id="d6f3a-236">選取 [建立]  來建立 VPN 閘道。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-236">Select **Create** to create the VPN gateway.</span></span> <span data-ttu-id="d6f3a-237">閘道設定會經過驗證，而 [部署虛擬網路閘道] 圖格會顯示在儀表板上。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-237">The gateway settings are validated and the "Deploying Virtual network gateway" tile is shown on your dashboard.</span></span>

   >[!Note]
   ><span data-ttu-id="d6f3a-238">建立閘道可能需要長達 45 分鐘。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-238">Creating a gateway can take up to 45 minutes.</span></span> <span data-ttu-id="d6f3a-239">您可能需要重新整理入口網站頁面，才能看到完成的狀態。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-239">You may need to refresh your portal page to see the completed status.</span></span>

    <span data-ttu-id="d6f3a-240">建立閘道之後，您可以在入口網站中查看虛擬網路，檢視已指派給閘道的 IP 位址。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-240">After the gateway is created, you can see the IP address assigned to it by looking at the virtual network in the portal.</span></span> <span data-ttu-id="d6f3a-241">閘道會顯示為已連接的裝置。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-241">The gateway appears as a connected device.</span></span> <span data-ttu-id="d6f3a-242">如需有關閘道的詳細資訊，請選取裝置。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-242">To see more information about the gateway, select the device.</span></span>

7. <span data-ttu-id="d6f3a-243">在您的 Azure Stack Hub 部署上重複上述步驟 (1-5)。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-243">Repeat the previous steps (1-5) on your Azure Stack Hub deployment.</span></span>

## <a name="create-the-local-network-gateway-in-azure-and-azure-stack-hub"></a><span data-ttu-id="d6f3a-244">在 Azure 與 Azure Stack Hub 中建立區域網路閘道</span><span class="sxs-lookup"><span data-stu-id="d6f3a-244">Create the local network gateway in Azure and Azure Stack Hub</span></span>

<span data-ttu-id="d6f3a-245">區域網路閘道通常是指您的內部部署位置。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-245">The local network gateway typically refers to your on-premises location.</span></span> <span data-ttu-id="d6f3a-246">您會為網站提供 Azure 或 Azure Stack Hub 可以參考的名稱，然後指定：</span><span class="sxs-lookup"><span data-stu-id="d6f3a-246">You give the site a name that Azure or Azure Stack Hub can refer to, and then specify:</span></span>

- <span data-ttu-id="d6f3a-247">內部部署 VPN 裝置的 IP 位址，而這是您要建立連線的 VPN 裝置。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-247">The IP address of the on-premises VPN device that you're creating a connection for.</span></span>
- <span data-ttu-id="d6f3a-248">IP 位址首碼，以供系統透過 VPN 閘道路由至 VPN 裝置。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-248">The IP address prefixes that will be routed through the VPN gateway to the VPN device.</span></span> <span data-ttu-id="d6f3a-249">您指定的位址首碼是位於內部部署網路上的首碼。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-249">The address prefixes you specify are the prefixes located on your on-premises network.</span></span>

  >[!Note]
  ><span data-ttu-id="d6f3a-250">如果您的內部部署網路有所變更，或者您需要變更 VPN 裝置的公用 IP 位址，您稍後可以更新這些值。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-250">If your on-premises network changes or you need to change the public IP address for the VPN device, you can update these values later.</span></span>

1. <span data-ttu-id="d6f3a-251">在入口網站中，選取 [+建立資源]  。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-251">In the portal, select **+Create a resource**.</span></span>
2. <span data-ttu-id="d6f3a-252">在搜尋方塊中輸入 **區域網路閘道**，然後選取 **Enter** 鍵進行搜尋。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-252">In the search box, enter **Local network gateway**, then select **Enter** to search.</span></span> <span data-ttu-id="d6f3a-253">會顯示結果清單。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-253">A list of results will display.</span></span>
3. <span data-ttu-id="d6f3a-254">選取 [區域網路閘道]  ，然後選取 [建立]  以開啟 [建立區域網路閘道]  頁面。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-254">Select **Local network gateway**, then select **Create** to open the **Create local network gateway** page.</span></span>
4. <span data-ttu-id="d6f3a-255">在 [建立區域網路閘道]  上，使用 [教學課程範例值]  指定區域網路閘道的值。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-255">On **Create local network gateway**, specify the values for your local network gateway using our **Tutorial example values**.</span></span> <span data-ttu-id="d6f3a-256">請納入下列額外值：</span><span class="sxs-lookup"><span data-stu-id="d6f3a-256">Include the following additional values:</span></span>

    - <span data-ttu-id="d6f3a-257">**IP 位址**：這是您要讓 Azure 或 Azure Stack Hub 連線之 VPN 裝置的公用 IP 位址。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-257">**IP address**: The public IP address of the VPN device that you want Azure or Azure Stack Hub to connect to.</span></span> <span data-ttu-id="d6f3a-258">指定不在 NAT 後方的有效公用 IP 位址，以便 Azure 到達此位址。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-258">Specify a valid public IP address that isn't behind a NAT so Azure can reach the address.</span></span> <span data-ttu-id="d6f3a-259">如果您目前沒有 IP 位址，您可以使用範例中的值作為預留位置。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-259">If you don't have the IP address right now, you can use a value from the example as a placeholder.</span></span> <span data-ttu-id="d6f3a-260">您後續必須回過頭將預留位置取代為 VPN 裝置的公用 IP 位址。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-260">You'll have to go back and replace the placeholder with the public IP address of your VPN device.</span></span> <span data-ttu-id="d6f3a-261">在您提供有效的位址前，Azure 無法連線到裝置。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-261">Azure can't connect to the device until you provide a valid address.</span></span>
    - <span data-ttu-id="d6f3a-262">**位址空間**：此區域網路所代表網路的位址範圍。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-262">**Address Space**: the address range for the network that this local network represents.</span></span> <span data-ttu-id="d6f3a-263">您可以加入多個位址空間範圍。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-263">You can add multiple address space ranges.</span></span> <span data-ttu-id="d6f3a-264">確定您指定的範圍，不會與您要連線的其他網路範圍重疊。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-264">Make sure that the ranges you specify don't overlap with ranges of other networks that you want to connect to.</span></span> <span data-ttu-id="d6f3a-265">Azure 會將您指定的位址範圍路由傳送至內部部署 VPN 裝置 IP 位址。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-265">Azure will route the address range that you specify to the on-premises VPN device IP address.</span></span> <span data-ttu-id="d6f3a-266">如果您想要連線至內部部署網站，請使用您自己的值，而不是範例值。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-266">Use your own values if you want to connect to your on-premises site, not an example value.</span></span>
    - <span data-ttu-id="d6f3a-267">**設定 BGP 設定**：僅在設定 BGP 時才使用。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-267">**Configure BGP settings**: Use only when configuring BGP.</span></span> <span data-ttu-id="d6f3a-268">否則，請勿選取此選項。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-268">Otherwise, don't select this option.</span></span>
    - <span data-ttu-id="d6f3a-269">訂用帳戶  ：請確認已顯示正確的訂用帳戶。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-269">**Subscription**: Verify that the correct subscription is showing.</span></span>
    - <span data-ttu-id="d6f3a-270">**資源群組**：選取您想要使用的資源群組。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-270">**Resource Group**: Select the resource group that you want to use.</span></span> <span data-ttu-id="d6f3a-271">您可以建立新的資源群組，或選取您已建立的資源群組。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-271">You can either create a new resource group or select one that you've already created.</span></span>
    - <span data-ttu-id="d6f3a-272">**位置**：選取將要建立此物件的位置。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-272">**Location**: Select the location that this object will be created in.</span></span> <span data-ttu-id="d6f3a-273">您可以選取 VNet 所在的相同位置，但您不需要這麼做。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-273">You may want to select the same location that your VNet resides in, but you're not required to do so.</span></span>
5. <span data-ttu-id="d6f3a-274">當您完成必要值的指定時，請選取 [建立]  以建立區域網路閘道。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-274">When you finish specifying the required values, select **Create** to create the local network gateway.</span></span>
6. <span data-ttu-id="d6f3a-275">在您的 Azure Stack Hub 部署上重複這些步驟 (1-5)。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-275">Repeat these steps (1-5) on your Azure Stack Hub deployment.</span></span>

## <a name="configure-your-connection"></a><span data-ttu-id="d6f3a-276">設定您的連線</span><span class="sxs-lookup"><span data-stu-id="d6f3a-276">Configure your connection</span></span>

<span data-ttu-id="d6f3a-277">內部部署網路的站對站連線需要 VPN 裝置。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-277">Site-to-site connections to an on-premises network require a VPN device.</span></span> <span data-ttu-id="d6f3a-278">您設定的 VPN 裝置，稱為連線。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-278">The VPN device you configure is referred to as a connection.</span></span> <span data-ttu-id="d6f3a-279">若要設定連線，您必須：</span><span class="sxs-lookup"><span data-stu-id="d6f3a-279">To configure your connection, you need:</span></span>

- <span data-ttu-id="d6f3a-280">共用金鑰。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-280">A shared key.</span></span> <span data-ttu-id="d6f3a-281">這個金鑰與您建立站對站 VPN 連線時指定的共用金鑰相同。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-281">This key is the same shared key that you specify when creating your site-to-site VPN connection.</span></span> <span data-ttu-id="d6f3a-282">在我們的範例中，我們會使用基本的共用金鑰。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-282">In our examples, we use a basic shared key.</span></span> <span data-ttu-id="d6f3a-283">我們建議您產生更複雜的金鑰以供使用。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-283">We recommend that you generate a more complex key to use.</span></span>
- <span data-ttu-id="d6f3a-284">虛擬網路閘道的公用 IP 位址。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-284">The public IP address of your virtual network gateway.</span></span> <span data-ttu-id="d6f3a-285">您可以使用 Azure 入口網站、PowerShell 或 CLI 來檢視公用 IP 位址。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-285">You can view the public IP address by using the Azure portal, PowerShell, or CLI.</span></span> <span data-ttu-id="d6f3a-286">若要使用 Azure 入口網站尋找 VPN 閘道的公用 IP 位址，請移至虛擬網路閘道，然後選取閘道名稱。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-286">To find the public IP address of your VPN gateway using the Azure portal, go to virtual network gateways, then select the name of your gateway.</span></span>

<span data-ttu-id="d6f3a-287">使用下列步驟，在虛擬網路閘道與內部部署 VPN 裝置之間建立站對站 VPN 連線。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-287">Use the following steps to create a site-to-site VPN connection between your virtual network gateway and your on-premises VPN device.</span></span>

1. <span data-ttu-id="d6f3a-288">在 Azure 入口網站中，選取 [+建立資源]  。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-288">In the Azure portal, select **+Create a resource**.</span></span>
2. <span data-ttu-id="d6f3a-289">搜尋 [連線]  。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-289">Search for **connections**.</span></span>
3. <span data-ttu-id="d6f3a-290">在 [結果]  中，選取 [連線]  。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-290">In **Results**, select **Connections**.</span></span>
4. <span data-ttu-id="d6f3a-291">在 [連線]  上，選取 [建立]  。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-291">On **Connection**, select **Create**.</span></span>
5. <span data-ttu-id="d6f3a-292">在 [建立連線]  上，進行下列設定：</span><span class="sxs-lookup"><span data-stu-id="d6f3a-292">On **Create Connection**, configure the following settings:</span></span>

    - <span data-ttu-id="d6f3a-293">**連線類型**：選取 [站對站 \(IPSec\)]。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-293">**Connection type**: Select site-to-site (IPSec).</span></span>
    - <span data-ttu-id="d6f3a-294">**資源群組**：選取您的測試資源群組。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-294">**Resource Group**: Select your test resource group.</span></span>
    - <span data-ttu-id="d6f3a-295">**虛擬網路閘道**：選取您所建立的虛擬網路閘道。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-295">**Virtual Network Gateway**: Select the virtual network gateway you created.</span></span>
    - <span data-ttu-id="d6f3a-296">**區域網路閘道**：選取您所建立的區域網路閘道。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-296">**Local Network Gateway**: Select the local network gateway you created.</span></span>
    - <span data-ttu-id="d6f3a-297">**連線名稱**：此名稱會自動填入來自兩個閘道的值。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-297">**Connection Name**: This name is autopopulated using the values from the two gateways.</span></span>
    - <span data-ttu-id="d6f3a-298">**共用金鑰**：此值必須與您用於本機內部部署 VPN 裝置的值相符。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-298">**Shared Key**: This value must match the value that you're using for your local on-premises VPN device.</span></span> <span data-ttu-id="d6f3a-299">教學課程範例會使用 'abc123'，但是您應該使用更為複雜的值。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-299">The tutorial example uses 'abc123', but you should use something more complex.</span></span> <span data-ttu-id="d6f3a-300">重要的是，此值 *必須* 與您在設定 VPN 裝置時指定的值相同。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-300">The important thing is that this value *must* be the same value that you specify when configuring your VPN device.</span></span>
    - <span data-ttu-id="d6f3a-301">[訂用帳戶]  、[資源群組]  和 [位置]  的值是固定的。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-301">The values for **Subscription**, **Resource Group**, and **Location** are fixed.</span></span>

6. <span data-ttu-id="d6f3a-302">選取 [確定]  來建立連線。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-302">Select **OK** to create your connection.</span></span>

<span data-ttu-id="d6f3a-303">您可以在虛擬網路閘道的 [連線]  頁面中看見此連線。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-303">You can see the connection in the **Connections** page of the virtual network gateway.</span></span> <span data-ttu-id="d6f3a-304">狀態將會從 [未知]  變成 [連線中]  ，然後變成 [成功]  。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-304">The status will go from *Unknown* to *Connecting*, and then to *Succeeded*.</span></span>

## <a name="next-steps"></a><span data-ttu-id="d6f3a-305">後續步驟</span><span class="sxs-lookup"><span data-stu-id="d6f3a-305">Next steps</span></span>

- <span data-ttu-id="d6f3a-306">若要深入了解 Azure 雲端模式，請參閱[雲端設計模式](/azure/architecture/patterns)。</span><span class="sxs-lookup"><span data-stu-id="d6f3a-306">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
