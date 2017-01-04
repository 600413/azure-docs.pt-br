---
title: Conectar um dispositivo usando o C no Linux | Microsoft Docs
description: "Descreve como conectar um dispositivo à solução pré-configurada de monitoramento remoto do Azure IoT Suite usando um aplicativo escrito em C em execução no Linux."
services: 
suite: iot-suite
documentationcenter: na
author: dominicbetts
manager: timlt
editor: 
ms.assetid: 0c7c8039-0bbf-4bb5-9e79-ed8cff433629
ms.service: iot-suite
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/05/2016
ms.author: dobett
translationtype: Human Translation
ms.sourcegitcommit: 6fdcdc323cff07d7debd46ab10b17ba7e9d8781a
ms.openlocfilehash: d1a6ffe59c5cb4fd0575ba02aa9768548461b869


---
# <a name="connect-your-device-to-the-remote-monitoring-preconfigured-solution-linux"></a>Conectar seu dispositivo à solução pré-configurada de monitoramento remoto (Linux)
[!INCLUDE [iot-suite-selector-connecting](../../includes/iot-suite-selector-connecting.md)]

## <a name="build-and-run-a-sample-c-client-linux"></a>Criar e executar um cliente C de exemplo Linux
Os procedimentos a seguir mostram como criar um aplicativo cliente, escrito em C, compilado e executado no Ubuntu Linux, que se comunica com a solução pré-configurada de monitoramento remoto. Para concluir essas etapas, você precisa de um dispositivo em execução na versão 15.04 ou 15.10 do Ubuntu. Antes de continuar, instale os pacotes de pré-requisitos no seu dispositivo Ubuntu usando o seguinte comando:

```
sudo apt-get install cmake gcc g++
```

## <a name="install-the-client-libraries-on-your-device"></a>Instalar as bibliotecas de cliente no dispositivo
As bibliotecas de cliente do Hub IoT do Azure estão disponíveis como um pacote que você pode instalar em seu dispositivo Ubuntu usando o comando **apt-get** . Conclua as seguintes etapas para instalar o pacote que contém os arquivos de biblioteca e cabeçalho do cliente Hub IoT no seu computador Ubuntu:

1. Adicione o repositório AzureIoT ao computador:
   
    ```
    sudo add-apt-repository ppa:aziotsdklinux/ppa-azureiot
    sudo apt-get update
    ```
2. Instale o pacote azure-iot-sdk-c-dev
   
    ```
    sudo apt-get install -y azure-iot-sdk-c-dev
    ```

## <a name="add-code-to-specify-the-behavior-of-the-device"></a>Adicionar código para especificar o comportamento do dispositivo
No computador Ubuntu, crie uma pasta chamada **remote\_monitoring**. Na pasta **remote\_monitoring**, crie quatro arquivos: **main.c**, **remote\_monitoring.c**, **remote\_monitoring.h** e **CMakeLists.txt**.

As bibliotecas de cliente serializador do Hub IoT usam um modelo para especificar o formato das mensagens que o dispositivo envia ao Hub IoT e os comandos recebidos do Hub IoT.

1. Em um editor de texto, abra o arquivo **remote\_monitoring.c**. Adicione as seguintes declarações de `#include` :
   
    ```
    #include "iothubtransportamqp.h"
    #include "schemalib.h"
    #include "iothub_client.h"
    #include "serializer.h"
    #include "schemaserializer.h"
    #include "azure_c_shared_utility/threadapi.h"
    #include "azure_c_shared_utility/platform.h"
    ```
2. Adicione as declarações de variável a seguir após as instruções `#include` . Substitua os valores do espaço reservado [ID do Dispositivo] e [Chave do Dispositivo] pelos valores do seu dispositivo no painel da solução de monitoramento remoto. Use o nome de host do Hub IoT no painel para substituir [IoTHub Name]. Por exemplo, se o nome de host do Hub IoT for **contoso.azure-devices.net**, substitua [Nome do HubIoT] por **contoso**:
   
    ```
    static const char* deviceId = "[Device Id]";
    static const char* deviceKey = "[Device Key]";
    static const char* hubName = "IoTHub Name]";
    static const char* hubSuffix = "azure-devices.net";
    ```
