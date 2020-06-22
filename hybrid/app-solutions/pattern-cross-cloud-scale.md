---
title: Azure Stack Hub 中的跨雲端調整模式
description: 了解如何在 Azure 和 Azure Stack Hub 上建立可跨雲端調整的應用程式。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: a830f96e97c347cbbcc09a1b17f4836ecb6eb3e6
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910123"
---
# <a name="cross-cloud-scaling-pattern"></a><span data-ttu-id="29b44-103">跨雲端調整模式</span><span class="sxs-lookup"><span data-stu-id="29b44-103">Cross-cloud scaling pattern</span></span>

<span data-ttu-id="29b44-104">自動將資源新增至現有的應用程式，以因應增加的負載。</span><span class="sxs-lookup"><span data-stu-id="29b44-104">Automatically add resources to an existing app to accommodate an increase in load.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="29b44-105">內容和問題</span><span class="sxs-lookup"><span data-stu-id="29b44-105">Context and problem</span></span>

<span data-ttu-id="29b44-106">您的應用程式無法增加容量，以因應非預期增加的需求。</span><span class="sxs-lookup"><span data-stu-id="29b44-106">Your app can't increase capacity to meet unexpected increases in demand.</span></span> <span data-ttu-id="29b44-107">缺少延展性會導致使用者在尖峰使用時間無法連線到應用程式。</span><span class="sxs-lookup"><span data-stu-id="29b44-107">This lack of scalability results in users not reaching the app during peak usage times.</span></span> <span data-ttu-id="29b44-108">應用程式可為固定數目的使用者提供服務。</span><span class="sxs-lookup"><span data-stu-id="29b44-108">The app can service a fixed number of users.</span></span>

<span data-ttu-id="29b44-109">國際企業需要安全、可靠且可用的雲端式應用程式。</span><span class="sxs-lookup"><span data-stu-id="29b44-109">Global enterprises require secure, reliable, and available cloud-based apps.</span></span> <span data-ttu-id="29b44-110">因應增加的需求，並使用正確的基礎結構來支援這項需求，是非常重要的。</span><span class="sxs-lookup"><span data-stu-id="29b44-110">Meeting increases in demand and using the right infrastructure to support that demand is critical.</span></span> <span data-ttu-id="29b44-111">企業為了在成本和維護，以及商務資料安全性、儲存和即時可用性之間取得平衡，都費盡了心力。</span><span class="sxs-lookup"><span data-stu-id="29b44-111">Businesses struggle to balance costs and maintenance with business data security, storage, and real-time availability.</span></span>

<span data-ttu-id="29b44-112">您可能無法在公用雲端中執行應用程式。</span><span class="sxs-lookup"><span data-stu-id="29b44-112">You may not be able to run your app in the public cloud.</span></span> <span data-ttu-id="29b44-113">不過，若讓企業在內部部署環境中維持用來處理應用程式需求高峰的容量，似乎不切實際。</span><span class="sxs-lookup"><span data-stu-id="29b44-113">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="29b44-114">透過此模式，您將可透過內部部署解決方案使用公用雲端的靈活性。</span><span class="sxs-lookup"><span data-stu-id="29b44-114">With this pattern, you can use the elasticity of the public cloud with your on-premises solution.</span></span>

## <a name="solution"></a><span data-ttu-id="29b44-115">解決方法</span><span class="sxs-lookup"><span data-stu-id="29b44-115">Solution</span></span>

<span data-ttu-id="29b44-116">跨雲端調整模式會使用公用雲端資源擴充位於本機雲端的應用程式。</span><span class="sxs-lookup"><span data-stu-id="29b44-116">The cross-cloud scaling pattern extends an app located in a local cloud with public cloud resources.</span></span> <span data-ttu-id="29b44-117">此模式會在需求增加或減少時觸發，並分別新增或移除雲端中的資源。</span><span class="sxs-lookup"><span data-stu-id="29b44-117">The pattern is triggered by an increase or decrease in demand, and respectively adds or removes resources in the cloud.</span></span> <span data-ttu-id="29b44-118">這些資源提供了備援性、快速的可用性，和異地相容的路由。</span><span class="sxs-lookup"><span data-stu-id="29b44-118">These resources provide redundancy, rapid availability, and geo-compliant routing.</span></span>

