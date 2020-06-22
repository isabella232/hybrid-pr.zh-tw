---
title: Azure 和 Azure Stack Hub 中的混合式轉送模式
description: 使用 Azure 與 Azure Stack Hub 中的混合式轉送模式來連線到受防火牆保護的邊緣資源。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 03b20a20a04f620c977fb20e1ea26f5982e42721
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910127"
---
# <a name="hybrid-relay-pattern"></a>Hybrid 轉送模式

瞭解如何使用混合式轉送模式和 Azure 轉送來連線到邊緣資源或受防火牆保護的裝置。

## <a name="context-and-problem"></a>內容和問題

邊緣裝置通常位於公司防火牆或 NAT 裝置後方。 雖然這些裝置是安全的，但可能無法與公用雲端或其他公司網路上的邊緣裝置通訊。 可能必須以安全的方式，向公用雲端中的使用者公開特定連接埠與功能。

## <a name="solution"></a>解決方法

混合式轉送模式會使用 Azure 轉送，在無法直接通訊的兩個端點之間建立 Websocket 通道。 不是內部部署但需要連線到內部部署端點的裝置，將會連線至公用雲端中的端點。 此端點會在安全通道上，透過預先定義的路由將流量重新導向。 內部部署環境內部的端點會接收流量，並將其路由傳送到正確的目的地。

![混合式轉送模式解決方案架構](media/pattern-hybrid-relay/solution-architecture.png)

以下是混合式轉送模式的運作方式：

1. 裝置透過預先定義的連接埠連線至 Azure 中的虛擬機器 (VM)。
2. 流量會轉送到 Azure 中的 Azure 轉送。
3. Azure Stack 中樞上的 VM 已建立與 Azure 轉送的長時間連線，會接收流量並將其轉送至目的地。
4. 內部部署服務或端點會處理要求。

## <a name="components"></a>元件

此解決方案使用下列元件：

| 階層 | 元件 | 描述 |
|----------|-----------|-------------|
| Azure | Azure VM | Azure VM 提供內部部署資源的可公開存取端點。 |
| | Azure 轉送 | [Azure 轉送](/azure/azure-relay/)提供基礎結構來維護 Azure vm 與 AZURE STACK Hub vm 之間的通道和連線。|
| Azure Stack Hub | 計算 | Azure Stack Hub VM 提供混合式轉送通道的伺服器端。 |
| | 儲存體 | 已部署到 Azure Stack Hub 中的 AKS 引擎叢集會提供可調整的彈性引擎以執行臉部 API 容器。|

## <a name="issues-and-considerations"></a>問題和考量

決定如何實作此解決方案時，請考慮下列幾點：

### <a name="scalability"></a>延展性

此模式只允許在用戶端與伺服器上進行 1:1 連接埠對應。 例如，如果 Azure 端點上的一個服務在連接埠 80 建立通道，它就無法供另一個服務使用。 應據以規劃連接埠對應。 Azure 轉送和 Vm 應該適當地調整來處理流量。

### <a name="availability"></a>可用性

這些通道與連線並不是多餘的。 若要確保高可用性，您可以實作錯誤檢查程式碼。 另一個選項是在負載平衡器後方有 Azure 轉送連接的 Vm 集區。

### <a name="manageability"></a>管理能力

此解決方案可能會跨越許多裝置和位置而變得不好管理。 Azure 的 IoT 服務可讓新的位置與裝置自動上線，並讓其保持最新狀態。

### <a name="security"></a>安全性

這種模式可讓您從邊緣對內部裝置上的連接埠進行自由存取。 請考慮為內部裝置上的服務，或在混合式轉送端點前方新增驗證機制。

## <a name="next-steps"></a>後續步驟

深入了解此文章介紹的更多相關主題：

- 此模式會使用 Azure 轉送。 如需詳細資訊，請參閱[Azure 轉送檔](/azure/azure-relay/)。
- 若要深入了解最佳做法並獲得任何其他問題的解答，請參閱[混合式應用程式設計考量](overview-app-design-considerations.md)。
- 若要深入了解產品與解決方案的完整組合，請參閱 [Azure Stack 系列產品和解決方案](/azure-stack)。

當您準備好測試解決方案範例時，請繼續參閱[混合式轉送解決方案部署指南](https://aka.ms/hybridrelaydeployment)。 部署指南提供部署及測試其元件的逐步指示。