3. Adicione o código a seguir para definir o modelo que permite ao dispositivo se comunicar com o Hub IoT. Esse modelo especifica que o dispositivo envia temperatura, temperatura externa, umidade e uma ID de dispositivo como telemetria. O dispositivo também envia metadados sobre o dispositivo ao Hub IoT, incluindo uma lista de comandos permitidos pelo dispositivo. Esse dispositivo responde aos comandos **SetTemperature** e **SetHumidity**:
   
    ```
    // Define the Model
    BEGIN_NAMESPACE(Contoso);
   
    DECLARE_STRUCT(SystemProperties,
      ascii_char_ptr, DeviceID,
      _Bool, Enabled
    );
   
    DECLARE_STRUCT(DeviceProperties,
      ascii_char_ptr, DeviceID,
      _Bool, HubEnabledState
    );
   
    DECLARE_MODEL(Thermostat,
   
      /* Event data (temperature, external temperature and humidity) */
      WITH_DATA(int, Temperature),
      WITH_DATA(int, ExternalTemperature),
      WITH_DATA(int, Humidity),
      WITH_DATA(ascii_char_ptr, DeviceId),
   
      /* Device Info - This is command metadata + some extra fields */
      WITH_DATA(ascii_char_ptr, ObjectType),
      WITH_DATA(_Bool, IsSimulatedDevice),
      WITH_DATA(ascii_char_ptr, Version),
      WITH_DATA(DeviceProperties, DeviceProperties),
      WITH_DATA(ascii_char_ptr_no_quotes, Commands),
   
      /* Commands implemented by the device */
      WITH_ACTION(SetTemperature, int, temperature),
      WITH_ACTION(SetHumidity, int, humidity)
    );
   
    END_NAMESPACE(Contoso);
    ```

### <a name="add-code-to-implement-the-behavior-of-the-device"></a>Adicionar código para implementar o comportamento do dispositivo
Adicione as funções a serem executadas quando o dispositivo receber um comando do hub e o código para enviar telemetria simulada ao hub.

1. Adicione as seguintes funções que serão executadas quando o dispositivo receber os comandos **SetTemperature** e **SetHumidity** definidos no modelo:
   
    ```
    EXECUTE_COMMAND_RESULT SetTemperature(Thermostat* thermostat, int temperature)
    {
      (void)printf("Received temperature %d\r\n", temperature);
      thermostat->Temperature = temperature;
      return EXECUTE_COMMAND_SUCCESS;
    }
   
    EXECUTE_COMMAND_RESULT SetHumidity(Thermostat* thermostat, int humidity)
    {
      (void)printf("Received humidity %d\r\n", humidity);
      thermostat->Humidity = humidity;
      return EXECUTE_COMMAND_SUCCESS;
    }
    ```
2. Adicione a seguinte função que envia uma mensagem ao Hub IoT:
   
    ```
    static void sendMessage(IOTHUB_CLIENT_HANDLE iotHubClientHandle, const unsigned char* buffer, size_t size)
    {
      IOTHUB_MESSAGE_HANDLE messageHandle = IoTHubMessage_CreateFromByteArray(buffer, size);
      if (messageHandle == NULL)
      {
        printf("unable to create a new IoTHubMessage\r\n");
      }
      else
      {
        if (IoTHubClient_SendEventAsync(iotHubClientHandle, messageHandle, NULL, NULL) != IOTHUB_CLIENT_OK)
        {
          printf("failed to hand over the message to IoTHubClient");
        }
        else
        {
          printf("IoTHubClient accepted the message for delivery\r\n");
        }
   
        IoTHubMessage_Destroy(messageHandle);
      }
    free((void*)buffer);
    }
    ```
3. Adicione a seguinte função que conecta a biblioteca de serialização no SDK:
   
    ```
    static IOTHUBMESSAGE_DISPOSITION_RESULT IoTHubMessage(IOTHUB_MESSAGE_HANDLE message, void* userContextCallback)
    {
      IOTHUBMESSAGE_DISPOSITION_RESULT result;
      const unsigned char* buffer;
      size_t size;
      if (IoTHubMessage_GetByteArray(message, &buffer, &size) != IOTHUB_MESSAGE_OK)
      {
        printf("unable to IoTHubMessage_GetByteArray\r\n");
        result = EXECUTE_COMMAND_ERROR;
      }
      else
      {
        /*buffer is not zero terminated*/
        char* temp = malloc(size + 1);
        if (temp == NULL)
        {
          printf("failed to malloc\r\n");
          result = EXECUTE_COMMAND_ERROR;
        }
        else
        {
          memcpy(temp, buffer, size);
          temp[size] = '\0';
          EXECUTE_COMMAND_RESULT executeCommandResult = EXECUTE_COMMAND(userContextCallback, temp);
          result =
            (executeCommandResult == EXECUTE_COMMAND_ERROR) ? IOTHUBMESSAGE_ABANDONED :
            (executeCommandResult == EXECUTE_COMMAND_SUCCESS) ? IOTHUBMESSAGE_ACCEPTED :
            IOTHUBMESSAGE_REJECTED;
          free(temp);
        }
      }
      return result;
    }
    ```
