---
title: Como alterar, excluir ou gerenciar seus grupos de gerenciamento - Azure | Microsoft Docs
description: Saiba como manter e atualizar sua hierarquia de grupos de gerenciamento.
author: rthorn17
manager: rithorn
editor: 
ms.assetid: 
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 2/22/2018
ms.author: rithorn
ms.openlocfilehash: 975f572d9bd0f32825e6a618cd31bbc263885030
ms.sourcegitcommit: 12fa5f8018d4f34077d5bab323ce7c919e51ce47
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/23/2018
---
# <a name="manage-your-resources-with-management-groups"></a>Gerenciar seus recursos com grupos de gerenciamento 
Grupos de gerenciamento são contêineres que o ajudarão a gerenciar o acesso, a política e a conformidade entre várias assinaturas. Você pode alterar, excluir e gerenciar esses contêineres para ter hierarquias que podem ser usadas com o [Azure Policy](../azure-policy/azure-policy-introduction.md) e os [Controles de Acesso Baseados em Função (RBAC) do Azure](../active-directory/role-based-access-control-what-is.md). Para saber mais sobre grupos de gerenciamento, consulte [Organizar seus recursos com grupos de gerenciamento do Azure](management-groups-overview.md).

O recurso do grupo de gerenciamento está disponível em uma visualização pública. Para começar a usar os grupos de gerenciamento, faça logon no [Portal do Azure](https://portal.azure.com) ou use [Azure PowerShell](https://www.powershellgallery.com/packages/AzureRM.ManagementGroups/0.0.1-preview), [CLI do Azure](https://docs.microsoft.com/en-us/cli/azure/extension?view=azure-cli-latest#az_extension_list_available) ou [API REST](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/managementgroups/resource-manager/Microsoft.Management/preview/2018-01-01-preview) para gerenciar seus grupos de gerenciamento.

Para fazer alterações em um grupo de gerenciamento, você deve ter uma função de Proprietário ou Colaborador no grupo de gerenciamento. Para ver quais permissões você tem, selecione o grupo de gerenciamento e, em seguida, selecione **IAM**. Para saber mais sobre as funções de RBAC, consulte [Gerenciar acesso e permissões com RBAC](../active-directory/role-based-access-control-what-is.md).

## <a name="change-the-name-of-a-management-group"></a>Alterar o nome de um grupo de gerenciamento 
Você pode alterar o nome do grupo de gerenciamento usando o portal, o PowerShell ou a CLI do Azure.

### <a name="change-the-name-in-the-portal"></a>Alterar o nome no portal

1. Fazer logon no [Portal do Azure](https://portal.azure.com)
2. Selecione **Todos os serviços** > **Grupos de gerenciamento**  
3. Selecione o grupo de gerenciamento que gostaria de renomear. 
4. Selecione a opção **Renomear grupo** na parte superior da página. 

    ![Renomear Grupo](media/management-groups/detail_action_small.png)
5. Quando o menu for aberto, digite o novo nome que gostaria de exibir.

    ![Renomear Grupo](media/management-groups/rename_context.png) 
4. Selecione **Salvar**. 

### <a name="change-the-name-in-powershell"></a>Alterar o nome no PowerShell

Para atualizar o nome de exibição, use **Update-AzureRmManagementGroup**. Por exemplo, para alterar um nome de grupos de gerenciamento de "Contoso IT" para "Contoso Group", execute o seguinte comando: 

```azurepowershell-inetractive
C:\> Update-AzureRmManagementGroup -GroupName ContosoIt -DisplayName "Contoso Group"  
``` 

### <a name="change-the-name-in-azure-cli"></a>Alterar o nome na CLI do Azure

Para a CLI do Azure, use o comando de atualização. 

```azure-cli
C:\> az account management-group update --group-name Contoso --display-name "Contoso Group" 
```

---
 
## <a name="delete-a-management-group"></a>Excluir um grupo de gerenciamento
Para a exclusão de um grupo de gerenciamento, os seguintes requisitos deverão ser atendidos:
1. Não existem grupos de gerenciamento filhos ou assinaturas no grupo de gerenciamento. 
    - Para mover uma assinatura de um grupo de gerenciamento, consulte [Mover assinatura para outro grupo de gerenciamento](#Move-subscriptions-in-the-hierarchy). 
    - Para mover um grupo de gerenciamento para outro grupo de gerenciamento, consulte [Mover grupos de gerenciamento na hierarquia](#Move-management-groups-in-the-hierarchy). 
2. Você tem permissões de gravação na função de Proprietário ou Colaborador do grupo de gerenciamento no grupo de gerenciamento. Para ver quais permissões você tem, selecione o grupo de gerenciamento e, em seguida, selecione **IAM**. Para saber mais sobre as funções de RBAC, consulte [Gerenciar acesso e permissões com RBAC](../active-directory/role-based-access-control-what-is.md).  

### <a name="delete-in-the-portal"></a>Excluir no portal

1. Fazer logon no [Portal do Azure](https://portal.azure.com)
2. Selecione **Todos os serviços** > **Grupos de gerenciamento**  
3. Selecione o grupo de gerenciamento que gostaria de excluir. 
    
    ![Excluir Grupo](media/management-groups/delete.png)
4. Selecione **Excluir**. 
    - Se o ícone estiver desativado, passar o seletor de mouse sobre o ícone mostrará o motivo. 
5. Há uma janela que abre confirmando que deseja excluir o grupo de gerenciamento. 

    ![Excluir Grupo](media/management-groups/delete_confirm.png) 
6. Selecione **Sim** 


### <a name="delete-in-powershell"></a>Excluir no PowerShell

Use o comando **Remove-AzureRmManagementGroup** do PowerShell para excluir grupos de gerenciamento. 

```azurepowershell-interactive
Remove-AzureRmManagementGroup -GroupName Contoso
```

### <a name="delete-in-azure-cli"></a>Exclusão na CLI do Azure
Com a CLI do Azure, use o comando az account management-group delete. 

```azure-cli
C:\> az account management-group delete --group-name Contoso
```
---

## <a name="view-management-groups"></a>Exibir grupos de gerenciamento
Você pode exibir qualquer grupo de gerenciamento no qual você tem uma função de RBAC direta ou herdada.  

### <a name="view-in-the-portal"></a>Exibir no portal
1. Fazer logon no [Portal do Azure](https://portal.azure.com)
2. Selecione **Todos os serviços** > **Grupos de gerenciamento** 
3. A página de hierarquia Grupo de Gerenciamento é carregada em todos os grupos aos quais você tem acesso. 
    ![Principal](media/management-groups/main.png)
4. Selecione um grupo de gerenciamento individual para os detalhes  

### <a name="view-in-powershell"></a>Exibir no PowerShell
Você pode usar o comando Get-AzureRmManagementGroup para recuperar todos os grupos.  

```azurepowershell-interactive
Get-AzureRmManagementGroup
```
Para obter informações de um único grupo de gerenciamento, use o parâmetro -GroupName

```azurepowershell-interactive
Get-AzureRmManagementGroup -GroupName Contoso
```

### <a name="view-in-azure-cli"></a>Exibir na CLI do Azure
Você pode usar o comando list para recuperar todos os grupos.  

```azure-cli
az account management-group list
```
Para obter informações de um único grupo de gerenciamento, use o comando show

```azurepowershell-interactive
az account management-group show --group-name Contoso
```
---

## <a name="move-subscriptions-in-the-hierarchy"></a>Mover assinaturas na hierarquia
Um motivo para criar um grupo de gerenciamento é agrupar assinaturas. Somente grupos de gerenciamento e assinaturas podem ser tornados filhos de outro grupo de gerenciamento. Uma assinatura movida para um grupo de gerenciamento herda todos os acessos de usuário e políticas do grupo de gerenciamento pai. 

Para mover a assinatura, há várias permissões que você deve ter: 
- Função de "Proprietário" na assinatura filho.
- Função de "Proprietário" ou "Colaborador" no novo grupo de gerenciamento pai. 
- Função de "Proprietário" ou "Colaborador" no antigo grupo de gerenciamento pai.
Para ver quais permissões você tem, selecione o grupo de gerenciamento e, em seguida, selecione **IAM**. Para saber mais sobre as funções de RBAC, consulte [Gerenciar acesso e permissões com RBAC](../active-directory/role-based-access-control-what-is.md). 

### <a name="move-subscriptions-in-the-portal"></a>Mover assinaturas no portal

**Adicionar uma assinatura existente a um grupo de gerenciamento**
1. Fazer logon no [Portal do Azure](https://portal.azure.com)
2. Selecione **Todos os serviços** > **Grupos de gerenciamento** 
3. Selecione o grupo de gerenciamento o qual planeja que seja o pai.      
5. Na parte superior da página, selecione **Adicionar existente**. 
6. No menu aberto, selecione o **Tipo de Recurso** do item que você está tentando mover, que é **Assinatura**.
7. Selecione a assinatura na lista com a ID correta. 

    ![Filhos](media/management-groups/add_context_2.png)
8. Selecione "Salvar"

**Remover uma assinatura de um grupo de gerenciamento**
1. Fazer logon no [Portal do Azure](https://portal.azure.com)
2. Selecione **Todos os serviços** > **Grupos de gerenciamento** 
3. Selecione o grupo de gerenciamento o qual planeja que seja o pai atual.  
4. Selecione a elipse no final da linha da assinatura na lista que você deseja mover.

    ![Mover](media/management-groups/move_small.png)
5. Selecione **Mover**
6. No menu aberto, selecione o **Grupo de gerenciamento pai**.

    ![Mover](media/management-groups/move_small_context.png)
7. Selecione **Salvar**

### <a name="move-subscriptions-in-powershell"></a>Mover assinaturas no PowerShell
Para mover uma assinatura no PowerShell, use o comando Add-AzureRmManagementGroupSubscription.  

```azurepowershell-interactive
Add-AzureRmManagementGroupSubscription -GroupName Contoso -SubscriptionId 12345678-1234-1234-1234-123456789012
```

Para remover o vínculo entre a assinatura e o grupo de gerenciamento, use o comando Remove-AzureRmManagementGroupSubscription.

```azurepowershell-interactive
Remove-AzureRmManagementGroupSubscription -GroupName Contoso -SubscriptionId 12345678-1234-1234-1234-123456789012
```

### <a name="move-subscriptions-in-azure-cli"></a>Mover assinaturas na CLI do Azure
Para mover uma assinatura na CLI, use o comando add. 

```azure-cli
C:\> az account management-group add --group-name Contoso --subscription 12345678-1234-1234-1234-123456789012
```

Para remover a assinatura do grupo de gerenciamento, use o comando subscription remove.  

```azure-cli
C:\> az account management-group remove --group-name Contoso --subscription 12345678-1234-1234-1234-123456789012
```

---

## <a name="move-management-groups-in-the-hierarchy"></a>Mover grupos de gerenciamento na hierarquia  
Quando você move um grupo de gerenciamento pai, todos os recursos filhos que contêm grupos de gerenciamento, assinaturas, grupos de recursos e recursos são movidos com o pai.   

### <a name="move-management-groups-in-the-portal"></a>Mover grupos de gerenciamento no portal

1. Fazer logon no [Portal do Azure](https://portal.azure.com)
2. Selecione **Todos os serviços** > **Grupos de gerenciamento** 
3. Selecione o grupo de gerenciamento o qual planeja que seja o pai.      
5. Na parte superior da página, selecione **Adicionar existente**.
6. No menu aberto, selecione o **Tipo de Recurso** do item que você está tentando mover, que é **Grupo de Gerenciamento**.
7. Selecione o grupo de gerenciamento com a ID e o Nome correto.

    ![mover](media/management-groups/add_context.png)
8. Selecione **Salvar**

### <a name="move-management-groups-in-powershell"></a>Mover grupos de gerenciamento no PowerShell
Use o comando Update-AzureRmManagementGroup no PowerShell para mover um grupo de gerenciamento em um grupo diferente.  

```powershell
C:\> Update-AzureRmManagementGroup -GroupName Contoso  -ParentName ContosoIT
```  
### <a name="move-management-groups-in-azure-cli"></a>Mover grupos de gerenciamento na CLI do Azure
Use o comando update para mover um grupo de gerenciamento com a CLI do Azure. 

```azure-cli
C:/> az account management-group udpate --group-name Contoso --parent-id "Contoso Tenant" 
``` 

---

## <a name="next-steps"></a>Próximas etapas 
Para saber mais sobre grupos de gerenciamento, consulte: 
- [Organizar seus recursos com grupos de gerenciamento do Azure ](management-groups-overview.md)
- [Criar grupos de gerenciamento para organizar recursos do Azure](management-groups-create.md)
- [Instalar o módulo Azure PowerShell](https://www.powershellgallery.com/packages/AzureRM.ManagementGroups/0.0.1-preview)
- [Revisar as Especificações de API REST](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/managementgroups/resource-manager/Microsoft.Management/preview/2018-01-01-preview)
- [Instalar a Extensão CLI do Azure](https://docs.microsoft.com/en-us/cli/azure/extension?view=azure-cli-latest#az_extension_list_available)