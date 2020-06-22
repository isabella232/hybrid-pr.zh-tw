---
title: 使用 Azure 和 Azure Stack Edge 的缺貨偵測
description: 了解如何使用 Azure 與 Azure Stack Edge 服務實作缺貨偵測功能。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 865f63bc4234e50ed169aa29cefdb1886750594c
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910139"
---
# <a name="out-of-stock-detection-at-the-edge-pattern"></a><span data-ttu-id="c7e24-103">位於邊緣的缺貨偵測模式</span><span class="sxs-lookup"><span data-stu-id="c7e24-103">Out of stock detection at the edge pattern</span></span>

<span data-ttu-id="c7e24-104">此模式示範如何使用 Azure Stack Edge 或 Azure IoT Edge 裝置與網路攝影機，判斷貨架上是否有商品缺貨。</span><span class="sxs-lookup"><span data-stu-id="c7e24-104">This pattern illustrates how to determine if shelves have out of stock items using an Azure Stack Edge or Azure IoT Edge device and network cameras.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="c7e24-105">內容和問題</span><span class="sxs-lookup"><span data-stu-id="c7e24-105">Context and problem</span></span>

<span data-ttu-id="c7e24-106">實體零售商店失去銷售機會，因為當客戶尋找某個商品時，貨架上沒有該商品。</span><span class="sxs-lookup"><span data-stu-id="c7e24-106">Physical retail stores lose sales because when customers look for an item, it's not present on the shelf.</span></span> <span data-ttu-id="c7e24-107">不過，商品可能位於店面後面，尚未補貨上架。</span><span class="sxs-lookup"><span data-stu-id="c7e24-107">However, the item could have been in the back of the store and not been restocked.</span></span> <span data-ttu-id="c7e24-108">商店想要提高員工工作效率，並在商品需要補貨時自動接收通知。</span><span class="sxs-lookup"><span data-stu-id="c7e24-108">Stores would like to use their staff more efficiently and get automatically notified when items need restocking.</span></span>

## <a name="solution"></a><span data-ttu-id="c7e24-109">解決方法</span><span class="sxs-lookup"><span data-stu-id="c7e24-109">Solution</span></span>

<span data-ttu-id="c7e24-110">解決方案範例會使用邊緣裝置 (例如每家店面中的一個 Azure Stack Edge)，以有效率地處理來自店面中相機的資料。</span><span class="sxs-lookup"><span data-stu-id="c7e24-110">The solution example uses an edge device, like an Azure Stack Edge in each store, which efficiently processes data from cameras in the store.</span></span> <span data-ttu-id="c7e24-111">這種最佳化的設計可讓店面只將相關的事件與影像傳送到雲端。</span><span class="sxs-lookup"><span data-stu-id="c7e24-111">This optimized design lets stores send only relevant events and images to the cloud.</span></span> <span data-ttu-id="c7e24-112">此設計可節省頻寬、儲存空間，並確保客戶隱私權。</span><span class="sxs-lookup"><span data-stu-id="c7e24-112">The design saves bandwidth, storage space, and ensures customer privacy.</span></span> <span data-ttu-id="c7e24-113">從每個相機讀取畫面時，ML 模型會處理影像，並傳回任何缺貨區域。</span><span class="sxs-lookup"><span data-stu-id="c7e24-113">As frames are read from each camera, an ML model processes the image and returns any out of stock areas.</span></span> <span data-ttu-id="c7e24-114">影像與缺貨區域會顯示在本機 Web 應用程式上。</span><span class="sxs-lookup"><span data-stu-id="c7e24-114">The image and out of stock areas are displayed on a local web app.</span></span> <span data-ttu-id="c7e24-115">此資料可以傳送至時間序列深入解析環境，以在 Power BI 中顯示見解。</span><span class="sxs-lookup"><span data-stu-id="c7e24-115">This data can be sent to a Time Series Insight environment to show insights in Power BI.</span></span>

![邊緣缺貨解決方案架構](media/pattern-out-of-stock-at-edge/solution-architecture.png)

<span data-ttu-id="c7e24-117">解決方案的運作方式如下：</span><span class="sxs-lookup"><span data-stu-id="c7e24-117">Here's how the solution works:</span></span>