4. Adicione a função a seguir para se conectar ao Hub IoT, enviar e receber mensagens e se desconectar do hub. Observe como o dispositivo envia metadados sobre si, incluindo os comandos que ele suporta, ao hub IoT quando ele se conecta. Esses metadados permitem que a solução atualize o status do dispositivo para **Executando** no painel:
   
    ```
    void remote_monitoring_run(void)
    {
      if (platform_init() != 0)
      {
        printf("Failed to initialize the platform.\r\n");
      }
      else
      {
        if (serializer_init(NULL) != SERIALIZER_OK)
        {
          printf("Failed on serializer_init\r\n");
        }
        else
        {
          IOTHUB_CLIENT_CONFIG config;
          IOTHUB_CLIENT_HANDLE iotHubClientHandle;
   
          config.deviceId = deviceId;
          config.deviceKey = deviceKey;
          config.deviceSasToken = NULL;
          config.iotHubName = hubName;
          config.iotHubSuffix = hubSuffix;
          config.protocol = AMQP_Protocol;
          iotHubClientHandle = IoTHubClient_Create(&config);
          if (iotHubClientHandle == NULL)
          {
            (void)printf("Failed on IoTHubClient_CreateFromConnectionString\r\n");
          }
          else
          {
            Thermostat* thermostat = CREATE_MODEL_INSTANCE(Contoso, Thermostat);
            if (thermostat == NULL)
            {
              (void)printf("Failed on CREATE_MODEL_INSTANCE\r\n");
            }
            else
            {
              STRING_HANDLE commandsMetadata;
   
              if (IoTHubClient_SetMessageCallback(iotHubClientHandle, IoTHubMessage, thermostat) != IOTHUB_CLIENT_OK)
              {
                printf("unable to IoTHubClient_SetMessageCallback\r\n");
              }
              else
              {
   
                /* send the device info upon startup so that the cloud app knows
                   what commands are available and the fact that the device is up */
                thermostat->ObjectType = "DeviceInfo";
                thermostat->IsSimulatedDevice = false;
                thermostat->Version = "1.0";
                thermostat->DeviceProperties.HubEnabledState = true;
                thermostat->DeviceProperties.DeviceID = (char*)deviceId;
   
                commandsMetadata = STRING_new();
                if (commandsMetadata == NULL)
                {
                  (void)printf("Failed on creating string for commands metadata\r\n");
                }
                else
                {
                  /* Serialize the commands metadata as a JSON string before sending */
                  if (SchemaSerializer_SerializeCommandMetadata(GET_MODEL_HANDLE(Contoso, Thermostat), commandsMetadata) != SCHEMA_SERIALIZER_OK)
                  {
                    (void)printf("Failed serializing commands metadata\r\n");
                  }
                  else
                  {
                    unsigned char* buffer;
                    size_t bufferSize;
                    thermostat->Commands = (char*)STRING_c_str(commandsMetadata);
   
                    /* Here is the actual send of the Device Info */
                    if (SERIALIZE(&buffer, &bufferSize, thermostat->ObjectType, thermostat->Version, thermostat->IsSimulatedDevice, thermostat->DeviceProperties, thermostat->Commands) != CODEFIRST_OK)
                    {
                      (void)printf("Failed serializing\r\n");
                    }
                    else
                    {
                      sendMessage(iotHubClientHandle, buffer, bufferSize);
                    }
   
                  }
   
                  STRING_delete(commandsMetadata);
                }
   
                thermostat->Temperature = 50;
                thermostat->ExternalTemperature = 55;
                thermostat->Humidity = 50;
                thermostat->DeviceId = (char*)deviceId;
   
                while (1)
                {
                  unsigned char*buffer;
                  size_t bufferSize;
   
                  (void)printf("Sending sensor value Temperature = %d, Humidity = %d\r\n", thermostat->Temperature, thermostat->Humidity);
   
                  if (SERIALIZE(&buffer, &bufferSize, thermostat->DeviceId, thermostat->Temperature, thermostat->Humidity, thermostat->ExternalTemperature) != CODEFIRST_OK)
                  {
                    (void)printf("Failed sending sensor value\r\n");
                  }
                  else
                  {
                    sendMessage(iotHubClientHandle, buffer, bufferSize);
                  }
   
                  ThreadAPI_Sleep(1000);
                }
              }
   
              DESTROY_MODEL_INSTANCE(thermostat);
            }
            IoTHubClient_Destroy(iotHubClientHandle);
          }
          serializer_deinit();
        }
        platform_deinit();
      }
    }
    ```
   
    Para referência, veja um exemplo de mensagem de **DeviceInfo** enviada ao Hub IoT na inicialização:
   
    ```
    {
      "ObjectType":"DeviceInfo",
      "Version":"1.0",
      "IsSimulatedDevice":false,
      "DeviceProperties":
      {
        "DeviceID":"mydevice01", "HubEnabledState":true
      }, 
      "Commands":
      [
        {"Name":"SetHumidity", "Parameters":[{"Name":"humidity","Type":"double"}]},
        { "Name":"SetTemperature", "Parameters":[{"Name":"temperature","Type":"double"}]}
      ]
    }
    ```
   
    Para referência, veja um exemplo de mensagem de **Telemetria** enviada ao Hub IoT:
   
    ```
    {"DeviceId":"mydevice01", "Temperature":50, "Humidity":50, "ExternalTemperature":55}
    ```
   
    Para referência, veja um exemplo de **Comando** recebido do Hub IoT:
   
    ```
    {
      "Name":"SetHumidity",
      "MessageId":"2f3d3c75-3b77-4832-80ed-a5bb3e233391",
      "CreatedTime":"2016-03-11T15:09:44.2231295Z",
      "Parameters":{"humidity":23}
    }
    ```