![跨雲端調整模式](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> <span data-ttu-id="29b44-120">此模式僅適用於應用程式的無狀態元件。</span><span class="sxs-lookup"><span data-stu-id="29b44-120">This pattern applies only to stateless components of your app.</span></span>

## <a name="components"></a><span data-ttu-id="29b44-121">元件</span><span class="sxs-lookup"><span data-stu-id="29b44-121">Components</span></span>

<span data-ttu-id="29b44-122">跨雲端調整模式由下列元件組成。</span><span class="sxs-lookup"><span data-stu-id="29b44-122">The cross-cloud scaling pattern consists of the following components.</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="29b44-123">雲端外部</span><span class="sxs-lookup"><span data-stu-id="29b44-123">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="29b44-124">流量管理員</span><span class="sxs-lookup"><span data-stu-id="29b44-124">Traffic Manager</span></span>

<span data-ttu-id="29b44-125">在此圖中，其位於公用雲端群組以外，但必須能夠同時協調本機資料中心和公用雲端中的流量。</span><span class="sxs-lookup"><span data-stu-id="29b44-125">In the diagram, this is located outside of the public cloud group, but it would need to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="29b44-126">平衡器會監視端點，並在必要時提供容錯移轉轉散發，藉以提供應用程式的高可用性。</span><span class="sxs-lookup"><span data-stu-id="29b44-126">The balancer delivers high availability for app by monitoring endpoints and providing failover redistribution when required.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="29b44-127">網域名稱系統 (DNS)</span><span class="sxs-lookup"><span data-stu-id="29b44-127">Domain Name System (DNS)</span></span>

<span data-ttu-id="29b44-128">網域名稱系統 (DNS) 負責將網站或服務名稱轉譯 (或解析) 為其 IP 位址。</span><span class="sxs-lookup"><span data-stu-id="29b44-128">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="cloud"></a><span data-ttu-id="29b44-129">Cloud</span><span class="sxs-lookup"><span data-stu-id="29b44-129">Cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="29b44-130">裝載的組建伺服器</span><span class="sxs-lookup"><span data-stu-id="29b44-130">Hosted build server</span></span>

<span data-ttu-id="29b44-131">裝載組建管線的環境。</span><span class="sxs-lookup"><span data-stu-id="29b44-131">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="29b44-132">應用程式資源</span><span class="sxs-lookup"><span data-stu-id="29b44-132">App resources</span></span>

<span data-ttu-id="29b44-133">應用程式資源必須能夠縮減和擴增，例如虛擬機器擴展集和容器。</span><span class="sxs-lookup"><span data-stu-id="29b44-133">The app resources need to be able to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="29b44-134">自訂網域名稱</span><span class="sxs-lookup"><span data-stu-id="29b44-134">Custom domain name</span></span>

<span data-ttu-id="29b44-135">將自訂變數名稱用於路由要求 GLOB。</span><span class="sxs-lookup"><span data-stu-id="29b44-135">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="29b44-136">公用 IP 位址</span><span class="sxs-lookup"><span data-stu-id="29b44-136">Public IP addresses</span></span>

<span data-ttu-id="29b44-137">公用 IP 位址可透過流量管理員用來將傳入流量路由到公用雲端應用程式資源端點。</span><span class="sxs-lookup"><span data-stu-id="29b44-137">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-cloud"></a><span data-ttu-id="29b44-138">本機雲端</span><span class="sxs-lookup"><span data-stu-id="29b44-138">Local cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="29b44-139">裝載的組建伺服器</span><span class="sxs-lookup"><span data-stu-id="29b44-139">Hosted build server</span></span>

<span data-ttu-id="29b44-140">裝載組建管線的環境。</span><span class="sxs-lookup"><span data-stu-id="29b44-140">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="29b44-141">應用程式資源</span><span class="sxs-lookup"><span data-stu-id="29b44-141">App resources</span></span>

<span data-ttu-id="29b44-142">應用程式資源必須能夠縮減和擴增，例如虛擬機器擴展集和容器。</span><span class="sxs-lookup"><span data-stu-id="29b44-142">The app resources need the ability to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="29b44-143">自訂網域名稱</span><span class="sxs-lookup"><span data-stu-id="29b44-143">Custom domain name</span></span>

<span data-ttu-id="29b44-144">將自訂變數名稱用於路由要求 GLOB。</span><span class="sxs-lookup"><span data-stu-id="29b44-144">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="29b44-145">公用 IP 位址</span><span class="sxs-lookup"><span data-stu-id="29b44-145">Public IP addresses</span></span>

<span data-ttu-id="29b44-146">公用 IP 位址可透過流量管理員用來將傳入流量路由到公用雲端應用程式資源端點。</span><span class="sxs-lookup"><span data-stu-id="29b44-146">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="29b44-147">問題和考量</span><span class="sxs-lookup"><span data-stu-id="29b44-147">Issues and considerations</span></span>

<span data-ttu-id="29b44-148">當您決定如何實作此模式時，請考慮下列幾點：</span><span class="sxs-lookup"><span data-stu-id="29b44-148">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="29b44-149">延展性</span><span class="sxs-lookup"><span data-stu-id="29b44-149">Scalability</span></span>

<span data-ttu-id="29b44-150">跨雲端調整的關鍵元件是提供隨選調整的能力。</span><span class="sxs-lookup"><span data-stu-id="29b44-150">The key component of cross-cloud scaling is the ability to deliver on-demand scaling.</span></span> <span data-ttu-id="29b44-151">必須在公用和本機雲端基礎結構之間進行調整，並根據需求提供一致且可靠的服務。</span><span class="sxs-lookup"><span data-stu-id="29b44-151">Scaling must happen between public and local cloud infrastructure and provide a consistent, reliable service per the demand.</span></span>

### <a name="availability"></a><span data-ttu-id="29b44-152">可用性</span><span class="sxs-lookup"><span data-stu-id="29b44-152">Availability</span></span>

<span data-ttu-id="29b44-153">確定已透過內部部署硬體設定和軟體部署，針對本機部署應用程式的高可用性進行設定。</span><span class="sxs-lookup"><span data-stu-id="29b44-153">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="29b44-154">管理能力</span><span class="sxs-lookup"><span data-stu-id="29b44-154">Manageability</span></span>

<span data-ttu-id="29b44-155">跨雲端模式可確保在不同環境間都能有順暢的管理和熟悉的介面。</span><span class="sxs-lookup"><span data-stu-id="29b44-155">The cross-cloud pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="29b44-156">使用此模式的時機</span><span class="sxs-lookup"><span data-stu-id="29b44-156">When to use this pattern</span></span>

<span data-ttu-id="29b44-157">使用此模式：</span><span class="sxs-lookup"><span data-stu-id="29b44-157">Use this pattern:</span></span>

- <span data-ttu-id="29b44-158">當您需要增加應用程式容量以因應非預期的需求或定期需求時。</span><span class="sxs-lookup"><span data-stu-id="29b44-158">When you need to increase your app capacity with unexpected demands or periodic demands in demand.</span></span>
- <span data-ttu-id="29b44-159">當您不想投資於只有尖峰期間才會用到的資源時。</span><span class="sxs-lookup"><span data-stu-id="29b44-159">When you don't want to invest in resources that will only be used during peaks.</span></span> <span data-ttu-id="29b44-160">您用多少就付多少。</span><span class="sxs-lookup"><span data-stu-id="29b44-160">Pay for what you use.</span></span>

<span data-ttu-id="29b44-161">在下列情況下不建議使用此模式：</span><span class="sxs-lookup"><span data-stu-id="29b44-161">This pattern isn't recommended when:</span></span>

- <span data-ttu-id="29b44-162">使用者在使用您的解決方案時需要透過網際網路連線。</span><span class="sxs-lookup"><span data-stu-id="29b44-162">Your solution requires users connecting over the internet.</span></span>
- <span data-ttu-id="29b44-163">您的企業有當地的法規，要求起始連線必須來自現場電話。</span><span class="sxs-lookup"><span data-stu-id="29b44-163">Your business has local regulations that require that the originating connection to come from an onsite call.</span></span>
- <span data-ttu-id="29b44-164">您的網路出現將使調整效能受限的一般瓶頸。</span><span class="sxs-lookup"><span data-stu-id="29b44-164">Your network experiences regular bottlenecks that would restrict the performance of the scaling.</span></span>
- <span data-ttu-id="29b44-165">您的環境已中斷網際網路連線，因此無法連接到公用雲端。</span><span class="sxs-lookup"><span data-stu-id="29b44-165">Your environment is disconnected from the internet and can't reach the public cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="29b44-166">後續步驟</span><span class="sxs-lookup"><span data-stu-id="29b44-166">Next steps</span></span>

<span data-ttu-id="29b44-167">深入了解此文章介紹的更多相關主題：</span><span class="sxs-lookup"><span data-stu-id="29b44-167">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="29b44-168">若要深入了解此 DNS 型流量負載平衡器的運作方式，請參閱 [Azure 流量管理員概觀](/azure/traffic-manager/traffic-manager-overview)。</span><span class="sxs-lookup"><span data-stu-id="29b44-168">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="29b44-169">若要深入了解最佳做法並獲得任何其他問題的解答，請參閱[混合式應用程式設計考量](overview-app-design-considerations.md)。</span><span class="sxs-lookup"><span data-stu-id="29b44-169">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="29b44-170">若要深入了解產品與解決方案的完整組合，請參閱 [Azure Stack 系列產品和解決方案](/azure-stack)。</span><span class="sxs-lookup"><span data-stu-id="29b44-170">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="29b44-171">當您準備好測試解決方案範例時，請繼續參閱[跨雲端調整解決方案部署指南](solution-deployment-guide-cross-cloud-scaling.md)。</span><span class="sxs-lookup"><span data-stu-id="29b44-171">When you're ready to test the solution example, continue with the [Cross-cloud scaling solution deployment guide](solution-deployment-guide-cross-cloud-scaling.md).</span></span> <span data-ttu-id="29b44-172">部署指南提供部署及測試其元件的逐步指示。</span><span class="sxs-lookup"><span data-stu-id="29b44-172">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="29b44-173">了解如何建立可提供手動觸發程序的跨雲端解決方案，從 Azure Stack Hub 託管的 Web 應用程式切換到 Azure 託管的 Web 應用程式。</span><span class="sxs-lookup"><span data-stu-id="29b44-173">You learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app.</span></span> <span data-ttu-id="29b44-174">您也將了解如何透過流量管理員使用自動調整功能，以確保在負載下時有彈性且可調整的雲端公用程式。</span><span class="sxs-lookup"><span data-stu-id="29b44-174">You also learn how to use autoscaling via traffic manager, ensuring flexible and scalable cloud utility when under load.</span></span>
