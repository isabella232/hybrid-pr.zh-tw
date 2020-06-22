---
title: 以邊緣模式定型機器學習模型
description: 了解如何使用 Azure 和 Azure Stack Hub，在邊緣進行機器學習模型定型。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: da1012cc8847f221de6bb540ba4e191e43920f41
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910114"
---
# <a name="train-machine-learning-model-at-the-edge-pattern"></a><span data-ttu-id="cf883-103">以邊緣模式定型機器學習模型</span><span class="sxs-lookup"><span data-stu-id="cf883-103">Train machine learning model at the edge pattern</span></span>

<span data-ttu-id="cf883-104">從僅存在於內部部署的資料產生可攜式機器學習 (ML) 模型。</span><span class="sxs-lookup"><span data-stu-id="cf883-104">Generate portable machine learning (ML) models from data that only exists on-premises.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="cf883-105">內容和問題</span><span class="sxs-lookup"><span data-stu-id="cf883-105">Context and problem</span></span>

<span data-ttu-id="cf883-106">許多組織希望使用其資料科學家了解的工具，從其內部部署或舊版資料中獲取見解。</span><span class="sxs-lookup"><span data-stu-id="cf883-106">Many organizations would like to unlock insights from their on-premises or legacy data using tools that their data scientists understand.</span></span> <span data-ttu-id="cf883-107">[Azure Machine Learning](/azure/machine-learning/) 提供的雲端原生工具可定型、調整及部署 ML 和深度學習模型。</span><span class="sxs-lookup"><span data-stu-id="cf883-107">[Azure Machine Learning](/azure/machine-learning/) provides cloud-native tooling to train, tune, and deploy ML and deep learning models.</span></span>  

<span data-ttu-id="cf883-108">不過，某些資料太大，無法傳送至雲端，或礙於法規因素而無法傳送至雲端。</span><span class="sxs-lookup"><span data-stu-id="cf883-108">However, some data is too large send to the cloud or can't be sent to the cloud for regulatory reasons.</span></span> <span data-ttu-id="cf883-109">使用此模式時，資料科學家可以使用 Azure Machine Learning，利用內部部署資料和計算來定型模型。</span><span class="sxs-lookup"><span data-stu-id="cf883-109">Using this pattern, data scientists can use Azure Machine Learning to train models using on-premises data and compute.</span></span>

## <a name="solution"></a><span data-ttu-id="cf883-110">解決方法</span><span class="sxs-lookup"><span data-stu-id="cf883-110">Solution</span></span>

<span data-ttu-id="cf883-111">邊緣模式的訓練會使用在 Azure Stack Hub 上執行的虛擬機器 (VM)。</span><span class="sxs-lookup"><span data-stu-id="cf883-111">The training at the edge pattern uses a virtual machine (VM) running on Azure Stack Hub.</span></span> <span data-ttu-id="cf883-112">VM 已在 Azure ML 中註冊為計算目標，使其能存取僅適用於內部部署的資料。</span><span class="sxs-lookup"><span data-stu-id="cf883-112">The VM is registered as a compute target in Azure ML, letting it access data only available on-premises.</span></span> <span data-ttu-id="cf883-113">在此情況下，資料會儲存在 Azure Stack Hub 的 Blob 儲存體中。</span><span class="sxs-lookup"><span data-stu-id="cf883-113">In this case, the data is stored in Azure Stack Hub's blob storage.</span></span>

<span data-ttu-id="cf883-114">模型定型之後，便會向 Azure ML 註冊、對其進行容器化，並將其新增至 Azure Container Registry 以進行部署。</span><span class="sxs-lookup"><span data-stu-id="cf883-114">Once the model is trained, it's registered with Azure ML, containerized, and added to an Azure Container Registry for deployment.</span></span> <span data-ttu-id="cf883-115">若要進行此模式的反覆項目，必須可透過公用網際網路連線至 Azure Stack Hub 訓練 VM。</span><span class="sxs-lookup"><span data-stu-id="cf883-115">For this iteration of the pattern, the Azure Stack Hub training VM must be reachable over the public internet.</span></span>

