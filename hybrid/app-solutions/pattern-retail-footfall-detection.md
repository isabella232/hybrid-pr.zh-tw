---
title: 使用 Azure 和 Azure Stack Hub 的客流量偵測模式
description: 瞭解如何使用 Azure 和 Azure Stack Hub 來實行以 AI 為基礎的客流量偵測解決方案，以便分析零售商店的流量。
author: BryanLa
ms.topic: article
ms.date: 10/31/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 10/31/2019
ms.openlocfilehash: 79fb39d418bed53ef6a78980fcd9188bdf6e57ae
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281273"
---
# <a name="footfall-detection-pattern"></a>客流量偵測模式

此模式概述如何實行以 AI 為基礎的客流量偵測解決方案，以便在零售商店中分析訪客的流量。 該解決方案會使用 Azure、Azure Stack Hub 和自訂視覺 AI 開發套件，從真實世界的動作產生深入解析。

## <a name="context-and-problem"></a>內容和問題

Contoso 商店想要深入瞭解客戶如何接收其目前的產品（相對於儲存版面配置）。 他們無法在每一節中放置員工，而且讓分析師的團隊審視整個商店的攝影機素材沒有效率。 此外，其每一家店都沒有足夠頻寬可將影片從所有攝影機串流到雲端進行分析。

Contoso 想要以不顯眼且不侵犯隱私的方式來判斷其客戶的人口統計資訊、忠誠度以及對於商店的陳列和產品有何反應。

## <a name="solution"></a>解決方法

此零售分析模式使用階層式方法在邊緣進行推斷。 藉由使用自訂視覺 AI 開發套件，系統只會將有人臉的影像傳送至執行 Azure 認知服務的私人 Azure Stack Hub 進行分析。 系統會將匿名的彙總資料傳送至 Azure，以便彙總所有商店的資料，並在 Power BI 中以視覺方式呈現。 結合 edge 和公用雲端可讓 Contoso 利用現代化的 AI 技術，同時也保有與其公司原則的合規性，並尊重客戶的隱私權。

[![客流量偵測模式解決方案](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)

以下摘要說明該解決方案的運作方式：

1. 自訂視覺 AI 開發套件會從 IoT 中樞取得設定，以安裝 IoT Edge 執行階段和 ML 模型。
2. 如果該模型發現有人，便會拍照並上傳至 Azure Stack Hub Blob 儲存體。
3. Blob 服務會在 Azure Stack Hub 上觸發 Azure Function。
4. Azure Function 會呼叫有人臉識別 API 的容器，以從該影像取得人口統計和表情方面的資料。
5. 系統會將匿名資料傳送至 Azure 事件中樞叢集。
6. 事件中樞叢集會將資料推送至串流分析。
7. 串流分析會匯總資料並推送至 Power BI。

## <a name="components"></a>元件

此解決方案使用下列元件：

| 階層 | 元件 | 描述 |
|----------|-----------|-------------|
| 店內硬體 | [自訂視覺 AI 開發套件](https://azure.github.io/Vision-AI-DevKit-Pages/) | 提供使用本機 ML 模型的店內篩選功能，並只擷取有人的影像來進行分析。 透過 IoT 中樞安全地進行佈建和更新。<br><br>|
| Azure | [Azure 事件中樞](/azure/event-hubs/) | Azure 事件中樞會提供可調整的平台，以便用來擷取可與 Azure 串流分析完美整合的匿名資料。 |
|  | [Azure 串流分析](/azure/stream-analytics/) | Azure 串流分析作業會彙總匿名資料，並將其分組為 15 秒長的時段而以視覺方式加以呈現。 |
|  | [Microsoft Power BI](https://powerbi.microsoft.com/) | Power BI 會提供便於使用的儀表板介面，以供您檢視來自 Azure 串流分析的輸出。 |
| Azure Stack Hub | [App Service](/azure-stack/operator/azure-stack-app-service-overview) | App Service 資源提供者 (RP) 提供 edge 元件的基礎，包括 web apps/Api 和函式的裝載和管理功能。 |
| | Azure Kubernetes Service [(AKS) 引擎](https://github.com/Azure/aks-engine)叢集 | AKS-Engine 叢集部署到 Azure Stack Hub 的 AKS RP 會提供可調整的彈性引擎以執行臉部 API 容器。 |
| | Azure 認知服務[人臉識別 API 容器](/azure/cognitive-services/face/face-how-to-install-containers)| 具有人臉識別 API 容器的 Azure 認知服務 RP 會在 Contoso 的私人網路上提供人口統計、表情和獨特訪客的偵測能力。 |
| | Blob 儲存體 | 從 AI 開發套件所擷取的映像會上傳至 Azure Stack Hub 的 Blob 儲存體。 |
| | Azure Functions | 在 Azure Stack Hub 上執行的 Azure Function 會接收來自 blob 儲存體的輸入，並管理與臉部 API 的互動。 其會將匿名資料發至位於 Azure 的事件中樞叢集。<br><br>|

## <a name="issues-and-considerations"></a>問題和考量

決定如何實作此解決方案時，請考慮下列幾點：

### <a name="scalability"></a>延展性

若要讓此解決方案可擴大為多個攝影機和位置，您必須確定所有元件都能應付增加的負載。 您可能需要採取如下的動作：

- 增加串流分析串流單位數目。
- 向外擴充臉部 API 部署。
- 增加事件中樞叢集輸送量。
- 在極端情況下，可能必須從 Azure Functions 遷移至虛擬機器。

### <a name="availability"></a>可用性

由於此解決方案採用階層式方法，因此請務必考慮如何處理網路或電源故障。 視商務需求而定，您可能會想要執行機制來在本機快取影像，然後在連接傳回時轉送至 Azure Stack Hub。 如果位置夠大，將具有人臉識別 API 容器的 Data Box Edge 部署到該位置，或許會更好。

### <a name="manageability"></a>管理能力

此解決方案可能會跨越許多裝置和位置而變得不好管理。 [Azure 的 IoT 服務](/azure/iot-fundamentals/) 可以用來自動將新的位置和裝置上線，並讓它們保持在最新狀態。

### <a name="security"></a>安全性

此解決方案會擷取客戶影像，因此一定要考量到安全性。 請確定所有的儲存體帳戶都使用適當的存取原則來保護，並定期輪替金鑰。 請確定儲存體帳戶和事件中樞有保留原則，且符合公司和政府的隱私權規定。 另外，請確保對使用者存取層級進行分層。 分層設定可確保使用者只能存取其角色所需的資料。

## <a name="next-steps"></a>下一步

若要深入了解本文介紹的更多相關主題：

- 請參閱客流量偵測模式使用的[分層資料模式](https://aka.ms/tiereddatadeploy)。
- 若要深入了解如何使用自訂視覺，請參閱[自訂視覺 AI 開發套件](https://azure.github.io/Vision-AI-DevKit-Pages/)。 

當您準備好測試解決方案範例時，請繼續參閱 [客流量偵測部署指南](/azure/architecture/hybrid/deployments/solution-deployment-guide-retail-footfall-detection)。 部署指南提供部署及測試其元件的逐步指示。