---
title: Azure Stack Hub 中的異地分散式應用程式模式
description: 了解如何使用 Azure 與 Azure Stack Hub 的智慧邊緣所適用的異地分散式應用程式模式。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 1f6243927390c7a520c2607c722664b2d31fc07f
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910051"
---
# <a name="geo-distributed-app-pattern"></a><span data-ttu-id="1e2f1-103">異地分散式應用程式模式</span><span class="sxs-lookup"><span data-stu-id="1e2f1-103">Geo-distributed app pattern</span></span>

<span data-ttu-id="1e2f1-104">了解如何跨多個區域提供應用程式端點，並根據位置與合規性需求來路由傳送使用者流量。</span><span class="sxs-lookup"><span data-stu-id="1e2f1-104">Learn how to provide app endpoints across multiple regions and route user traffic based on location and compliance needs.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="1e2f1-105">內容和問題</span><span class="sxs-lookup"><span data-stu-id="1e2f1-105">Context and problem</span></span>

<span data-ttu-id="1e2f1-106">遍及廣大地理位置的組織都積極設法要安全且正確地散佈資料，並使其可供存取，同時確保各區域的每個使用者、位置和裝置的安全性、合規性和效能都能達到必要的水準。</span><span class="sxs-lookup"><span data-stu-id="1e2f1-106">Organizations with wide-reaching geographies strive to securely and accurately distribute and enable access to data while ensuring required levels of security, compliance and performance per user, location, and device across borders.</span></span>

## <a name="solution"></a><span data-ttu-id="1e2f1-107">解決方法</span><span class="sxs-lookup"><span data-stu-id="1e2f1-107">Solution</span></span>

<span data-ttu-id="1e2f1-108">Azure Stack Hub 地理流量路由傳送模式 (或異地分散式應用程式) 會讓流量根據各種計量導向至特定端點。</span><span class="sxs-lookup"><span data-stu-id="1e2f1-108">The Azure Stack Hub geographic traffic routing pattern, or geo-distributed apps, lets traffic be directed to specific endpoints based on various metrics.</span></span> <span data-ttu-id="1e2f1-109">使用以地理位置為基礎的路由和端點組態來建立流量管理員，可根據區域需求、公司與國際法規和資料需求將流量路由至端點。</span><span class="sxs-lookup"><span data-stu-id="1e2f1-109">Creating a Traffic Manager with geographic-based routing and endpoint configuration routes traffic to endpoints based on regional requirements, corporate and international regulation, and data needs.</span></span>

