---
title: 適用于 Azure 和 Azure Stack Hub 的混合式模式和解決方案範例
description: 介紹在 Azure 上學習和建立混合式解決方案的混合式模式和解決方案範例，並 Azure Stack Hub。
author: BryanLa
ms.topic: overview
ms.date: 05/24/2021
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 05/24/2021
ms.openlocfilehash: 9f3f13c23bec31c5132c7e90294356b9463fd72b
ms.sourcegitcommit: cf2c4033d1b169f5b63980ce1865281366905e2e
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/25/2021
ms.locfileid: "110343853"
---
# <a name="hybrid-solution-patterns-and-examples-for-azure-and-azure-stack"></a><span data-ttu-id="3695e-103">適用于 Azure 和 Azure Stack 的混合式解決方案模式和範例</span><span class="sxs-lookup"><span data-stu-id="3695e-103">Hybrid solution patterns and examples for Azure and Azure Stack</span></span>

<span data-ttu-id="3695e-104">Microsoft 會以單一且一致的 Azure 生態系統形式來提供 Azure 和 Azure Stack 產品與解決方案。</span><span class="sxs-lookup"><span data-stu-id="3695e-104">Microsoft provides Azure and Azure Stack products and solutions as one consistent Azure ecosystem.</span></span> <span data-ttu-id="3695e-105">Microsoft Azure Stack 系列是 Azure 的擴充功能。</span><span class="sxs-lookup"><span data-stu-id="3695e-105">The Microsoft Azure Stack family is an extension of Azure.</span></span>

## <a name="the-hybrid-cloud-and-hybrid-apps"></a><span data-ttu-id="3695e-106">混合式雲端和混合式應用程式</span><span class="sxs-lookup"><span data-stu-id="3695e-106">The hybrid cloud and hybrid apps</span></span>

<span data-ttu-id="3695e-107">Azure Stack 藉由啟用 *混合式雲端*，將雲端運算的靈活性帶入您的內部部署環境和邊緣。</span><span class="sxs-lookup"><span data-stu-id="3695e-107">Azure Stack brings the agility of cloud computing to your on-premises environment and the edge by enabling a *hybrid cloud*.</span></span> <span data-ttu-id="3695e-108">Azure Stack Hub、Azure Stack HCI 和 Azure Stack Edge 會將 Azure 從雲端延伸至您的主權資料中心、分公司、現場和更遠的範圍。</span><span class="sxs-lookup"><span data-stu-id="3695e-108">Azure Stack Hub, Azure Stack HCI, and Azure Stack Edge extend Azure from the cloud into your sovereign datacenters, branch offices, field, and beyond.</span></span> <span data-ttu-id="3695e-109">使用這組多樣化的功能，您將可以：</span><span class="sxs-lookup"><span data-stu-id="3695e-109">With this diverse set of capabilities, you can:</span></span>

- <span data-ttu-id="3695e-110">重複使用程式碼並在 Azure 與內部部署環境中一致地執行雲端原生應用程式。</span><span class="sxs-lookup"><span data-stu-id="3695e-110">Reuse code and run cloud-native apps consistently across Azure and your on-premises environments.</span></span>
- <span data-ttu-id="3695e-111">使用對 Azure 服務的選擇性連線來執行傳統的虛擬化工作負載。</span><span class="sxs-lookup"><span data-stu-id="3695e-111">Run traditional virtualized workloads with optional connections to Azure services.</span></span>
- <span data-ttu-id="3695e-112">將資料傳輸至雲端，或將保留在主權資料中心以維持合規性。</span><span class="sxs-lookup"><span data-stu-id="3695e-112">Transfer data to the cloud, or keep it in your sovereign datacenter to maintain compliance.</span></span>
- <span data-ttu-id="3695e-113">執行硬體加速機器學習、容器化或虛擬化的工作負載，全都在智慧邊緣進行。</span><span class="sxs-lookup"><span data-stu-id="3695e-113">Run hardware-accelerated machine-learning, containerized, or virtualized workloads, all at the intelligent edge.</span></span>

<span data-ttu-id="3695e-114">跨雲端的應用程式也稱為混合式 *應用程式*。</span><span class="sxs-lookup"><span data-stu-id="3695e-114">Apps that span clouds are also referred to as *hybrid apps*.</span></span> <span data-ttu-id="3695e-115">您可以在 Azure 中建置混合式雲端應用程式，並將其部署到任何位置的已連線或中斷連線的資料中心。</span><span class="sxs-lookup"><span data-stu-id="3695e-115">You can build hybrid cloud apps in Azure and deploy them to your connected or disconnected datacenter located anywhere.</span></span>

<span data-ttu-id="3695e-116">混合式應用程式案例與可供開發的資源有很大的差異。</span><span class="sxs-lookup"><span data-stu-id="3695e-116">Hybrid app scenarios vary greatly with the resources that are available for development.</span></span> <span data-ttu-id="3695e-117">它們也會涵蓋地理位置、安全性、網際網路存取等考慮。</span><span class="sxs-lookup"><span data-stu-id="3695e-117">They also span considerations such as geography, security, internet access, and others.</span></span> <span data-ttu-id="3695e-118">雖然此處所述的解決方案模式和範例可能無法解決所有需求，但它們會提供在執行混合式解決方案時探索和重複使用的指導方針和範例。</span><span class="sxs-lookup"><span data-stu-id="3695e-118">Although the solution patterns and examples described here may not address all requirements, they provide guidelines and examples to explore and reuse while implementing hybrid solutions.</span></span>

