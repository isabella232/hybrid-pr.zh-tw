---
title: Azure Stack Hub 中的 DevOps 模式
description: 深入了解 DevOps 模式，以確保 Azure 和 Azure Stack Hub 中的部署之間能保有一致性。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 306cc9604a8e919724f9f76b7e5122d534d2d1ae
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910119"
---
# <a name="devops-pattern"></a><span data-ttu-id="9b10a-103">DevOps 模式</span><span class="sxs-lookup"><span data-stu-id="9b10a-103">DevOps pattern</span></span>

<span data-ttu-id="9b10a-104">從單一位置撰寫程式碼，並部署至開發、測試和生產環境中的多個目標 (這些環境可位於您的本機資料中心、私人雲端或公用雲端中)。</span><span class="sxs-lookup"><span data-stu-id="9b10a-104">Code from a single location and deploy to multiple targets in development, test, and production environments that may be in your local datacenter, private clouds, or the public cloud.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="9b10a-105">內容和問題</span><span class="sxs-lookup"><span data-stu-id="9b10a-105">Context and problem</span></span>

<span data-ttu-id="9b10a-106">應用程式部署的持續性、安全性和可靠性是組織不可或缺的，且對開發小組而言至關重要。</span><span class="sxs-lookup"><span data-stu-id="9b10a-106">Application deployment continuity, security, and reliability are essential to organizations and critical to development teams.</span></span>

<span data-ttu-id="9b10a-107">應用程式通常需要重構的程式碼，才能在每個目標環境中執行。</span><span class="sxs-lookup"><span data-stu-id="9b10a-107">Apps often require refactored code to run in each target environment.</span></span> <span data-ttu-id="9b10a-108">這表示應用程式不一定是可攜的。</span><span class="sxs-lookup"><span data-stu-id="9b10a-108">This means that an app isn't completely portable.</span></span> <span data-ttu-id="9b10a-109">它在每個環境間移動時，必須進行更新、測試和驗證。</span><span class="sxs-lookup"><span data-stu-id="9b10a-109">It must be updated, tested, and validated as it moves through each environment.</span></span> <span data-ttu-id="9b10a-110">例如，在開發環境中撰寫的程式碼必須經過重寫才能在測試環境中運作，且在最後進入生產環境時也必須重寫。</span><span class="sxs-lookup"><span data-stu-id="9b10a-110">For example, code written in a development environment must then be rewritten to work in a test environment and rewritten when it finally lands in a production environment.</span></span> <span data-ttu-id="9b10a-111">此外，此程式碼會按特性繫結至主機。</span><span class="sxs-lookup"><span data-stu-id="9b10a-111">Furthermore, this code is specifically tied to the host.</span></span> <span data-ttu-id="9b10a-112">這可能會提高維護應用程式的成本和複雜度。</span><span class="sxs-lookup"><span data-stu-id="9b10a-112">This increases the cost and complexity of maintaining your app.</span></span> <span data-ttu-id="9b10a-113">應用程式的每個版本會分別繫結至各個環境。</span><span class="sxs-lookup"><span data-stu-id="9b10a-113">Each version of the app is tied to each environment.</span></span> <span data-ttu-id="9b10a-114">提高的複雜度和重複性會使安全性和程式碼品質承受更高的風險。</span><span class="sxs-lookup"><span data-stu-id="9b10a-114">The increased complexity and duplication increase the risk of security and code quality.</span></span> <span data-ttu-id="9b10a-115">此外，當您移除還原失敗的主機或部署其他主機以處理增加的需求時，程式碼無法直接重新部署。</span><span class="sxs-lookup"><span data-stu-id="9b10a-115">In addition, the code can't be readily redeployed when you remove restore failed hosts or deploy additional hosts to handle increases in demand.</span></span>

## <a name="solution"></a><span data-ttu-id="9b10a-116">解決方法</span><span class="sxs-lookup"><span data-stu-id="9b10a-116">Solution</span></span>

