---
title: 使用 Azure 和 Azure Stack Hub 的客流量偵測模式
description: 瞭解如何使用 Azure 和 Azure Stack Hub 來實行以 AI 為基礎的客流量偵測解決方案，以便分析零售商店的流量。
author: BryanLa
ms.topic: article
ms.date: 10/31/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 10/31/2019
ms.openlocfilehash: 866557ec3af2337e9f034da84cf417675508563b
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895324"
---
# <a name="footfall-detection-pattern"></a><span data-ttu-id="5f120-103">客流量偵測模式</span><span class="sxs-lookup"><span data-stu-id="5f120-103">Footfall detection pattern</span></span>

<span data-ttu-id="5f120-104">此模式概述如何實行以 AI 為基礎的客流量偵測解決方案，以便在零售商店中分析訪客的流量。</span><span class="sxs-lookup"><span data-stu-id="5f120-104">This pattern provides an overview for implementing an AI-based footfall detection solution for analyzing visitor traffic in retail stores.</span></span> <span data-ttu-id="5f120-105">該解決方案會使用 Azure、Azure Stack Hub 和自訂視覺 AI 開發套件，從真實世界的動作產生深入解析。</span><span class="sxs-lookup"><span data-stu-id="5f120-105">The solution generates insights from real world actions, using Azure, Azure Stack Hub, and the Custom Vision AI Dev Kit.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="5f120-106">內容和問題</span><span class="sxs-lookup"><span data-stu-id="5f120-106">Context and problem</span></span>

<span data-ttu-id="5f120-107">Contoso 商店想要深入瞭解客戶如何接收其目前的產品（相對於儲存版面配置）。</span><span class="sxs-lookup"><span data-stu-id="5f120-107">Contoso Stores would like to gain insights on how customers are receiving their current products in relation to store layout.</span></span> <span data-ttu-id="5f120-108">他們無法在每一節中放置員工，而且讓分析師的團隊審視整個商店的攝影機素材沒有效率。</span><span class="sxs-lookup"><span data-stu-id="5f120-108">They're unable to place staff in every section and it's inefficient to have a team of analysts review an entire store's camera footage.</span></span> <span data-ttu-id="5f120-109">此外，其每一家店都沒有足夠頻寬可將影片從所有攝影機串流到雲端進行分析。</span><span class="sxs-lookup"><span data-stu-id="5f120-109">In addition, none of their stores have enough bandwidth to stream video from all their cameras to the cloud for analysis.</span></span>

<span data-ttu-id="5f120-110">Contoso 想要以不顯眼且不侵犯隱私的方式來判斷其客戶的人口統計資訊、忠誠度以及對於商店的陳列和產品有何反應。</span><span class="sxs-lookup"><span data-stu-id="5f120-110">Contoso would like to find an unobtrusive, privacy-friendly way to determine their customers' demographics, loyalty, and reactions to store displays and products.</span></span>

## <a name="solution"></a><span data-ttu-id="5f120-111">解決方法</span><span class="sxs-lookup"><span data-stu-id="5f120-111">Solution</span></span>

<span data-ttu-id="5f120-112">此零售分析模式使用階層式方法在邊緣進行推斷。</span><span class="sxs-lookup"><span data-stu-id="5f120-112">This retail analytics pattern uses a tiered approach to inferencing at the edge.</span></span> <span data-ttu-id="5f120-113">藉由使用自訂視覺 AI 開發套件，系統只會將有人臉的影像傳送至執行 Azure 認知服務的私人 Azure Stack Hub 進行分析。</span><span class="sxs-lookup"><span data-stu-id="5f120-113">By using the Custom Vision AI Dev Kit, only images with human faces are sent for analysis to a private Azure Stack Hub that runs Azure Cognitive Services.</span></span> <span data-ttu-id="5f120-114">系統會將匿名的彙總資料傳送至 Azure，以便彙總所有商店的資料，並在 Power BI 中以視覺方式呈現。</span><span class="sxs-lookup"><span data-stu-id="5f120-114">Anonymized, aggregated data is sent to Azure for aggregation across all stores and visualization in Power BI.</span></span> <span data-ttu-id="5f120-115">結合 edge 和公用雲端可讓 Contoso 利用現代化的 AI 技術，同時也保有與其公司原則的合規性，並尊重客戶的隱私權。</span><span class="sxs-lookup"><span data-stu-id="5f120-115">Combining the edge and public cloud lets Contoso take advantage of modern AI technology while also remaining in compliance with their corporate policies and respecting their customers' privacy.</span></span>

