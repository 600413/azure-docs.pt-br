---
title: "Solucionar problemas configurações de delegação restrita de Kerberos para o Proxy do Aplicativo | Microsoft Docs"
description: "Solucionar problemas configurações de delegação restrita de Kerberos para o Proxy do Aplicativo."
services: active-directory
documentationcenter: 
author: MarkusVi
manager: mtillman
ms.assetid: 
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/09/2018
ms.author: markvi
ms.reviewer: harshja
ms.openlocfilehash: a580b0afbd34623986ea8a3f60147a937c423e5e
ms.sourcegitcommit: d1f35f71e6b1cbeee79b06bfc3a7d0914ac57275
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/22/2018
---
# <a name="troubleshoot-kerberos-constrained-delegation-configurations-for-application-proxy"></a>Solucionar problemas configurações de delegação restrita de Kerberos para o Proxy do Aplicativo

Os métodos disponíveis para a obtenção de SSO a aplicativos publicados podem variar um pouco de um aplicativo para outro. Uma das opções que o Proxy de Aplicativo do Azure oferece por padrão é a Delegação Restrita de Kerberos (KCD). Você pode configurar um conector para realizar a autenticação restrita de Kerberos para aplicativos de back-end, em nome dos usuários.

O procedimento real para habilitar o KCD é relativamente simples e geralmente exige não mais do que uma compreensão geral dos vários componentes e fluxo de autenticação que facilita o SSO. Localizar boas fontes de informações para ajudar a solucionar cenários em que o KCD SSO não funciona conforme o esperado, pode ser esparso.

Assim, este artigo tenta fornecer um único ponto de referência que deve ajudar a solucionar problemas e resolver alguns dos problemas mais comuns vistos por conta própria. Ao mesmo tempo, oferece orientações adicionais para diagnóstico da implementação mais complexa e com problemas.

Este artigo pressupõe o seguinte:

-   A implantação do Application Proxy do Azure por [documentação](https://docs.microsoft.com/azure/active-directory/active-directory-application-proxy-enable) e o acesso geral a aplicativos que não são KCD não está funcionando conforme o esperado.

-   O aplicativo de destino publicado baseia-se na implementação do Kerberos da IIS e da Microsoft.

-   Os hosts de servidor e aplicativo residem em um único domínio do Active Directory. Informações detalhadas sobre entre cenários de floresta e de domínio podem ser encontradas no [documento técnico de KCD](https://aka.ms/KCDPaper).

-   O aplicativo de assunto é publicado no Azure locatário com pré-autenticação habilitado e os usuários devem autenticar no Azure por meio da autenticação baseada em formulários. Cenários de autenticação de cliente avançados não são cobertos por este artigo, mas são adicionados em algum momento no futuro.

## <a name="prerequisites"></a>pré-requisitos

Application Proxy do Azure pode ser implantado em muitos tipos de ambientes ou infraestruturas, e as arquiteturas sem dúvida variam de acordo com a empresa. Uma das causas mais comuns de problemas relacionados a KCD não são os ambientes em si, mas simples configurações ou supervisões incorretas.

Por esse motivo, é sempre melhor iniciar, certificando-se de ter atendido todos os pré-requisitos dispostos no artigo [Usando KCD SSO com o Application Proxy](https://docs.microsoft.com/azure/active-directory/active-directory-application-proxy-sso-using-kcd), antes de iniciar a solução de problemas.

Particularmente, a seção sobre como configurar o KCD em 2012R2, pois isso emprega uma abordagem fundamentalmente diferente para configurar o KCD em versões anteriores do Windows, mas também enquanto estiver sendo atento a várias outras considerações:

-   Não é incomum que um servidor membro do domínio abra uma caixa de diálogo de canal seguro com um controlador de domínio específico. Então passe para outra caixa de diálogo a qualquer momento, de modo que hosts de conector não devem estar restritos a poderem comunicar-se apenas com os DCs locais.

-   Semelhante ao ponto anterior, cenários entre domínios dependem de referências para direcionarem um host de conector para controladores de domínio que estejam fora do perímetro da rede local. Neste cenário, é igualmente importante garantir que também permita tráfego para os controladores de domínio que representam outros respectivos domínios ou falha de delegação else.

-   Sempre que possível, você deve evitar colocar todos os dispositivos IDS/IPS ativos entre hosts de conector e controladores de domínio, pois esses são às vezes intrusivos demais e interferem com o tráfego RPC de núcleo

É necessário testar a delegação nos cenários mais simples. Quanto mais variáveis você introduzir, com mais você pode ter que lidar. Por exemplo, limitar seus testes a um único conector pode economizar tempo valioso e conectores adicionais podem ser adicionados depois que os problemas tiverem sido resolvidos.

Alguns fatores ambientais também podem estar contribuindo para um problema. Durante o teste, minimize a arquitetura para mínimo para evitar esses fatores ambientais. Por exemplo, as ACLs de firewall internas mal configuradas não são incomuns, portanto, se possível, tenha todo o tráfego de um conector permitido direto para os controles de domínio e o aplicativo de back-end. 

Na verdade, o melhor local absoluto para posicionar os conectores é mais próximo possível de seus destinos. Ter um firewall embutido durante testes simplesmente adiciona complexidade desnecessária e apenas pode prolongar as investigações.

Portanto, então o que constitui um problema KCD? Há várias indicações clássicas de KCD SSO falhando e os primeiros sinais de um problema geralmente se manifestam no navegador.

   ![Erro de configuração do KCD incorreto](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic1.png)

   ![A autorização falhou devido à falta de permissões](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic2.png)

todos que tenham o mesmo sintoma de falha ao executar o SSO e, consequentemente, negando o acesso do usuário para o aplicativo.

## <a name="troubleshooting"></a>solução de problemas

Como você soluciona depende do problema e dos sintomas observados. Antes de prosseguir, explore os seguintes links, pois contêm informações úteis que você ainda não encontrou:

-   [Solucionar problemas e mensagens de erro do Application Proxy](https://docs.microsoft.com/azure/active-directory/active-directory-application-proxy-troubleshoot)

-   [Sintomas e erros de Kerberos](https://docs.microsoft.com/azure/active-directory/active-directory-application-proxy-troubleshoot#kerberos-errors)

-   [Trabalhando com o SSO as identidades locais e na nuvem não são idênticas](https://docs.microsoft.com/azure/active-directory/active-directory-application-proxy-sso-using-kcd#working-with-sso-when-on-premises-and-cloud-identities-are-not-identical)

Se você chegou até aqui, então, agora, o problema principal definitivamente existe. Comece separando o fluxo em três estágios distintos que você pode solucionar.

**Pré-autenticação de cliente** – o usuário externo realizando autenticação no Azure por meio de um navegador.

Ser capaz de autenticar previamente no Azure é obrigatório para KCD SSO funcionar. Se ocorrer algum problema, você deverá testar e corrigir o problema. A fase de pré-autenticação não tem nenhuma relação com o KCD ou o aplicativo publicado. Deve ser bastante fácil corrigir discrepâncias por meio da verificação de integridade da conta, se ela existe no Azure, e se ela não está desabilitada/bloqueada. A resposta de erro no navegador é normalmente descritiva para entender a causa. Você também poderá verificar nossos outros documentos de solução de problemas para verificar, se você não tiver certeza.

**Serviço de delegação** – o conector de Proxy do Azure obtendo um tíquete de serviço do Kerberos por meio de um KDC (Centro de distribuição de Kerberos), em nome dos usuários.

As comunicações externas entre o cliente e o front-end do Azure não devem ter nenhuma influência sobre o KCD, a não ser garantir que ele funciona. Isso é para que o serviço de Proxy do Azure possa ser fornecido com uma ID de usuário válida que é usada para obter um tíquete do Kerberos. Sem isso, o KCD não é possível e falharia.

Conforme mencionado anteriormente, as mensagens de erro do navegador geralmente fornecem algumas boas dicas do por que as coisas estão falhando. Certifique-se de observar a ID da atividade e o carimbo de data/hora na resposta, pois isso lhe permite correlacionar o comportamento à reais eventos no log de eventos do Proxy do Azure.

   ![Erro de configuração do KCD incorreto](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic3.png)

E as entradas correspondentes visualizadas no log de eventos seria visualizadas como eventos 13019 ou 12027. Você pode encontrar os logs de eventos do conector em **Logs de aplicativos e serviços**&gt;**Microsoft**&gt;**AadApplicationProxy**&gt;**Conector**&gt;**Admin**.

   ![Evento 13019 do log de eventos do Application Proxy](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic4.png)

   ![Evento 12027 do log de eventos do Application Proxy](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic5.png)

-   Usar um registro A no seu DNS interno para o endereço do aplicativo, e não um CName

-   Confirme que o host do conector tenha recebido direitos para delegar para o SPN da conta de destino designada e que **Usar qualquer protocolo de autenticação** esteja selecionado. Para obter mais informações sobre este tópico, consulte o [artigo de configuração de SSO](https://docs.microsoft.com/azure/active-directory/active-directory-application-proxy-sso-using-kcd)

-   Verifique se há apenas uma única instância do SPN em existência no AD, emitindo um `setspn -x` de um prompt de comando em qualquer host de membro de domínio

-   Verifique se uma política de domínio é imposta para limitar o [máximo de tamanho de tokens de Kerberos emitidos](https://blogs.technet.microsoft.com/askds/2012/09/12/maxtokensize-and-windows-8-and-windows-server-2012/), pois isso impede que o conector obtenha um token se considerado excessivo

Um rastreamento de rede capturando as trocas entre o host de conector e um domínio de KDC seria a próxima etapa recomendada na obtenção de detalhes de nível mais baixo sobre os problemas. Para obter mais informações, consulte [aprofundamento do papel de solução de problemas](https://aka.ms/proxytshootpaper).

Se o tíquete parece bom, você verá um evento nos logs, indicando que a autenticação falhou devido ao aplicativo que retorna um 401. Isso geralmente indica que o aplicativo de destino rejeita seu tíquete, então continue com o próximo estágio a seguir:

**Aplicativo de destino** – O consumidor do tíquete do Kerberos fornecido pelo conector

Nesse estágio, é esperado que o conector tenha enviado um tíquete de serviço do Kerberos para o back-end, como um cabeçalho dentro da primeira solicitação de aplicativo.

-   Usando a URL do aplicativo interno definida no portal, valide que o aplicativo é acessível diretamente do navegador no host do conector. Em seguida, você pode fazer logon com êxito. Detalhes sobre como fazer isso podem ser encontrados na página de solução de problemas do conector.

-   Ainda no host do conector, confirme se a autenticação entre o navegador e o aplicativo está usando Kerberos, seguindo um destes procedimentos:

1.  Execute as ferramentas de desenvolvimento (**F12**) no Internet Explorer, ou use [Fiddler](https://blogs.msdn.microsoft.com/crminthefield/2012/10/10/using-fiddler-to-check-for-kerberos-auth/) do host de conector. Vá para o aplicativo usando a URL interna e inspecione os cabeçalhos de autorização WWW oferecidos e retornados na resposta do aplicativo, para garantir que negotiate ou Kerberos está presente. Um blob de Kerberos subsequente retornado na resposta do navegador para o aplicativo geralmente começa com **YII**, portanto, isso é uma boa indicação do Kerberos em execução. NTLM, por outro lado, sempre começa com **TlRMTVNTUAAB**, que lê NTLMSSP quando decodificado da Base64. Se você vê **TlRMTVNTUAAB** no início do blob, isso significa que o Kerberos **não** está disponível. Se você não vê isso, é provável que o Kerberos esteja disponível.
    > [!NOTE]
    > Se ao usar o Fiddler, esse método requer desabilitar temporariamente a proteção estendida na configuração do aplicativo no IIS.

     ![Janela de inspeção da rede do navegador](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic6.png)

    *Figura:* uma vez que isso não começa com TIRMTVNTUAAB, este é um exemplo de que o Kerberos está disponível. Este é um exemplo de um Blob de Kerberos que não começa com YII.

2.  Remova temporariamente NTLM na lista de provedores no IIS local e aplicativo de acesso diretamente do IE no host do conector. Com NTLM não mais na lista de provedores, você deve ser capaz de acessar o aplicativo usando apenas o Kerberos. Se isso falhar, isso indicará que há um problema com a configuração do aplicativo e a autenticação de Kerberos não está funcionando.

Se o Kerberos não estiver disponível, verifique as configurações de autenticação do aplicativo no IIS para verificar se negotiate está listado na parte superior, com NTLM logo abaixo dele. (Não negociar: kerberos ou Negotiate: PKU2U). Só continue se Kerberos está funcional.

   ![Fornecedores de autenticação do Windows](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic7.png)
   
-   Com o Kerberos e NTLM implementados, permite agora desabilitar temporariamente a pré-autenticação para o aplicativo no portal. Veja se você pode acessá-lo pela Internet usando a URL externa. Você deve ser solicitado a autenticar e deve ser capaz de fazer isso com a mesma conta usada na etapa anterior. Caso contrário, isso indica um problema com o aplicativo de back-end e não KCD todo.

-   Agora reabilite a pré-autenticação no portal e autentique por meio do Azure, tentando se conectar ao aplicativo por meio de sua URL externa. Se houver falha em SSO, você verá uma mensagem de erro proibido do navegador, além de evento 13022 no log:

    *Conector do Application Proxy do Microsoft AAD não pode autenticar o usuário porque o servidor back-end responde às tentativas de autenticação do Kerberos com um erro HTTP 401.*

    ![Erro proibido HTTTP 401](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic8.png)

-   Verifique o aplicativo do IIS para garantir que o pool de aplicativos configurado esteja configurado para usar a mesma conta que o SPN foi configurado no AD, navegando no IIS como a ilustração a seguir

    ![Janela de configuração de aplicativo IIS](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic9.png)

    Se você souber a identidade, execute o seguinte em um prompt de comando para verificar se essa conta está definitivamente configurada com o SPN em questão. Por exemplo, `setspn –q http/spn.wacketywack.com`

    ![Janela de comando SetSPN](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic10.png)

-   Verifique se o SPN definido nas configurações do aplicativo no portal é o mesmo SPN configurado em relação à conta do AD de destino usada pelo pool de aplicativos do aplicativo

   ![Configuração de SPN no Portal do Azure](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic11.png)
   
-   Vá para o IIS e selecione a opção do **Editor de configuração** para o aplicativo e navegue até **system.webServer/security/authentication/windowsAuthentication** para certificar-se de que o valor **UseAppPoolCredentials** é **Verdadeiro**

   ![Opção de credencial de pools de aplicativo da configuração do IIS](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic12.png)

Depois de alterar esse valor para **True**, todos os tíquetes Kerberos em cache precisarão ser removidos do servidor back-end. É possível fazer isso executando o seguinte comando:

```powershell
Get-WmiObject Win32_LogonSession | Where-Object {$_.AuthenticationPackage -ne 'NTLM'} | ForEach-Object {klist.exe purge -li ([Convert]::ToString($_.LogonId, 16))}
``` 

Para obter mais informações, consulte [Limpar o cache de tíquete do cliente Kerberos de todas as sessões](https://gallery.technet.microsoft.com/scriptcenter/Purge-the-Kerberos-client-b56987bf).



Embora seja útil para melhorar o desempenho das operações de Kerberos, deixar o modo de Kernel habilitado também faz com que a permissão para que o serviço solicitado seja descriptografada com a conta do computador. Isso também é chamado de sistema Local, então, ter essa definição como verdadeira quebra o KCD quando o aplicativo é hospedado em vários servidores em uma fazenda.

-   Como uma verificação adicional, você também pode ser necessário desabilitar a proteção **estendida** também. Foram encontrados cenários em que isso mostrou quebrar o KCD quando habilitado em configurações específicas, em que um aplicativo é publicado como uma subpasta do site da Web padrão. Esse em si é configurado para autenticação anônima, deixando as caixas de diálogo todas em cinza, sugerindo que objetos filho não iriam herdar as configurações ativas. Mas onde possível, sempre recomendamos ter esse recurso habilitado, então, de qualquer forma, faça o teste, mas não se esqueça de restaurar para habilitado.

Essas verificações adicionais devem ter colocado você no caminho certo para começar a usar seu aplicativo publicado. Você pode prosseguir e criar conectores adicionais que também são configurados para delegar, mas, caso não haja nenhum outro, sugerimos uma leitura de nosso passo a passo técnico mais aprofundado, [O guia completo para Solucionar Problemas do Application Proxy do Azure AD](https://aka.ms/proxytshootpaper)

Se você ainda não conseguir progredir no problema, o suporte ficará mais do que feliz em ajudar e continuar daqui. Crie um tíquete de suporte diretamente dentro do portal (um engenheiro entrará em contato).

## <a name="other-scenarios"></a>Outros cenários

-   Application Proxy do Azure solicita um tíquete do Kerberos antes de enviar a solicitação para um aplicativo. Alguns aplicativos de terceiros, como Tableau Server, não gostam deste método de autenticação, e em vez disso, esperam que as negociações mais convencionais ocorram. A primeira solicitação é anônima, permitindo que o aplicativo responda com os tipos de autenticação que ele oferece suporte através de um 401.

-   Autenticação com salto duplo – comumente usado em cenários nos quais um aplicativo é hierárquico, com um back-end e front-end, ambos exigindo autenticação, como o Serviços de Relatórios SQL.

## <a name="next-steps"></a>Próximas etapas
[Configurar a KCD (delegação Restrita de Kerberos) em um domínio gerenciado](https://docs.microsoft.com/azure/active-directory-domain-services/active-directory-ds-enable-kcd)
