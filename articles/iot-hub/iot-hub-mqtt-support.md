---
title: Entender o suporte do MQTT do Hub IoT do Azure | Microsoft Docs
description: "Guia do desenvolvedor ‑ suporte para dispositivos que se conectam a um ponto de extremidade do Hub IoT voltado para o dispositivo usando o protocolo MQTT. Inclui informações sobre suporte interno do MQTT nos SDKs do dispositivo IoT do Azure."
services: iot-hub
documentationcenter: .net
author: fsautomata
manager: timlt
editor: 
ms.assetid: 1d71c27c-b466-4a40-b95b-d6550cf85144
ms.service: iot-hub
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/19/2018
ms.author: elioda
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: a22c20a26ee4750c79c23fbba69de72a0084dfe7
ms.sourcegitcommit: d1f35f71e6b1cbeee79b06bfc3a7d0914ac57275
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/22/2018
---
# <a name="communicate-with-your-iot-hub-using-the-mqtt-protocol"></a>Comunicar com o hub IoT usando o protocolo MQTT

O Hub IoT permite que os dispositivos comuniquem-se com os pontos de extremidade do dispositivo de Hub IoT usando:

* [MQTT v3.1.1][lnk-mqtt-org] na porta 8883
* MQTT v3.1.1 sobre WebSocket na porta 443.

Todas as comunicações de dispositivo com o Hub IoT devem ser protegidas usando TLS/SSL. Portanto, o Hub IoT não fornece suporte para conexões não seguras na porta 1883.

## <a name="connecting-to-iot-hub"></a>Conectando-se ao Hub IoT

Um dispositivo pode usar o protocolo MQTT para conectar-se a um Hub IoT usando:

* Ou as bibliotecas nos [SDKs de IoT do Azure][lnk-device-sdks].
* Ou o protocolo MQTT diretamente.

## <a name="using-the-device-sdks"></a>Usando os SDKs de dispositivo

Os [SDKs do cliente do dispositivo][lnk-device-sdks] que são compatíveis com o protocolo MQTT estão disponíveis para Java, Node.js, C, C# e Python. Os SDKs do dispositivo usam a cadeia de conexão do Hub IoT padrão para estabelecer uma conexão com um Hub IoT. Para usar o protocolo MQTT, o parâmetro do protocolo do cliente deve ser definido como **MQTT**. Por padrão, os SDKs do dispositivo se conectam a um Hub IoT com o sinalizador **CleanSession** definido como **0** e usam **QoS 1** para troca de mensagens com o Hub IoT.

Quando um dispositivo está conectado a um Hub IoT, os SDKs do dispositivo fornecem métodos que permitem que o dispositivo troque mensagens com um Hub IoT.

A tabela a seguir contém links para exemplos de código de cada linguagem compatível e especifica o parâmetro a ser usado para estabelecer uma conexão com o Hub IoT usando o protocolo MQTT.

