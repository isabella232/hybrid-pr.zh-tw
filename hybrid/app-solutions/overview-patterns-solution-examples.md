---
title: 適用于 Azure 和 Azure Stack Hub 的混合式模式和解決方案範例
description: 介紹在 Azure 上學習和建立混合式解決方案的混合式模式和解決方案範例，並 Azure Stack Hub。
author: BryanLa
ms.topic: overview
ms.date: 05/24/2021
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 05/24/2021
ms.openlocfilehash: 9f3f13c23bec31c5132c7e90294356b9463fd72b
ms.sourcegitcommit: cf2c4033d1b169f5b63980ce1865281366905e2e
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/25/2021
ms.locfileid: "110343853"
---
# <a name="hybrid-solution-patterns-and-examples-for-azure-and-azure-stack"></a>適用于 Azure 和 Azure Stack 的混合式解決方案模式和範例

Microsoft 會以單一且一致的 Azure 生態系統形式來提供 Azure 和 Azure Stack 產品與解決方案。 Microsoft Azure Stack 系列是 Azure 的擴充功能。

## <a name="the-hybrid-cloud-and-hybrid-apps"></a>混合式雲端和混合式應用程式

Azure Stack 藉由啟用 *混合式雲端*，將雲端運算的靈活性帶入您的內部部署環境和邊緣。 Azure Stack Hub、Azure Stack HCI 和 Azure Stack Edge 會將 Azure 從雲端延伸至您的主權資料中心、分公司、現場和更遠的範圍。 使用這組多樣化的功能，您將可以：

- 重複使用程式碼並在 Azure 與內部部署環境中一致地執行雲端原生應用程式。
- 使用對 Azure 服務的選擇性連線來執行傳統的虛擬化工作負載。
- 將資料傳輸至雲端，或將保留在主權資料中心以維持合規性。
- 執行硬體加速機器學習、容器化或虛擬化的工作負載，全都在智慧邊緣進行。

跨雲端的應用程式也稱為混合式 *應用程式*。 您可以在 Azure 中建置混合式雲端應用程式，並將其部署到任何位置的已連線或中斷連線的資料中心。

混合式應用程式案例與可供開發的資源有很大的差異。 它們也會涵蓋地理位置、安全性、網際網路存取等考慮。 雖然此處所述的解決方案模式和範例可能無法解決所有需求，但它們會提供在執行混合式解決方案時探索和重複使用的指導方針和範例。

## <a name="solution-patterns"></a>解決方案模式

解決方案模式可從真實世界的客戶案例和經驗中挑選一般化可重複的設計指引。 所擷取出來的模式可適用於不同類型的案例或垂直產業。 每個模式都會記載內容和問題，並提供解決方案範例的概觀。 解決方案範例的本意是要作為模式的可能實作。

模式文章有兩種類型：

- 單一模式：提供一般用途單一案例的設計指引。
- 多重模式：提供應用時會使用多個模式的設計指引。 解決更複雜的案例或產業特定的問題時，通常需要此模式。

## <a name="solution-deployment-guides"></a>解決方案部署指南

逐步部署指南可協助您部署解決方案範例。 此指南可能也會參考隨附的程式碼範例，期儲存位置在 GitHub 的[解決方案範例存放庫](https://github.com/Azure-Samples/azure-intelligent-edge-patterns)中。

## <a name="next-steps"></a>下一步

- 若要深入了解產品與解決方案的完整組合，請參閱 [Azure Stack 系列產品和解決方案](/azure-stack)。
- 探索目錄的「模式」和「解決方案部署指南」章節，以深入瞭解各項。
- 閱讀 [混合式應用程式設計考慮](overview-app-design-considerations.md) ，以複習設計、部署及操作混合式應用程式的軟體品質要素。
- [在 Azure Stack 上設定開發環境](/azure-stack/user/azure-stack-dev-start)並在 Azure Stack 上[部署第一個應用程式](/azure-stack/user/azure-stack-dev-start-deploy-app)。
