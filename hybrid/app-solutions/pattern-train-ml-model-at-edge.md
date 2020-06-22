---
title: 以邊緣模式定型機器學習模型
description: 了解如何使用 Azure 和 Azure Stack Hub，在邊緣進行機器學習模型定型。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: da1012cc8847f221de6bb540ba4e191e43920f41
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910114"
---
# <a name="train-machine-learning-model-at-the-edge-pattern"></a>以邊緣模式定型機器學習模型

從僅存在於內部部署的資料產生可攜式機器學習 (ML) 模型。

## <a name="context-and-problem"></a>內容和問題

許多組織希望使用其資料科學家了解的工具，從其內部部署或舊版資料中獲取見解。 [Azure Machine Learning](/azure/machine-learning/) 提供的雲端原生工具可定型、調整及部署 ML 和深度學習模型。  

不過，某些資料太大，無法傳送至雲端，或礙於法規因素而無法傳送至雲端。 使用此模式時，資料科學家可以使用 Azure Machine Learning，利用內部部署資料和計算來定型模型。

## <a name="solution"></a>解決方法

邊緣模式的訓練會使用在 Azure Stack Hub 上執行的虛擬機器 (VM)。 VM 已在 Azure ML 中註冊為計算目標，使其能存取僅適用於內部部署的資料。 在此情況下，資料會儲存在 Azure Stack Hub 的 Blob 儲存體中。

模型定型之後，便會向 Azure ML 註冊、對其進行容器化，並將其新增至 Azure Container Registry 以進行部署。 若要進行此模式的反覆項目，必須可透過公用網際網路連線至 Azure Stack Hub 訓練 VM。

[![在邊緣架構定型 ML 模型](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)

此模式的運作方式如下：

1. 已部署 Azure Stack Hub VM，並透過 Azure ML 註冊為計算目標。
2. 在 Azure ML 中建立了一個實驗，該實驗使用 Azure Stack Hub VM 作為計算目標。
3. 模型定型之後，會進行註冊並容器化。
4. 現在可以將模型部署到內部部署或雲端中的位置。

## <a name="components"></a>元件

此解決方案使用下列元件：

| 階層 | 元件 | 描述 |
|----------|-----------|-------------|
| Azure | Azure Machine Learning | [Azure Machine Learning](/azure/machine-learning/) 可協調 ML 模型的訓練。 |
| | Azure Container Registry | Azure ML 會將模型封裝到容器中，並將其儲存在用於部署的 [Azure Container Registry](/azure/container-registry/) 中。|
| Azure Stack Hub | App Service 方案 | [具有 App Service 的 Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) 可為邊緣的元件提供基礎映像。 |
| | 計算 | 使用 Docker 執行 Ubuntu 的 Azure Stack Hub VM 可用於定型 ML 模型。 |
| | 儲存體 | 私人資料可以裝載在 Azure Stack Hub Blob 儲存體中。 |

## <a name="issues-and-considerations"></a>問題和考量

決定如何實作此解決方案時，請考慮下列幾點：

### <a name="scalability"></a>延展性

為了讓這項解決方案得以擴展，您必須在 Azure Stack Hub 上建立適當大小的 VM 以進行訓練。

### <a name="availability"></a>可用性

請確定訓練指令碼和 Azure Stack Hub VM 可以存取用於訓練的內部部署資料。

### <a name="manageability"></a>管理能力

請確定模型和實驗已進行適當的註冊、建立版本，並加上標籤，以避免在模型部署期間造成混淆。

### <a name="security"></a>安全性

此模式可讓 Azure ML 存取可能的內部部署敏感性資料。 請確定用來透過 SSH 連線至 Azure Stack Hub VM 的帳戶具有強式密碼，且訓練指令碼不會保留資料或將資料上傳到雲端。

## <a name="next-steps"></a>後續步驟

深入了解此文章介紹的更多相關主題：

- 如需 ML 和相關主題的概觀，請參閱 [Azure Machine Learning 說明文件](/azure/machine-learning)。
- 若要了解如何組建、儲存和管理容器部署的映像，請參閱 [Azure Container Registry](/azure/container-registry/)。
- 若要深入了解資源提供者和如何部署的詳細資訊，請參閱 [Azure Stack Hub 上的 App Service](/azure-stack/operator/azure-stack-app-service-overview)。
- 若要深入了解最佳做法並獲得任何其他問題的解答，請參閱[混合式應用程式設計考量](overview-app-design-considerations.md)。
- 若要深入了解產品與解決方案的完整組合，請參閱 [Azure Stack 系列產品和解決方案](/azure-stack)。

當您準備好測試解決方案範例時，請繼續參閱[在邊緣定型 ML 模型部署指南](https://aka.ms/edgetrainingdeploy)。 部署指南提供部署及測試其元件的逐步指示。
