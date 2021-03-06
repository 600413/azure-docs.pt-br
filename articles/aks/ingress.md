---
title: "Configurar entrada com o cluster do AKS (Serviço de Contêiner do Azure)"
description: "Instale e configure um controlador de entrada NGINX em um cluster do AKS (Serviço de Contêiner do Azure)."
services: container-service
author: neilpeterson
manager: timlt
ms.service: container-service
ms.topic: article
ms.date: 2/21/2018
ms.author: nepeters
ms.custom: mvc
ms.openlocfilehash: 4d5f1ddbb3d56d4c7af90ddb4f7d37f082a751c8
ms.sourcegitcommit: c765cbd9c379ed00f1e2394374efa8e1915321b9
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/28/2018
---
# <a name="https-ingress-on-azure-container-service-aks"></a>Entrada HTTPS no AKS (Serviço de Contêiner do Azure)

Um controlador de entrada é uma parte do software que fornece proxy reverso, roteamento de tráfego configurável e terminação TLS para serviços de Kubernetes. Os recursos de entrada de Kubernetes são usados para configurar as regras de entrada e as rotas para os serviços de Kubernetes individuais. Usando um controlador de entrada e regras de ingresso, um único endereço externo pode ser usado para rotear tráfego a vários serviços em um cluster de Kubernetes.

Este documento descreve uma implantação de exemplo do [Controlador de entrada NGINX][nginx-ingress] no cluster do AKS (Serviço de Contêiner do Azure). Além disso, o projeto [KUBE-LEGO][kube-lego] é usado para gerar e configurar automaticamente certificados [Vamos Criptografar][lets-encrypt]. Finalmente, vários aplicativos executam no cluster do AKS, cada um dos quais é acessível em um único endereço.

## <a name="install-an-ingress-controller"></a>Instalar um controlador de entrada

Use Helm para instalar o controlador de entrada NGINX. Consulte a [documentação][nginx-ingress] do controlador de entrada NGINX para obter informações detalhadas sobre a implantação. 

```
helm install stable/nginx-ingress
```

Durante a instalação, um endereço IP público do Azure é criado para o controlador de entrada. Para obter o endereço IP público, use o comando de serviço kubectl get. Talvez demore um pouco até que o endereço IP seja atribuído ao serviço.

```console
$ kubectl get service

NAME                                          TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)                      AGE
kubernetes                                    ClusterIP      10.0.0.1       <none>           443/TCP                      3d
toned-spaniel-nginx-ingress-controller        LoadBalancer   10.0.236.223   52.224.125.195   80:30927/TCP,443:31144/TCP   18m
toned-spaniel-nginx-ingress-default-backend   ClusterIP      10.0.5.86      <none>           80/TCP                       18m
```

Como não foram criadas regras de entrada, se navegar para o endereço IP público você será encaminhado para a página 404 padrão dos controladores de entrada NGINX.

![Back-end de NGINX padrão](media/ingress/default-back-end.png)

## <a name="configure-dns-name"></a>Configurar nome DNS

Como são utilizados certificados HTTPS, será necessário configurar um nome FQDN para o endereço IP dos controladores de entrada. Para esse exemplo, um FQDN do Azure é criado com a CLI do Azure. Atualize o script com o endereço IP do controlador de entrada e o nome que você pretender usar no FQDN.

```
#!/bin/bash

# Public IP address
IP="52.224.125.195"

# Name to associate with public IP address
DNSNAME="demo-aks-ingress"

# Get resource group and public ip name
RESOURCEGROUP=$(az network public-ip list --query "[?contains(ipAddress, '$IP')].[resourceGroup]" --output tsv)
PIPNAME=$(az network public-ip list --query "[?contains(ipAddress, '$IP')].[name]" --output tsv)

# Update public ip address with dns name
az network public-ip update --resource-group $RESOURCEGROUP --name  $PIPNAME --dns-name $DNSNAME
```

Se necessário, execute o comando a seguir para recuperar o FQDN. Atualize o valor do endereço IP com o do controlador de entrada.

```azurecli
az network public-ip list --query "[?contains(ipAddress, '52.224.125.195')].[dnsSettings.fqdn]" --output tsv
```

O controlador de entrada agora está acessível por meio do FQDN.

## <a name="install-kube-lego"></a>Instalar KUBE-LEGO

