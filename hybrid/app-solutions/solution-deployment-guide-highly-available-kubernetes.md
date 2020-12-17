---
title: 在 Azure Stack Hub 上部署高可用性 Kubernetes 叢集
description: 了解如何使用 Azure 和 Azure Stack Hub 部署 Kubernetes 叢集來提供高可用性。
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: 91f5856aa670bf3810baa5e5f07dbb7dafc9e3f3
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/09/2020
ms.locfileid: "96911919"
---
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a><span data-ttu-id="fb17f-103">在 Azure Stack Hub 上部署高可用性 Kubernetes 叢集</span><span class="sxs-lookup"><span data-stu-id="fb17f-103">Deploy a high availability Kubernetes cluster on Azure Stack Hub</span></span>

<span data-ttu-id="fb17f-104">本文將說明如何在不同的實體位置中，建置高可用性且部署在多個 Azure Stack Hub 執行個體上的 Kubernetes 叢集環境。</span><span class="sxs-lookup"><span data-stu-id="fb17f-104">This article will show you how to build a highly available Kubernetes cluster environment, deployed on multiple Azure Stack Hub instances, in different physical locations.</span></span>

<span data-ttu-id="fb17f-105">在此解決方案部署指南中，您將了解如何：</span><span class="sxs-lookup"><span data-stu-id="fb17f-105">In this solution deployment guide, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="fb17f-106">下載並準備 AKS 引擎</span><span class="sxs-lookup"><span data-stu-id="fb17f-106">Download and prepare the AKS Engine</span></span>
> - <span data-ttu-id="fb17f-107">連線到 AKS 引擎協助程式 VM</span><span class="sxs-lookup"><span data-stu-id="fb17f-107">Connect to the AKS Engine Helper VM</span></span>
> - <span data-ttu-id="fb17f-108">部署 Kubernetes 叢集</span><span class="sxs-lookup"><span data-stu-id="fb17f-108">Deploy a Kubernetes cluster</span></span>
> - <span data-ttu-id="fb17f-109">連線至 Kubernetes 叢集</span><span class="sxs-lookup"><span data-stu-id="fb17f-109">Connect to the Kubernetes cluster</span></span>
> - <span data-ttu-id="fb17f-110">將 Azure Pipelines 連線至 Kubernetes 叢集</span><span class="sxs-lookup"><span data-stu-id="fb17f-110">Connect Azure Pipelines to Kubernetes cluster</span></span>
> - <span data-ttu-id="fb17f-111">設定監視</span><span class="sxs-lookup"><span data-stu-id="fb17f-111">Configure monitoring</span></span>
> - <span data-ttu-id="fb17f-112">部署應用程式</span><span class="sxs-lookup"><span data-stu-id="fb17f-112">Deploy application</span></span>
> - <span data-ttu-id="fb17f-113">自動調整應用程式</span><span class="sxs-lookup"><span data-stu-id="fb17f-113">Autoscale application</span></span>
> - <span data-ttu-id="fb17f-114">設定流量管理員</span><span class="sxs-lookup"><span data-stu-id="fb17f-114">Configure Traffic Manager</span></span>
> - <span data-ttu-id="fb17f-115">升級 Kubernetes</span><span class="sxs-lookup"><span data-stu-id="fb17f-115">Upgrade Kubernetes</span></span>
> - <span data-ttu-id="fb17f-116">調整 Kubernetes</span><span class="sxs-lookup"><span data-stu-id="fb17f-116">Scale Kubernetes</span></span>

