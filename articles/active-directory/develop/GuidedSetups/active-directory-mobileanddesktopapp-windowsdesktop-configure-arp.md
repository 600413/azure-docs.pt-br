---
title: "Introdução à Área de Trabalho do Windows no Azure AD v2 – Configuração | Microsoft Docs"
description: "Como um aplicativo .NET da Área de Trabalho do Windows (XAML) pode obter um token de acesso e chamar uma API protegida pelo ponto de extremidade do Azure Active Directory v2."
services: active-directory
documentationcenter: dev-center-name
author: andretms
manager: mbaldwin
editor: 
ms.assetid: 820acdb7-d316-4c3b-8de9-79df48ba3b06
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/09/2017
ms.author: andret
ms.custom: aaddev
ms.translationtype: Human Translation
ms.sourcegitcommit: 71fea4a41b2e3a60f2f610609a14372e678b7ec4
ms.openlocfilehash: c46837af57978b047242b2869243f83d372ee43e
ms.contentlocale: pt-br
ms.lasthandoff: 05/10/2017


---

## <a name="add-the-applications-registration-information-to-your-app"></a>Adicionar as informações de registro do aplicativo ao aplicativo
Nesta etapa, você precisa adicionar a ID do Aplicativo ao projeto.

1.    Abra `App.xaml.cs` e substitua a linha que contém a `ClientId` por:

```csharp
private static string ClientId = "[Enter the application Id here]";
```

### <a name="what-is-next"></a>Próximas etapas

[Testar e validar](active-directory-mobileanddesktopapp-windowsdesktop-test.md)