### <a name="add-code-to-invoke-the-remotemonitoringrun-function"></a>Adicionar código para invocar a função remote_monitoring_run
Em um editor de texto, abra o arquivo **remote_monitoring.h**. Adicione os códigos a seguir:

```
void remote_monitoring_run(void);
```

Em um editor de texto, abra o arquivo **main.c** . Adicione os códigos a seguir:

```
#include "remote_monitoring.h"

int main(void)
{
    remote_monitoring_run();

    return 0;
}
```

## <a name="use-cmake-to-build-the-client-application"></a>Usar CMake para criar o aplicativo cliente
As etapas a seguir descrevem como usar *CMake* para compilar o aplicativo cliente.

1. Em um editor de texto, abra o arquivo **CMakeLists.txt** na pasta **remote_monitoring**.
2. Adicione as seguintes instruções para definir como criar o aplicativo cliente:
   
    ```
    cmake_minimum_required(VERSION 2.8.11)
   
    set(AZUREIOT_INC_FOLDER ".." "/usr/include/azureiot")
   
    include_directories(${AZUREIOT_INC_FOLDER})
   
    set(sample_application_c_files
        ./remote_monitoring.c
        ./main.c
    )
   
    set(sample_application_h_files
        ./remote_monitoring.h
    )
   
    add_executable(sample_app ${sample_application_c_files} ${sample_application_h_files})
   
    target_link_libraries(sample_app
        serializer
        iothub_client
        iothub_client_amqp_transport
        aziotsharedutil
        uamqp
        pthread
        curl
        ssl
        crypto
        m
    )
    ```
3. Na pasta **remote_monitoring**, crie uma pasta para armazenar os arquivos *make* que CMake gera e execute os comandos **cmake** e **make** como se segue:
   
    ```
    mkdir cmake
    cd cmake
    cmake ../
    make
    ```
4. Execute o aplicativo cliente e envie telemetria ao Hub IoT:
   
    ```
    ./sample_app
    ```

[!INCLUDE [iot-suite-visualize-connecting](../../includes/iot-suite-visualize-connecting.md)]




<!--HONumber=Nov16_HO3-->


