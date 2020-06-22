---
title: 適用於 Azure 和 Azure Stack Hub 的混合式模式和解決方案範例
description: 概述混合式模式和解決方案範例，用來在 Azure 和 Azure Stack Hub 上學習及建置混合式解決方案。
author: BryanLa
ms.topic: overview
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ab0eb885e7b0fefaca8991522712652f979d8712
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910062"
---
# <a name="hybrid-patterns-and-solution-examples-for-azure-and-azure-stack"></a><span data-ttu-id="98484-103">適用於 Azure 和 Azure Stack 的混合式模式和解決方案範例</span><span class="sxs-lookup"><span data-stu-id="98484-103">Hybrid patterns and solution examples for Azure and Azure Stack</span></span>

<span data-ttu-id="98484-104">Microsoft 會以單一且一致的 Azure 生態系統形式來提供 Azure 和 Azure Stack 產品與解決方案。</span><span class="sxs-lookup"><span data-stu-id="98484-104">Microsoft provides Azure and Azure Stack products and solutions as one consistent Azure ecosystem.</span></span> <span data-ttu-id="98484-105">Microsoft Azure Stack 系列是 Azure 的擴充功能。</span><span class="sxs-lookup"><span data-stu-id="98484-105">The Microsoft Azure Stack family is an extension of Azure.</span></span>

## <a name="the-hybrid-cloud-and-hybrid-apps"></a><span data-ttu-id="98484-106">混合式雲端和混合式應用程式</span><span class="sxs-lookup"><span data-stu-id="98484-106">The hybrid cloud and hybrid apps</span></span>

<span data-ttu-id="98484-107">Azure Stack 藉由實現「混合式雲端」  ，將雲端運算的靈活度帶到內部部署環境和邊緣。</span><span class="sxs-lookup"><span data-stu-id="98484-107">Azure Stack brings the agility of cloud computing to your on-premises environment and the edge by enabling a *hybrid cloud*.</span></span> <span data-ttu-id="98484-108">Azure Stack Hub、Azure Stack HCI 和 Azure Stack Edge 會將 Azure 從雲端延伸至您的主權資料中心、分公司、現場和更遠的範圍。</span><span class="sxs-lookup"><span data-stu-id="98484-108">Azure Stack Hub, Azure Stack HCI, and Azure Stack Edge extend Azure from the cloud into your sovereign datacenters, branch offices, field, and beyond.</span></span> <span data-ttu-id="98484-109">使用這組多樣化的功能，您將可以：</span><span class="sxs-lookup"><span data-stu-id="98484-109">With this diverse set of capabilities, you can:</span></span>

- <span data-ttu-id="98484-110">重複使用程式碼並在 Azure 與內部部署環境中一致地執行雲端原生應用程式。</span><span class="sxs-lookup"><span data-stu-id="98484-110">Reuse code and run cloud-native apps consistently across Azure and your on-premises environments.</span></span>
- <span data-ttu-id="98484-111">使用對 Azure 服務的選擇性連線來執行傳統的虛擬化工作負載。</span><span class="sxs-lookup"><span data-stu-id="98484-111">Run traditional virtualized workloads with optional connections to Azure services.</span></span>
- <span data-ttu-id="98484-112">將資料傳輸至雲端，或將保留在主權資料中心以維持合規性。</span><span class="sxs-lookup"><span data-stu-id="98484-112">Transfer data to the cloud, or keep it in your sovereign datacenter to maintain compliance.</span></span>
- <span data-ttu-id="98484-113">執行硬體加速機器學習、容器化或虛擬化的工作負載，全都在智慧邊緣進行。</span><span class="sxs-lookup"><span data-stu-id="98484-113">Run hardware-accelerated machine-learning, containerized, or virtualized workloads, all at the intelligent edge.</span></span>

<span data-ttu-id="98484-114">跨雲端應用程式也稱為*混合式應用程式*。</span><span class="sxs-lookup"><span data-stu-id="98484-114">Apps that span clouds are also referred to as *hybrid apps*.</span></span> <span data-ttu-id="98484-115">您可以在 Azure 中建置混合式雲端應用程式，並將其部署到任何位置的已連線或中斷連線的資料中心。</span><span class="sxs-lookup"><span data-stu-id="98484-115">You can build hybrid cloud apps in Azure and deploy them to your connected or disconnected datacenter located anywhere.</span></span>

<span data-ttu-id="98484-116">混合式應用程式案例會隨著可供開發的資源而有很大的變化。</span><span class="sxs-lookup"><span data-stu-id="98484-116">Hybrid app scenarios vary greatly with the resources that are available for development.</span></span> <span data-ttu-id="98484-117">這些案例的注意事項也會跨及地理位置、安全性、網際網路存取等方面。</span><span class="sxs-lookup"><span data-stu-id="98484-117">They also span considerations such as geography, security, internet access, and others.</span></span> <span data-ttu-id="98484-118">雖然這裡所說的模式和解決方案可能無法因應所有需求，但卻可以提供指導方針和範例讓您探究並供您在實作混合式解決方案時重複使用。</span><span class="sxs-lookup"><span data-stu-id="98484-118">Although the patterns and solutions described here may not address all requirements, they provide guidelines and examples to explore and reuse while implementing hybrid solutions.</span></span>

