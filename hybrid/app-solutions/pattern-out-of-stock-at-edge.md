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
# <a name="out-of-stock-detection-at-the-edge-pattern"></a>位於邊緣的缺貨偵測模式

此模式示範如何使用 Azure Stack Edge 或 Azure IoT Edge 裝置與網路攝影機，判斷貨架上是否有商品缺貨。

## <a name="context-and-problem"></a>內容和問題

實體零售商店失去銷售機會，因為當客戶尋找某個商品時，貨架上沒有該商品。 不過，商品可能位於店面後面，尚未補貨上架。 商店想要提高員工工作效率，並在商品需要補貨時自動接收通知。

## <a name="solution"></a>解決方法

解決方案範例會使用邊緣裝置 (例如每家店面中的一個 Azure Stack Edge)，以有效率地處理來自店面中相機的資料。 這種最佳化的設計可讓店面只將相關的事件與影像傳送到雲端。 此設計可節省頻寬、儲存空間，並確保客戶隱私權。 從每個相機讀取畫面時，ML 模型會處理影像，並傳回任何缺貨區域。 影像與缺貨區域會顯示在本機 Web 應用程式上。 此資料可以傳送至時間序列深入解析環境，以在 Power BI 中顯示見解。

![邊緣缺貨解決方案架構](media/pattern-out-of-stock-at-edge/solution-architecture.png)

解決方案的運作方式如下：

1. 影像是透過 HTTP 或 RTSP 透過網路攝影機來擷取的。
2. 影像會調整大小並傳送至推斷驅動程式，這會與 ML 模型通訊以判斷是否有任何存貨影像。
3. ML 模型會傳回任何缺貨區域。
4. 推斷驅動程式會將未經處理的影像上傳到 Blob (若已指定)，並將模型的結果傳送到 Azure IoT 中樞，以及裝置上的週框方塊處理器。
5. 週框方塊處理器會將週框方塊新增至影像，並在記憶體內部資料庫中快取影像路徑。
6. Web 應用程式會查詢影像，並依收到的順序顯示它們。
7. 在時間序列深入解析 (TSI) 中彙總來自 IoT 中樞的訊息。
8. Power BI 會顯示一段時間內缺貨商品的互動式報表，其中包括時間序列深入解析資料。


## <a name="components"></a>元件

此解決方案使用下列元件：

| 階層 | 元件 | 描述 |
|----------|-----------|-------------|
| 內部部署硬體 | 網路攝影機 | 網路攝影機是必要裝置，搭配 HTTP 或 RTSP 摘要，即可提供要用於進行推斷的影像。 |
| Azure | Azure IoT 中樞 | [Azure IoT 中樞](/azure/iot-hub/)會處理裝置佈建，以及邊緣裝置的傳訊作業。 |
|  | Azure Time Series Insights | [Azure 時間序列深入解析](/azure/time-series-insights/)會儲存來自 IoT 中樞的訊息以用於視覺效果。 |
|  | Power BI | [Microsoft Power BI](https://powerbi.microsoft.com/) 可提供缺貨事件的商業導向報告。 Power BI 會提供便於使用的儀表板介面，以供您檢視來自 Azure 串流分析的輸出。 |
| Azure Stack Edge 或<br>Azure IoT Edge 裝置 | Azure IoT Edge | [Azure IoT Edge](/azure/iot-edge/) 會協調內部部署容器的執行時間，並處理裝置管理和更新。|
| | Azure Project Brainwave | 在 Azure Stack Edge 裝置上，[Project Brainwave](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) 會使用可現場程式化閘道陣列 (FPGA) 來加速 ML 推斷。|

## <a name="issues-and-considerations"></a>問題和考量

決定如何實作此解決方案時，請考慮下列幾點：

### <a name="scalability"></a>延展性

根據所提供的硬體而定，大部分機器學習模型只能在特定每秒畫面格數的情況下執行。 從您的攝影機判斷最佳的取樣率，以確保 ML 管線不會備份。 不同類型的硬體會處理不同數目的攝影機與畫面播放速率。

### <a name="availability"></a>可用性

請務必考量邊緣裝置失去連線時可能會發生的情況。 請考慮時間序列深入解析與 Power BI 儀表板可能會遺失哪些資料。 提供的範例解決方案並未設計為具備高可用性。

### <a name="manageability"></a>管理能力

此解決方案可能會跨越許多裝置和位置而變得不好管理。 Azure 的 IoT 服務可讓新的位置與裝置自動上線，並讓其保持最新狀態。 您也必須遵循適當的資料控管程序。

### <a name="security"></a>安全性

此模式會處理潛在的敏感性資料。 請確定金鑰會定期輪替，並已正確設定 Azure 儲存體帳戶與本機共用的權限。

## <a name="next-steps"></a>後續步驟

深入了解此文章介紹的更多相關主題：
- 在此模式中會使用多個 IoT 相關服務，包括 [Azure IoT Edge](/azure/iot-edge/)、[Azure IoT 中樞](/azure/iot-hub/)與 [Azure 時間序列深入解析](/azure/time-series-insights/)。
- 若要深入了解 Microsoft Project Brainwave，請參閱[部落格公告](https://blogs.microsoft.com/ai/build-2018-project-brainwave/)，並參閱[Azure 運用 Project Brainwave 加速機器學習影片](https://www.youtube.com/watch?v=DJfMobMjCX0)。
- 若要深入了解最佳做法並獲得其他問題的解答，請參閱[混合式應用程式設計考量](overview-app-design-considerations.md)。
- 若要深入了解產品與解決方案的完整組合，請參閱 [Azure Stack 系列產品和解決方案](/azure-stack)。

當您準備好測試解決方案範例時，請繼續參閱[階層式資料分析解決方案部署指南](https://aka.ms/edgeinferencingdeploy)。 部署指南提供部署及測試其元件的逐步指示。
