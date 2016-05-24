<properties
	pageTitle="Adicione a API do OneDrive ao PowerApps Enterprise | Microsoft Azure"
	description="Crie ou configure uma nova API do OneDrive no ambiente de serviço de aplicativo da sua organização"
	services=""
    suite="powerapps"
	documentationCenter="" 
	authors="rajeshramabathiran"
	manager="dwrede"
	editor=""/>

<tags
   ms.service="powerapps"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na" 
   ms.date="03/29/2016"
   ms.author="litran"/>

# Criar uma nova API do OneDrive no PowerApps Enterprise

> [AZURE.IMPORTANT] Esse tópico foi arquivado e será removido em breve. Venha ver as novidades do [PowerApps](https://powerapps.microsoft.com).
> 
> - Para saber mais sobre PowerApps e para começar, vá para [PowerApps](https://powerapps.microsoft.com).  
> - Para saber mais sobre conexões disponíveis no PowerApps, vá para [Conexões Disponíveis](https://powerapps.microsoft.com/tutorials/connections-list/). 

<!--Archived
Add the Onedrive API to your organization's (tenant) app service environment. 

## Create the API in the Azure portal

1. No [Portal do Azure](https://portal.azure.com/), entre com sua conta de trabalho. Por exemplo, entre com *Seunomedeusuário*@*SuaEmpresa*.com. Quando fizer isso, entrará automaticamente na assinatura de sua empresa.
 
2. Select **Browse** in the task bar:  
![][14]

3. In the list, you can scroll to find PowerApps or type in *powerapps*:  
![][15]  

4. In **PowerApps**, select **Manage APIs**:    
![Browse to registered apis][1]

5. In **Manage APIs**, select **Add** to add the new API:  
![Add API][2]

6. Enter a descriptive **name** for your API.  
	
7. In **Source**, select **Available APIs** to select the pre-built APIs, and select **OneDrive**:  
![select OneDrive api][3]

8. Select **Settings - Configure required settings**:  
![configure OneDrive API settings][4]

9. Enter the *App Key* and *App Secret* of your OneDrive application. If you don't have one, see the "Register a OneDrive app for use with PowerApps" section in this topic to create the key and secret values you need.  

	> [AZURE.IMPORTANT] Save the **redirect URL**. You may need this value later in this topic.

10. Select **OK** to complete the steps.

When finished, a new OneDrive API is added to your app service environment.

## Optional: Register a OneDrive app for use with PowerApps

If you don't have an existing OneDrive app with the key and secret values, then use the following steps to create the application, and get the values you need. 

1. Go to the [app creation page][5] in _Microsoft account developer center_ and sign in with your _Microsoft Account_.

2. Enter your **Application name**, and select **I accept**:  
![OneDrive new app][6]

3. In the settings page:  

	1. Select **API Settings**.  
	2. Set the redirect URL to the redirect URL you received when you added the new OneDrive API in the Azure Portal (in this topic).  
	3. Select **Save**.  

	![OneDrive app API settings][7]

A new OneDrive app is created. You can use this app in your OneDrive API configuration in the Azure portal. 

## See the REST APIs

[Onedrive REST API](../connectors/connectors-create-api-onedrive.md) reference.

## Summary and next steps
In this topic, you added the OneDrive API to your PowersApps Enterprise. Next, give users access to the API so it can be added to their apps: 

[Add a connection and give users access](powerapps-manage-api-connection-user-access.md)
-->


<!--References-->
[1]: ./media/powerapps-create-api-onedrive/browse-to-registered-apis.PNG
[2]: ./media/powerapps-create-api-onedrive/add-api.PNG
[3]: ./media/powerapps-create-api-onedrive/select-onedrive-api.PNG
[4]: ./media/powerapps-create-api-onedrive/configure-onedrive-api.PNG
[5]: https://account.live.com/developers/applications/create
[6]: ./media/powerapps-create-api-onedrive/onedrive-new-app.PNG
[7]: ./media/powerapps-create-api-onedrive/onedrive-app-api-settings.PNG
[14]: ./media/powerapps-create-api-onedrive/browseall.png
[15]: ./media/powerapps-create-api-onedrive/allresources.png

<!---HONumber=AcomDC_0511_2016-->