## <a name="solution-patterns"></a><span data-ttu-id="3695e-119">解決方案模式</span><span class="sxs-lookup"><span data-stu-id="3695e-119">Solution patterns</span></span>

<span data-ttu-id="3695e-120">解決方案模式可從真實世界的客戶案例和經驗中挑選一般化可重複的設計指引。</span><span class="sxs-lookup"><span data-stu-id="3695e-120">Solution patterns cull generalized repeatable design guidance, from real world customer scenarios and experiences.</span></span> <span data-ttu-id="3695e-121">所擷取出來的模式可適用於不同類型的案例或垂直產業。</span><span class="sxs-lookup"><span data-stu-id="3695e-121">A pattern is abstract, allowing it to be applicable to different types of scenarios or vertical industries.</span></span> <span data-ttu-id="3695e-122">每個模式都會記載內容和問題，並提供解決方案範例的概觀。</span><span class="sxs-lookup"><span data-stu-id="3695e-122">Each pattern documents the context and problem, and provides an overview of a solution example.</span></span> <span data-ttu-id="3695e-123">解決方案範例的本意是要作為模式的可能實作。</span><span class="sxs-lookup"><span data-stu-id="3695e-123">The solution example is meant as a possible implementation of the pattern.</span></span>

<span data-ttu-id="3695e-124">模式文章有兩種類型：</span><span class="sxs-lookup"><span data-stu-id="3695e-124">There are two types of pattern articles:</span></span>

- <span data-ttu-id="3695e-125">單一模式：提供一般用途單一案例的設計指引。</span><span class="sxs-lookup"><span data-stu-id="3695e-125">Single pattern: provides design guidance for a single general-purpose scenario.</span></span>
- <span data-ttu-id="3695e-126">多重模式：提供應用時會使用多個模式的設計指引。</span><span class="sxs-lookup"><span data-stu-id="3695e-126">Multi-pattern: provides design guidance where the application of multiple patterns is used.</span></span> <span data-ttu-id="3695e-127">解決更複雜的案例或產業特定的問題時，通常需要此模式。</span><span class="sxs-lookup"><span data-stu-id="3695e-127">This pattern is frequently required for solving more complex scenarios or industry-specific problems.</span></span>

## <a name="solution-deployment-guides"></a><span data-ttu-id="3695e-128">解決方案部署指南</span><span class="sxs-lookup"><span data-stu-id="3695e-128">Solution deployment guides</span></span>

<span data-ttu-id="3695e-129">逐步部署指南可協助您部署解決方案範例。</span><span class="sxs-lookup"><span data-stu-id="3695e-129">Step-by-step deployment guides assist in deploying a solution example.</span></span> <span data-ttu-id="3695e-130">此指南可能也會參考隨附的程式碼範例，期儲存位置在 GitHub 的[解決方案範例存放庫](https://github.com/Azure-Samples/azure-intelligent-edge-patterns)中。</span><span class="sxs-lookup"><span data-stu-id="3695e-130">The guide may also refer to a companion code sample, stored in the GitHub [solutions sample repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>

## <a name="next-steps"></a><span data-ttu-id="3695e-131">下一步</span><span class="sxs-lookup"><span data-stu-id="3695e-131">Next steps</span></span>

- <span data-ttu-id="3695e-132">若要深入了解產品與解決方案的完整組合，請參閱 [Azure Stack 系列產品和解決方案](/azure-stack)。</span><span class="sxs-lookup"><span data-stu-id="3695e-132">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>
- <span data-ttu-id="3695e-133">探索目錄的「模式」和「解決方案部署指南」章節，以深入瞭解各項。</span><span class="sxs-lookup"><span data-stu-id="3695e-133">Explore the "Patterns" and the "Solution deployment guides" sections of the TOC to learn more about each.</span></span>
- <span data-ttu-id="3695e-134">閱讀 [混合式應用程式設計考慮](overview-app-design-considerations.md) ，以複習設計、部署及操作混合式應用程式的軟體品質要素。</span><span class="sxs-lookup"><span data-stu-id="3695e-134">Read about [Hybrid app design considerations](overview-app-design-considerations.md) to review pillars of software quality for designing, deploying, and operating hybrid apps.</span></span>
- <span data-ttu-id="3695e-135">[在 Azure Stack 上設定開發環境](/azure-stack/user/azure-stack-dev-start)並在 Azure Stack 上[部署第一個應用程式](/azure-stack/user/azure-stack-dev-start-deploy-app)。</span><span class="sxs-lookup"><span data-stu-id="3695e-135">[Set up a development environment on Azure Stack](/azure-stack/user/azure-stack-dev-start) and [deploy your first app](/azure-stack/user/azure-stack-dev-start-deploy-app) on Azure Stack.</span></span>