<span data-ttu-id="9b10a-117">DevOps 模式可讓您建置、測試和部署在多個雲端上執行的應用程式。</span><span class="sxs-lookup"><span data-stu-id="9b10a-117">The DevOps Pattern enables you to build, test, and deploy an app that runs on multiple clouds.</span></span> <span data-ttu-id="9b10a-118">此模式結合了持續整合和持續傳遞的實務。</span><span class="sxs-lookup"><span data-stu-id="9b10a-118">This pattern unites the practice of continuous integration and continuous delivery.</span></span> <span data-ttu-id="9b10a-119">透過持續整合，在每次小組成員對版本控制認可變更時，都會建置並測試程式碼。</span><span class="sxs-lookup"><span data-stu-id="9b10a-119">With continuous integration, code is built and tested every time a team member commits a change to version control.</span></span> <span data-ttu-id="9b10a-120">持續傳遞會讓組建中的每個步驟在生產環境中自動執行。</span><span class="sxs-lookup"><span data-stu-id="9b10a-120">Continuous delivery automates each step from a build to a production environment.</span></span> <span data-ttu-id="9b10a-121">這些程序會共同建立一個發行流程，支援跨不同環境的部署。</span><span class="sxs-lookup"><span data-stu-id="9b10a-121">Together, these processes create a release process that supports deployment across diverse environments.</span></span> <span data-ttu-id="9b10a-122">透過此模式，您可以草擬程式碼，然後將相同的程式碼部署至內部部署環境、不同的私人雲端和公用雲端。</span><span class="sxs-lookup"><span data-stu-id="9b10a-122">With this pattern, you can draft your code and then deploy the same code to a premise environment, different private clouds, and the public clouds.</span></span> <span data-ttu-id="9b10a-123">環境中的差異需要以組態檔的變更來調整，而不是變更程式碼。</span><span class="sxs-lookup"><span data-stu-id="9b10a-123">Differences in environment require a change to a configuration file rather than changes to the code.</span></span>

![DevOps 模式](media/pattern-cicd-pipeline/hybrid-ci-cd.png)

<span data-ttu-id="9b10a-125">透過在內部部署、私人雲端和公用雲端環境間皆一致的一組開發工具，您可以實作持續整合和持續傳遞的實務。</span><span class="sxs-lookup"><span data-stu-id="9b10a-125">With a consistent set of development tools across on-premises, private cloud, and public cloud environments, you can implement a practice of continuous integration and continuous delivery.</span></span> <span data-ttu-id="9b10a-126">使用 DevOps 模式部署的應用程式和服務具有互通性，且可在其中任一位置執行，而充分運用內部部署和公用雲端的特性與功能。</span><span class="sxs-lookup"><span data-stu-id="9b10a-126">Apps and services deployed using the DevOps Pattern are interchangeable and can run in any of these locations, taking advantage of on-premises and public cloud features and capabilities.</span></span>

<span data-ttu-id="9b10a-127">使用 DevOps 發行管線可協助您：</span><span class="sxs-lookup"><span data-stu-id="9b10a-127">Using a DevOps release pipeline helps you:</span></span>

- <span data-ttu-id="9b10a-128">根據對單一存放庫的程式碼認可起始新的組建。</span><span class="sxs-lookup"><span data-stu-id="9b10a-128">Initiate a new build based on code commits to a single repository.</span></span>
- <span data-ttu-id="9b10a-129">將您新建置的程式碼自動部署至公用雲端以進行使用者驗收測試。</span><span class="sxs-lookup"><span data-stu-id="9b10a-129">Automatically deploy your newly built code to the public cloud for user acceptance testing.</span></span>
- <span data-ttu-id="9b10a-130">在您的程式碼通過測試後自動部署至私人雲端。</span><span class="sxs-lookup"><span data-stu-id="9b10a-130">Automatically deploy to a private cloud once your code has passed testing.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="9b10a-131">問題和考量</span><span class="sxs-lookup"><span data-stu-id="9b10a-131">Issues and considerations</span></span>

