---
title: Azure Stack Hub 中的跨雲端調整模式
description: 了解如何在 Azure 和 Azure Stack Hub 上建立可跨雲端調整的應用程式。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: a830f96e97c347cbbcc09a1b17f4836ecb6eb3e6
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910123"
---
# <a name="cross-cloud-scaling-pattern"></a>跨雲端調整模式

自動將資源新增至現有的應用程式，以因應增加的負載。

## <a name="context-and-problem"></a>內容和問題

您的應用程式無法增加容量，以因應非預期增加的需求。 缺少延展性會導致使用者在尖峰使用時間無法連線到應用程式。 應用程式可為固定數目的使用者提供服務。

國際企業需要安全、可靠且可用的雲端式應用程式。 因應增加的需求，並使用正確的基礎結構來支援這項需求，是非常重要的。 企業為了在成本和維護，以及商務資料安全性、儲存和即時可用性之間取得平衡，都費盡了心力。

您可能無法在公用雲端中執行應用程式。 不過，若讓企業在內部部署環境中維持用來處理應用程式需求高峰的容量，似乎不切實際。 透過此模式，您將可透過內部部署解決方案使用公用雲端的靈活性。

## <a name="solution"></a>解決方法

跨雲端調整模式會使用公用雲端資源擴充位於本機雲端的應用程式。 此模式會在需求增加或減少時觸發，並分別新增或移除雲端中的資源。 這些資源提供了備援性、快速的可用性，和異地相容的路由。

![跨雲端調整模式](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> 此模式僅適用於應用程式的無狀態元件。

## <a name="components"></a>元件

跨雲端調整模式由下列元件組成。

### <a name="outside-the-cloud"></a>雲端外部

#### <a name="traffic-manager"></a>流量管理員

在此圖中，其位於公用雲端群組以外，但必須能夠同時協調本機資料中心和公用雲端中的流量。 平衡器會監視端點，並在必要時提供容錯移轉轉散發，藉以提供應用程式的高可用性。

#### <a name="domain-name-system-dns"></a>網域名稱系統 (DNS)

網域名稱系統 (DNS) 負責將網站或服務名稱轉譯 (或解析) 為其 IP 位址。

### <a name="cloud"></a>Cloud

#### <a name="hosted-build-server"></a>裝載的組建伺服器

裝載組建管線的環境。

#### <a name="app-resources"></a>應用程式資源

應用程式資源必須能夠縮減和擴增，例如虛擬機器擴展集和容器。

#### <a name="custom-domain-name"></a>自訂網域名稱

將自訂變數名稱用於路由要求 GLOB。

#### <a name="public-ip-addresses"></a>公用 IP 位址

公用 IP 位址可透過流量管理員用來將傳入流量路由到公用雲端應用程式資源端點。  

### <a name="local-cloud"></a>本機雲端

#### <a name="hosted-build-server"></a>裝載的組建伺服器

裝載組建管線的環境。

#### <a name="app-resources"></a>應用程式資源

應用程式資源必須能夠縮減和擴增，例如虛擬機器擴展集和容器。

#### <a name="custom-domain-name"></a>自訂網域名稱

將自訂變數名稱用於路由要求 GLOB。

#### <a name="public-ip-addresses"></a>公用 IP 位址

公用 IP 位址可透過流量管理員用來將傳入流量路由到公用雲端應用程式資源端點。

## <a name="issues-and-considerations"></a>問題和考量

當您決定如何實作此模式時，請考慮下列幾點：

### <a name="scalability"></a>延展性

跨雲端調整的關鍵元件是提供隨選調整的能力。 必須在公用和本機雲端基礎結構之間進行調整，並根據需求提供一致且可靠的服務。

### <a name="availability"></a>可用性

確定已透過內部部署硬體設定和軟體部署，針對本機部署應用程式的高可用性進行設定。

### <a name="manageability"></a>管理能力

跨雲端模式可確保在不同環境間都能有順暢的管理和熟悉的介面。

## <a name="when-to-use-this-pattern"></a>使用此模式的時機

使用此模式：

- 當您需要增加應用程式容量以因應非預期的需求或定期需求時。
- 當您不想投資於只有尖峰期間才會用到的資源時。 您用多少就付多少。

在下列情況下不建議使用此模式：

- 使用者在使用您的解決方案時需要透過網際網路連線。
- 您的企業有當地的法規，要求起始連線必須來自現場電話。
- 您的網路出現將使調整效能受限的一般瓶頸。
- 您的環境已中斷網際網路連線，因此無法連接到公用雲端。

## <a name="next-steps"></a>後續步驟

深入了解此文章介紹的更多相關主題：

- 若要深入了解此 DNS 型流量負載平衡器的運作方式，請參閱 [Azure 流量管理員概觀](/azure/traffic-manager/traffic-manager-overview)。
- 若要深入了解最佳做法並獲得任何其他問題的解答，請參閱[混合式應用程式設計考量](overview-app-design-considerations.md)。
- 若要深入了解產品與解決方案的完整組合，請參閱 [Azure Stack 系列產品和解決方案](/azure-stack)。

當您準備好測試解決方案範例時，請繼續參閱[跨雲端調整解決方案部署指南](solution-deployment-guide-cross-cloud-scaling.md)。 部署指南提供部署及測試其元件的逐步指示。 了解如何建立可提供手動觸發程序的跨雲端解決方案，從 Azure Stack Hub 託管的 Web 應用程式切換到 Azure 託管的 Web 應用程式。 您也將了解如何透過流量管理員使用自動調整功能，以確保在負載下時有彈性且可調整的雲端公用程式。
