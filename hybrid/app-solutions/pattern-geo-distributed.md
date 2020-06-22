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
# <a name="geo-distributed-app-pattern"></a>異地分散式應用程式模式

了解如何跨多個區域提供應用程式端點，並根據位置與合規性需求來路由傳送使用者流量。

## <a name="context-and-problem"></a>內容和問題

遍及廣大地理位置的組織都積極設法要安全且正確地散佈資料，並使其可供存取，同時確保各區域的每個使用者、位置和裝置的安全性、合規性和效能都能達到必要的水準。

## <a name="solution"></a>解決方法

Azure Stack Hub 地理流量路由傳送模式 (或異地分散式應用程式) 會讓流量根據各種計量導向至特定端點。 使用以地理位置為基礎的路由和端點組態來建立流量管理員，可根據區域需求、公司與國際法規和資料需求將流量路由至端點。

![地理位置分散模式](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a>元件

### <a name="outside-the-cloud"></a>雲端外部

#### <a name="traffic-manager"></a>流量管理員

在此圖中，流量管理員位於公用雲端以外，但必須能夠同時協調本機資料中心與公用雲端中的流量。 平衡器會將流量路由到地理位置。

#### <a name="domain-name-system-dns"></a>網域名稱系統 (DNS)

網域名稱系統 (DNS) 負責將網站或服務名稱轉譯 (或解析) 為其 IP 位址。

### <a name="public-cloud"></a>公用雲端

#### <a name="cloud-endpoint"></a>雲端端點

公用 IP 位址可透過流量管理員用來將傳入流量路由到公用雲端應用程式資源端點。  

### <a name="local-clouds"></a>本機雲端

#### <a name="local-endpoint"></a>本機端點

公用 IP 位址可透過流量管理員用來將傳入流量路由到公用雲端應用程式資源端點。

## <a name="issues-and-considerations"></a>問題和考量

當您決定如何實作此模式時，請考慮下列幾點：

### <a name="scalability"></a>延展性

此模式會處理地理流量路由，而不是根據流量增加的情況進行調整。 不過，您可以將此模式與其他 Azure 和內部部署解決方案結合。 例如，此模式可以與跨雲端調整模式搭配使用。

### <a name="availability"></a>可用性

確定已透過內部部署硬體設定和軟體部署，針對本機部署應用程式的高可用性進行設定。

### <a name="manageability"></a>管理能力

此模式可確保在不同環境間都能有順暢的管理和熟悉的介面。

## <a name="when-to-use-this-pattern"></a>使用此模式的時機

- 我的組織有海外分公司，且需要自訂的區域安全性和發佈原則。
- 我的每個組織辦公室皆提取員工、商務和設備資料，而需要各自的當地法規和時區的報告活動。
- 只要對單一區域內和跨區域的多個應用程式部署進行應用程式的水平相應放大，即可達到高延展性需求，以處理極高的負載需求。
- 應用程式必須具有高度可用性，且即使在單一區域中斷時也能回應用戶端要求。

## <a name="next-steps"></a>後續步驟

深入了解此文章介紹的更多相關主題：

- 若要深入了解此 DNS 型流量負載平衡器的運作方式，請參閱 [Azure 流量管理員概觀](/azure/traffic-manager/traffic-manager-overview)。
- 若要深入了解最佳做法並獲得其他問題的解答，請參閱[混合式應用程式設計考量](overview-app-design-considerations.md)。
- 若要深入了解產品與解決方案的完整組合，請參閱 [Azure Stack 系列產品和解決方案](/azure-stack)。

當您準備好測試解決方案範例時，請繼續參閱[異地分散式應用程式解決方案部署指南](solution-deployment-guide-geo-distributed.md)。 部署指南提供部署及測試其元件的逐步指示。 您將會了解如何根據各種計量，使用異地分散式應用程式模式將流量導向至特定端點。 使用以地理位置為基礎的路由和端點組態來建立流量管理員設定檔，可確保資訊會根據區域需求、公司與國際法規和您的資料需求路由至端點。