1. <span data-ttu-id="c7e24-118">影像是透過 HTTP 或 RTSP 透過網路攝影機來擷取的。</span><span class="sxs-lookup"><span data-stu-id="c7e24-118">Images are captured from a network camera over HTTP or RTSP.</span></span>
2. <span data-ttu-id="c7e24-119">影像會調整大小並傳送至推斷驅動程式，這會與 ML 模型通訊以判斷是否有任何存貨影像。</span><span class="sxs-lookup"><span data-stu-id="c7e24-119">The image is resized and sent to the inference driver, which communicates with the ML model to determine if there are any out of stock images.</span></span>
3. <span data-ttu-id="c7e24-120">ML 模型會傳回任何缺貨區域。</span><span class="sxs-lookup"><span data-stu-id="c7e24-120">The ML model returns any out of stock areas.</span></span>
4. <span data-ttu-id="c7e24-121">推斷驅動程式會將未經處理的影像上傳到 Blob (若已指定)，並將模型的結果傳送到 Azure IoT 中樞，以及裝置上的週框方塊處理器。</span><span class="sxs-lookup"><span data-stu-id="c7e24-121">The inferencing driver uploads the raw image to a blob (if specified), and sends the results from the model to Azure IoT Hub and a bounding box processor on the device.</span></span>
5. <span data-ttu-id="c7e24-122">週框方塊處理器會將週框方塊新增至影像，並在記憶體內部資料庫中快取影像路徑。</span><span class="sxs-lookup"><span data-stu-id="c7e24-122">The bounding box processor adds bounding boxes to the image and caches the image path in an in-memory database.</span></span>
6. <span data-ttu-id="c7e24-123">Web 應用程式會查詢影像，並依收到的順序顯示它們。</span><span class="sxs-lookup"><span data-stu-id="c7e24-123">The web app queries for images and shows them in the order received.</span></span>
7. <span data-ttu-id="c7e24-124">在時間序列深入解析 (TSI) 中彙總來自 IoT 中樞的訊息。</span><span class="sxs-lookup"><span data-stu-id="c7e24-124">Messages from IoT Hub are aggregated in Time Series Insights.</span></span>
8. <span data-ttu-id="c7e24-125">Power BI 會顯示一段時間內缺貨商品的互動式報表，其中包括時間序列深入解析資料。</span><span class="sxs-lookup"><span data-stu-id="c7e24-125">Power BI displays an interactive report of out of stock items over time with the data from Time Series Insights.</span></span>


## <a name="components"></a><span data-ttu-id="c7e24-126">元件</span><span class="sxs-lookup"><span data-stu-id="c7e24-126">Components</span></span>

<span data-ttu-id="c7e24-127">此解決方案使用下列元件：</span><span class="sxs-lookup"><span data-stu-id="c7e24-127">This solution uses the following components:</span></span>