<span data-ttu-id="cf883-116">[![在邊緣架構定型 ML 模型](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span><span class="sxs-lookup"><span data-stu-id="cf883-116">[![Train ML model at the edge architecture](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span></span>

<span data-ttu-id="cf883-117">此模式的運作方式如下：</span><span class="sxs-lookup"><span data-stu-id="cf883-117">Here's how the pattern works:</span></span>

1. <span data-ttu-id="cf883-118">已部署 Azure Stack Hub VM，並透過 Azure ML 註冊為計算目標。</span><span class="sxs-lookup"><span data-stu-id="cf883-118">The Azure Stack Hub VM is deployed and registered as a compute target with Azure ML.</span></span>
2. <span data-ttu-id="cf883-119">在 Azure ML 中建立了一個實驗，該實驗使用 Azure Stack Hub VM 作為計算目標。</span><span class="sxs-lookup"><span data-stu-id="cf883-119">An experiment is created in Azure ML that uses the Azure Stack Hub VM as a compute target.</span></span>
3. <span data-ttu-id="cf883-120">模型定型之後，會進行註冊並容器化。</span><span class="sxs-lookup"><span data-stu-id="cf883-120">Once the model is trained, it's registered and containerized.</span></span>
4. <span data-ttu-id="cf883-121">現在可以將模型部署到內部部署或雲端中的位置。</span><span class="sxs-lookup"><span data-stu-id="cf883-121">The model can now be deployed to locations that are either on-premises or in the cloud.</span></span>

## <a name="components"></a><span data-ttu-id="cf883-122">元件</span><span class="sxs-lookup"><span data-stu-id="cf883-122">Components</span></span>

<span data-ttu-id="cf883-123">此解決方案使用下列元件：</span><span class="sxs-lookup"><span data-stu-id="cf883-123">This solution uses the following components:</span></span>

| <span data-ttu-id="cf883-124">階層</span><span class="sxs-lookup"><span data-stu-id="cf883-124">Layer</span></span> | <span data-ttu-id="cf883-125">元件</span><span class="sxs-lookup"><span data-stu-id="cf883-125">Component</span></span> | <span data-ttu-id="cf883-126">描述</span><span class="sxs-lookup"><span data-stu-id="cf883-126">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="cf883-127">Azure</span><span class="sxs-lookup"><span data-stu-id="cf883-127">Azure</span></span> | <span data-ttu-id="cf883-128">Azure Machine Learning</span><span class="sxs-lookup"><span data-stu-id="cf883-128">Azure Machine Learning</span></span> | <span data-ttu-id="cf883-129">[Azure Machine Learning](/azure/machine-learning/) 可協調 ML 模型的訓練。</span><span class="sxs-lookup"><span data-stu-id="cf883-129">[Azure Machine Learning](/azure/machine-learning/) orchestrates the training of the ML model.</span></span> |
| | <span data-ttu-id="cf883-130">Azure Container Registry</span><span class="sxs-lookup"><span data-stu-id="cf883-130">Azure Container Registry</span></span> | <span data-ttu-id="cf883-131">Azure ML 會將模型封裝到容器中，並將其儲存在用於部署的 [Azure Container Registry](/azure/container-registry/) 中。</span><span class="sxs-lookup"><span data-stu-id="cf883-131">Azure ML packages the model into a container and stores it in an [Azure Container Registry](/azure/container-registry/) for deployment.</span></span>|
| <span data-ttu-id="cf883-132">Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="cf883-132">Azure Stack Hub</span></span> | <span data-ttu-id="cf883-133">App Service 方案</span><span class="sxs-lookup"><span data-stu-id="cf883-133">App Service</span></span> | <span data-ttu-id="cf883-134">[具有 App Service 的 Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) 可為邊緣的元件提供基礎映像。</span><span class="sxs-lookup"><span data-stu-id="cf883-134">[Azure Stack Hub with App Service](/azure-stack/operator/azure-stack-app-service-overview) provides the base for the components at the edge.</span></span> |
| | <span data-ttu-id="cf883-135">計算</span><span class="sxs-lookup"><span data-stu-id="cf883-135">Compute</span></span> | <span data-ttu-id="cf883-136">使用 Docker 執行 Ubuntu 的 Azure Stack Hub VM 可用於定型 ML 模型。</span><span class="sxs-lookup"><span data-stu-id="cf883-136">An Azure Stack Hub VM running Ubuntu with Docker is used to train the ML model.</span></span> |
| | <span data-ttu-id="cf883-137">儲存體</span><span class="sxs-lookup"><span data-stu-id="cf883-137">Storage</span></span> | <span data-ttu-id="cf883-138">私人資料可以裝載在 Azure Stack Hub Blob 儲存體中。</span><span class="sxs-lookup"><span data-stu-id="cf883-138">Private data can be hosted in Azure Stack Hub blob storage.</span></span> |

## <a name="issues-and-considerations"></a><span data-ttu-id="cf883-139">問題和考量</span><span class="sxs-lookup"><span data-stu-id="cf883-139">Issues and considerations</span></span>

<span data-ttu-id="cf883-140">決定如何實作此解決方案時，請考慮下列幾點：</span><span class="sxs-lookup"><span data-stu-id="cf883-140">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="cf883-141">延展性</span><span class="sxs-lookup"><span data-stu-id="cf883-141">Scalability</span></span>

<span data-ttu-id="cf883-142">為了讓這項解決方案得以擴展，您必須在 Azure Stack Hub 上建立適當大小的 VM 以進行訓練。</span><span class="sxs-lookup"><span data-stu-id="cf883-142">To enable this solution to scale, you'll need to create an appropriately sized VM on Azure Stack Hub for training.</span></span>

### <a name="availability"></a><span data-ttu-id="cf883-143">可用性</span><span class="sxs-lookup"><span data-stu-id="cf883-143">Availability</span></span>

<span data-ttu-id="cf883-144">請確定訓練指令碼和 Azure Stack Hub VM 可以存取用於訓練的內部部署資料。</span><span class="sxs-lookup"><span data-stu-id="cf883-144">Ensure that the training scripts and Azure Stack Hub VM have access to the on-premises data used for training.</span></span>

### <a name="manageability"></a><span data-ttu-id="cf883-145">管理能力</span><span class="sxs-lookup"><span data-stu-id="cf883-145">Manageability</span></span>

<span data-ttu-id="cf883-146">請確定模型和實驗已進行適當的註冊、建立版本，並加上標籤，以避免在模型部署期間造成混淆。</span><span class="sxs-lookup"><span data-stu-id="cf883-146">Ensure that models and experiments are appropriately registered, versioned, and tagged to avoid confusion during model deployment.</span></span>

### <a name="security"></a><span data-ttu-id="cf883-147">安全性</span><span class="sxs-lookup"><span data-stu-id="cf883-147">Security</span></span>

<span data-ttu-id="cf883-148">此模式可讓 Azure ML 存取可能的內部部署敏感性資料。</span><span class="sxs-lookup"><span data-stu-id="cf883-148">This pattern lets Azure ML access possible sensitive data on-premises.</span></span> <span data-ttu-id="cf883-149">請確定用來透過 SSH 連線至 Azure Stack Hub VM 的帳戶具有強式密碼，且訓練指令碼不會保留資料或將資料上傳到雲端。</span><span class="sxs-lookup"><span data-stu-id="cf883-149">Ensure the account used to SSH into Azure Stack Hub VM has a strong password and training scripts don't preserve or upload data to the cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="cf883-150">後續步驟</span><span class="sxs-lookup"><span data-stu-id="cf883-150">Next steps</span></span>

<span data-ttu-id="cf883-151">深入了解此文章介紹的更多相關主題：</span><span class="sxs-lookup"><span data-stu-id="cf883-151">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="cf883-152">如需 ML 和相關主題的概觀，請參閱 [Azure Machine Learning 說明文件](/azure/machine-learning)。</span><span class="sxs-lookup"><span data-stu-id="cf883-152">See the [Azure Machine Learning documentation](/azure/machine-learning) for an overview of ML and related topics.</span></span>
- <span data-ttu-id="cf883-153">若要了解如何組建、儲存和管理容器部署的映像，請參閱 [Azure Container Registry](/azure/container-registry/)。</span><span class="sxs-lookup"><span data-stu-id="cf883-153">See [Azure Container Registry](/azure/container-registry/) to learn how to build, store, and manage images for container deployments.</span></span>
- <span data-ttu-id="cf883-154">若要深入了解資源提供者和如何部署的詳細資訊，請參閱 [Azure Stack Hub 上的 App Service](/azure-stack/operator/azure-stack-app-service-overview)。</span><span class="sxs-lookup"><span data-stu-id="cf883-154">Refer to [App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) to learn more about the resource provider and how to deploy.</span></span>
- <span data-ttu-id="cf883-155">若要深入了解最佳做法並獲得任何其他問題的解答，請參閱[混合式應用程式設計考量](overview-app-design-considerations.md)。</span><span class="sxs-lookup"><span data-stu-id="cf883-155">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get any additional questions answered.</span></span>
- <span data-ttu-id="cf883-156">若要深入了解產品與解決方案的完整組合，請參閱 [Azure Stack 系列產品和解決方案](/azure-stack)。</span><span class="sxs-lookup"><span data-stu-id="cf883-156">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="cf883-157">當您準備好測試解決方案範例時，請繼續參閱[在邊緣定型 ML 模型部署指南](https://aka.ms/edgetrainingdeploy)。</span><span class="sxs-lookup"><span data-stu-id="cf883-157">When you're ready to test the solution example, continue with the [Train ML model at the edge deployment guide](https://aka.ms/edgetrainingdeploy).</span></span> <span data-ttu-id="cf883-158">部署指南提供部署及測試其元件的逐步指示。</span><span class="sxs-lookup"><span data-stu-id="cf883-158">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>
