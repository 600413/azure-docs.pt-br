---
title: "Criar um gateway de aplicativo com redirecionamento interno – CLI do Azure | Microsoft Docs"
description: "Saiba como criar um gateway de aplicativo que redireciona o tráfego interno da Web ao pool adequado, usando a CLI do Azure."
services: application-gateway
author: davidmu1
manager: timlt
editor: tysonn
ms.service: application-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/24/2018
ms.author: davidmu
ms.openlocfilehash: 4228a3f534a5dc58ab2efa3c5cf0edd4caee43c9
ms.sourcegitcommit: 059dae3d8a0e716adc95ad2296843a45745a415d
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/09/2018
---
# <a name="create-an-application-gateway-with-internal-redirection-using-the-azure-cli"></a>Criar um gateway de aplicativo com redirecionamento interno usando a CLI do Azure

Você pode usar a CLI do Azure para configurar o [redirecionamento do tráfego da Web](application-gateway-multi-site-overview.md) ao criar um [gateway de aplicativo](application-gateway-introduction.md). Neste tutorial, você cria pools de back-end usando um conjunto de dimensionamento de máquinas virtuais. Em seguida, configura ouvintes e regras com base em domínios que você tem, para garantir que o tráfego da Web chegue ao pool apropriado. Este tutorial presume que você tem vários domínios e usa os exemplos *www.contoso.com* e *www.contoso.org*.

Neste artigo, você aprenderá a:

> [!div class="checklist"]
> * Configurar a rede
> * Criar um Gateway de Aplicativo
> * Adicionar ouvintes e regras de redirecionamento
> * Criar um conjunto de dimensionamento de máquinas virtuais com o pool de back-end
> * Criar um registro CNAME no seu domínio

