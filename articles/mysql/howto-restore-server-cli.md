---
title: Como fazer backup e restaurar um servidor no Banco de Dados do Azure para MySQL
description: Saiba como fazer backup e restaurar um servidor no Banco de Dados do Azure para MySQL usando a CLI do Azure.
services: mysql
author: jasonwhowell
ms.author: jasonh
manager: kfile
editor: jasonwhowell
ms.service: mysql-database
ms.devlang: azure-cli
ms.topic: article
ms.date: 02/28/2018
ms.openlocfilehash: b954e26c9ecb1767b971117fc9102e8573beaaac
ms.sourcegitcommit: c765cbd9c379ed00f1e2394374efa8e1915321b9
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/28/2018
---
# <a name="how-to-backup-and-restore-a-server-in-azure-database-for-mysql-by-using-the-azure-cli"></a>Como fazer backup e restaurar um servidor no Banco de Dados do Azure para MySQL usando a CLI do Azure

Use o Banco de Dados do Azure para MySQL para restaurar um banco de dados do servidor para uma data anterior que abranja de sete a 35 dias.

## <a name="prerequisites"></a>pré-requisitos
Para concluir este guia de instruções, você precisa:
- Um [banco de dados e servidor do Banco de Dados do Azure para MySQL](quickstart-create-mysql-server-database-using-azure-portal.md)

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

> [!IMPORTANT]
> Se você instalar e usar a CLI do Azure localmente, este guia de instruções exige que você use a CLI do Azure versão 2.0 ou posterior. Para confirmar a versão, no prompt de comando da CLI do Azure, digite `az --version`. Para instalar ou atualizar, consulte [Instalar a CLI 2.0 do Azure]( /cli/azure/install-azure-cli).

## <a name="backup-happens-automatically"></a>O backup ocorre automaticamente
Ao usar o Banco de Dados do Azure para MySQL, o serviço de banco de dados faz automaticamente um backup do serviço a cada cinco minutos. 

Para a Camada Básica, os backups estão disponíveis por sete dias. Para a Camada Padrão, os backups estão disponíveis por 35 dias. Para saber mais, confira [Tipos de preço do Banco de Dados do Azure para MySQL](concepts-pricing-tiers.md).

Com esse recurso de backup automático, você pode restaurar o servidor e seus bancos de dados para uma data em momento anterior.

## <a name="restore-a-database-to-a-previous-point-in-time-by-using-the-azure-cli"></a>Restaurar um banco de dados para um momento anterior usando a CLI do Azure
Use o Banco de Dados do Azure para MySQL para restaurar o servidor para um momento anterior. Os dados restaurados são copiados para um novo servidor e o servidor existente é deixado como está. Por exemplo, se uma tabela tiver sido descartada acidentalmente ao meio-dia de hoje, você poderá restaurá-la para um horário um pouco antes do meio-dia. Depois, você pode recuperar a tabela e os dados ausentes da cópia restaurada do servidor. 

Para restaurar o servidor, use o comando [az mysql server restore](/cli/azure/mysql/server#az_mysql_server_restore) da CLI do Azure.

### <a name="run-the-restore-command"></a>Executar o comando restore

Para restaurar o servidor, no prompt de comando da CLI do Azure, digite o seguinte comando:

```azurecli-interactive
az mysql server restore --resource-group myresourcegroup --name myserver-restored --restore-point-in-time 2017-04-13T13:59:00Z --source-server mydemoserver
```

O comando `az mysql server restore` exige os seguintes parâmetros:
| Configuração | Valor sugerido | DESCRIÇÃO  |
| --- | --- | --- |
| resource-group | myresourcegroup |  O grupo de recursos em que o servidor de origem existe.  |
| Nome | myserver-restored | O nome do novo servidor que é criado pelo comando de restauração. |
| Restauração-point-in-time | 2017-04-13T13:59:00Z | Selecione um ponto no tempo para o qual restaurar. Essa data e hora devem estar dentro do período de retenção de backup do servidor de origem. Use o formato ISO8601 de data e hora. Por exemplo, você pode usar seu fuso horário local, como `2017-04-13T05:59:00-08:00`. Você também pode usar o formato UTC Zulu, por exemplo, `2017-04-13T13:59:00Z`. |
| source-server | mydemoserver | O nome ou ID para restaurar a partir do servidor de origem. |

Quando você restaura um servidor para um ponto anterior no tempo, é criado um novo servidor. O servidor original e seus bancos de dados do ponto no tempo especificado são copiados para o novo servidor.

Os valores de local e tipo de preço para o servidor restaurado permanecem iguais aos do servidor de origem. 

O comando `az mysql server restore` é síncrono. Depois que o servidor é restaurado, você pode usá-lo novamente para repetir o processo para outro ponto no tempo. 

Depois que o processo de restauração é concluído, localize o novo servidor e verifique se os dados são restaurados como esperado.

## <a name="next-steps"></a>Próximas etapas
[Bibliotecas de conexão para o Banco de Dados do Azure para MySQL](concepts-connection-libraries.md)