<span data-ttu-id="9b10a-132">DevOps 模式的目的是要確保不論目標環境為何，都能有跨部署的一致性。</span><span class="sxs-lookup"><span data-stu-id="9b10a-132">The DevOps Pattern is intended to ensure consistency across deployments regardless of the target environment.</span></span> <span data-ttu-id="9b10a-133">不過，雲端和內部部署環境中的功能會有所不同。</span><span class="sxs-lookup"><span data-stu-id="9b10a-133">However, capabilities vary across cloud and on-premises environments.</span></span> <span data-ttu-id="9b10a-134">請考慮下列幾點：</span><span class="sxs-lookup"><span data-stu-id="9b10a-134">Consider the following points:</span></span>

- <span data-ttu-id="9b10a-135">在目標部署位置中是否可以使用您部署中的函式、端點、服務和其他資源？</span><span class="sxs-lookup"><span data-stu-id="9b10a-135">Are the functions, endpoints, services, and other resources in your deployment available in the target deployment locations?</span></span>
- <span data-ttu-id="9b10a-136">組態成品是否儲存於可跨雲端存取的位置？</span><span class="sxs-lookup"><span data-stu-id="9b10a-136">Are configuration artifacts stored in locations that are accessible across clouds?</span></span>
- <span data-ttu-id="9b10a-137">部署參數是否將適用於所有目標環境中？</span><span class="sxs-lookup"><span data-stu-id="9b10a-137">Will deployment parameters work in all the target environments?</span></span>
- <span data-ttu-id="9b10a-138">是否在所有目標雲端中都可以使用資源特有的屬性？</span><span class="sxs-lookup"><span data-stu-id="9b10a-138">Are resource-specific properties available in all target clouds?</span></span>

