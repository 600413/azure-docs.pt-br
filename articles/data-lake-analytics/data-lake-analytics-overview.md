---
title: "Visão geral da do Microsoft Azure Data Lake Analytics | Microsoft Docs"
description: "O Data Lake Analytics é um serviço de computação de Big Data do Azure que permite usar dados para impulsionar seus negócios com as informações obtidas de seus dados na nuvem, independentemente de onde eles estiverem e de seu tamanho."
services: data-lake-analytics
documentationcenter: 
author: saveenr
manager: saveenr
editor: cgronlun
ms.assetid: 1e1d443a-48a2-47fb-bc00-bf88274222de
ms.service: data-lake-analytics
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 06/23/2017
ms.author: saveenr
ms.openlocfilehash: d327a3c28e928550b361c07569df74060cfcac0d
ms.sourcegitcommit: 83ea7c4e12fc47b83978a1e9391f8bb808b41f97
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/28/2018
---
# <a name="overview-of-microsoft-azure-data-lake-analytics"></a>Visão geral da Análise Microsoft Azure Data Lake
## <a name="what-is-azure-data-lake-analytics"></a>O que é a Análise Azure Data Lake?
O Azure Data Lake Analytics é um serviço de análise sob demanda que simplifica big data. Em vez de implantar, configurar e ajustar o hardware, você escreve consultas para transformar seus dados e extrair informações importantes. O serviço de análise pode manipular trabalhos de qualquer escala de maneira instantânea, simplesmente configurando o controle para a quantidade de potência necessária. Você só paga pelo trabalho quando ele está em execução, o que o torna econômico. O suporte ao serviço de análise inclui U-SQL, uma linguagem que reúne os benefícios do SQL com a potência do código obrigatório. O U-SQL permite que você analise dados no Data Lake Store, no SQL Server no Azure, no Banco de Dados SQL do Azure e no SQL Data Warehouse do Azure.

## <a name="key-capabilities"></a>Principais recursos
* **Escala dinâmica**
  
    O Data Lake Analytics foi projetado desde o início para proporcionar escala e desempenho na nuvem.  Ela provisiona recursos de maneira dinâmica e permitir a análise de dados em terabytes ou até mesmo em exabytes. Ao final do trabalho, o serviço deixa de usar os recursos automaticamente, e você paga apenas pela potência de processamento usada. À medida que o tamanho dos dados armazenados ou da quantidade de recursos de computação usados aumenta ou diminui, você não precisa reescrever os códigos. Você pode se concentrar apenas na lógica dos seus negócios e não em como você processa e armazena grandes conjuntos de dados.
* **Use ferramentas com as quais você está familiarizado para desenvolver, depurar e otimizar de maneira mais rápida e inteligente**
  
    O Data Lake Analytics apresenta uma ampla integração com o Visual Studio, para que você possa usar ferramentas com as quais já está familiarizado para executar, depurar e ajustar códigos. A visualização dos trabalhos do U-SQL permite ver como ocorre a execução de códigos em escala, para que você possa identificar gargalos de desempenho e otimizar custos.
* **U-SQL: simples e familiar, poderoso e extensível**
  
    A Análise Data Lake inclui o U-SQL, uma linguagem de consulta que estende a natureza declarativa simples e familiar do SQL com o poder expressivo do C#. A linguagem U-SQL é baseada no mesmo tempo de execução distribuído que alimenta os sistemas de Big Data na Microsoft. Milhões de desenvolvedores do SQL e do .NET agora podem processar e analisar dados com as habilidades que eles já têm.
* **Integração total com seus investimentos de TI**
  
    A Análise Data Lake pode usar seus investimentos de TI existentes para identidade, gerenciamento, segurança e armazenamento de dados. Isso simplifica a governança de dados e facilita estender seus aplicativos de dados atuais. O Data Lake Analytics é integrado ao Active Directory para o gerenciamento e permissões de usuário e é fornecido com o monitoramento e auditoria internos.

* **Financeiramente viável e econômico**
  
    A Análise Data Lake é uma solução econômica para executar cargas de trabalho de Big Data. Você paga por trabalho quando os dados são processados. Não é necessário nenhum hardware, licenças ou contratos de suporte específico do serviço. O sistema é automaticamente escalado vertical ou horizontalmente conforme a tarefa é iniciada e concluída, para que você só pague pelo serviço de que precisa. [Saiba mais sobre o controle de custos e como reduzir os custos](https://1drv.ms/f/s!AvdZLquGMt47h213Hg3rhl-Tym1c).
    
* **Funciona com todos os seus Dados do Azure**
  
    O Data Lake Analytics é especialmente otimizado para funcionar com o Azure Data Lake, fornecendo o mais alto nível de desempenho, taxa de transferência e paralelização para suas cargas de trabalho de big data.  O Data Lake Analytics também pode trabalhar com Armazenamento de Blobs do Azure e Banco de Dados SQL do Azure.

## <a name="next-steps"></a>Próximas etapas
 
  * Introdução ao Data Lake Analytics usando: o [Portal do Azure](data-lake-analytics-get-started-portal.md) | [Azure PowerShell](data-lake-analytics-get-started-powershell.md) | [CLI](data-lake-analytics-get-started-cli2.md)
  * Gerencie o Azure Data Lake Analytics usando [portal do Azure](data-lake-analytics-manage-use-portal.md) | [Azure PowerShell](data-lake-analytics-manage-use-powershell.md) | [CLI](data-lake-analytics-manage-use-cli.md) | [SDK do Azure .NET](data-lake-analytics-manage-use-dotnet-sdk.md) | [Node.js](data-lake-analytics-manage-use-nodejs.md)
  * [Como controlar os custos e economizar dinheiro com o Data Lake Analytics](https://1drv.ms/f/s!AvdZLquGMt47h213Hg3rhl-Tym1c)