Se você não tiver uma assinatura do Azure, crie uma [conta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de começar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Se você optar por instalar e usar a CLI localmente, este guia de início rápido exigirá a execução da CLI do Azure versão 2.0.4 ou posterior. Para saber qual é a versão, execute `az --version`. Se você precisa instalar ou atualizar, consulte [Instalar a CLI 2.0 do Azure](/cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Criar um grupo de recursos

Um grupo de recursos é um contêiner lógico no qual os recursos do Azure são implantados e gerenciados. Criar um grupo de recursos usando [az group create](/cli/azure/group#create).

O exemplo a seguir cria um grupo de recursos denominado *myResourceGroupAG* no local *eastus*.

```azurecli-interactive 
az group create --name myResourceGroupAG --location eastus
```

## <a name="create-network-resources"></a>Criar recursos da rede 

Criar a rede virtual denominada *myVNet* e a sub-rede denominada *myAGSubnet* usando [az network vnet create](/cli/azure/network/vnet#az_net). É possível adicionar a sub-rede denominada *myBackendSubnet*, que é necessária para os pools de servidores de back-end usando [az network vnet subnet create](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_create). Crie o endereço IP público denominado *myAGPublicIPAddress* usando [az network public-ip create](/cli/azure/public-ip#az_network_public_ip_create).

```azurecli-interactive
az network vnet create \
  --name myVNet \
  --resource-group myResourceGroupAG \
  --location eastus \
  --address-prefix 10.0.0.0/16 \
  --subnet-name myAGSubnet \
  --subnet-prefix 10.0.1.0/24
az network vnet subnet create \
  --name myBackendSubnet \
  --resource-group myResourceGroupAG \
  --vnet-name myVNet \
  --address-prefix 10.0.2.0/24
az network public-ip create \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress
```

## <a name="create-the-application-gateway"></a>Criar o gateway de aplicativo

Você pode usar [az network application-gateway create](/cli/azure/application-gateway#create) para criar o gateway do aplicativo denominado *myAppGateway*. Quando você cria um gateway de aplicativo usando a CLI do Azure, você pode especificar informações de configuração, como configurações de HTTP, sku e capacidade. O gateway de aplicativo é atribuído a *myAGSubnet* e *myAGPublicIPAddress* que você criou anteriormente. 

```azurecli-interactive
az network application-gateway create \
  --name myAppGateway \
  --location eastus \
  --resource-group myResourceGroupAG \
  --vnet-name myVNet \
  --subnet myAGsubnet \
  --capacity 2 \
  --sku Standard_Medium \
  --http-settings-cookie-based-affinity Disabled \
  --frontend-port 80 \
  --http-settings-port 80 \
  --http-settings-protocol Http \
  --public-ip-address myAGPublicIPAddress
```

Pode levar vários minutos para o gateway de aplicativo ser criado. Depois de criar o gateway de aplicativo, você pode ver esses novos recursos:

- *appGatewayBackendPool* - Um gateway de aplicativo deve ter pelo menos um pool de endereços de back-end.
- *appGatewayBackendHttpSettings* - Especifica que a porta 80 e um protocolo HTTP são usados para comunicação.
- *appGatewayHttpListener* - O ouvinte padrão associado ao *appGatewayBackendPool*.
- *appGatewayFrontendIP* - Atribui *myAGPublicIPAddress* ao *appGatewayHttpListener*.
- *rule1* - A regra padrão de roteamento que está associada ao *appGatewayHttpListener*.


## <a name="add-listeners-and-rules"></a>Adicionar ouvintes e regras 

Um ouvinte é necessário para habilitar o gateway de aplicativo para rotear o tráfego corretamente para o pool de back-end. Neste tutorial, você cria dois ouvintes para os seus dois domínios. Neste exemplo, são criados ouvintes para os domínios *www.contoso.com* e *www.contoso.org*.

Adicione ouvintes de back-end necessários para redirecionar o tráfego usando [az network application-gateway http-listener create](/cli/azure/application-gateway#az_network_application_gateway_http_listener_create).

```azurecli-interactive
az network application-gateway http-listener create \
  --name contosoComListener \
  --frontend-ip appGatewayFrontendIP \
  --frontend-port appGatewayFrontendPort \
  --resource-group myResourceGroupAG \
  --gateway-name myAppGateway \
  --host-name www.contoso.com
az network application-gateway http-listener create \
  --name contosoOrgListener \
  --frontend-ip appGatewayFrontendIP \
  --frontend-port appGatewayFrontendPort \
  --resource-group myResourceGroupAG \
  --gateway-name myAppGateway \
  --host-name www.contoso.org   
  ```

### <a name="add-the-redirection-configuration"></a>Adicionar a configuração de redirecionamento

Adicione a configuração de redirecionamento que envia tráfego de *www.contoso.org* para o ouvinte de *www.contoso.com* no gateway do aplicativo usando [az network application-gateway redirect-config create](/cli/azure/network/application-gateway/redirect-config#az_network_application_gateway_redirect_config_create).

```azurecli-interactive
az network application-gateway redirect-config create \
  --name orgToCom \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --type Permanent \
  --target-listener contosoComListener \
  --include-path true \
  --include-query-string true
```

### <a name="add-routing-rules"></a>Adicionar regras de redirecionamento

As regras são processadas na ordem em que são criadas, e o tráfego é redirecionado usando a primeira regra que corresponde à URL enviada para o gateway de aplicativo. A regra básica padrão que foi criada não é necessária neste tutorial. Neste exemplo, você criará duas novas regras, denominadas *contosoComRule* e *contosoOrgRule* e excluirá a regra padrão que foi criada.  Você pode adicionar as regras usando [az network application-gateway rule create](/cli/azure/application-gateway#az_network_application_gateway_rule_create).

```azurecli-interactive
az network application-gateway rule create \
  --gateway-name myAppGateway \
  --name contosoComRule \
  --resource-group myResourceGroupAG \
  --http-listener contosoComListener \
  --rule-type Basic \
  --address-pool appGatewayBackendPool
az network application-gateway rule create \
  --gateway-name myAppGateway \
  --name contosoOrgRule \
  --resource-group myResourceGroupAG \
  --http-listener contosoOrgListener \
  --rule-type Basic \
  --redirect-config orgToCom
az network application-gateway rule delete \
  --gateway-name myAppGateway \
  --name rule1 \
  --resource-group myResourceGroupAG
```

## <a name="create-virtual-machine-scale-sets"></a>Criar conjuntos de dimensionamento de máquinas virtuais

Neste exemplo, você criará um conjunto de dimensionamento de máquinas virtuais que oferece suporte ao pool de back-end padrão criado. O conjunto de dimensionamento criado é chamado *myvmss* e contém duas instâncias de máquina virtual em que você instala o NGINX.

```azurecli-interactive
az vmss create \
  --name myvmss \
  --resource-group myResourceGroupAG \
  --image UbuntuLTS \
  --admin-username azureuser \
  --admin-password Azure123456! \
  --instance-count 2 \
  --vnet-name myVNet \
  --subnet myBackendSubnet \
  --vm-sku Standard_DS2 \
  --upgrade-policy-mode Automatic \
  --app-gateway myAppGateway \
  --backend-pool-name appGatewayBackendPool
```

### <a name="install-nginx"></a>Instalar o NGINX

```azurecli-interactive
az vmss extension set \
  --publisher Microsoft.Azure.Extensions \
  --version 2.0 \
  --name CustomScript \
  --resource-group myResourceGroupAG \
  --vmss-name myvmss \
  --settings '{ "fileUris": ["https://raw.githubusercontent.com/davidmu1/samplescripts/master/install_nginx.sh"],
  "commandToExecute": "./install_nginx.sh" }'
```

## <a name="create-cname-record-in-your-domain"></a>Criar um registro CNAME em seu domínio

Depois de criar o gateway de aplicativo com seu endereço IP público, é possível obter o endereço DNS e usá-lo para criar um registro CNAME em seu domínio. Use [az network public-ip show](/cli/azure/network/public-ip#az_network_public_ip_show) para obter o endereço DNS do gateway de aplicativo. Copie o valor de *fqdn* em DNSSettings e use-o como o valor do registro CNAME a ser criado. O uso de registros A não é recomendado, pois o VIP pode mudar quando o gateway de aplicativo for reiniciado.

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress \
  --query [dnsSettings.fqdn] \
  --output tsv
```

## <a name="test-the-application-gateway"></a>Testar o gateway de aplicativo

Digite seu nome de domínio na barra de endereços do navegador. Por exemplo, http://www.contoso.com.

![Testar o site contoso no gateway do aplicativo](./media/tutorial-internal-site-redirect-cli/application-gateway-nginxtest.png)

Altere o endereço para o seu outro domínio, por exemplo http://www.contoso.org, e você verá que o tráfego foi redirecionado de volta para o ouvinte de www.contoso.com.

## <a name="next-steps"></a>Próximas etapas

Neste tutorial, você aprendeu como:

> [!div class="checklist"]
> * Configurar a rede
> * Criar um Gateway de Aplicativo
> * Adicionar ouvintes e regras de redirecionamento
> * Criar um conjunto de dimensionamento de máquinas virtuais com o pool de back-end
> * Criar um registro CNAME no seu domínio

> [!div class="nextstepaction"]
> [Saiba mais sobre o que você pode fazer com o gateway de aplicativo](./application-gateway-introduction.md)