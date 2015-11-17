<properties
	pageTitle="Importar dados para a Pesquisa do Azure usando a API REST | Microsoft Azure | Serviço de pesquisa de nuvem hospedado"
	description="Como carregar dados para um índice na Pesquisa do Azure usando a API REST."
	services="search"
	documentationCenter=""
	authors="HeidiSteen"
	manager="mblythe"
	editor=""
    tags=""/>

<tags
	ms.service="search"
	ms.devlang="rest-api"
	ms.workload="search"
	ms.topic="get-started-article"
	ms.tgt_pltfrm="na"
	ms.date="11/09/2015"
	ms.author="heidist"/>

# Importar dados para a Pesquisa do Azure usando a API REST
> [AZURE.SELECTOR]
- [Overview](search-what-is-data-import.md)
- [Portal](search-import-data-portal.md)
- [.NET](search-import-data-dotnet.md)
- [REST](search-import-data-rest-api.md)
- [Indexers](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers-2015-02-28.md)

Este artigo mostra como preencher um índice usando a [API REST da Pesquisa do Azure](https://msdn.microsoft.com/library/azure/dn798935.aspx). Parte do conteúdo abaixo é de [Adicionar, atualizar ou excluir documentos] (API REST da Pesquisa do Azure] (https://msdn.microsoft.com/library/azure/dn798930.aspx). Consulte o artigo pai para obter mais contexto.

Os pré-requisitos para a importação incluem ter um índice já existente para receber os dados.

Ao usar a API REST, a ingestão de dados se baseará em uma solicitação Put ou HTTP POST. Os trechos de código são do [exemplo de perfis de pontuação](search-get-started-scoring-profiles.md).

        static bool PostDocuments(string fileName)
        {
            // Add some documents to the newly created index
            Uri requestUri = new Uri(serviceUrl + indexName + "/docs/index?" + ApiVersion);

            // Load the json containing the data from an external file
            string json = File.ReadAllText(fileName);

            using (HttpClient client = new HttpClient())
            {
                // Create the index
                client.DefaultRequestHeaders.Add("api-key", primaryKey);
                HttpResponseMessage response = client.PostAsync(requestUri,       // To add data use POST
                    new StringContent(json, Encoding.UTF8, "application/json")).Result;

                if (response.StatusCode == HttpStatusCode.OK)
                {
                    Console.WriteLine("Documents posted from file {0}. \r\n", fileName);
                    return true;
                }

                Console.WriteLine("Documents failed to upload: {0} {1} \r\n", (int)response.StatusCode, response.Content.ReadAsStringAsync().Result.ToString());
                return false;

            }
        }

<!---HONumber=Nov15_HO3-->