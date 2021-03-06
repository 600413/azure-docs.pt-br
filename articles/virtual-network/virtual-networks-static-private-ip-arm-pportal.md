---
title: "Configurar endereços IP privados para VMs - Portal do Azure | Microsoft Docs"
description: "Saiba como configurar endereços IP para máquinas virtuais usando o Portal do Azure."
services: virtual-network
documentationcenter: na
author: jimdial
manager: timlt
editor: tysonn
tags: azure-resource-manager
ms.assetid: 11245645-357d-4358-9a14-dd78e367b494
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/04/2016
ms.author: jdial
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: c2679c7cb75c438402f3ab64c3e14c964cb8a85d
ms.sourcegitcommit: d87b039e13a5f8df1ee9d82a727e6bc04715c341
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/21/2018
---
# <a name="configure-private-ip-addresses-for-a-virtual-machine-using-the-azure-portal"></a>Configurar endereços IP particulares para uma máquina virtual usando o Portal do Azure

> [!div class="op_single_selector"]
> * [portal do Azure](virtual-networks-static-private-ip-arm-pportal.md)
> * [PowerShell](virtual-networks-static-private-ip-arm-ps.md)
> * [CLI do Azure](virtual-networks-static-private-ip-arm-cli.md)
> * [Portal do Azure (Clássico)](virtual-networks-static-private-ip-classic-pportal.md)
> * [PowerShell (Clássico)](virtual-networks-static-private-ip-classic-ps.md)
> * [CLI do Azure (Clássica)](virtual-networks-static-private-ip-classic-cli.md)


[!INCLUDE [virtual-networks-static-private-ip-intro-include](../../includes/virtual-networks-static-private-ip-intro-include.md)]

[!INCLUDE [azure-arm-classic-important-include](../../includes/azure-arm-classic-important-include.md)]

Este artigo aborda o modelo de implantação do Gerenciador de Recursos. Você também pode [gerenciar o endereço IP privado estático no modelo de implantação clássico](virtual-networks-static-private-ip-classic-pportal.md).

[!INCLUDE [virtual-networks-static-ip-scenario-include](../../includes/virtual-networks-static-ip-scenario-include.md)]

As etapas de exemplo abaixo esperam um ambiente simples já criado. Se você quiser executar as etapas da forma como elas aparecem neste documento, primeiro crie o ambiente de teste descrito em [criar uma vnet](virtual-networks-create-vnet-arm-pportal.md).

## <a name="how-to-create-a-vm-for-testing-static-private-ip-addresses"></a>Como criar uma VM para testar endereços IP privados estáticos
Você não pode definir um endereço IP privado estático durante a criação de uma VM no modo de implantação do Gerenciador de Recursos usando o portal do Azure. Você deve criar a VM primeiro e, em seguida, definir seu IP privado como estático.

Para criar uma VM denominada *DNS01* na sub-rede*FrontEnd* de uma VNet denominada *TestVNet*, execute estas etapas:

1. Em um navegador, navegue até http://portal.azure.com e, se necessário, entre com sua conta do Azure.
2. Clique em **Criar um recurso** > **Serviços de Computação** > **Windows Server 2012 R2 Datacenter**, observe que a lista **Selecionar uma implantação modelo** já mostra o **Gerenciador de Recursos**e, em seguida, clique em **Criar**, como mostrado na figura a seguir.
   
    ![Criar VM no portal do Azure](./media/virtual-networks-static-ip-arm-pportal/figure01.png)
3. No painel **Noções básicas**, insira o nome da VM a ser criada (*DNS01* em nosso cenário), a conta de administrador local e a senha, como mostrado na figura a seguir.
   
    ![Painel de Noções básicas](./media/virtual-networks-static-ip-arm-pportal/figure02.png)
4. Certifique-se que o **Local** selecionado é *Centro dos EUA*, em seguida, clique em **Selecionar existente** em **Grupo de recursos**, em seguida, clique em **Grupo de recursos** novamente, em seguida, clique em *TestRG* e em **OK**.
   
    ![Painel de Noções básicas](./media/virtual-networks-static-ip-arm-pportal/figure03.png)
5. No painel **Escolha um tamanho**, selecione **Padrão A1** e, em seguida, clique em **Selecionar**.
   
    ![Escolha um tamanho de painel](./media/virtual-networks-static-ip-arm-pportal/figure04.png)    
6. No painel **Configurações**, certifique-se de que as propriedades estão definidas com os valores a seguir e, em seguida, clique em **OK**.
   
    -**Conta de armazenamento**: *vnetstorage*
   
   * **Rede**: *TestVNet*
   * **Sub-rede**: *FrontEnd*
     
     ![Escolha um tamanho de painel](./media/virtual-networks-static-ip-arm-pportal/figure05.png)     
7. No painel **Resumo**, clique em **OK**. Observe o seguinte bloco exibido em seu painel.
   
    ![Criar VM no portal do Azure](./media/virtual-networks-static-ip-arm-pportal/figure06.png)

## <a name="how-to-retrieve-static-private-ip-address-information-for-a-vm"></a>Como recuperar informações do endereço IP privado estático de uma VM
Para exibir as informações de endereço IP privado estático para a VM criada com as etapas acima, execute as seguintes etapas.

1. No portal do Azure, clique em **PROCURAR TUDO** > **Máquinas virtuais** > **DNS01** > **Todas as configurações** > **Adaptadores de rede** e clique no único adaptador de rede listado.
   
    ![Implantando o bloco VM](./media/virtual-networks-static-ip-arm-pportal/figure07.png)
2. No painel **Adaptador de rede**, clique em **Todas as configurações** > **Endereços IP** e observe os valores de **Atribuição** e **Endereço IP**.
   
    ![Implantando o bloco VM](./media/virtual-networks-static-ip-arm-pportal/figure08.png)

## <a name="how-to-add-a-static-private-ip-address-to-an-existing-vm"></a>Como adicionar um endereço IP privado estático a uma VM existente
Para adicionar um IP privado estático à VM criada usando as etapas acima, siga estas etapas:

1. No painel **Endereços IP** mostrado acima, clique em **Estático** em **Atribuição**.
2. Digite *192.168.1.101* para **Endereço IP**, e, em seguida, clique em **Salvar**.
   
    ![Criar VM no portal do Azure](./media/virtual-networks-static-ip-arm-pportal/figure09.png)

> [!NOTE]
> Se depois de clicar em **Salvar** você observar que a atribuição ainda está definida como **Dinâmico**, isso significa que o endereço IP digitado já está em uso. Tente um endereço IP diferente.
> 
> 

## <a name="how-to-remove-a-static-private-ip-address-from-a-vm"></a>Como remover um endereço IP privado estático de uma VM
Para remover o endereço IP privado estático da VM criada acima, realize a seguinte etapa:

No painel **Endereços IP** mostrado acima, clique em **Dinâmico** em **Atribuição** e, em seguida, clique em **Salvar**.

## <a name="next-steps"></a>Próximas etapas
* Saiba mais sobre endereços [IP públicos reservados](virtual-networks-reserved-public-ip.md) .
* Saiba mais sobre endereços [ILPIP (IP público em nível de instância)](virtual-networks-instance-level-public-ip.md) .
* Consulte as [APIs REST de IP reservado](https://msdn.microsoft.com/library/azure/dn722420.aspx).