<span data-ttu-id="5f120-116">[![客流量偵測模式解決方案](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)</span><span class="sxs-lookup"><span data-stu-id="5f120-116">[![Footfall detection pattern solution](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)</span></span>

<span data-ttu-id="5f120-117">以下摘要說明該解決方案的運作方式：</span><span class="sxs-lookup"><span data-stu-id="5f120-117">Here's a summary of how the solution works:</span></span>

1. <span data-ttu-id="5f120-118">自訂視覺 AI 開發套件會從 IoT 中樞取得設定，以安裝 IoT Edge 執行階段和 ML 模型。</span><span class="sxs-lookup"><span data-stu-id="5f120-118">The Custom Vision AI Dev Kit gets a configuration from IoT Hub, which installs the IoT Edge Runtime and an ML model.</span></span>
2. <span data-ttu-id="5f120-119">如果該模型發現有人，便會拍照並上傳至 Azure Stack Hub Blob 儲存體。</span><span class="sxs-lookup"><span data-stu-id="5f120-119">If the model sees a person, it takes a picture and uploads it to Azure Stack Hub blob storage.</span></span>
3. <span data-ttu-id="5f120-120">Blob 服務會在 Azure Stack Hub 上觸發 Azure Function。</span><span class="sxs-lookup"><span data-stu-id="5f120-120">The blob service triggers an Azure Function on Azure Stack Hub.</span></span>
4. <span data-ttu-id="5f120-121">Azure Function 會呼叫有人臉識別 API 的容器，以從該影像取得人口統計和表情方面的資料。</span><span class="sxs-lookup"><span data-stu-id="5f120-121">The Azure Function calls a container with the Face API to get demographic and emotion data from the image.</span></span>
5. <span data-ttu-id="5f120-122">系統會將匿名資料傳送至 Azure 事件中樞叢集。</span><span class="sxs-lookup"><span data-stu-id="5f120-122">The data is anonymized and sent to an Azure Event Hubs cluster.</span></span>
6. <span data-ttu-id="5f120-123">事件中樞叢集會將資料推送至串流分析。</span><span class="sxs-lookup"><span data-stu-id="5f120-123">The Event Hubs cluster pushes the data to Stream Analytics.</span></span>
7. <span data-ttu-id="5f120-124">串流分析會匯總資料並推送至 Power BI。</span><span class="sxs-lookup"><span data-stu-id="5f120-124">Stream Analytics aggregates the data and pushes it to Power BI.</span></span>

## <a name="components"></a><span data-ttu-id="5f120-125">元件</span><span class="sxs-lookup"><span data-stu-id="5f120-125">Components</span></span>

<span data-ttu-id="5f120-126">此解決方案使用下列元件：</span><span class="sxs-lookup"><span data-stu-id="5f120-126">This solution uses the following components:</span></span>

| <span data-ttu-id="5f120-127">階層</span><span class="sxs-lookup"><span data-stu-id="5f120-127">Layer</span></span> | <span data-ttu-id="5f120-128">元件</span><span class="sxs-lookup"><span data-stu-id="5f120-128">Component</span></span> | <span data-ttu-id="5f120-129">描述</span><span class="sxs-lookup"><span data-stu-id="5f120-129">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="5f120-130">店內硬體</span><span class="sxs-lookup"><span data-stu-id="5f120-130">In-store hardware</span></span> | [<span data-ttu-id="5f120-131">自訂視覺 AI 開發套件</span><span class="sxs-lookup"><span data-stu-id="5f120-131">Custom Vision AI Dev Kit</span></span>](https://azure.github.io/Vision-AI-DevKit-Pages/) | <span data-ttu-id="5f120-132">提供使用本機 ML 模型的店內篩選功能，並只擷取有人的影像來進行分析。</span><span class="sxs-lookup"><span data-stu-id="5f120-132">Provides in-store filtering using a local ML model that only captures images of people for analysis.</span></span> <span data-ttu-id="5f120-133">透過 IoT 中樞安全地進行佈建和更新。</span><span class="sxs-lookup"><span data-stu-id="5f120-133">Securely provisioned and updated through IoT Hub.</span></span><br><br>|
| <span data-ttu-id="5f120-134">Azure</span><span class="sxs-lookup"><span data-stu-id="5f120-134">Azure</span></span> | [<span data-ttu-id="5f120-135">Azure 事件中樞</span><span class="sxs-lookup"><span data-stu-id="5f120-135">Azure Event Hubs</span></span>](/azure/event-hubs/) | <span data-ttu-id="5f120-136">Azure 事件中樞會提供可調整的平台，以便用來擷取可與 Azure 串流分析完美整合的匿名資料。</span><span class="sxs-lookup"><span data-stu-id="5f120-136">Azure Event Hubs provides a scalable platform for ingesting anonymized data that integrates neatly with Azure Stream Analytics.</span></span> |
|  | [<span data-ttu-id="5f120-137">Azure 串流分析</span><span class="sxs-lookup"><span data-stu-id="5f120-137">Azure Stream Analytics</span></span>](/azure/stream-analytics/) | <span data-ttu-id="5f120-138">Azure 串流分析作業會彙總匿名資料，並將其分組為 15 秒長的時段而以視覺方式加以呈現。</span><span class="sxs-lookup"><span data-stu-id="5f120-138">An Azure Stream Analytics job aggregates the anonymized data and groups it into 15-second windows for visualization.</span></span> |
|  | [<span data-ttu-id="5f120-139">Microsoft Power BI</span><span class="sxs-lookup"><span data-stu-id="5f120-139">Microsoft Power BI</span></span>](https://powerbi.microsoft.com/) | <span data-ttu-id="5f120-140">Power BI 會提供便於使用的儀表板介面，以供您檢視來自 Azure 串流分析的輸出。</span><span class="sxs-lookup"><span data-stu-id="5f120-140">Power BI provides an easy-to-use dashboard interface for viewing the output from Azure Stream Analytics.</span></span> |
| <span data-ttu-id="5f120-141">Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="5f120-141">Azure Stack Hub</span></span> | [<span data-ttu-id="5f120-142">App Service</span><span class="sxs-lookup"><span data-stu-id="5f120-142">App Service</span></span>](/azure-stack/operator/azure-stack-app-service-overview) | <span data-ttu-id="5f120-143">App Service 資源提供者 (RP) 提供 edge 元件的基礎，包括 web apps/Api 和函式的裝載和管理功能。</span><span class="sxs-lookup"><span data-stu-id="5f120-143">The App Service resource provider (RP) provides a base for edge components, including hosting and management features for web apps/APIs and Functions.</span></span> |
| | <span data-ttu-id="5f120-144">Azure Kubernetes Service [(AKS) 引擎](https://github.com/Azure/aks-engine)叢集</span><span class="sxs-lookup"><span data-stu-id="5f120-144">Azure Kubernetes Service [(AKS) Engine](https://github.com/Azure/aks-engine) cluster</span></span> | <span data-ttu-id="5f120-145">AKS-Engine 叢集部署到 Azure Stack Hub 的 AKS RP 會提供可調整的彈性引擎以執行臉部 API 容器。</span><span class="sxs-lookup"><span data-stu-id="5f120-145">The AKS RP with AKS-Engine cluster deployed into Azure Stack Hub provides a scalable, resilient engine to run the Face API container.</span></span> |
| | <span data-ttu-id="5f120-146">Azure 認知服務[人臉識別 API 容器](/azure/cognitive-services/face/face-how-to-install-containers)</span><span class="sxs-lookup"><span data-stu-id="5f120-146">Azure Cognitive Services [Face API containers](/azure/cognitive-services/face/face-how-to-install-containers)</span></span>| <span data-ttu-id="5f120-147">具有人臉識別 API 容器的 Azure 認知服務 RP 會在 Contoso 的私人網路上提供人口統計、表情和獨特訪客的偵測能力。</span><span class="sxs-lookup"><span data-stu-id="5f120-147">The Azure Cognitive Services RP with Face API containers provides demographic, emotion, and unique visitor detection on Contoso's private network.</span></span> |
| | <span data-ttu-id="5f120-148">Blob 儲存體</span><span class="sxs-lookup"><span data-stu-id="5f120-148">Blob Storage</span></span> | <span data-ttu-id="5f120-149">從 AI 開發套件所擷取的映像會上傳至 Azure Stack Hub 的 Blob 儲存體。</span><span class="sxs-lookup"><span data-stu-id="5f120-149">Images captured from the AI Dev Kit are uploaded to Azure Stack Hub's blob storage.</span></span> |
| | <span data-ttu-id="5f120-150">Azure Functions</span><span class="sxs-lookup"><span data-stu-id="5f120-150">Azure Functions</span></span> | <span data-ttu-id="5f120-151">在 Azure Stack Hub 上執行的 Azure Function 會接收來自 blob 儲存體的輸入，並管理與臉部 API 的互動。</span><span class="sxs-lookup"><span data-stu-id="5f120-151">An Azure Function running on Azure Stack Hub receives input from blob storage and manages the interactions with the Face API.</span></span> <span data-ttu-id="5f120-152">其會將匿名資料發至位於 Azure 的事件中樞叢集。</span><span class="sxs-lookup"><span data-stu-id="5f120-152">It emits anonymized data to an Event Hubs cluster located in Azure.</span></span><br><br>|

## <a name="issues-and-considerations"></a><span data-ttu-id="5f120-153">問題和考量</span><span class="sxs-lookup"><span data-stu-id="5f120-153">Issues and considerations</span></span>

<span data-ttu-id="5f120-154">決定如何實作此解決方案時，請考慮下列幾點：</span><span class="sxs-lookup"><span data-stu-id="5f120-154">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="5f120-155">延展性</span><span class="sxs-lookup"><span data-stu-id="5f120-155">Scalability</span></span>

<span data-ttu-id="5f120-156">若要讓此解決方案可擴大為多個攝影機和位置，您必須確定所有元件都能應付增加的負載。</span><span class="sxs-lookup"><span data-stu-id="5f120-156">To enable this solution to scale across multiple cameras and locations, you'll need to make sure that all of the components can handle the increased load.</span></span> <span data-ttu-id="5f120-157">您可能需要採取如下的動作：</span><span class="sxs-lookup"><span data-stu-id="5f120-157">You may need to take actions like:</span></span>

- <span data-ttu-id="5f120-158">增加串流分析串流單位數目。</span><span class="sxs-lookup"><span data-stu-id="5f120-158">Increase the number of Stream Analytics streaming units.</span></span>
- <span data-ttu-id="5f120-159">向外擴充臉部 API 部署。</span><span class="sxs-lookup"><span data-stu-id="5f120-159">Scale out the Face API deployment.</span></span>
- <span data-ttu-id="5f120-160">增加事件中樞叢集輸送量。</span><span class="sxs-lookup"><span data-stu-id="5f120-160">Increase the Event Hubs cluster throughput.</span></span>
- <span data-ttu-id="5f120-161">在極端情況下，可能必須從 Azure Functions 遷移至虛擬機器。</span><span class="sxs-lookup"><span data-stu-id="5f120-161">For extreme cases, migrate from Azure Functions to a virtual machine may be necessary.</span></span>

### <a name="availability"></a><span data-ttu-id="5f120-162">可用性</span><span class="sxs-lookup"><span data-stu-id="5f120-162">Availability</span></span>

<span data-ttu-id="5f120-163">由於此解決方案採用階層式方法，因此請務必考慮如何處理網路或電源故障。</span><span class="sxs-lookup"><span data-stu-id="5f120-163">Since this solution is tiered, it's important to think about how to deal with networking or power failures.</span></span> <span data-ttu-id="5f120-164">視商務需求而定，您可能會想要執行機制來在本機快取影像，然後在連接傳回時轉送至 Azure Stack Hub。</span><span class="sxs-lookup"><span data-stu-id="5f120-164">Depending on business needs, you might want to implement a mechanism to cache images locally, then forward to Azure Stack Hub when connectivity returns.</span></span> <span data-ttu-id="5f120-165">如果位置夠大，將具有人臉識別 API 容器的 Data Box Edge 部署到該位置，或許會更好。</span><span class="sxs-lookup"><span data-stu-id="5f120-165">If the location is large enough, deploying a Data Box Edge with the Face API container to that location might be a better option.</span></span>

### <a name="manageability"></a><span data-ttu-id="5f120-166">管理能力</span><span class="sxs-lookup"><span data-stu-id="5f120-166">Manageability</span></span>

<span data-ttu-id="5f120-167">此解決方案可能會跨越許多裝置和位置而變得不好管理。</span><span class="sxs-lookup"><span data-stu-id="5f120-167">This solution can span many devices and locations, which could get unwieldy.</span></span> <span data-ttu-id="5f120-168">[Azure 的 IoT 服務](/azure/iot-fundamentals/) 可以用來自動將新的位置和裝置上線，並讓它們保持在最新狀態。</span><span class="sxs-lookup"><span data-stu-id="5f120-168">[Azure's IoT services](/azure/iot-fundamentals/) can be used to automatically bring new locations and devices online and keep them up to date.</span></span>

### <a name="security"></a><span data-ttu-id="5f120-169">安全性</span><span class="sxs-lookup"><span data-stu-id="5f120-169">Security</span></span>

<span data-ttu-id="5f120-170">此解決方案會擷取客戶影像，因此一定要考量到安全性。</span><span class="sxs-lookup"><span data-stu-id="5f120-170">This solution captures customer images, making security a paramount consideration.</span></span> <span data-ttu-id="5f120-171">請確定所有的儲存體帳戶都使用適當的存取原則來保護，並定期輪替金鑰。</span><span class="sxs-lookup"><span data-stu-id="5f120-171">Make sure all storage accounts are secured with the proper access policies and rotate keys regularly.</span></span> <span data-ttu-id="5f120-172">請確定儲存體帳戶和事件中樞有保留原則，且符合公司和政府的隱私權規定。</span><span class="sxs-lookup"><span data-stu-id="5f120-172">Ensure storage accounts and Event Hubs have retention policies that meet corporate and government privacy regulations.</span></span> <span data-ttu-id="5f120-173">另外，請確保對使用者存取層級進行分層。</span><span class="sxs-lookup"><span data-stu-id="5f120-173">Also make sure to tier the user access levels.</span></span> <span data-ttu-id="5f120-174">分層設定可確保使用者只能存取其角色所需的資料。</span><span class="sxs-lookup"><span data-stu-id="5f120-174">Tiering ensures that users only have access to the data they need for their role.</span></span>

## <a name="next-steps"></a><span data-ttu-id="5f120-175">下一步</span><span class="sxs-lookup"><span data-stu-id="5f120-175">Next steps</span></span>

<span data-ttu-id="5f120-176">若要深入了解本文介紹的更多相關主題：</span><span class="sxs-lookup"><span data-stu-id="5f120-176">To learn more about the topics introduced in this article:</span></span>

- <span data-ttu-id="5f120-177">請參閱客流量偵測模式使用的[分層資料模式](https://aka.ms/tiereddatadeploy)。</span><span class="sxs-lookup"><span data-stu-id="5f120-177">See the [Tiered Data pattern](https://aka.ms/tiereddatadeploy), which is leveraged by the footfall detection pattern.</span></span>
- <span data-ttu-id="5f120-178">若要深入了解如何使用自訂視覺，請參閱[自訂視覺 AI 開發套件](https://azure.github.io/Vision-AI-DevKit-Pages/)。</span><span class="sxs-lookup"><span data-stu-id="5f120-178">See the [Custom Vision AI Dev Kit](https://azure.github.io/Vision-AI-DevKit-Pages/) to learn more about using custom vision.</span></span> 

<span data-ttu-id="5f120-179">當您準備好測試解決方案範例時，請繼續參閱 [客流量偵測部署指南](solution-deployment-guide-retail-footfall-detection.md)。</span><span class="sxs-lookup"><span data-stu-id="5f120-179">When you're ready to test the solution example, continue with the [Footfall detection deployment guide](solution-deployment-guide-retail-footfall-detection.md).</span></span> <span data-ttu-id="5f120-180">部署指南提供部署及測試其元件的逐步指示。</span><span class="sxs-lookup"><span data-stu-id="5f120-180">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>
