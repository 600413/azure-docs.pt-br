---
title: "Visualização de dependência em Migrações para Azure | Microsoft Docs"
description: "Fornece uma visão geral dos cálculos de avaliação no serviço Migrações para Azure."
author: rayne-wiselman
ms.service: azure-migrate
ms.topic: conceptual
ms.date: 2/21/2018
ms.author: raynew
ms.openlocfilehash: 886977764517f1fec89eee77fc3263d30ff9ab31
ms.sourcegitcommit: 12fa5f8018d4f34077d5bab323ce7c919e51ce47
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/23/2018
---
# <a name="dependency-visualization"></a>Visualização de dependência

Os serviços [Migrações para Azure](migrate-overview.md) avaliam grupos de computadores locais para a migração para o Azure. Para agrupar computadores, você pode usar a visualização de dependência. Este artigo fornece informações sobre este recurso.


## <a name="overview"></a>Visão geral

A visualização de dependência em Migrações para Azure permite criar grupos para avaliar a migração com maior confiança. Usando a visualização de dependência, você pode exibir as dependências de rede de computadores específicos ou em um grupo de computadores. Isso é útil para garantir que nenhuma funcionalidade seja perdida (ou computadores esquecidos) no processo de migração, quando os aplicativos e as cargas de trabalho são executados em vários computadores.  

## <a name="how-does-it-work"></a>Como ele funciona?

As Migrações para Azure usam a solução [Mapa do Serviço](../operations-management-suite/operations-management-suite-service-map.md) no [Log Analytics](../log-analytics/log-analytics-overview.md) para visualização de dependência.
- Quando você cria um projeto de Migrações para Azure, um espaço de trabalho do Log Analytics do OMS é criado na sua assinatura.
- O nome do espaço de trabalho é o nome que você especificar para o projeto de migração, prefixado com **migrate-** e, opcionalmente, o sufixo com um número. 
- Navegue até o espaço de trabalho do Log Analytics da seção **Essentials** da página **Visão geral** do projeto.
- O espaço de trabalho criado é marcado com a chave **MigrateProject** e o valor **nome do projeto**. Você pode usá-los para pesquisar no portal do Azure.  

    ![Espaço de trabalho do Log Analytics](./media/concepts-dependency-visualization/oms-workspace.png)

Para usar a visualização de dependência, você precisa baixar e instalar agentes em cada computador local que você deseja analisar.  

## <a name="do-i-need-to-pay-for-it"></a>Eu preciso pagar por ele?

Saiba mais sobre os preços de Migrações para Azure [aqui](https://azure.microsoft.com/pricing/details/azure-migrate/). 

## <a name="how-do-i-manage-the-workspace"></a>Como faço para gerenciar o espaço de trabalho?

Você pode usar o espaço de trabalho do Log Analytics fora de Migrações para Azure. Ele não será excluído se você excluir o projeto de migração em que ele foi criado. Se você não precisar mais do espaço de trabalho, [exclua-o](../log-analytics/log-analytics-manage-access.md) manualmente.

Não exclua o espaço de trabalho criado pelas Migrações para Azure, a menos que você exclua o projeto de migração. Se você fizer isso, as dependências não funcionarão conforme o esperado.

## <a name="next-steps"></a>Próximas etapas

[Agrupar máquinas usando dependências da máquina](how-to-create-group-machine-dependencies.md)