<span data-ttu-id="9b10a-139">如需詳細資訊，請參閱[針對雲端一致性開發 Azure Resource Manager 範本](https://docs.microsoft.com/azure/azure-resource-manager/templates-cloud-consistency)。</span><span class="sxs-lookup"><span data-stu-id="9b10a-139">For more information, see [Develop Azure Resource Manager templates for cloud consistency](https://docs.microsoft.com/azure/azure-resource-manager/templates-cloud-consistency).</span></span>

<span data-ttu-id="9b10a-140">此外，在決定此模式的實作方式時，請考量以下幾點：</span><span class="sxs-lookup"><span data-stu-id="9b10a-140">In addition, consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="9b10a-141">延展性</span><span class="sxs-lookup"><span data-stu-id="9b10a-141">Scalability</span></span>

<span data-ttu-id="9b10a-142">部署自動化系統是 DevOps 模式中的主要控制點。</span><span class="sxs-lookup"><span data-stu-id="9b10a-142">Deployment automation systems are the key control point in the DevOps Patterns.</span></span> <span data-ttu-id="9b10a-143">實作方式可能有所不同。</span><span class="sxs-lookup"><span data-stu-id="9b10a-143">Implementations can vary.</span></span> <span data-ttu-id="9b10a-144">伺服器大小的正確選擇，取決於預期的工作負載大小。</span><span class="sxs-lookup"><span data-stu-id="9b10a-144">The selection of the correct server size depends on the size of the expected workload.</span></span> <span data-ttu-id="9b10a-145">VM 的調整成本高於容器。</span><span class="sxs-lookup"><span data-stu-id="9b10a-145">VMs cost more to scale than containers.</span></span> <span data-ttu-id="9b10a-146">若要將容器用於調整大小，您的組建流程必須與容器一起搭配執行。</span><span class="sxs-lookup"><span data-stu-id="9b10a-146">To use containers for scaling, however, your build process must run with containers.</span></span>

### <a name="availability"></a><span data-ttu-id="9b10a-147">可用性</span><span class="sxs-lookup"><span data-stu-id="9b10a-147">Availability</span></span>

<span data-ttu-id="9b10a-148">DevPattern 內容中的可用性是指能夠復原與您的工作流程相關聯的任何狀態資訊，例如測試結果、程式碼相依性，或其他成品。</span><span class="sxs-lookup"><span data-stu-id="9b10a-148">Availability in the context of the DevPattern means being able to recover any state information associated with your workflow, such as test results, code dependencies, or other artifacts.</span></span> <span data-ttu-id="9b10a-149">若要評估您的可用性需求，請考量以下兩個計量：</span><span class="sxs-lookup"><span data-stu-id="9b10a-149">To assess your availability requirements, consider two common metrics:</span></span>

- <span data-ttu-id="9b10a-150">復原時間目標 (RTO) 會指定您在沒有系統的情況能運作多久的時間。</span><span class="sxs-lookup"><span data-stu-id="9b10a-150">Recovery Time Objective (RTO) specifies how long you can go without a system.</span></span>

- <span data-ttu-id="9b10a-151">復原點目標 (RPO) 是指當進行中的中斷影響道系統時，您可以承受多少資料的遺失。</span><span class="sxs-lookup"><span data-stu-id="9b10a-151">Recovery Point Objective (RPO) indicates how much data you can afford to lose if a disruption in service affects the system.</span></span>

<span data-ttu-id="9b10a-152">在實務上，RTO 和 RPO 代表備援性和備份。</span><span class="sxs-lookup"><span data-stu-id="9b10a-152">In practice, RTO, and RPO imply redundancy and backup.</span></span> <span data-ttu-id="9b10a-153">在全域 Azure 雲端上，可用性不是硬體復原的問題 (那是 Azure 的問題)，而是要確保您可以維護 DevOps 系統的狀態。</span><span class="sxs-lookup"><span data-stu-id="9b10a-153">On the global Azure cloud, availability isn't a question of hardware recovery—that's part of Azure—but rather ensuring you maintain the state of your DevOps systems.</span></span> <span data-ttu-id="9b10a-154">在 Azure Stack Hub 上，硬體復原可能是考量之一。</span><span class="sxs-lookup"><span data-stu-id="9b10a-154">On Azure Stack Hub, hardware recovery may be a consideration.</span></span>

<span data-ttu-id="9b10a-155">設計用於部署自動化的系統時，另一項主要考量是將服務部署至雲端環境所需的存取控制和適當的權限管理。</span><span class="sxs-lookup"><span data-stu-id="9b10a-155">Another major consideration when designing the system used for deployment automation is access control and the proper management of the rights needed to deploy services to cloud environments.</span></span> <span data-ttu-id="9b10a-156">建立、刪除或修改部署時需要哪些權限？</span><span class="sxs-lookup"><span data-stu-id="9b10a-156">What rights are needed to create, delete, or modify deployments?</span></span> <span data-ttu-id="9b10a-157">例如，在 Azure 中建立資源群組時通常需要一組權限，而在資源群組中部署服務時則需要另一組權限。</span><span class="sxs-lookup"><span data-stu-id="9b10a-157">For example, one set of rights is typically required to create a resource group in Azure and another to deploy services in the resource group.</span></span>

### <a name="manageability"></a><span data-ttu-id="9b10a-158">管理能力</span><span class="sxs-lookup"><span data-stu-id="9b10a-158">Manageability</span></span>

<span data-ttu-id="9b10a-159">以 DevOps 模式為基礎設計任何系統時，都必須考量整個組合中各項服務的自動化、記錄及警示。</span><span class="sxs-lookup"><span data-stu-id="9b10a-159">The design of any system based on the DevOps pattern must consider automation, logging, and alerting for each service across the portfolio.</span></span> <span data-ttu-id="9b10a-160">請使用共用服務和 (或) 應用程式小組，並追蹤安全性原則和治理情形。</span><span class="sxs-lookup"><span data-stu-id="9b10a-160">Use shared services, an application team, or both, and track security policies and governance as well.</span></span>

<span data-ttu-id="9b10a-161">請在 Azure 或 Azure Stack Hub 上的個別資源群組中部署生產環境和開發/測試環境。</span><span class="sxs-lookup"><span data-stu-id="9b10a-161">Deploy production environments and development/test environments in separate resource groups on Azure or Azure Stack Hub.</span></span> <span data-ttu-id="9b10a-162">然後，您可以監視每個環境的資源，並依資源群組彙總計費成本。</span><span class="sxs-lookup"><span data-stu-id="9b10a-162">Then you can monitor each environment's resources and roll up billing costs by resource group.</span></span> <span data-ttu-id="9b10a-163">您也可以刪除整組資源，這對於測試部署很有用。</span><span class="sxs-lookup"><span data-stu-id="9b10a-163">You can also delete resources as a set, which is useful for test deployments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="9b10a-164">使用此模式的時機</span><span class="sxs-lookup"><span data-stu-id="9b10a-164">When to use this pattern</span></span>

<span data-ttu-id="9b10a-165">使用此模式的時機：</span><span class="sxs-lookup"><span data-stu-id="9b10a-165">Use this pattern if:</span></span>

- <span data-ttu-id="9b10a-166">您可以在一個符合開發人員需求的環境中開發程式碼，並部署到您的解決方案專用、但可能難以開發新程式碼的環境中。</span><span class="sxs-lookup"><span data-stu-id="9b10a-166">You can develop code in one environment that meets the needs of your developers, and deploy to an environment specific to your solution where it may be difficult to develop new code.</span></span>
- <span data-ttu-id="9b10a-167">您可以使用開發人員想要的程式碼和工具，只要他們能夠遵循 DevOps 模式中的持續整合和持續傳遞程序即可。</span><span class="sxs-lookup"><span data-stu-id="9b10a-167">You can use the code and tools your developers would like, as long as they're able to follow the continuous integration and continuous delivery process in the DevOps Pattern.</span></span>

<span data-ttu-id="9b10a-168">不建議使用此模式：</span><span class="sxs-lookup"><span data-stu-id="9b10a-168">This pattern isn't recommended:</span></span>

- <span data-ttu-id="9b10a-169">如果您無法自動執行基礎結構、佈建資源、組態、身分識別和安全性工作。</span><span class="sxs-lookup"><span data-stu-id="9b10a-169">If you can't automate infrastructure, provisioning resources, configuration, identity, and security tasks.</span></span>
- <span data-ttu-id="9b10a-170">如果小組沒有相關權限可存取混合式雲端資源，以實作持續整合/持續開發 (CI/CD) 方法。</span><span class="sxs-lookup"><span data-stu-id="9b10a-170">If teams don't have access to hybrid cloud resources to implement a Continuous Integration/Continuous Development (CI/CD) approach.</span></span>

## <a name="next-steps"></a><span data-ttu-id="9b10a-171">後續步驟</span><span class="sxs-lookup"><span data-stu-id="9b10a-171">Next steps</span></span>

<span data-ttu-id="9b10a-172">深入了解此文章介紹的更多相關主題：</span><span class="sxs-lookup"><span data-stu-id="9b10a-172">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="9b10a-173">請參閱 [Azure DevOps 文件](/azure/devops)以深入了解 Azure DevOps 與相關工具，包括 Azure Repos 與 Azure Pipelines。</span><span class="sxs-lookup"><span data-stu-id="9b10a-173">See the [Azure DevOps documentation](/azure/devops) to learn more about Azure DevOps and related tools, including Azure Repos, and Azure Pipelines.</span></span>
- <span data-ttu-id="9b10a-174">若要深入了解產品與解決方案的完整組合，請參閱 [Azure Stack 系列產品和解決方案](/azure-stack)。</span><span class="sxs-lookup"><span data-stu-id="9b10a-174">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="9b10a-175">當您準備好測試解決方案範例時，請繼續參閱 [DevOps 混合式 CI/CD 解決方案部署指南](https://aka.ms/hybriddevopsdeploy)。</span><span class="sxs-lookup"><span data-stu-id="9b10a-175">When you're ready to test the solution example, continue with the [DevOps hybrid CI/CD solution deployment guide](https://aka.ms/hybriddevopsdeploy).</span></span> <span data-ttu-id="9b10a-176">部署指南提供部署及測試其元件的逐步指示。</span><span class="sxs-lookup"><span data-stu-id="9b10a-176">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="9b10a-177">您將會了解如何使用混合式持續整合/持續傳遞 (CI/CD) 管線，將應用程式部署至 Azure 與 Azure Stack Hub。</span><span class="sxs-lookup"><span data-stu-id="9b10a-177">You learn how to deploy an app to Azure and Azure Stack Hub using a hybrid continuous integration/continuous delivery (CI/CD) pipeline.</span></span>