O controlador de entrada NGINX dá suporte para terminação TLS. Embora existam várias maneiras de recuperar e configurar certificados para HTTPS, este documento demonstra o uso de [KUBE-LEGO][kube-lego], que fornece a funcionalidade de gerenciamento e geração de certificado automática [Vamos Criptografar][lets-encrypt].

Para instalar o controlador KUBE-LEGO, use o comando de instalação Helm a seguir. Atualize o endereço de email com um de sua organização.

```
helm install stable/kube-lego \
  --set config.LEGO_EMAIL=user@contoso.com \
  --set config.LEGO_URL=https://acme-v01.api.letsencrypt.org/directory
```

Para obter mais informações sobre a configuração do KUBE-LEGO, consulte a [documentação do KUBE-LEGO][kube-lego].

## <a name="run-application"></a>Executar aplicativo

Neste ponto, um controlador de entrada e uma solução de gerenciamento de certificados foram configurados. Agora, execute alguns aplicativos no cluster do AKS.

Para esse exemplo, o Helm é utilizado para executar várias instâncias de um aplicativo simples hello world.

Antes de executar o aplicativo, adicione o repositório Helm de exemplos do Azure no seu sistema de desenvolvimento.

```
helm repo add azure-samples https://azure-samples.github.io/helm-charts/
```

 Execute o gráfico hello world do AKS com o comando a seguir:

```
helm install azure-samples/aks-helloworld
```

Em seguida, instale uma segunda instância do aplicativo hello world.

Para a segunda instância, especifique um novo título para que os dois aplicativos sejam visualmente distintos. Você também precisa especificar um nome de serviço exclusivo. Essas configurações podem ser vistas no comando a seguir.

```console
helm install azure-samples/aks-helloworld --set title="AKS Ingress Demo" --set serviceName="ingress-demo"
```

## <a name="create-ingress-route"></a>Criar rota de entrada

Ambos os aplicativos estão em execução no cluster de Kubernetes, no entanto, foram configurados com um serviço do tipo `ClusterIP`. Dessa forma, os aplicativos não são acessíveis a partir da Internet. Para torná-los disponíveis, crie um recurso de entrada de Kubernetes. O recurso de entrada configura as regras que roteiam o tráfego para um dos dois aplicativos.

Crie um nome de arquivo `hello-world-ingress.yaml` e copie o YAML a seguir.

Observe que o tráfego para o endereço `https://demo-aks-ingress.eastus.cloudapp.azure.com/` é roteado para o serviço nomeado `aks-helloworld`. O tráfego para o endereço `https://demo-aks-ingress.eastus.cloudapp.azure.com/hello-world-two` é roteado para o serviço `ingress-demo`.

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: hello-world-ingress
  annotations:
    kubernetes.io/tls-acme: "true"
    ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
  - hosts:
    - demo-aks-ingress.eastus.cloudapp.azure.com
    secretName: tls-secret
  rules:
  - host: demo-aks-ingress.eastus.cloudapp.azure.com
    http:
      paths:
      - path: /
        backend:
          serviceName: aks-helloworld
          servicePort: 80
      - path: /hello-world-two
        backend:
          serviceName: ingress-demo
          servicePort: 80
```

Crie o recurso de entrada com o comando `kubectl apply`.

```console
kubectl apply -f hello-world-ingress.yaml
```

## <a name="test-the-ingress-configuration"></a>Testar a configuração de entrada

Navegue até o FQDN do controlador de entrada de Kubernetes e você deverá visualizar o aplicativo hello world.

![Exemplo de aplicativo um](media/ingress/app-one.png)

Em seguida, navegue até o FQDN do controlador de entrada com o caminho `/hello-world-two` e você deverá visualizar o aplicativo hello world com o título personalizado.

![Exemplo de aplicativo dois](media/ingress/app-two.png)

Observe também que a conexão é criptografada e que um certificado emitido pelo Vamos Criptografar é utilizado.

![Permitir criptografar o certificado](media/ingress/certificate.png)

## <a name="next-steps"></a>Próximas etapas

Saiba mais sobre o software demonstrado neste documento. 

- [Controlador de entrada NGINX ][nginx-ingress]
- [KUBE-LEGO][kube-lego]

<!-- LINKS - external -->
[kube-lego]: https://github.com/jetstack/kube-lego
[lets-encrypt]: https://letsencrypt.org/
[nginx-ingress]: https://github.com/kubernetes/ingress-nginx
