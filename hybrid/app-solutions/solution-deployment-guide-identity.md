---
title: 為 Azure 和 Azure Stack Hub 應用程式設定混合式雲端身分識別
description: 了解如何為 Azure 和 Azure Stack Hub 應用程式設定混合式雲端身分識別。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 650eef0f144ecafab4586d93f72e1defdf4a61ce
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477247"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a>為 Azure 和 Azure Stack Hub 應用程式設定混合式雲端身分識別

了解如何為 Azure 和 Azure Stack Hub 應用程式設定混合式雲端身分識別。

在全域 Azure 和 Azure Stack Hub 中，您都有兩個用來授與應用程式存取權的選項。

 * 當 Azure Stack Hub 可持續連線至網際網路時，您可以使用 Azure Active Directory (Azure AD)。
 * 當 Azure Stack Hub 的網際網路連線中斷時，您可以使用 Azure Directory 同盟服務 (AD FS)。

您可以使用服務主體來授與 Azure Stack Hub 應用程式存取權，以便在 Azure Stack Hub 中使用 Azure Resource Manager 進行部署或設定。

在本解決方案中，您會建置環境範例，用以：

> [!div class="checklist"]
> - 在全域 Azure 與 Azure Stack Hub 中建立混合式身分識別
> - 擷取用以存取 Azure Stack Hub API 的權杖。

您必須具有 Azure Stack Hub 操作員權限，才能執行本解決方案中的步驟。

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub 是 Azure 的延伸模組。 Azure Stack Hub 可將雲端運算的靈活性和創新能力導入您的內部部署環境中，並啟用獨特的混合式雲端，讓您能夠隨處建置及部署混合式應用程式。  
> 
> [混合式應用程式設計考量](overview-app-design-considerations.md)一文檢閱了設計、部署和操作混合式應用程式時的軟體品質要素 (放置、延展性、可用性、復原、管理性和安全性)。 這些設計考量有助於您設計出最佳的混合式應用程式，減少生產環境可能會遇到的挑戰。

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a>在入口網站中為 Azure AD 建立服務主體

如果您使用 Azure AD 作為身分識別存放區部署了 Azure Stack Hub，則可以如同對 Azure 般建立服務主體。 [使用應用程式身分識別來存取資源](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity)會說明如何透過入口網站逐步執行。 請確認您具備[必要的 Azure AD 權限](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions)。

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a>使用 PowerShell 為 AD FS 建立服務主體

如果您已使用 AD FS 部署 Azure Stack Hub，則可以使用 PowerShell 來建立服務主體、指派用於存取的角色，以及從 PowerShell 使用該身分識別登入。 [使用應用程式身分識別來存取資源](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity)會說明如何使用 PowerShell 逐步執行必要步驟。

## <a name="using-the-azure-stack-hub-api"></a>使用 Azure Stack Hub API

[Azure Stack Hub API](/azure-stack/user/azure-stack-rest-api-use.md) 解決方案會逐步解說擷取權杖以存取 Azure Stack Hub API 的程序。

## <a name="connect-to-azure-stack-hub-using-powershell"></a>使用 PowerShell 連線至 Azure Stack Hub

[在 Azure Stack Hub 中使用 PowerShell 啟動並執行](/azure-stack/operator/azure-stack-powershell-install.md)快速入門會引導您完成安裝 Azure PowerShell 並連線至 Azure Stack Hub 安裝所需的步驟。

### <a name="prerequisites"></a>Prerequisites

您需要使用您可以存取的訂用帳戶連線至 Azure AD 的 Azure Stack 安裝。 如果您沒有 Azure Stack Hub 安裝，您可以依照這些指示設定 [Azure Stack 開發套件 (ASDK)](/azure-stack/asdk/asdk-install.md)。

#### <a name="connect-to-azure-stack-hub-using-code"></a>使用程式碼連線至 Azure Stack Hub

若要使用程式碼連線至 Azure Stack Hub，請使用 Azure Resource Manager 端點 API 取得 Azure Stack Hub 安裝的驗證和圖形端點。 然後，使用 REST 要求進行驗證。 您可以在 [GitHub](https://github.com/shriramnat/HybridARMApplication) 找到範例用戶端應用程式。

>[!Note]
>除非您所選語言的 Azure SDK 支援 Azure API 設定檔，否則 SDK 可能無法與 Azure Stack Hub 搭配使用。 若要深入了解 Azure API 設定檔，請參閱[管理 API 版本設定檔](/azure-stack/user/azure-stack-version-profiles.md)一文。

## <a name="next-steps"></a>後續步驟

- 若要深入了解在 Azure Stack Hub 處理身分識別的方式，請參閱 [Azure Stack Hub 的身分識別架構](/azure-stack/operator/azure-stack-identity-architecture.md)。
- 若要深入了解 Azure 雲端模式，請參閱[雲端設計模式](/azure/architecture/patterns)。