| Linguagem | Parâmetro do protocolo |
| --- | --- |
| [Node.js][lnk-sample-node] |azure-iot-device-mqtt |
| [Java][lnk-sample-java] |IotHubClientProtocol.MQTT |
| [C][lnk-sample-c] |MQTT_Protocol |
| [C#][lnk-sample-csharp] |TransportType.Mqtt |
| [Python][lnk-sample-python] |IoTHubTransportProvider.MQTT |

### <a name="migrating-a-device-app-from-amqp-to-mqtt"></a>Migrando um aplicativo de dispositivo de AMQP para MQTT

Se você estiver usando o [SDKs de cliente de dispositivo][lnk-device-sdks], a mudança do uso de AMQP para MQTT exige alteração do parâmetro de protocolo na inicialização do cliente, conforme mencionado anteriormente.

Ao fazer isso, verifique os seguintes itens:

* AMQP retorna erros para várias condições, enquanto MQTT encerra a conexão. Como resultado, sua lógica de manipulação de exceções pode exigir algumas alterações.
* MQTT não dá suporte a operações de *rejeição* quando recebe [mensagens da nuvem para do dispositivo][lnk-messaging]. Se seu aplicativo de back-end precisar receber uma resposta do aplicativo do dispositivo, considere usar [métodos diretos][lnk-methods].

## <a name="using-the-mqtt-protocol-directly"></a>Usando o protocolo MQTT diretamente

Se um dispositivo não puder usar os SDKs do dispositivo, ele poderá se conectar com os pontos de extremidade públicos do dispositivo usando o protocolo MQTT na porta 8883. No pacote **CONNECT** , o dispositivo deve usar os seguintes valores:

* No campo **ClientId**, use o **deviceId**.

* Para o campo **Username**, use `{iothubhostname}/{device_id}/api-version=2016-11-14`, onde `{iothubhostname}` é o CName completo do Hub IoT.

    Por exemplo, se o nome de seu Hub IoT for **contoso.azure-devices.net** e se o nome do dispositivo for **MyDevice01**, o campo **Username** completo deverá conter:

    `contoso.azure-devices.net/MyDevice01/api-version=2016-11-14`

* No campo **Senha** use um token SAS. O formato do token SAS é o mesmo, conforme descrito para os protocolos HTTPS e AMQP:

  `SharedAccessSignature sig={signature-string}&se={expiry}&sr={URL-encoded-resourceURI}`

  > [!NOTE]
  > Se você usa a autenticação de certificado X.509, as senhas de token SAS não são necessárias. Para saber mais, confira [Configurar a segurança de X.509 em seu Hub IoT do Azure][lnk-x509]

  Para saber mais sobre como gerar tokens SAS, confira a seção de dispositivo de [Usar tokens de segurança do Hub IoT][lnk-sas-tokens].

  Durante o teste, você também pode usar a ferramenta [gerenciador de dispositivo][lnk-device-explorer] para gerar rapidamente um token SAS que pode ser copiado e colado em seu próprio código:

  1. Acesse a guia **Gerenciamento** no **Gerenciador de Dispositivo**.
  2. Clique em **Token SAS** (parte superior direita).
  3. Em **SASTokenForm**, selecione seu dispositivo no menu suspenso **DeviceID**. Defina o **TTL**.
  4. Clique em **Gerar** para criar o token.

     O token de SAS gerado tem a seguinte estrutura:

     `HostName={your hub name}.azure-devices.net;DeviceId=javadevice;SharedAccessSignature=SharedAccessSignature sr={your hub name}.azure-devices.net%2Fdevices%2FMyDevice01%2Fapi-version%3D2016-11-14&sig=vSgHBMUG.....Ntg%3d&se=1456481802`

     A parte desse token para usar como o campo **Senha** para conectar usando MQTT é:

     `SharedAccessSignature sr={your hub name}.azure-devices.net%2Fdevices%2FMyDevice01%2Fapi-version%3D2016-11-14&sig=vSgHBMUG.....Ntg%3d&se=1456481802`

Para que o MQTT conecte e desconecte pacotes, o Hub IoT emite um evento no canal **Monitoramento de Operações** . Este evento possui informações adicionais que podem ajudá-lo a solucionar problemas de conectividade.

### <a name="tlsssl-configuration"></a>Configuração de TLS/SSL

Para usar o protocolo MQTT diretamente, o cliente *deve* se conectar por TLS/SSL. Tentativas de ignorar essa etapa falham com erros de conexão.

Para estabelecer uma conexão TLS, você precisará baixar e referenciar o certificado de raiz DigiCert Baltimore. Esse certificado é utilizado pelo Azure para proteger a conexão. É possível localizar esse certificado no repositório [Azure-iot-sdk-c][lnk-sdk-c-certs]. Para obter mais informações sobre esses certificados, acesse o [site da Digicert][lnk-digicert-root-certs].

Um exemplo de como implementar isso usando a versão do Python da [biblioteca Paho MQTT] [ lnk-paho] pelo Eclipse Foundation se parece com o seguinte.

Primeiro, instale a biblioteca Paho do seu ambiente de linha de comando:

```cmd/sh
pip install paho-mqtt
```

Em seguida, implemente o cliente em um script Python. Substitua os espaços reservados conforme a seguir:

* `<local path to digicert.cer>` é o caminho para um arquivo local que contém o certificado DigiCert Baltimore Root. Você pode criar esse arquivo, copiando as informações sobre o certificado do [certs.c](https://github.com/Azure/azure-iot-sdk-c/blob/master/certs/certs.c) no SDK de IoT do Azure para C. Inclua as linhas `-----BEGIN CERTIFICATE-----` e `-----END CERTIFICATE-----`, remova as marcas `"` no início e no final de cada linha, e remova os caracteres `\r\n` no final de cada linha.
* `<device id from device registry>` é a ID de um dispositivo que você adicionou ao seu Hub IoT.
* `<generated SAS token>` é um token de SAS para o dispositivo criado, conforme descrito anteriormente neste artigo.
* `<iot hub name>` o nome do seu Hub IoT.

```python
from paho.mqtt import client as mqtt
import ssl

path_to_root_cert = "<local path to digicert.cer>"
device_id = "<device id from device registry>"
sas_token = "<generated SAS token>"
iot_hub_name = "<iot hub name>"

def on_connect(client, userdata, flags, rc):
  print ("Device connected with result code: " + str(rc))
def on_disconnect(client, userdata, rc):
  print ("Device disconnected with result code: " + str(rc))
def on_publish(client, userdata, mid):
  print ("Device sent message")

client = mqtt.Client(client_id=device_id, protocol=mqtt.MQTTv311)

client.on_connect = on_connect
client.on_disconnect = on_disconnect
client.on_publish = on_publish

client.username_pw_set(username=iot_hub_name+".azure-devices.net/" + device_id, password=sas_token)

client.tls_set(ca_certs=path_to_root_cert, certfile=None, keyfile=None, cert_reqs=ssl.CERT_REQUIRED, tls_version=ssl.PROTOCOL_TLSv1, ciphers=None)
client.tls_insecure_set(False)

client.connect(iot_hub_name+".azure-devices.net", port=8883)

client.publish("devices/" + device_id + "/messages/events/", "{id=123}", qos=1)
client.loop_forever()
```

### <a name="sending-device-to-cloud-messages"></a>Enviando mensagens de dispositivo para nuvem

Depois de fazer uma conexão bem-sucedida, um dispositivo pode enviar mensagens ao IoT Hub usando `devices/{device_id}/messages/events/` ou `devices/{device_id}/messages/events/{property_bag}` como um **nome de tópico**. O elemento `{property_bag}` habilita o dispositivo a enviar mensagens com propriedades adicionais em um formato codificado de URL. Por exemplo: 

```text
RFC 2396-encoded(<PropertyName1>)=RFC 2396-encoded(<PropertyValue1>)&RFC 2396-encoded(<PropertyName2>)=RFC 2396-encoded(<PropertyValue2>)…
```

> [!NOTE]
> Esse elemento `{property_bag}` usa a mesma codificação para cadeias de caracteres de consulta do protocolo HTTPS.

O aplicativo do dispositivo também pode usar `devices/{device_id}/messages/events/{property_bag}` como o **nome do tópico Will** para definir *mensagens Will* a serem encaminhadas como uma mensagem de telemetria.

* O Hub IoT não dá suporte a mensagens de QoS 2. Se um aplicativo de dispositivo publicar uma mensagem com o **QoS 2**, o Hub IoT fechará a conexão de rede.
* O Hub IoT não persiste mensagens de retenção. Se um dispositivo envia uma mensagem com o sinalizador **RETAIN** definido como 1, o Hub IoT adiciona a propriedade de aplicativo **x-opt-retain** à mensagem. Nesse caso, em vez de persistir a mensagem de retenção, o Hub IoT a transmite ao aplicativo de back-end.
* O Hub IoT oferece suporte apenas a uma conexão MQTT ativa por dispositivo. Qualquer nova conexão MQTT em nome da mesma ID de dispositivo faz com que o Hub IoT descarte a conexão existente.

Para obter mais informações, consulte [Guia do desenvolvedor do sistema de mensagens][lnk-messaging].

### <a name="receiving-cloud-to-device-messages"></a>Recebendo mensagens da nuvem para o dispositivo

Para receber mensagens do Hub IoT, um dispositivo deve fazer uma assinatura usando `devices/{device_id}/messages/devicebound/#` como um **Filtro do Tópico**. O curinga de vários níveis `#` no filtro de tópico é usado apenas para permitir que o dispositivo receba propriedades adicionais no nome do tópico. O Hub IoT não permite o uso dos caracteres curinga `#` ou `?` para filtragem de subtópicos. Como o Hub IoT não é um agente de mensagens pub-sub de uso geral, ele dá suporte somente para nomes de tópicos e filtros tópicos documentados.

O dispositivo só receberá mensagens do Hub IoT depois de assinar com êxito o ponto de extremidade específico ao dispositivo, representado pelo filtro de tópico `devices/{device_id}/messages/devicebound/#`. Depois que uma assinatura é estabelecida, o dispositivo receberá mensagens de nuvem para dispositivo que foram enviadas após o horário da assinatura. Se o dispositivo se conectar com o sinalizador **CleanSession** definido como **0**, a assinatura será persistida entre as diferentes sessões. Neste caso, a próxima vez que o dispositivo conectar-se com **CleanSession 0**, ele receberá todas as mensagens pendentes enviadas a ele enquanto estava desconectado. Se o dispositivo usar o sinalizador **CleanSession** definido como **1**, não receberá todas as mensagens do Hub IoT até que ele se inscreva no ponto de extremidade do dispositivo.

O Hub IoT entrega mensagens com o **Nome do Tópico** `devices/{device_id}/messages/devicebound/` ou `devices/{device_id}/messages/devicebound/{property_bag}` quando há propriedades de mensagens. `{property_bag}` contém pares de chave/valor codificados de URL das propriedades da mensagem. Somente propriedades de aplicativo e propriedades do sistema definível pelo usuário (como **messageId** ou **correlationId**) estão incluídas no recipiente de propriedades. Os nomes de propriedade do sistema têm o prefixo **$**; as propriedades de aplicativo usam o nome da propriedade original sem prefixo.

Quando um aplicativo do dispositivo assina um tópico com o **QoS 2**, o Hub IoT concede, no máximo, o nível 1 do QoS no pacote **SUBACK**. Depois disso, o Hub IoT entrega mensagens ao dispositivo usando QoS 1.

### <a name="retrieving-a-device-twins-properties"></a>Recuperando as propriedades de um dispositivo gêmeo

Primeiro, um dispositivo assina `$iothub/twin/res/#` para receber respostas da operação. Em seguida, ele envia uma mensagem vazia ao tópico `$iothub/twin/GET/?$rid={request id}`, com um valor preenchido como a **ID da solicitação**. O serviço então envia uma mensagem de resposta que contém os dados do dispositivo gêmeo no tópico `$iothub/twin/res/{status}/?$rid={request id}`, usando a mesma **ID de solicitação** da solicitação.

A ID da solicitação pode ser qualquer valor válido de um valor da propriedade de mensagem, de acordo com o [Guia do desenvolvedor do sistema de mensagens do Hub IoT][lnk-messaging] e o status é validado como um inteiro.

O corpo de resposta contém a seção de propriedades do dispositivo gêmeo. O trecho de código a seguir mostra o corpo da entrada de registro de identidade limitada ao membro "propriedades", por exemplo:

```json
{
    "properties": {
        "desired": {
            "telemetrySendFrequency": "5m",
            "$version": 12
        },
        "reported": {
            "telemetrySendFrequency": "5m",
            "batteryLevel": 55,
            "$version": 123
        }
    }
}
```

Os códigos de status possíveis são:

|Status | DESCRIÇÃO |
| ----- | ----------- |
| 200 | Sucesso |
| 429 | Número excessivo de solicitações (limitado), de acordo com a [Limitação do Hub IoT][lnk-quotas] |
| 5** | Erros do servidor |

Para obter mais informações, consulte [Guia do desenvolvedor de dispositivos gêmeos][lnk-devguide-twin].

### <a name="update-device-twins-reported-properties"></a>Atualizar as propriedades relatadas do dispositivo gêmeo

A sequência a seguir descreve como um dispositivo atualiza as propriedades relatadas no dispositivo gêmeo no Hub IoT:

1. Primeiro, um dispositivo deve assinar o tópico `$iothub/twin/res/#` para receber respostas da operação do Hub IoT.

1. Um dispositivo envia uma mensagem que contém a atualização do dispositivo gêmeo para o tópico `$iothub/twin/PATCH/properties/reported/?$rid={request id}`. Essa mensagem inclui um valor de **id da solicitação**.

1. Em seguida, o serviço envia uma mensagem de resposta que contém o novo valor de ETag para a coleção de propriedades relatadas no tópico `$iothub/twin/res/{status}/?$rid={request id}`. Essa mensagem de resposta usa a mesma **id de solicitação** da solicitação.

O corpo da mensagem de solicitação contém um documento JSON, que contém novos valores para propriedades relatadas. Cada membro no documento JSON atualiza ou adiciona o membro correspondente no documento do dispositivo gêmeo. Um membro definido como `null` exclui o membro do objeto recipiente. Por exemplo: 

```json
{
    "telemetrySendFrequency": "35m",
    "batteryLevel": 60
}
```

Os códigos de status possíveis são:

|Status | DESCRIÇÃO |
| ----- | ----------- |
| 200 | Sucesso |
| 400 | Solicitação inválida. JSON malformado |
| 429 | Número excessivo de solicitações (limitado), de acordo com a [Limitação do Hub IoT][lnk-quotas] |
| 5** | Erros do servidor |

Para obter mais informações, consulte [Guia do desenvolvedor de dispositivos gêmeos][lnk-devguide-twin].

### <a name="receiving-desired-properties-update-notifications"></a>Recebendo notificações de atualização de propriedades desejadas

Quando um dispositivo é conectado, o Hub IoT envia notificações para o tópico `$iothub/twin/PATCH/properties/desired/?$version={new version}`, que contêm o conteúdo da atualização executada pelo back-end da solução. Por exemplo: 

```json
{
    "telemetrySendFrequency": "5m",
    "route": null
}
```

Em relação às atualizações de propriedade, valores `null` significam que o membro do objeto JSON está sendo excluído.

> [!IMPORTANT]
> O Hub IoT gera notificações de alteração somente quando os dispositivos estão conectados. Lembre-se de implementar o [fluxo de reconexão do dispositivo][lnk-devguide-twin-reconnection] para manter as propriedades desejadas sincronizadas entre o Hub IoT e o aplicativo do dispositivo.

Para obter mais informações, consulte [Guia do desenvolvedor de dispositivos gêmeos][lnk-devguide-twin].

### <a name="respond-to-a-direct-method"></a>Responder a um método direto

Primeiro, um dispositivo precisa assinar `$iothub/methods/POST/#`. O Hub IoT envia solicitações de método para o tópico `$iothub/methods/POST/{method name}/?$rid={request id}` com um corpo vazio ou JSON válido.

Para responder, o dispositivo envia uma mensagem com um JSON válido ou um corpo vazio para o tópico `$iothub/methods/res/{status}/?$rid={request id}`. Nessa mensagem, a **ID da Solicitação** deve corresponder com o da mensagem de solicitação e o **status** deve ser um número inteiro.

Para obter mais informações, consulte [Guia do desenvolvedor do método direto][lnk-methods].

### <a name="additional-considerations"></a>Considerações adicionais

Como uma consideração final, se for necessário personalizar o comportamento do protocolo MQTT na nuvem, você deverá revisar o [gateway de protocolo de IoT do Azure ][lnk-azure-protocol-gateway]. Esse software permite que você implante um gateway de protocolo personalizado de alto desempenho que faz interface diretamente com o Hub IoT. O gateway do protocolo IoT do Azure permite que você personalize o protocolo de dispositivo para acomodar as implantações de MQTT de nível industrial ou outros protocolos personalizados. Essa abordagem exige, no entanto, que você execute e opere um gateway de protocolo personalizado.

## <a name="next-steps"></a>Próximas etapas

Para saber mais sobre o protocolo MQTT, consulte a [documentação do MQTT][lnk-mqtt-docs].

Para saber mais sobre como planejar sua implantação do Hub IoT, consulte:

* [Catálogo de dispositivos Azure Certified para IoT][lnk-devices]
* [Suporte a protocolos adicionais][lnk-protocols]
* [Comparar com Hubs de Eventos][lnk-compare]
* [Escala, alta disponibilidade e recuperação de desastre][lnk-scaling]

Para explorar melhor as funcionalidades do Hub IoT, consulte:

* [Guia do desenvolvedor do Hub IoT][lnk-devguide]
* [Implantação do IA em dispositivos de borda com o Azure IoT Edge][lnk-iotedge]

[lnk-device-sdks]: https://github.com/Azure/azure-iot-sdks
[lnk-mqtt-org]: http://mqtt.org/
[lnk-mqtt-docs]: http://mqtt.org/documentation
[lnk-sample-node]: https://github.com/Azure/azure-iot-sdk-node/blob/master/device/samples/simple_sample_device.js
[lnk-sample-java]: https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-samples/send-receive-sample/src/main/java/samples/com/microsoft/azure/sdk/iot/SendReceive.java
[lnk-sample-c]: https://github.com/Azure/azure-iot-sdk-c/tree/master/iothub_client/samples/iothub_client_sample_mqtt
[lnk-sample-csharp]: https://github.com/Azure/azure-iot-sdk-csharp/tree/master/device/samples
[lnk-sample-python]: https://github.com/Azure/azure-iot-sdk-python/tree/master/device/samples
[lnk-device-explorer]: https://github.com/Azure/azure-iot-sdk-csharp/blob/master/tools/DeviceExplorer
[lnk-sas-tokens]: iot-hub-devguide-security.md#use-sas-tokens-in-a-device-app
[lnk-azure-protocol-gateway]: iot-hub-protocol-gateway.md

[lnk-devices]: https://catalog.azureiotsuite.com/
[lnk-protocols]: iot-hub-protocol-gateway.md
[lnk-compare]: iot-hub-compare-event-hubs.md
[lnk-scaling]: iot-hub-scaling.md
[lnk-devguide]: iot-hub-devguide.md
[lnk-iotedge]: ../iot-edge/tutorial-simulate-device-linux.md
[lnk-x509]: iot-hub-security-x509-get-started.md

[lnk-methods]: iot-hub-devguide-direct-methods.md
[lnk-messaging]: iot-hub-devguide-messaging.md

[lnk-quotas]: iot-hub-devguide-quotas-throttling.md
[lnk-devguide-twin-reconnection]: iot-hub-devguide-device-twins.md#device-reconnection-flow
[lnk-devguide-twin]: iot-hub-devguide-device-twins.md
[lnk-sdk-c-certs]: https://github.com/Azure/azure-iot-sdk-c/blob/master/certs/certs.c
[lnk-digicert-root-certs]: https://www.digicert.com/digicert-root-certificates.htm
[lnk-paho]: https://pypi.python.org/pypi/paho-mqtt