## <a name="design-patterns"></a><span data-ttu-id="98484-119">設計模式</span><span class="sxs-lookup"><span data-stu-id="98484-119">Design patterns</span></span>

<span data-ttu-id="98484-120">設計模式會從真實的客戶案例和經驗中，挑選出普遍且可重複的設計指引。</span><span class="sxs-lookup"><span data-stu-id="98484-120">Design patterns cull generalized repeatable design guidance, from real world customer scenarios and experiences.</span></span> <span data-ttu-id="98484-121">所擷取出來的模式可適用於不同類型的案例或垂直產業。</span><span class="sxs-lookup"><span data-stu-id="98484-121">A pattern is abstract, allowing it to be applicable to different types of scenarios or vertical industries.</span></span> <span data-ttu-id="98484-122">每個模式都會記載內容和問題，並提供解決方案範例的概觀。</span><span class="sxs-lookup"><span data-stu-id="98484-122">Each pattern documents the context and problem, and provides an overview of a solution example.</span></span> <span data-ttu-id="98484-123">解決方案範例的本意是要作為模式的可能實作。</span><span class="sxs-lookup"><span data-stu-id="98484-123">The solution example is meant as a possible implementation of the pattern.</span></span>

<span data-ttu-id="98484-124">模式文章有兩種類型：</span><span class="sxs-lookup"><span data-stu-id="98484-124">There are two types of pattern articles:</span></span>

- <span data-ttu-id="98484-125">單一模式：提供一般用途單一案例的設計指引。</span><span class="sxs-lookup"><span data-stu-id="98484-125">Single pattern: provides design guidance for a single general-purpose scenario.</span></span>
- <span data-ttu-id="98484-126">多重模式：提供應用時會使用多個模式的設計指引。</span><span class="sxs-lookup"><span data-stu-id="98484-126">Multi-pattern: provides design guidance where the application of multiple patterns is used.</span></span> <span data-ttu-id="98484-127">為了解決更複雜的案例或產業特有問題，經常會需要使用這種模式。</span><span class="sxs-lookup"><span data-stu-id="98484-127">This pattern is frequently required for solving more complex scenarios or industry-specific problems.</span></span>

## <a name="solution-deployment-guides"></a><span data-ttu-id="98484-128">解決方案部署指南</span><span class="sxs-lookup"><span data-stu-id="98484-128">Solution deployment guides</span></span>

<span data-ttu-id="98484-129">逐步部署指南可協助您部署解決方案範例。</span><span class="sxs-lookup"><span data-stu-id="98484-129">Step-by-step deployment guides assist in deploying a solution example.</span></span> <span data-ttu-id="98484-130">此指南可能也會參考隨附的程式碼範例，期儲存位置在 GitHub 的[解決方案範例存放庫](https://github.com/Azure-Samples/azure-intelligent-edge-patterns)中。</span><span class="sxs-lookup"><span data-stu-id="98484-130">The guide may also refer to a companion code sample, stored in the GitHub [solutions sample repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>

## <a name="next-steps"></a><span data-ttu-id="98484-131">後續步驟</span><span class="sxs-lookup"><span data-stu-id="98484-131">Next steps</span></span>

- <span data-ttu-id="98484-132">若要深入了解產品與解決方案的完整組合，請參閱 [Azure Stack 系列產品和解決方案](/azure-stack)。</span><span class="sxs-lookup"><span data-stu-id="98484-132">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>
- <span data-ttu-id="98484-133">探索目錄中的＜模式＞和＜解決方案部署指南＞章節，以分別深入了解。</span><span class="sxs-lookup"><span data-stu-id="98484-133">Explore the "Patterns" and the "Solution deployment guides" sections of the TOC to learn more about each.</span></span>
- <span data-ttu-id="98484-134">閱讀[混合式應用程式設計考量](overview-app-design-considerations.md)，以檢閱設計、部署及操作混合式應用程式的軟體品質要素。</span><span class="sxs-lookup"><span data-stu-id="98484-134">Read about [Hybrid app design considerations](overview-app-design-considerations.md) to review pillars of software quality for designing, deploying, and operating hybrid apps.</span></span>
- <span data-ttu-id="98484-135">[在 Azure Stack 上設定開發環境](/azure-stack/user/azure-stack-dev-start.md)並在 Azure Stack 上[部署第一個應用程式](/azure-stack/user/azure-stack-dev-start-deploy-app.md)。</span><span class="sxs-lookup"><span data-stu-id="98484-135">[Set up a development environment on Azure Stack](/azure-stack/user/azure-stack-dev-start.md) and [deploy your first app](/azure-stack/user/azure-stack-dev-start-deploy-app.md) on Azure Stack.</span></span>
