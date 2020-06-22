---
title: 使用 Azure 和 Azure Stack Hub 的階層式資料分析模式
description: 了解如何使用 Azure 與 Azure Stack Hub 來跨混合式雲端實作階層式資料解決方案。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: b671fa9f47fa51ab6e40633c04964957d613fec2
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910154"
---
# <a name="tiered-data-for-analytics-pattern"></a>階層式資料分析模式

此模式說明如何使用 Azure Stack Hub 與 Azure，跨多個內部部署與雲端位置暫存、分析、處理及儲存資料。

## <a name="context-and-problem"></a>內容和問題

現代化技術領域中的企業組織所面臨的其中一個問題就是保護資料儲存安全、處理及分析。 考量項目包括：

- 資料內容
- location
- 安全性與隱私權需求
- 存取權限
- 維護
- 儲存體倉儲

Azure 與 Azure Stack Hun 結合，可解決資料疑慮，並提供低成本解決方案。 此解決方案最適合透過分散式製造或物流公司來示範。

此解決方案是以下列案例為基礎：

- 大型的多分支製造組織。
- 需要在全球遠端位置和中央總部之間快速且安全地儲存、處理及散發資料。
- 必須維持安全的員工與機械活動、設施資訊與業務報告資料。 資料必須適當地散發，並符合區域合規性原則與業界法規。

## <a name="solution"></a>解決方法

使用內部部署與公用雲端環境來滿足擁有許多設備企業的需求。 Azure Stack Hub 提供快速、安全且彈性的解決方案來收集、處理、儲存及散發本機與遠端資料。 當安全性、機密性、公司政策和法規需求在位置和使用者之間有所差異時，此模式特別實用。

![階層式資料分析模式解決方案架構](media/pattern-tiered-data-analytics/solution-architecture.png)

## <a name="components"></a>元件

此模式使用下列元件：

| 階層 | 元件 | 描述 |
|----------|-----------|-------------|
| Azure | 儲存體 | [Azure 儲存體](/azure/storage/)帳戶提供乾淨的資料取用端點。 Azure 儲存體是 Microsoft 針對最新資料儲存體環境推出的雲端儲存體解決方案。 Azure 儲存體提供可大幅調整的資料物件存放區，以及雲端檔案系統服務。 其也提供可用於進行可靠傳訊的訊息存放區，以及 NoSQL 存放區。 |
| Azure Stack Hub | 儲存體 | [Azure Stack Hub 儲存體](/azure-stack/user/azure-stack-storage-overview)帳戶會用於建立多個服務：<br><br>用於儲存原始資料的 - **Blob 儲存體**。 Blob 儲存體可以儲存任何類型的文字或二進位資料，例如文件、媒體檔案或應用程式安裝程式。 每個 Blob 會在容器下形成組織。 容器提供對物件群組指派安全原則的實用方式。 儲存體帳戶可包含任意數目的容器，而容器可包含任意數目的 Blob，儲存體帳戶的容量限制高達 500 TB。<br>用於資料封存的- **Blob 儲存體**。 適用於非經常性存取層資料封存的低成本儲存體有其優點。 非經常性存取層資料的範例包括備份、媒體內容、科學資料、合規性與封存資料。 一般而言，未經常存取的任何資料都會被視為非經常性儲存層儲存體。 根據存取頻率與保留期限等屬性來分層的資料。 客戶資料並不會被經常存取，但需要與經常性存取層資料類似的延遲和效能。<br>用於儲存已處理資料的- **佇列儲存體**。 佇列儲存體可提供應用程式元件之間的雲端傳訊。 設計擴充性的應用程式時，會經常分離應用程式元件，以便進行個別擴充。 佇列儲存體可針對應用程式元件間的通訊，提供非同步傳訊，無論應用程式元件是在雲端、桌面、內部部署伺服器或行動裝置上執行。 佇列儲存體也支援管理非同步工作並建置處理工作流程。 |
| | Azure Functions | [Azure Functions](/azure/azure-functions/) 服務是由 [Azure Stack Hub 上的 Azure App Service](/azure-stack/operator/azure-stack-app-service-overview) 資源提供者提供。 Azure Functions 可讓您在簡單、無伺服器的環境中執行程式碼，以回應各種事件。 Azure Functions 可使用您選擇的程式設計語言調整規模以符合需求，不需要建立 VM 或發佈 Web 應用程式。 解決方案會使用函式來進行下列工作：<br><br>- **資料輸入**<br>- **資料淨化** 手動觸發的函式可以執行已排程的資料處理、清理和封存作業。 範例可能包括夜間客戶清單清理與每月報表處理。|

## <a name="issues-and-considerations"></a>問題和考量

決定如何實作此解決方案時，請考慮下列幾點：

### <a name="scalability"></a>延展性

Azure Functions 及儲存體解決方案會進行縮放，以滿足資料量和處理需求。 如需有關 Azure 延展性資訊與目標，請參閱 [Azure 儲存體延展性文件](/azure/storage/common/storage-scalability-targets)。

### <a name="availability"></a>可用性

儲存體是此模式的主要可用性考量。 大量資料的處理和散發都需要透過快速連結的連線。

### <a name="manageability"></a>管理能力

此解決方案的管理性取決於所使用的製作工具，以及是否採用原始程式碼控制。

## <a name="next-steps"></a>後續步驟

深入了解此文章介紹的更多相關主題：

- 請參閱 [Azure 儲存體](/azure/storage/)與 [Azure Functions](/azure/azure-functions/) 文件。 此模式會大量使用 Azure 與 Azure Stack Hub 上的 Azure 儲存體帳戶和 Azure Functions。
- 若要深入了解最佳做法並獲得其他問題的解答，請參閱[混合式應用程式設計考量](overview-app-design-considerations.md)。
- 若要深入了解產品與解決方案的完整組合，請參閱 [Azure Stack 系列產品和解決方案](/azure-stack)。

當您準備好測試解決方案範例時，請繼續參閱[階層式資料分析解決方案部署指南](https://aka.ms/tiereddatadeploy)。 部署指南提供部署及測試其元件的逐步指示。