> [!Tip]  
> <span data-ttu-id="fb17f-117">![混合式支柱](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="fb17f-117">![Hybrid pillars](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="fb17f-118">Microsoft Azure Stack Hub 是 Azure 的延伸模組。</span><span class="sxs-lookup"><span data-stu-id="fb17f-118">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="fb17f-119">Azure Stack Hub 可將雲端運算的靈活性與創新能力導入您的內部部署環境中，並啟用獨特的混合式雲端，讓您能夠隨處建置及部署混合式應用程式。</span><span class="sxs-lookup"><span data-stu-id="fb17f-119">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="fb17f-120">[混合式應用程式設計考量](overview-app-design-considerations.md)一文檢閱了設計、部署和操作混合式應用程式時的軟體品質要素 (放置、延展性、可用性、復原、管理性和安全性)。</span><span class="sxs-lookup"><span data-stu-id="fb17f-120">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="fb17f-121">這些設計考量有助於您設計出最佳的混合式應用程式，減少生產環境可能會遇到的挑戰。</span><span class="sxs-lookup"><span data-stu-id="fb17f-121">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="fb17f-122">Prerequisites</span><span class="sxs-lookup"><span data-stu-id="fb17f-122">Prerequisites</span></span>

<span data-ttu-id="fb17f-123">開始使用此部署指南之前，請務必先：</span><span class="sxs-lookup"><span data-stu-id="fb17f-123">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="fb17f-124">請參閱[高可用性 Kubernetes 叢集模式](pattern-highly-available-kubernetes.md)一文。</span><span class="sxs-lookup"><span data-stu-id="fb17f-124">Review the [High availability Kubernetes cluster pattern](pattern-highly-available-kubernetes.md) article.</span></span>
- <span data-ttu-id="fb17f-125">請參閱[附屬 GitHub 存放庫](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub)的內容，其中包含本文中所參考的其他資產。</span><span class="sxs-lookup"><span data-stu-id="fb17f-125">Review the contents of the [companion GitHub repository](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), which contains additional assets referenced in this article.</span></span>
- <span data-ttu-id="fb17f-126">擁有可存取 [Azure Stack Hub 使用者入口網站](/azure-stack/user/azure-stack-use-portal)的帳戶，且至少具有[「參與者」權限](/azure-stack/user/azure-stack-manage-permissions)。</span><span class="sxs-lookup"><span data-stu-id="fb17f-126">Have an account that can access the [Azure Stack Hub user portal](/azure-stack/user/azure-stack-use-portal), with at least ["contributor" permissions](/azure-stack/user/azure-stack-manage-permissions).</span></span>

## <a name="download-and-prepare-aks-engine"></a><span data-ttu-id="fb17f-127">下載並準備 AKS 引擎</span><span class="sxs-lookup"><span data-stu-id="fb17f-127">Download and prepare AKS Engine</span></span>

<span data-ttu-id="fb17f-128">AKS 引擎是一個二進位檔，可以從任何可連線至 Azure Stack Hub Azure Resource Manager 端點的 Windows 或 Linux 主機中使用。</span><span class="sxs-lookup"><span data-stu-id="fb17f-128">AKS Engine is a binary that can be used from any Windows or Linux host that can reach the Azure Stack Hub Azure Resource Manager endpoints.</span></span> <span data-ttu-id="fb17f-129">本指南說明如何在 Azure Stack Hub 上部署新的 Linux (或 Windows) VM。</span><span class="sxs-lookup"><span data-stu-id="fb17f-129">This guide describes deploying a new Linux (or Windows) VM on Azure Stack Hub.</span></span> <span data-ttu-id="fb17f-130">稍後會在 AKS 引擎部署 Kubernetes 叢集時用到。</span><span class="sxs-lookup"><span data-stu-id="fb17f-130">It will be used later when AKS Engine deploys the Kubernetes clusters.</span></span>

> [!NOTE]
> <span data-ttu-id="fb17f-131">您也可以在現有的 Windows 或 Linux VM 上，使用 AKS 引擎在 Azure Stack Hub 上部署 Kubernetes 叢集。</span><span class="sxs-lookup"><span data-stu-id="fb17f-131">You can also use an existing Windows or Linux VM to deploy a Kubernetes cluster on Azure Stack Hub using AKS Engine.</span></span>

<span data-ttu-id="fb17f-132">AKS 引擎的逐步程序和需求記載於此處：</span><span class="sxs-lookup"><span data-stu-id="fb17f-132">The step-by-step process and requirements for AKS Engine are documented here:</span></span>

* <span data-ttu-id="fb17f-133">[在 Azure Stack Hub 中的 Linux 上安裝 AKS 引擎](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (或使用 [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))</span><span class="sxs-lookup"><span data-stu-id="fb17f-133">[Install the AKS Engine on Linux in Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (or using [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))</span></span>

<span data-ttu-id="fb17f-134">AKS 引擎是一個協助程式工具，可用於部署和操作 (非受控) Kubernetes 叢集 (在 Azure 和 Azure Stack Hub 中)。</span><span class="sxs-lookup"><span data-stu-id="fb17f-134">AKS Engine is a helper tool to deploy and operate (unmanaged) Kubernetes clusters (in Azure and Azure Stack Hub).</span></span>

<span data-ttu-id="fb17f-135">如需 Azure Stack Hub 上的 AKS 引擎詳細資料和差異，請參閱：</span><span class="sxs-lookup"><span data-stu-id="fb17f-135">The details and differences of AKS Engine on Azure Stack Hub are described here:</span></span>

* [<span data-ttu-id="fb17f-136">Azure Stack Hub 上的 AKS 引擎是什麼？</span><span class="sxs-lookup"><span data-stu-id="fb17f-136">What is the AKS Engine on Azure Stack Hub?</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* <span data-ttu-id="fb17f-137">[Azure Stack Hub 上的 AKS 引擎](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (GitHub 上)</span><span class="sxs-lookup"><span data-stu-id="fb17f-137">[AKS Engine on Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (on GitHub)</span></span>

<span data-ttu-id="fb17f-138">範例環境會使用 Terraform 將 AKS 引擎 VM 的部署自動化。</span><span class="sxs-lookup"><span data-stu-id="fb17f-138">The sample environment will use Terraform to automate the deployment of the AKS Engine VM.</span></span> <span data-ttu-id="fb17f-139">您可以在[附屬的 GitHub 存放庫中找到詳細資料和程式碼](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md)。</span><span class="sxs-lookup"><span data-stu-id="fb17f-139">You can find the [details and code in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).</span></span>

<span data-ttu-id="fb17f-140">此步驟會在 Azure Stack Hub 上產生新的資源群組，且資源群組中會包含 AKS 引擎協助程式 VM 和相關資源：</span><span class="sxs-lookup"><span data-stu-id="fb17f-140">The result of this step is a new resource group on Azure Stack Hub that contains the AKS Engine helper VM and related resources:</span></span>

![Azure Stack Hub 中的 AKS 引擎 VM 資源](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> <span data-ttu-id="fb17f-142">如果您必須在中斷連線的網路隔絕環境中部署 AKS 引擎，請參閱[中斷連線的 Azure Stack Hub 執行個體](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances)以深入了解。</span><span class="sxs-lookup"><span data-stu-id="fb17f-142">If you have to deploy AKS Engine in a disconnected air-gapped environment, review [Disconnected Azure Stack Hub Instances](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) to learn more.</span></span>

<span data-ttu-id="fb17f-143">在下一個步驟中，我們將使用新部署的 AKS 引擎 VM 來部署 Kubernetes 叢集。</span><span class="sxs-lookup"><span data-stu-id="fb17f-143">In the next step, we'll use the newly deployed AKS Engine VM to deploy a Kubernetes cluster.</span></span>

## <a name="connect-to-the-aks-engine-helper-vm"></a><span data-ttu-id="fb17f-144">連線到 AKS 引擎協助程式 VM</span><span class="sxs-lookup"><span data-stu-id="fb17f-144">Connect to the AKS Engine helper VM</span></span>

<span data-ttu-id="fb17f-145">首先，您必須連線到先前建立的 AKS 引擎協助程式 VM。</span><span class="sxs-lookup"><span data-stu-id="fb17f-145">First you must connect to the previously created AKS Engine helper VM.</span></span>

<span data-ttu-id="fb17f-146">VM 應具有公用 IP 位址，而且應該可透過 SSH (連接埠 22/TCP) 存取。</span><span class="sxs-lookup"><span data-stu-id="fb17f-146">The VM should have a Public IP Address and should be accessible via SSH (Port 22/TCP).</span></span>

![AKS 引擎 VM 概觀頁面](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> <span data-ttu-id="fb17f-148">您可以使用您選擇的工具 (例如 Windows 10 中的 MobaXterm、puTTY 或 PowerShell)，以使用 SSH 連線到 Linux VM。</span><span class="sxs-lookup"><span data-stu-id="fb17f-148">You can use a tool of your choice like MobaXterm, puTTY or PowerShell in Windows 10 to connect to a Linux VM using SSH.</span></span>

```console
ssh <username>@<ipaddress>
```

<span data-ttu-id="fb17f-149">連線之後，請執行 `aks-engine` 命令。</span><span class="sxs-lookup"><span data-stu-id="fb17f-149">After connecting, run the command `aks-engine`.</span></span> <span data-ttu-id="fb17f-150">若要深入了解 AKS 引擎和 Kubernetes 版本，請移至[支援的 AKS 引擎版本](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions)。</span><span class="sxs-lookup"><span data-stu-id="fb17f-150">Go to [Supported AKS Engine Versions](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) to learn more about the AKS Engine and Kubernetes versions.</span></span>

![aks-engine 命令列範例](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a><span data-ttu-id="fb17f-152">部署 Kubernetes 叢集</span><span class="sxs-lookup"><span data-stu-id="fb17f-152">Deploy a Kubernetes cluster</span></span>

<span data-ttu-id="fb17f-153">AKS 引擎協助程式 VM 本身尚未在我們的 Azure Stack Hub 上建立 Kubernetes 叢集。</span><span class="sxs-lookup"><span data-stu-id="fb17f-153">The AKS Engine helper VM itself hasn't created a Kubernetes cluster on our Azure Stack Hub, yet.</span></span> <span data-ttu-id="fb17f-154">建立叢集是要在 AKS 引擎協助程式 VM 中採取的第一個動作。</span><span class="sxs-lookup"><span data-stu-id="fb17f-154">Creating the cluster is the first action to take in the AKS Engine helper VM.</span></span>

<span data-ttu-id="fb17f-155">逐步程序記載於此處：</span><span class="sxs-lookup"><span data-stu-id="fb17f-155">The step-by-step process is documented here:</span></span>

* [<span data-ttu-id="fb17f-156">在 Azure Stack Hub 上使用 AKS 引擎部署 Kubernetes 叢集</span><span class="sxs-lookup"><span data-stu-id="fb17f-156">Deploy a Kubernetes cluster with the AKS engine on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

<span data-ttu-id="fb17f-157">`aks-engine deploy` 命令和先前步驟中準備工作的最終結果是功能完整的 Kubernetes 叢集，並且會部署到第一個 Azure Stack Hub 執行個體的租用戶空間。</span><span class="sxs-lookup"><span data-stu-id="fb17f-157">The end result of the `aks-engine deploy` command and the preparations in the previous steps is a fully featured Kubernetes cluster deployed into the tenant space of the first Azure Stack Hub instance.</span></span> <span data-ttu-id="fb17f-158">叢集本身包含 Azure IaaS 元件，例如 VM、負載平衡器、Vnet、磁碟等等。</span><span class="sxs-lookup"><span data-stu-id="fb17f-158">The cluster itself consists of Azure IaaS components like VMs, load balancers, VNets, disks, and so on.</span></span>

![Azure Stack Hub 入口網站的叢集 IaaS 元件](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) <span data-ttu-id="fb17f-160">Azure 負載平衡器 (K8s API 端點)</span><span class="sxs-lookup"><span data-stu-id="fb17f-160">Azure load balancer (K8s API Endpoint)</span></span>
2) <span data-ttu-id="fb17f-161">背景工作節點 (代理程式集區)</span><span class="sxs-lookup"><span data-stu-id="fb17f-161">Worker Nodes (Agent Pool)</span></span>
3) <span data-ttu-id="fb17f-162">主要節點</span><span class="sxs-lookup"><span data-stu-id="fb17f-162">Master Nodes</span></span>

<span data-ttu-id="fb17f-163">叢集現在已啟動且正在執行，而且在下一個步驟中，我們將連線到該叢集。</span><span class="sxs-lookup"><span data-stu-id="fb17f-163">The cluster is now up-and-running and in the next step we'll connect to it.</span></span>

## <a name="connect-to-the-kubernetes-cluster"></a><span data-ttu-id="fb17f-164">連線至 Kubernetes 叢集</span><span class="sxs-lookup"><span data-stu-id="fb17f-164">Connect to the Kubernetes cluster</span></span>

<span data-ttu-id="fb17f-165">您現在可以透過 SSH 連線到先前建立的 Kubernetes 叢集 (使用部署中指定的 SSH 金鑰) 或透過 `kubectl` (建議)。</span><span class="sxs-lookup"><span data-stu-id="fb17f-165">You can now connect to the previously created Kubernetes cluster, either via SSH (using the SSH key specified as part of the deployment) or via `kubectl` (recommended).</span></span> <span data-ttu-id="fb17f-166">若要了解適用於 Windows、Linux 和 macOS 的 Kubernetes 命令列工具，請參閱`kubectl`[此處](https://kubernetes.io/docs/tasks/tools/install-kubectl/)。</span><span class="sxs-lookup"><span data-stu-id="fb17f-166">The Kubernetes command-line tool `kubectl` is available for Windows, Linux, and macOS [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).</span></span> <span data-ttu-id="fb17f-167">其已在叢集的主要節點上預先安裝並設定。</span><span class="sxs-lookup"><span data-stu-id="fb17f-167">It's already pre-installed and configured on the master nodes of our cluster.</span></span>

```console
ssh azureuser@<k8s-master-lb-ip>
```

![在主要節點上執行 kubectl](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

<span data-ttu-id="fb17f-169">不建議使用主要節點作為系統管理工作的 jumpbox。</span><span class="sxs-lookup"><span data-stu-id="fb17f-169">It's not recommended to use the master node as a jumpbox for administrative tasks.</span></span> <span data-ttu-id="fb17f-170">`kubectl` 設定會儲存在主要節點和 AKS 引擎 VM 上的 `.kube/config` 中。</span><span class="sxs-lookup"><span data-stu-id="fb17f-170">The `kubectl` configuration is stored in `.kube/config` on the master node(s) as well as on the AKS Engine VM.</span></span> <span data-ttu-id="fb17f-171">您可以將設定複製到具有 Kubernetes 叢集連線的管理員機器，並在該處使用 `kubectl` 命令。</span><span class="sxs-lookup"><span data-stu-id="fb17f-171">You can copy the configuration to an admin machine with connectivity to the Kubernetes cluster and use the `kubectl` command there.</span></span> <span data-ttu-id="fb17f-172">`.kube/config` 檔案稍後也會用來設定 Azure Pipelines 中的服務連線。</span><span class="sxs-lookup"><span data-stu-id="fb17f-172">The `.kube/config` file is also used later to configure a service connection in Azure Pipelines.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="fb17f-173">保護這些檔案的安全，因為其中包含 Kubernetes 叢集的認證。</span><span class="sxs-lookup"><span data-stu-id="fb17f-173">Keep these files secure because they contain the credentials for your Kubernetes cluster.</span></span> <span data-ttu-id="fb17f-174">具有檔案存取權的攻擊者會具有足夠的資訊來取得其管理員存取權。</span><span class="sxs-lookup"><span data-stu-id="fb17f-174">An attacker with access to the file has enough information to gain administrator access to it.</span></span> <span data-ttu-id="fb17f-175">使用初始 `.kube/config` 檔案完成的所有動作都是使用叢集管理員帳戶完成。</span><span class="sxs-lookup"><span data-stu-id="fb17f-175">All actions that are done using the initial `.kube/config` file are done using a cluster-admin account.</span></span>

<span data-ttu-id="fb17f-176">您現在可以使用 `kubectl` 來嘗試各種命令，以檢查叢集的狀態。</span><span class="sxs-lookup"><span data-stu-id="fb17f-176">You can now try various commands using `kubectl` to check the status of your cluster.</span></span>

```bash
kubectl get nodes
```

```console
NAME                       STATUS   ROLE     VERSION
k8s-linuxpool-35064155-0   Ready    agent    v1.14.8
k8s-linuxpool-35064155-1   Ready    agent    v1.14.8
k8s-linuxpool-35064155-2   Ready    agent    v1.14.8
k8s-master-35064155-0      Ready    master   v1.14.8
```

```bash
kubectl cluster-info
```

```console
Kubernetes master is running at https://aks.***
CoreDNS is running at https://aks.**_/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
kubernetes-dashboard is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy
Metrics-server is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
```

> [!IMPORTANT]
> <span data-ttu-id="fb17f-177">Kubernetes 有自己的「角色型存取控制 (RBAC)」模型，可讓您建立更細緻的角色定義和角色繫結。</span><span class="sxs-lookup"><span data-stu-id="fb17f-177">Kubernetes has its own _ *Role-based Access Control (RBAC)*\* model that allows you to create fine-grained role definitions and role bindings.</span></span> <span data-ttu-id="fb17f-178">比起給予叢集管理員權限，這是較理想的叢集存取控制方式。</span><span class="sxs-lookup"><span data-stu-id="fb17f-178">This is the preferable way to control access to the cluster instead of handing out cluster-admin permissions.</span></span>

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a><span data-ttu-id="fb17f-179">將 Azure Pipelines 連線至 Kubernetes 叢集</span><span class="sxs-lookup"><span data-stu-id="fb17f-179">Connect Azure Pipelines to Kubernetes clusters</span></span>

<span data-ttu-id="fb17f-180">若要將 Azure Pipelines 連線到新部署的 Kubernetes 叢集，我們需要其 kube config (`.kube/config`) 檔案，如上一個步驟所述。</span><span class="sxs-lookup"><span data-stu-id="fb17f-180">To connect Azure Pipelines to the newly deployed Kubernetes cluster, we need its kube config (`.kube/config`) file as explained in the previous step.</span></span>

* <span data-ttu-id="fb17f-181">連線到 Kubernetes 叢集的其中一個主要節點。</span><span class="sxs-lookup"><span data-stu-id="fb17f-181">Connect to one of the master nodes of your Kubernetes cluster.</span></span>
* <span data-ttu-id="fb17f-182">複製 `.kube/config` 檔案的內容。</span><span class="sxs-lookup"><span data-stu-id="fb17f-182">Copy the content of the `.kube/config` file.</span></span>
* <span data-ttu-id="fb17f-183">移至 [Azure DevOps] > [專案設定] > [服務連線]，以建立新的 "Kubernetes" 服務連線 (使用 KubeConfig 作為驗證方法)</span><span class="sxs-lookup"><span data-stu-id="fb17f-183">Go to Azure DevOps > Project Settings > Service Connections to create a new "Kubernetes" service connection (use KubeConfig as Authentication method)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="fb17f-184">Azure Pipelines (或其組建代理程式) 必須具有 Kubernetes API 的存取權。</span><span class="sxs-lookup"><span data-stu-id="fb17f-184">Azure Pipelines (or its build agents) must have access to the Kubernetes API.</span></span> <span data-ttu-id="fb17f-185">如果存在從 Azure Pipelines 到 Azure Stack Hub Kubernetes 叢集的網際網路連線，您將需要部署自我裝載的 Azure Pipelines 組建代理程式。</span><span class="sxs-lookup"><span data-stu-id="fb17f-185">If there is an Internet connection from Azure Pipelines to the Azure Stack Hub Kubernetes clusetr, you'll need to deploy a self-hosted Azure Pipelines Build Agent.</span></span>

<span data-ttu-id="fb17f-186">為 Azure Pipelines 部署自我裝載式代理程式時，您可以在 Azure Stack Hub 上或在具有網路連線的機器上，對所有必要的管理端點進行部署。</span><span class="sxs-lookup"><span data-stu-id="fb17f-186">When deploying self-hosted Agents for Azure Pipelines, you may deploy either on Azure Stack Hub, or on a machine with network connectivity to all required management endpoints.</span></span> <span data-ttu-id="fb17f-187">請在此參閱詳細資料：</span><span class="sxs-lookup"><span data-stu-id="fb17f-187">See the details here:</span></span>

* <span data-ttu-id="fb17f-188">[Windows](/azure/devops/pipelines/agents/v2-windows) 或 [Linux](/azure/devops/pipelines/agents/v2-linux) 上的 [Azure Pipelines 代理程式](/azure/devops/pipelines/agents/agents)</span><span class="sxs-lookup"><span data-stu-id="fb17f-188">[Azure Pipelines agents](/azure/devops/pipelines/agents/agents) on [Windows](/azure/devops/pipelines/agents/v2-windows) or [Linux](/azure/devops/pipelines/agents/v2-linux)</span></span>

<span data-ttu-id="fb17f-189">[部署 (CI/CD) 考量](pattern-highly-available-kubernetes.md#deployment-cicd-considerations)一節包含的決策流程可協助您了解要使用 Microsoft 裝載的代理程式或自我裝載的代理程式：</span><span class="sxs-lookup"><span data-stu-id="fb17f-189">The pattern [Deployment (CI/CD) considerations](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) section contains a decision flow that helps you to understand whether to use Microsoft-hosted agents or self-hosted agents:</span></span>

<span data-ttu-id="fb17f-190">[![決策流程自我裝載式代理程式](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="fb17f-190">[![decision flow self hosted agents](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span></span>

<span data-ttu-id="fb17f-191">此範例解決方案中的拓撲包含每個 Azure Stack Hub 執行個體上的自我裝載式組建代理程式。</span><span class="sxs-lookup"><span data-stu-id="fb17f-191">In this sample solution, the topology includes a self-hosted build agent on each Azure Stack Hub instance.</span></span> <span data-ttu-id="fb17f-192">代理程式可以存取 Azure Stack Hub 管理端點和 Kubernetes 叢集 API 端點。</span><span class="sxs-lookup"><span data-stu-id="fb17f-192">The agent can access the Azure Stack Hub Management Endpoints and the Kubernetes cluster API endpoints.</span></span>

<span data-ttu-id="fb17f-193">[![僅輸出流量](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="fb17f-193">[![only outbound traffic](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span></span>

<span data-ttu-id="fb17f-194">這項設計可滿足一般的法規需求，也就是要有來自應用程式解決方案的輸出連線。</span><span class="sxs-lookup"><span data-stu-id="fb17f-194">This design fulfills a common regulatory requirement, which is to have only outbound connections from the application solution.</span></span>

## <a name="configure-monitoring"></a><span data-ttu-id="fb17f-195">設定監視</span><span class="sxs-lookup"><span data-stu-id="fb17f-195">Configure monitoring</span></span>

<span data-ttu-id="fb17f-196">您可以使用適用於容器的 [Azure 監視器](/azure/azure-monitor/) 來監視解決方案中的容器。</span><span class="sxs-lookup"><span data-stu-id="fb17f-196">You can use [Azure Monitor](/azure/azure-monitor/) for containers to monitor the containers in the solution.</span></span> <span data-ttu-id="fb17f-197">這會將 Azure 監視器指向 Azure Stack Hub 上由 AKS 引擎部署的 Kubernetes 叢集。</span><span class="sxs-lookup"><span data-stu-id="fb17f-197">This points Azure Monitor to the AKS Engine-deployed Kubernetes cluster on Azure Stack Hub.</span></span>

<span data-ttu-id="fb17f-198">有兩種方式可以為您的叢集啟用 Azure 監視器。</span><span class="sxs-lookup"><span data-stu-id="fb17f-198">There are two ways to enable Azure Monitor on your cluster.</span></span> <span data-ttu-id="fb17f-199">這兩種方式都需要您在 Azure 中設定 Log Analytics 工作區。</span><span class="sxs-lookup"><span data-stu-id="fb17f-199">Both ways require you to set up a Log Analytics workspace in Azure.</span></span>

* <span data-ttu-id="fb17f-200">[方法一](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one)是使用 Helm 圖表</span><span class="sxs-lookup"><span data-stu-id="fb17f-200">[Method one](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) uses a Helm Chart</span></span>
* <span data-ttu-id="fb17f-201">[方法二](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two)是作為 AKS 引擎叢集規格的一部分</span><span class="sxs-lookup"><span data-stu-id="fb17f-201">[Method two](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) as part of the AKS Engine cluster specification</span></span>

<span data-ttu-id="fb17f-202">範例拓撲會使用「方法一」，這可允許程序自動化並輕鬆地安裝更新。</span><span class="sxs-lookup"><span data-stu-id="fb17f-202">In the sample topology, "Method one" is used, which allows automation of the process and updates can be installed more easily.</span></span>

<span data-ttu-id="fb17f-203">在下一個步驟中，您的機器上需要有 Azure LogAnalytics 工作區 (識別碼和金鑰)、`Helm` (第3版) 和 `kubectl`。</span><span class="sxs-lookup"><span data-stu-id="fb17f-203">For the next step, you need an Azure LogAnalytics Workspace (ID and Key), `Helm` (version 3), and `kubectl` on your machine.</span></span>

<span data-ttu-id="fb17f-204">Helm 是 Kubernetes 套件管理員，可作為在 macOS、Windows 和 Linux 上執行的二進位檔。</span><span class="sxs-lookup"><span data-stu-id="fb17f-204">Helm is a Kubernetes package manager, available as a binary that is runs on macOS, Windows, and Linux.</span></span> <span data-ttu-id="fb17f-205">可以在此下載：[helm.sh](https://helm.sh/docs/intro/quickstart/) Helm 會依賴用於 `kubectl` 命令的 Kubernetes 設定檔。</span><span class="sxs-lookup"><span data-stu-id="fb17f-205">It can be downloaded here: [helm.sh](https://helm.sh/docs/intro/quickstart/) Helm relies on the Kubernetes configuration file used for the `kubectl` command.</span></span>

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

<span data-ttu-id="fb17f-206">此命令會在您的 Kubernetes 叢集上安裝 Azure 監視器代理程式：</span><span class="sxs-lookup"><span data-stu-id="fb17f-206">This command will install the Azure Monitor agent on your Kubernetes cluster:</span></span>

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

<span data-ttu-id="fb17f-207">Kubernetes 叢集上的 Operations Management Suite (OMS) 代理程式會將監視資料傳送至您的 Azure Log Analytics 工作區 (使用輸出 HTTPS)。</span><span class="sxs-lookup"><span data-stu-id="fb17f-207">The Operations Management Suite (OMS) Agent on your Kubernetes cluster will send monitoring data to your Azure Log Analytics Workspace (using outbound HTTPS).</span></span> <span data-ttu-id="fb17f-208">您現在可以使用 Azure 監視器，在 Azure Stack Hub 上取得更深入的 Kubernetes 叢集深入解析。</span><span class="sxs-lookup"><span data-stu-id="fb17f-208">You can now use Azure Monitor to get deeper insights about your Kubernetes clusters on Azure Stack Hub.</span></span> <span data-ttu-id="fb17f-209">此設計有利於說明透過應用程式叢集自動部署的分析有多強大。</span><span class="sxs-lookup"><span data-stu-id="fb17f-209">This design is a powerful way to demonstrate the power of analytics that can be automatically deployed with your application's clusters.</span></span>

<span data-ttu-id="fb17f-210">[![在 Azure 監視器中 Azure Stack Hub 叢集](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="fb17f-210">[![Azure Stack Hub clusters in Azure monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span></span>

<span data-ttu-id="fb17f-211">[![Azure 監視器叢集詳細資料](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="fb17f-211">[![Azure Monitor cluster details](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="fb17f-212">如果 Azure 監視器未顯示任何 Azure Stack Hub 資料，請確定您已遵循＜[如何將 AzureMonitor-Containers 解決方案新增至 Azure Loganalytics 工作區](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md)＞的指示進行。</span><span class="sxs-lookup"><span data-stu-id="fb17f-212">If Azure Monitor does not show any Azure Stack Hub data, please make sure that you have followed the instructions on [how to add AzureMonitor-Containers solution to a Azure Loganalytics workspace](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) carefully.</span></span>

## <a name="deploy-the-application"></a><span data-ttu-id="fb17f-213">部署應用程式</span><span class="sxs-lookup"><span data-stu-id="fb17f-213">Deploy the application</span></span>

<span data-ttu-id="fb17f-214">在安裝範例應用程式之前，還有另一個在 Kubernetes 叢集上設定 nginx 型輸入控制器的步驟。</span><span class="sxs-lookup"><span data-stu-id="fb17f-214">Before installing our sample application, there's another step to configure the nginx-based Ingress controller on our Kubernetes cluster.</span></span> <span data-ttu-id="fb17f-215">輸入控制器會當做第 7 層負載平衡器使用，以根據主機、路徑或通訊協定來路由叢集中的流量。</span><span class="sxs-lookup"><span data-stu-id="fb17f-215">The Ingress controller is used as a layer 7 load balancer to route traffic in our cluster based on host, path, or protocol.</span></span> <span data-ttu-id="fb17f-216">Nginx 輸入是以 Helm 圖表的形式提供。</span><span class="sxs-lookup"><span data-stu-id="fb17f-216">Nginx-ingress is available as a Helm Chart.</span></span> <span data-ttu-id="fb17f-217">如需詳細指示，請參閱 [Helm 圖表的 GitHub 存放庫](https://github.com/helm/charts/tree/master/stable/nginx-ingress)。</span><span class="sxs-lookup"><span data-stu-id="fb17f-217">For detailed instructions, refer to the [Helm Chart GitHub repository](https://github.com/helm/charts/tree/master/stable/nginx-ingress).</span></span>

<span data-ttu-id="fb17f-218">我們的範例應用程式也會封裝為 Helm 圖表，例如上一個步驟中的 [Azure 監視代理程式](#configure-monitoring)。</span><span class="sxs-lookup"><span data-stu-id="fb17f-218">Our sample application is also packaged as a Helm Chart, like the [Azure Monitoring Agent](#configure-monitoring) in the previous step.</span></span> <span data-ttu-id="fb17f-219">因此，可直接將應用程式部署到我們的 Kubernetes 叢集上。</span><span class="sxs-lookup"><span data-stu-id="fb17f-219">As such, it's straightforward to deploy the application onto our Kubernetes cluster.</span></span> <span data-ttu-id="fb17f-220">您可以在[附屬的 GitHub 存放庫中找到 Helm 圖表檔案](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)</span><span class="sxs-lookup"><span data-stu-id="fb17f-220">You can find the [Helm Chart files in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)</span></span>

<span data-ttu-id="fb17f-221">範例應用程式是三層應用程式，並且已部署到兩個 Azure Stack Hub 執行個體的 Kubernetes 叢集上。</span><span class="sxs-lookup"><span data-stu-id="fb17f-221">The sample application is a three tier application, deployed onto a Kubernetes cluster on each of two Azure Stack Hub instances.</span></span> <span data-ttu-id="fb17f-222">應用程式會使用 MongoDB 資料庫。</span><span class="sxs-lookup"><span data-stu-id="fb17f-222">The application uses a MongoDB database.</span></span> <span data-ttu-id="fb17f-223">若要深入了解如何在多個執行個體之間複寫資料，請參閱[資料和儲存體考量](pattern-highly-available-kubernetes.md#data-and-storage-considerations)一節。</span><span class="sxs-lookup"><span data-stu-id="fb17f-223">You can learn more about how to get the data replicated across multiple instances in the pattern [Data and Storage considerations](pattern-highly-available-kubernetes.md#data-and-storage-considerations).</span></span>

<span data-ttu-id="fb17f-224">部署應用程式的 Helm 圖表之後，您會看到應用程式的三個層級以單一 Pod 表示為部署和 StatefulSet (適用於資料庫)：</span><span class="sxs-lookup"><span data-stu-id="fb17f-224">After deploying the Helm Chart for the application, you'll see all three tiers of your application represented as deployments and stateful sets (for the database) with a single pod:</span></span>

```kubectl
kubectl get pod,deployment,statefulset
```

```console
NAME                                         READY   STATUS
pod/ratings-api-569d7f7b54-mrv5d             1/1     Running
pod/ratings-mongodb-0                        1/1     Running
pod/ratings-web-85667bfb86-l6vxz             1/1     Running

NAME                                         READY
deployment.extensions/ratings-api            1/1
deployment.extensions/ratings-web            1/1

NAME                                         READY
statefulset.apps/ratings-mongodb             1/1
```

<span data-ttu-id="fb17f-225">在服務端上，您會找到 nginx 型輸入控制器及其公用 IP 位址：</span><span class="sxs-lookup"><span data-stu-id="fb17f-225">On the services, side you'll find the nginx-based Ingress Controller and its public IP address:</span></span>

```kubectl
kubectl get service
```

```console
NAME                                         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)
kubernetes                                   ClusterIP      10.0.0.1       <none>        443/TCP
nginx-ingress-1588931383-controller          LoadBalancer   10.0.114.180   *public-ip*   443:30667/TCP
nginx-ingress-1588931383-default-backend     ClusterIP      10.0.76.54     <none>        80/TCP
ratings-api                                  ClusterIP      10.0.46.69     <none>        80/TCP
ratings-web                                  ClusterIP      10.0.161.124   <none>        80/TCP
```

<span data-ttu-id="fb17f-226">「外部 IP」位址是我們的「應用程式端點」。</span><span class="sxs-lookup"><span data-stu-id="fb17f-226">The "External IP" address is our "application endpoint".</span></span> <span data-ttu-id="fb17f-227">這是使用者連線並開啟應用程式的方式，而且也會作為下一個步驟＜[設定流量管理員](#configure-traffic-manager)＞的端點。</span><span class="sxs-lookup"><span data-stu-id="fb17f-227">It's how users will connect to open the application and will also be used as the endpoint for our next step [Configure Traffic Manager](#configure-traffic-manager).</span></span>

## <a name="autoscale-the-application"></a><span data-ttu-id="fb17f-228">自動調整應用程式</span><span class="sxs-lookup"><span data-stu-id="fb17f-228">Autoscale the application</span></span>
<span data-ttu-id="fb17f-229">您可以選擇性地設定[水平 Pod 自動調整程式](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)，以根據特定的計量 (例如 CPU 使用率) 來擴大或縮小規模。</span><span class="sxs-lookup"><span data-stu-id="fb17f-229">You can optionally configure the [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) to scale up or down based on certain metrics like CPU utilization.</span></span> <span data-ttu-id="fb17f-230">下列命令會建立水平 Pod 自動調整程式，以維護 1 到 10 個由 ratings-web 部署控制的 Pod 複本。</span><span class="sxs-lookup"><span data-stu-id="fb17f-230">The following command will create a Horizontal Pod Autoscaler that maintains 1 to 10 replicas of the Pods controlled by the ratings-web deployment.</span></span> <span data-ttu-id="fb17f-231">HPA 會增加及減少複本的數目 (透過部署)，將所有 Pod 的平均 CPU 使用率維持在 80%。</span><span class="sxs-lookup"><span data-stu-id="fb17f-231">HPA will increase and decrease the number of replicas (via the deployment) to maintain an average CPU utilization across all Pods of 80%.</span></span>

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
<span data-ttu-id="fb17f-232">您可以執行下列動作來檢查自動調整程式的目前狀態：</span><span class="sxs-lookup"><span data-stu-id="fb17f-232">You may check the current status of autoscaler by running:</span></span>

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a><span data-ttu-id="fb17f-233">設定流量管理員</span><span class="sxs-lookup"><span data-stu-id="fb17f-233">Configure Traffic Manager</span></span>

<span data-ttu-id="fb17f-234">為了在應用程式的兩個 (或多個) 部署之間分散流量，我們將使用 [Azure 流量管理員](/azure/traffic-manager/traffic-manager-overview)。</span><span class="sxs-lookup"><span data-stu-id="fb17f-234">To distribute traffic between two (or more) deployments of the application, we'll use [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview).</span></span> <span data-ttu-id="fb17f-235">Azure 流量管理員是 Azure 中的 DNS 型流量負載平衡器。</span><span class="sxs-lookup"><span data-stu-id="fb17f-235">Azure Traffic Manager is a DNS-based traffic load balancer in Azure.</span></span>

> [!NOTE]
> <span data-ttu-id="fb17f-236">流量管理員會使用 DNS，根據流量路由方法和端點的健康情況，將用戶端要求導向最適當的服務端點。</span><span class="sxs-lookup"><span data-stu-id="fb17f-236">Traffic Manager uses DNS to direct client requests to the most appropriate service endpoint, based on a traffic-routing method and the health of the endpoints.</span></span>

<span data-ttu-id="fb17f-237">您也可以使用裝載於內部部署環境的其他全域負載平衡解決方案，而不是使用 Azure 流量管理員。</span><span class="sxs-lookup"><span data-stu-id="fb17f-237">Instead of using Azure Traffic Manager you can also use other global load-balancing solutions hosted on-premises.</span></span> <span data-ttu-id="fb17f-238">在範例案例中，我們會使用 Azure 流量管理員在應用程式的兩個執行個體之間分散流量。</span><span class="sxs-lookup"><span data-stu-id="fb17f-238">In the sample scenario, we'll use Azure Traffic Manager to distribute traffic between two instances of our application.</span></span> <span data-ttu-id="fb17f-239">其可以在相同或不同位置的 Azure Stack Hub 執行個體上執行：</span><span class="sxs-lookup"><span data-stu-id="fb17f-239">They can run on Azure Stack Hub instances in the same or different locations:</span></span>

![內部部署流量管理員](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

<span data-ttu-id="fb17f-241">在 Azure 中，我們會將流量管理員設定為指向應用程式的兩個不同執行個體：</span><span class="sxs-lookup"><span data-stu-id="fb17f-241">In Azure, we configure Traffic Manager to point to the two different instances of our application:</span></span>

<span data-ttu-id="fb17f-242">[![TM 端點設定檔](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="fb17f-242">[![TM endpoint profile](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span></span>

<span data-ttu-id="fb17f-243">如您所見，這兩個端點會指向[上一節](#deploy-the-application)中已部署的兩個應用程式執行個體。</span><span class="sxs-lookup"><span data-stu-id="fb17f-243">As you can see, the two endpoints point to the two instances of the deployed application from the [previous section](#deploy-the-application).</span></span>

<span data-ttu-id="fb17f-244">至此：</span><span class="sxs-lookup"><span data-stu-id="fb17f-244">At this point:</span></span>
- <span data-ttu-id="fb17f-245">您已建立 Kubernetes 基礎結構，包括輸入控制器。</span><span class="sxs-lookup"><span data-stu-id="fb17f-245">The Kubernetes infrastructure has been created, including an Ingress Controller.</span></span>
- <span data-ttu-id="fb17f-246">叢集已部署到兩個 Azure Stack Hub 執行個體。</span><span class="sxs-lookup"><span data-stu-id="fb17f-246">Clusters have been deployed across two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="fb17f-247">已設定監視。</span><span class="sxs-lookup"><span data-stu-id="fb17f-247">Monitoring has been configured.</span></span>
- <span data-ttu-id="fb17f-248">Azure 流量管理員會將流量負載平衡到兩個 Azure Stack Hub 執行個體。</span><span class="sxs-lookup"><span data-stu-id="fb17f-248">Azure Traffic Manager will load balance traffic across the two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="fb17f-249">在此基礎結構之上，三層式應用程式範例已使用 Helm 圖表自動地進行部署。</span><span class="sxs-lookup"><span data-stu-id="fb17f-249">On top of this infrastructure, the sample three-tier application has been deployed in an automated way using Helm Charts.</span></span> 

<span data-ttu-id="fb17f-250">解決方案現在應該已啟動並可供使用者存取！</span><span class="sxs-lookup"><span data-stu-id="fb17f-250">The solution should now be up and accessible to users!</span></span>

<span data-ttu-id="fb17f-251">此外，還有一些部署後的操作考量值得討論，這會在下面兩節中討論。</span><span class="sxs-lookup"><span data-stu-id="fb17f-251">There are also some post-deployment operational considerations worth discussing, which are covered in the next two sections.</span></span>

## <a name="upgrade-kubernetes"></a><span data-ttu-id="fb17f-252">升級 Kubernetes</span><span class="sxs-lookup"><span data-stu-id="fb17f-252">Upgrade Kubernetes</span></span>

<span data-ttu-id="fb17f-253">升級 Kubernetes 叢集時，請考慮下列主題：</span><span class="sxs-lookup"><span data-stu-id="fb17f-253">Consider the following topics when upgrading the Kubernetes cluster:</span></span>

- <span data-ttu-id="fb17f-254">升級 Kubernetes 叢集是可使用 AKS 引擎來完成但複雜的第 2 天 (Day 2) 作業。</span><span class="sxs-lookup"><span data-stu-id="fb17f-254">Upgrading a Kubernetes cluster is a complex Day 2 operation that can be done using AKS Engine.</span></span> <span data-ttu-id="fb17f-255">如需詳細資訊，請參閱[升級 Azure Stack Hub 上的 Kubernetes 叢集](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade)。</span><span class="sxs-lookup"><span data-stu-id="fb17f-255">For more information, see [Upgrade a Kubernetes cluster on Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).</span></span>
- <span data-ttu-id="fb17f-256">AKS 引擎可讓您將叢集升級為較新的 Kubernetes 和基礎 OS 映像版本。</span><span class="sxs-lookup"><span data-stu-id="fb17f-256">AKS Engine allows you to upgrade clusters to newer Kubernetes and base OS image versions.</span></span> <span data-ttu-id="fb17f-257">如需詳細資訊，請參閱[升級至較新 Kubernetes 版本的步驟](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version)。</span><span class="sxs-lookup"><span data-stu-id="fb17f-257">For more information, see [Steps to upgrade to a newer Kubernetes version](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version).</span></span> 
- <span data-ttu-id="fb17f-258">您也可以只將基礎節點升級為較新的基礎 OS 映像版本。</span><span class="sxs-lookup"><span data-stu-id="fb17f-258">You can also upgrade only the underlaying nodes to newer base OS image versions.</span></span> <span data-ttu-id="fb17f-259">如需詳細資訊，請參閱[僅升級 OS 映像的步驟](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image)。</span><span class="sxs-lookup"><span data-stu-id="fb17f-259">For more information, see [Steps to only upgrade the OS image](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).</span></span>

<span data-ttu-id="fb17f-260">較新的基礎 OS 映像包含安全性和核心更新。</span><span class="sxs-lookup"><span data-stu-id="fb17f-260">Newer base OS images contain security and kernel updates.</span></span> <span data-ttu-id="fb17f-261">叢集操作員必須負責監視較新的 Kubernetes 版本和 OS 映像的可用性。</span><span class="sxs-lookup"><span data-stu-id="fb17f-261">It's the cluster operator's responsibility to monitor the availability of newer Kubernetes Versions and OS Images.</span></span> <span data-ttu-id="fb17f-262">操作員應該使用 AKS 引擎來規劃和執行這些升級。</span><span class="sxs-lookup"><span data-stu-id="fb17f-262">The operator should plan and execute these upgrades using AKS Engine.</span></span> <span data-ttu-id="fb17f-263">基礎 OS 映像必須由 Azure Stack Hub 操作員從 Azure Stack Hub Marketplace 下載。</span><span class="sxs-lookup"><span data-stu-id="fb17f-263">The base OS images must be downloaded from the Azure Stack Hub Marketplace by the Azure Stack Hub Operator.</span></span>

## <a name="scale-kubernetes"></a><span data-ttu-id="fb17f-264">調整 Kubernetes</span><span class="sxs-lookup"><span data-stu-id="fb17f-264">Scale Kubernetes</span></span>

<span data-ttu-id="fb17f-265">調整是另一個第 2 天 (Day 2) 作業，可以使用 AKS 引擎進行協調。</span><span class="sxs-lookup"><span data-stu-id="fb17f-265">Scale is another Day 2 operation that can be orchestrated using AKS Engine.</span></span>

<span data-ttu-id="fb17f-266">調整命令會重複使用輸出目錄內的叢集設定檔 (apimodel.json)，以作為新 Azure Resource Manager 部署的輸入。</span><span class="sxs-lookup"><span data-stu-id="fb17f-266">The scale command reuses your cluster configuration file (apimodel.json) in the output directory, as input for a new Azure Resource Manager deployment.</span></span> <span data-ttu-id="fb17f-267">AKS 引擎會針對特定的代理程式集區執行調整作業。</span><span class="sxs-lookup"><span data-stu-id="fb17f-267">AKS Engine executes the scale operation against a specific agent pool.</span></span> <span data-ttu-id="fb17f-268">當調整作業完成時，AKS 引擎就會更新該相同 apimodel.json 檔案中的叢集定義。</span><span class="sxs-lookup"><span data-stu-id="fb17f-268">When the scale operation is complete, AKS Engine updates the cluster definition in that same apimodel.json file.</span></span> <span data-ttu-id="fb17f-269">叢集定義會反映新的節點計數，以反映更新後的目前叢集設定。</span><span class="sxs-lookup"><span data-stu-id="fb17f-269">The cluster definition reflects the new node count in order to reflect the updated, current cluster configuration.</span></span>

- [<span data-ttu-id="fb17f-270">在 Azure Stack Hub 上調整 Kubernetes 叢集</span><span class="sxs-lookup"><span data-stu-id="fb17f-270">Scale a Kubernetes cluster on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a><span data-ttu-id="fb17f-271">後續步驟</span><span class="sxs-lookup"><span data-stu-id="fb17f-271">Next steps</span></span>

- <span data-ttu-id="fb17f-272">深入了解[混合式應用程式設計考量](overview-app-design-considerations.md)</span><span class="sxs-lookup"><span data-stu-id="fb17f-272">Learn more about [Hybrid app design considerations](overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="fb17f-273">檢閱 [GitHub 上此範例的程式碼](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub)並提出改進建議。</span><span class="sxs-lookup"><span data-stu-id="fb17f-273">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).</span></span>