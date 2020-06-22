---
title: 適用於 Azure 和 Azure Stack Hub 的混合式模式和解決方案範例
description: 概述混合式模式和解決方案範例，用來在 Azure 和 Azure Stack Hub 上學習及建置混合式解決方案。
author: BryanLa
ms.topic: overview
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ab0eb885e7b0fefaca8991522712652f979d8712
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910062"
---
# <a name="hybrid-patterns-and-solution-examples-for-azure-and-azure-stack"></a>適用於 Azure 和 Azure Stack 的混合式模式和解決方案範例

Microsoft 會以單一且一致的 Azure 生態系統形式來提供 Azure 和 Azure Stack 產品與解決方案。 Microsoft Azure Stack 系列是 Azure 的擴充功能。

## <a name="the-hybrid-cloud-and-hybrid-apps"></a>混合式雲端和混合式應用程式

Azure Stack 藉由實現「混合式雲端」  ，將雲端運算的靈活度帶到內部部署環境和邊緣。 Azure Stack Hub、Azure Stack HCI 和 Azure Stack Edge 會將 Azure 從雲端延伸至您的主權資料中心、分公司、現場和更遠的範圍。 使用這組多樣化的功能，您將可以：

- 重複使用程式碼並在 Azure 與內部部署環境中一致地執行雲端原生應用程式。
- 使用對 Azure 服務的選擇性連線來執行傳統的虛擬化工作負載。
- 將資料傳輸至雲端，或將保留在主權資料中心以維持合規性。
- 執行硬體加速機器學習、容器化或虛擬化的工作負載，全都在智慧邊緣進行。

跨雲端應用程式也稱為*混合式應用程式*。 您可以在 Azure 中建置混合式雲端應用程式，並將其部署到任何位置的已連線或中斷連線的資料中心。

混合式應用程式案例會隨著可供開發的資源而有很大的變化。 這些案例的注意事項也會跨及地理位置、安全性、網際網路存取等方面。 雖然這裡所說的模式和解決方案可能無法因應所有需求，但卻可以提供指導方針和範例讓您探究並供您在實作混合式解決方案時重複使用。

## <a name="design-patterns"></a>設計模式

設計模式會從真實的客戶案例和經驗中，挑選出普遍且可重複的設計指引。 所擷取出來的模式可適用於不同類型的案例或垂直產業。 每個模式都會記載內容和問題，並提供解決方案範例的概觀。 解決方案範例的本意是要作為模式的可能實作。

模式文章有兩種類型：

- 單一模式：提供一般用途單一案例的設計指引。
- 多重模式：提供應用時會使用多個模式的設計指引。 為了解決更複雜的案例或產業特有問題，經常會需要使用這種模式。

## <a name="solution-deployment-guides"></a>解決方案部署指南

逐步部署指南可協助您部署解決方案範例。 此指南可能也會參考隨附的程式碼範例，期儲存位置在 GitHub 的[解決方案範例存放庫](https://github.com/Azure-Samples/azure-intelligent-edge-patterns)中。

## <a name="next-steps"></a>後續步驟

- 若要深入了解產品與解決方案的完整組合，請參閱 [Azure Stack 系列產品和解決方案](/azure-stack)。
- 探索目錄中的＜模式＞和＜解決方案部署指南＞章節，以分別深入了解。
- 閱讀[混合式應用程式設計考量](overview-app-design-considerations.md)，以檢閱設計、部署及操作混合式應用程式的軟體品質要素。
- [在 Azure Stack 上設定開發環境](/azure-stack/user/azure-stack-dev-start.md)並在 Azure Stack 上[部署第一個應用程式](/azure-stack/user/azure-stack-dev-start-deploy-app.md)。