![地理位置分散模式](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a><span data-ttu-id="1e2f1-111">元件</span><span class="sxs-lookup"><span data-stu-id="1e2f1-111">Components</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="1e2f1-112">雲端外部</span><span class="sxs-lookup"><span data-stu-id="1e2f1-112">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="1e2f1-113">流量管理員</span><span class="sxs-lookup"><span data-stu-id="1e2f1-113">Traffic Manager</span></span>

<span data-ttu-id="1e2f1-114">在此圖中，流量管理員位於公用雲端以外，但必須能夠同時協調本機資料中心與公用雲端中的流量。</span><span class="sxs-lookup"><span data-stu-id="1e2f1-114">In the diagram, Traffic Manager is located outside of the public cloud, but it needs to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="1e2f1-115">平衡器會將流量路由到地理位置。</span><span class="sxs-lookup"><span data-stu-id="1e2f1-115">The balancer routes traffic to geographical locations.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="1e2f1-116">網域名稱系統 (DNS)</span><span class="sxs-lookup"><span data-stu-id="1e2f1-116">Domain Name System (DNS)</span></span>

<span data-ttu-id="1e2f1-117">網域名稱系統 (DNS) 負責將網站或服務名稱轉譯 (或解析) 為其 IP 位址。</span><span class="sxs-lookup"><span data-stu-id="1e2f1-117">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="public-cloud"></a><span data-ttu-id="1e2f1-118">公用雲端</span><span class="sxs-lookup"><span data-stu-id="1e2f1-118">Public cloud</span></span>

#### <a name="cloud-endpoint"></a><span data-ttu-id="1e2f1-119">雲端端點</span><span class="sxs-lookup"><span data-stu-id="1e2f1-119">Cloud Endpoint</span></span>

<span data-ttu-id="1e2f1-120">公用 IP 位址可透過流量管理員用來將傳入流量路由到公用雲端應用程式資源端點。</span><span class="sxs-lookup"><span data-stu-id="1e2f1-120">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-clouds"></a><span data-ttu-id="1e2f1-121">本機雲端</span><span class="sxs-lookup"><span data-stu-id="1e2f1-121">Local clouds</span></span>

#### <a name="local-endpoint"></a><span data-ttu-id="1e2f1-122">本機端點</span><span class="sxs-lookup"><span data-stu-id="1e2f1-122">Local endpoint</span></span>

<span data-ttu-id="1e2f1-123">公用 IP 位址可透過流量管理員用來將傳入流量路由到公用雲端應用程式資源端點。</span><span class="sxs-lookup"><span data-stu-id="1e2f1-123">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="1e2f1-124">問題和考量</span><span class="sxs-lookup"><span data-stu-id="1e2f1-124">Issues and considerations</span></span>

<span data-ttu-id="1e2f1-125">當您決定如何實作此模式時，請考慮下列幾點：</span><span class="sxs-lookup"><span data-stu-id="1e2f1-125">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="1e2f1-126">延展性</span><span class="sxs-lookup"><span data-stu-id="1e2f1-126">Scalability</span></span>

<span data-ttu-id="1e2f1-127">此模式會處理地理流量路由，而不是根據流量增加的情況進行調整。</span><span class="sxs-lookup"><span data-stu-id="1e2f1-127">The pattern handles geographical traffic routing rather than scaling to meet increases in traffic.</span></span> <span data-ttu-id="1e2f1-128">不過，您可以將此模式與其他 Azure 和內部部署解決方案結合。</span><span class="sxs-lookup"><span data-stu-id="1e2f1-128">However, you can combine this pattern with other Azure and on-premises solutions.</span></span> <span data-ttu-id="1e2f1-129">例如，此模式可以與跨雲端調整模式搭配使用。</span><span class="sxs-lookup"><span data-stu-id="1e2f1-129">For example, this pattern can be used with the cross-cloud scaling Pattern.</span></span>

### <a name="availability"></a><span data-ttu-id="1e2f1-130">可用性</span><span class="sxs-lookup"><span data-stu-id="1e2f1-130">Availability</span></span>

<span data-ttu-id="1e2f1-131">確定已透過內部部署硬體設定和軟體部署，針對本機部署應用程式的高可用性進行設定。</span><span class="sxs-lookup"><span data-stu-id="1e2f1-131">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="1e2f1-132">管理能力</span><span class="sxs-lookup"><span data-stu-id="1e2f1-132">Manageability</span></span>

<span data-ttu-id="1e2f1-133">此模式可確保在不同環境間都能有順暢的管理和熟悉的介面。</span><span class="sxs-lookup"><span data-stu-id="1e2f1-133">The pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="1e2f1-134">使用此模式的時機</span><span class="sxs-lookup"><span data-stu-id="1e2f1-134">When to use this pattern</span></span>

- <span data-ttu-id="1e2f1-135">我的組織有海外分公司，且需要自訂的區域安全性和發佈原則。</span><span class="sxs-lookup"><span data-stu-id="1e2f1-135">My organization has international branches requiring custom regional security and distribution policies.</span></span>
- <span data-ttu-id="1e2f1-136">我的每個組織辦公室皆提取員工、商務和設備資料，而需要各自的當地法規和時區的報告活動。</span><span class="sxs-lookup"><span data-stu-id="1e2f1-136">Each of my organization's offices pulls employee, business, and facility data, requiring reporting activity per local regulations and time zone.</span></span>
- <span data-ttu-id="1e2f1-137">只要對單一區域內和跨區域的多個應用程式部署進行應用程式的水平相應放大，即可達到高延展性需求，以處理極高的負載需求。</span><span class="sxs-lookup"><span data-stu-id="1e2f1-137">High-scale requirements can be met by horizontally scaling out apps, with multiple app deployments being made within a single region and across regions to handle extreme load requirements.</span></span>
- <span data-ttu-id="1e2f1-138">應用程式必須具有高度可用性，且即使在單一區域中斷時也能回應用戶端要求。</span><span class="sxs-lookup"><span data-stu-id="1e2f1-138">The apps must be highly available and responsive to client requests even in single-region outages.</span></span>

## <a name="next-steps"></a><span data-ttu-id="1e2f1-139">後續步驟</span><span class="sxs-lookup"><span data-stu-id="1e2f1-139">Next steps</span></span>

<span data-ttu-id="1e2f1-140">深入了解此文章介紹的更多相關主題：</span><span class="sxs-lookup"><span data-stu-id="1e2f1-140">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="1e2f1-141">若要深入了解此 DNS 型流量負載平衡器的運作方式，請參閱 [Azure 流量管理員概觀](/azure/traffic-manager/traffic-manager-overview)。</span><span class="sxs-lookup"><span data-stu-id="1e2f1-141">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="1e2f1-142">若要深入了解最佳做法並獲得其他問題的解答，請參閱[混合式應用程式設計考量](overview-app-design-considerations.md)。</span><span class="sxs-lookup"><span data-stu-id="1e2f1-142">See [Hybrid app design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="1e2f1-143">若要深入了解產品與解決方案的完整組合，請參閱 [Azure Stack 系列產品和解決方案](/azure-stack)。</span><span class="sxs-lookup"><span data-stu-id="1e2f1-143">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="1e2f1-144">當您準備好測試解決方案範例時，請繼續參閱[異地分散式應用程式解決方案部署指南](solution-deployment-guide-geo-distributed.md)。</span><span class="sxs-lookup"><span data-stu-id="1e2f1-144">When you're ready to test the solution example, continue with the [Geo-distributed app solution deployment guide](solution-deployment-guide-geo-distributed.md).</span></span> <span data-ttu-id="1e2f1-145">部署指南提供部署及測試其元件的逐步指示。</span><span class="sxs-lookup"><span data-stu-id="1e2f1-145">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="1e2f1-146">您將會了解如何根據各種計量，使用異地分散式應用程式模式將流量導向至特定端點。</span><span class="sxs-lookup"><span data-stu-id="1e2f1-146">You learn how to direct traffic to specific endpoints, based on various metrics using the geo-distributed app pattern.</span></span> <span data-ttu-id="1e2f1-147">使用以地理位置為基礎的路由和端點組態來建立流量管理員設定檔，可確保資訊會根據區域需求、公司與國際法規和您的資料需求路由至端點。</span><span class="sxs-lookup"><span data-stu-id="1e2f1-147">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>