| <span data-ttu-id="c7e24-128">階層</span><span class="sxs-lookup"><span data-stu-id="c7e24-128">Layer</span></span> | <span data-ttu-id="c7e24-129">元件</span><span class="sxs-lookup"><span data-stu-id="c7e24-129">Component</span></span> | <span data-ttu-id="c7e24-130">描述</span><span class="sxs-lookup"><span data-stu-id="c7e24-130">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="c7e24-131">內部部署硬體</span><span class="sxs-lookup"><span data-stu-id="c7e24-131">On-premises hardware</span></span> | <span data-ttu-id="c7e24-132">網路攝影機</span><span class="sxs-lookup"><span data-stu-id="c7e24-132">Network camera</span></span> | <span data-ttu-id="c7e24-133">網路攝影機是必要裝置，搭配 HTTP 或 RTSP 摘要，即可提供要用於進行推斷的影像。</span><span class="sxs-lookup"><span data-stu-id="c7e24-133">A network camera is required, with either an HTTP or RTSP feed to provide the images for inference.</span></span> |
| <span data-ttu-id="c7e24-134">Azure</span><span class="sxs-lookup"><span data-stu-id="c7e24-134">Azure</span></span> | <span data-ttu-id="c7e24-135">Azure IoT 中樞</span><span class="sxs-lookup"><span data-stu-id="c7e24-135">Azure IoT Hub</span></span> | <span data-ttu-id="c7e24-136">[Azure IoT 中樞](/azure/iot-hub/)會處理裝置佈建，以及邊緣裝置的傳訊作業。</span><span class="sxs-lookup"><span data-stu-id="c7e24-136">[Azure IoT Hub](/azure/iot-hub/) handles device provisioning and messaging for the edge devices.</span></span> |
|  | <span data-ttu-id="c7e24-137">Azure Time Series Insights</span><span class="sxs-lookup"><span data-stu-id="c7e24-137">Azure Time Series Insights</span></span> | <span data-ttu-id="c7e24-138">[Azure 時間序列深入解析](/azure/time-series-insights/)會儲存來自 IoT 中樞的訊息以用於視覺效果。</span><span class="sxs-lookup"><span data-stu-id="c7e24-138">[Azure Time Series Insights](/azure/time-series-insights/) stores the messages from IoT Hub for visualization.</span></span> |
|  | <span data-ttu-id="c7e24-139">Power BI</span><span class="sxs-lookup"><span data-stu-id="c7e24-139">Power BI</span></span> | <span data-ttu-id="c7e24-140">[Microsoft Power BI](https://powerbi.microsoft.com/) 可提供缺貨事件的商業導向報告。</span><span class="sxs-lookup"><span data-stu-id="c7e24-140">[Microsoft Power BI](https://powerbi.microsoft.com/) provides business-focused reports of out of stock events.</span></span> <span data-ttu-id="c7e24-141">Power BI 會提供便於使用的儀表板介面，以供您檢視來自 Azure 串流分析的輸出。</span><span class="sxs-lookup"><span data-stu-id="c7e24-141">Power BI provides an easy-to-use dashboard interface for viewing the output from Azure Stream Analytics.</span></span> |
| <span data-ttu-id="c7e24-142">Azure Stack Edge 或</span><span class="sxs-lookup"><span data-stu-id="c7e24-142">Azure Stack Edge or</span></span><br><span data-ttu-id="c7e24-143">Azure IoT Edge 裝置</span><span class="sxs-lookup"><span data-stu-id="c7e24-143">Azure IoT Edge device</span></span> | <span data-ttu-id="c7e24-144">Azure IoT Edge</span><span class="sxs-lookup"><span data-stu-id="c7e24-144">Azure IoT Edge</span></span> | <span data-ttu-id="c7e24-145">[Azure IoT Edge](/azure/iot-edge/) 會協調內部部署容器的執行時間，並處理裝置管理和更新。</span><span class="sxs-lookup"><span data-stu-id="c7e24-145">[Azure IoT Edge](/azure/iot-edge/) orchestrates the runtime for the on-premises containers and handles device management and updates.</span></span>|
| | <span data-ttu-id="c7e24-146">Azure Project Brainwave</span><span class="sxs-lookup"><span data-stu-id="c7e24-146">Azure project brainwave</span></span> | <span data-ttu-id="c7e24-147">在 Azure Stack Edge 裝置上，[Project Brainwave](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) 會使用可現場程式化閘道陣列 (FPGA) 來加速 ML 推斷。</span><span class="sxs-lookup"><span data-stu-id="c7e24-147">On an Azure Stack Edge device, [Project Brainwave](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) uses Field-Programmable Gate Arrays (FPGAs) to accelerate ML inferencing.</span></span>|

## <a name="issues-and-considerations"></a><span data-ttu-id="c7e24-148">問題和考量</span><span class="sxs-lookup"><span data-stu-id="c7e24-148">Issues and considerations</span></span>

<span data-ttu-id="c7e24-149">決定如何實作此解決方案時，請考慮下列幾點：</span><span class="sxs-lookup"><span data-stu-id="c7e24-149">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="c7e24-150">延展性</span><span class="sxs-lookup"><span data-stu-id="c7e24-150">Scalability</span></span>

<span data-ttu-id="c7e24-151">根據所提供的硬體而定，大部分機器學習模型只能在特定每秒畫面格數的情況下執行。</span><span class="sxs-lookup"><span data-stu-id="c7e24-151">Most machine learning models can only run at a certain number of frames per second, depending on the provided hardware.</span></span> <span data-ttu-id="c7e24-152">從您的攝影機判斷最佳的取樣率，以確保 ML 管線不會備份。</span><span class="sxs-lookup"><span data-stu-id="c7e24-152">Determine the optimal sample rate from your camera(s) to ensure that the ML pipeline doesn't back up.</span></span> <span data-ttu-id="c7e24-153">不同類型的硬體會處理不同數目的攝影機與畫面播放速率。</span><span class="sxs-lookup"><span data-stu-id="c7e24-153">Different types of hardware will handle different numbers of cameras and frame rates.</span></span>

### <a name="availability"></a><span data-ttu-id="c7e24-154">可用性</span><span class="sxs-lookup"><span data-stu-id="c7e24-154">Availability</span></span>

<span data-ttu-id="c7e24-155">請務必考量邊緣裝置失去連線時可能會發生的情況。</span><span class="sxs-lookup"><span data-stu-id="c7e24-155">It's important to consider what might happen if the edge device loses connectivity.</span></span> <span data-ttu-id="c7e24-156">請考慮時間序列深入解析與 Power BI 儀表板可能會遺失哪些資料。</span><span class="sxs-lookup"><span data-stu-id="c7e24-156">Consider what data might be lost from the Time Series Insights and Power BI dashboard.</span></span> <span data-ttu-id="c7e24-157">提供的範例解決方案並未設計為具備高可用性。</span><span class="sxs-lookup"><span data-stu-id="c7e24-157">The example solution as provided isn't designed to be highly available.</span></span>

### <a name="manageability"></a><span data-ttu-id="c7e24-158">管理能力</span><span class="sxs-lookup"><span data-stu-id="c7e24-158">Manageability</span></span>

<span data-ttu-id="c7e24-159">此解決方案可能會跨越許多裝置和位置而變得不好管理。</span><span class="sxs-lookup"><span data-stu-id="c7e24-159">This solution can span many devices and locations, which could get unwieldy.</span></span> <span data-ttu-id="c7e24-160">Azure 的 IoT 服務可讓新的位置與裝置自動上線，並讓其保持最新狀態。</span><span class="sxs-lookup"><span data-stu-id="c7e24-160">Azure's IoT services can automatically bring new locations and devices online and keep them up to date.</span></span> <span data-ttu-id="c7e24-161">您也必須遵循適當的資料控管程序。</span><span class="sxs-lookup"><span data-stu-id="c7e24-161">Proper data governance procedures must be followed as well.</span></span>

### <a name="security"></a><span data-ttu-id="c7e24-162">安全性</span><span class="sxs-lookup"><span data-stu-id="c7e24-162">Security</span></span>

<span data-ttu-id="c7e24-163">此模式會處理潛在的敏感性資料。</span><span class="sxs-lookup"><span data-stu-id="c7e24-163">This pattern handles potentially sensitive data.</span></span> <span data-ttu-id="c7e24-164">請確定金鑰會定期輪替，並已正確設定 Azure 儲存體帳戶與本機共用的權限。</span><span class="sxs-lookup"><span data-stu-id="c7e24-164">Make sure keys are regularly rotated and the permissions on the Azure Storage Account and local shares are correctly set.</span></span>

## <a name="next-steps"></a><span data-ttu-id="c7e24-165">後續步驟</span><span class="sxs-lookup"><span data-stu-id="c7e24-165">Next steps</span></span>

<span data-ttu-id="c7e24-166">深入了解此文章介紹的更多相關主題：</span><span class="sxs-lookup"><span data-stu-id="c7e24-166">To learn more about topics introduced in this article:</span></span>
- <span data-ttu-id="c7e24-167">在此模式中會使用多個 IoT 相關服務，包括 [Azure IoT Edge](/azure/iot-edge/)、[Azure IoT 中樞](/azure/iot-hub/)與 [Azure 時間序列深入解析](/azure/time-series-insights/)。</span><span class="sxs-lookup"><span data-stu-id="c7e24-167">Multiple IoT related services are used in this pattern, including [Azure IoT Edge](/azure/iot-edge/), [Azure IoT Hub](/azure/iot-hub/), and [Azure Time Series Insights](/azure/time-series-insights/).</span></span>
- <span data-ttu-id="c7e24-168">若要深入了解 Microsoft Project Brainwave，請參閱[部落格公告](https://blogs.microsoft.com/ai/build-2018-project-brainwave/)，並參閱[Azure 運用 Project Brainwave 加速機器學習影片](https://www.youtube.com/watch?v=DJfMobMjCX0)。</span><span class="sxs-lookup"><span data-stu-id="c7e24-168">To learn more about Microsoft Project Brainwave, see [the blog announcement](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) and checkout out the [Azure Accelerated Machine Learning with Project Brainwave video](https://www.youtube.com/watch?v=DJfMobMjCX0).</span></span>
- <span data-ttu-id="c7e24-169">若要深入了解最佳做法並獲得其他問題的解答，請參閱[混合式應用程式設計考量](overview-app-design-considerations.md)。</span><span class="sxs-lookup"><span data-stu-id="c7e24-169">See [Hybrid app design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers to any additional questions.</span></span>
- <span data-ttu-id="c7e24-170">若要深入了解產品與解決方案的完整組合，請參閱 [Azure Stack 系列產品和解決方案](/azure-stack)。</span><span class="sxs-lookup"><span data-stu-id="c7e24-170">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="c7e24-171">當您準備好測試解決方案範例時，請繼續參閱[階層式資料分析解決方案部署指南](https://aka.ms/edgeinferencingdeploy)。</span><span class="sxs-lookup"><span data-stu-id="c7e24-171">When you're ready to test the solution example, continue with the [Tiered data for analytics solution deployment guide](https://aka.ms/edgeinferencingdeploy).</span></span> <span data-ttu-id="c7e24-172">部署指南提供部署及測試其元件的逐步指示。</span><span class="sxs-lookup"><span data-stu-id="c7e24-172">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>
