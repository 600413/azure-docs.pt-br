---
title: Usar fluxos de trabalho do Hadoop Oozie no HDInsight baseado em Linux | Microsoft Docs
description: Usar o Oozie do Hadoop no HDInsight baseado em Linux. Saiba como definir um fluxo de trabalho do Oozie e enviar um trabalho do Oozie.
services: hdinsight
documentationcenter: 
author: Blackmist
manager: jhubbard
editor: cgronlun
tags: azure-portal
ms.assetid: d7603471-5076-43d1-8b9a-dbc4e366ce5d
ms.service: hdinsight
ms.custom: hdinsightactive
ms.workload: big-data
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/22/2018
ms.author: larryfr
ms.openlocfilehash: 53c249c24af774499becdf18350ff75dbc2ab861
ms.sourcegitcommit: 1fbaa2ccda2fb826c74755d42a31835d9d30e05f
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/22/2018
---
# <a name="use-oozie-with-hadoop-to-define-and-run-a-workflow-on-linux-based-azure-hdinsight"></a>Usar o Oozie com Hadoop para definir e executar um fluxo de trabalho no Azure HDInsight baseado em Linux

[!INCLUDE [oozie-selector](../../includes/hdinsight-oozie-selector.md)]

Saiba como usar o Apache Oozie com o Hadoop no Azure HDInsight. O Oozie é um sistema de fluxo de trabalho e coordenação que gerencia trabalhos do Hadoop. O Oozie é integrado à pilha do Hadoop e dá suporte aos seguintes trabalhos:

* Apache MapReduce
* Apache Pig
* Apache Hive
* Apache Sqoop

Você também pode usar o Oozie para agendar trabalhos específicos para um sistema, como programas Java ou scripts de shell.

> [!NOTE]
> Outra opção para definir fluxos de trabalho com HDInsight é usar o Azure Data Factory. Para saber mais sobre o Data Factory, confira [Usar o Pig e o Hive com o Data Factory][azure-data-factory-pig-hive].

> [!IMPORTANT]
> O Oozie não está habilitado no HDInsight ingressado no domínio.

## <a name="prerequisites"></a>pré-requisitos

* **Um cluster hdinsight**: consulte [Introdução ao HDInsight no Linux](/hadoop/apache-hadoop-linux-tutorial-get-started.md)

> [!IMPORTANT]
> As etapas deste documento exigem um cluster HDInsight que usa Linux. O Linux é o único sistema operacional usado no HDInsight versão 3.4 ou posterior. Para obter mais informações, confira [baixa do HDInsight no Windows](hdinsight-component-versioning.md#hdinsight-windows-retirement).

## <a name="example-workflow"></a>Fluxo de trabalho de exemplo

O fluxo de trabalho usado neste documento contém duas ações. Ações são definições de tarefas, como a execução de Sqoop, Hive, MapReduce ou outro processo:

![Diagrama de fluxo de trabalho][img-workflow-diagram]

1. Uma ação do Hive executa um script HiveQL para extrair os registros de **hivesampletable** incluídos no HDInsight. Cada linha de dados descreve uma visita de um dispositivo móvel específico. O formato de registro se parece com o seguinte texto:

        8       18:54:20        en-US   Android Samsung SCH-i500        California     United States    13.9204007      0       0
        23      19:19:44        en-US   Android HTC     Incredible      Pennsylvania   United States    NULL    0       0
        23      19:19:46        en-US   Android HTC     Incredible      Pennsylvania   United States    1.4757422       0       1

    O script do Hive usado neste documento conta o total de visitas de cada plataforma,como Android ou iPhone, e armazena as contagens em uma nova tabela do Hive.

    Para obter mais informações sobre o Hive, consulte [Usar o Hive com o HDInsight][hdinsight-use-hive].

2. Uma ação do Sqoop exporta o conteúdo da nova tabela Hive para uma tabela criada em um Banco de Dados SQL do Azure. Para saber mais sobre o Sqoop, confira [Usar o Hadoop Sqoop com o HDInsight][hdinsight-use-sqoop].

> [!NOTE]
> Para obter as versões do Oozie com suporte em clusters HDInsight, confira [Novidades nas versões de clusters Hadoop fornecidas pelo HDInsight][hdinsight-versions].

## <a name="create-the-working-directory"></a>Criar o diretório de trabalho

O Oozie espera armazenar todos os recursos necessários para um trabalho no mesmo diretório. Este exemplo usa **wasb:///tutorials/useoozie**. Para criar o diretório, conclua as seguintes etapas:

1. Conecte-se ao cluster HDInsight usando SSH:

    ```bash
    ssh sshuser@clustername-ssh.azurehdinsight.net
    ```

    Substitua `sshuser` pelo nome de usuário SSH para o cluster. Substitua `clustername` pelo nome do cluster. Para obter mais informações, confira [Usar SSH com HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

2. Para criar o diretório, use o seguinte comando:

    ```bash
    hdfs dfs -mkdir -p /tutorials/useoozie/data
    ```

    > [!NOTE]
    > O parâmetro `-p` cria todos os diretórios no caminho. O diretório de **dados** é usado para armazenar dados usados pelo script **useooziewf.hql**.

3. Para certificar-se de que Oozie pode representar a conta de usuário, use o seguinte comando:

    ```bash
    sudo adduser username users
    ```

    Substitua `username` pelo nome de usuário SSH.

    > [!NOTE]
    > Você pode ignorar os erros que indicam o usuário como já sendo membro do grupo `users`.

## <a name="add-a-database-driver"></a>Adicionar um driver de banco de dados

Como esse fluxo de trabalho usa Sqoop para exportar dados para o banco de dados SQL, você deve fornecer uma cópia do driver JDBC usado para interagir com o Banco de Dados SQL. Para copiar o driver JDBC para o diretório de trabalho, use o seguinte comando da sessão SSH:

```bash
hdfs dfs -put /usr/share/java/sqljdbc_4.1/enu/sqljdbc*.jar /tutorials/useoozie/
```

Caso o fluxo de trabalho tenha usado outros recursos, como um jar que contém um aplicativo MapReduce, você precisará adicionar esses recursos também.

## <a name="define-the-hive-query"></a>Definir a consulta de Hive

Use as etapas a seguir para criar um script de HiveQL (linguagem de consulta Hive) que define uma consulta. Você usará a consulta em um fluxo de trabalho do Oozie neste documento.

1. Na conexão SSH, use o seguinte comando para criar um arquivo denominado `useooziewf.hql`:

    ```bash
    nano useooziewf.hql
    ```

3. Quando o editor nano GNU for aberto, use a seguinte consulta como o conteúdo do arquivo:

    ```hiveql
    DROP TABLE ${hiveTableName};
    CREATE EXTERNAL TABLE ${hiveTableName}(deviceplatform string, count string) ROW FORMAT DELIMITED
    FIELDS TERMINATED BY '\t' STORED AS TEXTFILE LOCATION '${hiveDataFolder}';
    INSERT OVERWRITE TABLE ${hiveTableName} SELECT deviceplatform, COUNT(*) as count FROM hivesampletable GROUP BY deviceplatform;
    ```

    Duas variáveis são usadas no script:

    * `${hiveTableName}`: contém o nome da tabela a ser criada.

    * `${hiveDataFolder}`: contém o local para armazenar os arquivos de dados para a tabela.

    O arquivo de definição do fluxo de trabalho, workflow.xml neste tutorial, transmite esses valores para o script HiveQL no tempo de execução.

4. Para sair do editor, selecione Ctrl+X. Quando solicitado, selecione `Y` para salvar o arquivo, digite `useooziewf.hql` como o nome de arquivo e selecione **Enter**.

5. Use os comandos a seguir para copiar `useooziewf.hql` para `wasb:///tutorials/useoozie/useooziewf.hql`:

    ```bash
    hdfs dfs -put useooziewf.hql /tutorials/useoozie/useooziewf.hql
    ```

    Esse comando armazena o arquivo `useooziewf.hql` no armazenamento compatível com o HDFS para o cluster.

## <a name="define-the-workflow"></a>Definir o fluxo de trabalho

As definições de fluxo de trabalho do Oozie são codificadas em hPDL (linguagem de definição de processo do Hadoop), que é uma linguagem de definição de processo XML. Use as etapas a seguir para definir o fluxo de trabalho:

1. Use a instrução a seguir para criar e editar um novo arquivo:

    ```bash
    nano workflow.xml
    ```

2. Quando o editor nano for aberto, insira o seguinte XML como conteúdo do arquivo:

    ```xml
    <workflow-app name="useooziewf" xmlns="uri:oozie:workflow:0.2">
        <start to = "RunHiveScript"/>
        <action name="RunHiveScript">
        <hive xmlns="uri:oozie:hive-action:0.2">
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <configuration>
            <property>
                <name>mapred.job.queue.name</name>
                <value>${queueName}</value>
            </property>
            </configuration>
            <script>${hiveScript}</script>
            <param>hiveTableName=${hiveTableName}</param>
            <param>hiveDataFolder=${hiveDataFolder}</param>
        </hive>
        <ok to="RunSqoopExport"/>
        <error to="fail"/>
        </action>
        <action name="RunSqoopExport">
        <sqoop xmlns="uri:oozie:sqoop-action:0.2">
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <configuration>
            <property>
                <name>mapred.compress.map.output</name>
                <value>true</value>
            </property>
            </configuration>
            <arg>export</arg>
            <arg>--connect</arg>
            <arg>${sqlDatabaseConnectionString}</arg>
            <arg>--table</arg>
            <arg>${sqlDatabaseTableName}</arg>
            <arg>--export-dir</arg>
            <arg>${hiveDataFolder}</arg>
            <arg>-m</arg>
            <arg>1</arg>
            <arg>--input-fields-terminated-by</arg>
            <arg>"\t"</arg>
            <archive>sqljdbc41.jar</archive>
            </sqoop>
        <ok to="end"/>
        <error to="fail"/>
        </action>
        <kill name="fail">
        <message>Job failed, error message[${wf:errorMessage(wf:lastErrorNode())}] </message>
        </kill>
        <end name="end"/>
    </workflow-app>
    ```

    Existem duas ações definidas no fluxo de trabalho:

   * `RunHiveScript`: essa ação é a ação inicial e executa o script do Hive `useooziewf.hql`.

   * `RunSqoopExport`: essa ação exporta os dados criados por meio do script do Hive para o Banco de Dados SQL usando o Sqoop. Essa ação será executada apenas se a ação `RunHiveScript` for bem-sucedida.

     O fluxo de trabalho tem várias entradas, como `${jobTracker}`. Você substituirá essas entradas pelos valores usados na definição do trabalho. Você criará a definição de trabalho mais tarde neste documento.

     Observe também a entrada `<archive>sqljdbc4.jar</archive>` na seção Sqoop. Essa entrada instrui o Oozie a disponibilizar esse arquivo morto ao Sqoop quando essa ação é executada.

3. Para salvar o arquivo, selecione Ctrl+X, digite `Y`e selecione **Enter**. 

4. Use o seguinte comando para copiar o arquivo `workflow.xml` para `/tutorials/useoozie/workflow.xml`:

    ```bash
    hdfs dfs -put workflow.xml /tutorials/useoozie/workflow.xml
    ```

## <a name="create-the-database"></a>Criar o banco de dados

Para criar um Banco de Dados SQL, siga as etapas do documento [Criar um Banco de Dados SQL](../sql-database/sql-database-get-started.md). Ao criar o banco de dados, use `oozietest` como o nome do banco de dados. Além disso, anote o nome do servidor de banco de dados.

### <a name="create-the-table"></a>Criar a tabela.

> [!NOTE]
> Há várias maneiras para se conectar ao Banco de Dados SQL para criar uma tabela. As seguintes etapas usam [FreeTDS](http://www.freetds.org/) do cluster HDInsight.


1. Use o seguinte comando para instalar o FreeTDS no cluster do HDInsight:

    ```bash
    sudo apt-get --assume-yes install freetds-dev freetds-bin
    ```

2. Quando o FreeTDS tiver sido instalado, use o seguinte comando para conectar-se ao Banco de Dados SQL Server criado anteriormente:

    ```bash
    TDSVER=8.0 tsql -H <serverName>.database.windows.net -U <sqlLogin> -P <sqlPassword> -p 1433 -D oozietest
    ```

    Você receberá uma saída parecida com o seguinte texto:

        locale is "en_US.UTF-8"
        locale charset is "UTF-8"
        using default charset "UTF-8"
        Default database being set to oozietest
        1>

3. Ao prompt `1>` , insira o seguinte:

    ```sql
    CREATE TABLE [dbo].[mobiledata](
    [deviceplatform] [nvarchar](50),
    [count] [bigint])
    GO
    CREATE CLUSTERED INDEX mobiledata_clustered_index on mobiledata(deviceplatform)
    GO
    ```

    Quando a instrução `GO` for inserida, as instruções anteriores serão avaliadas. Essas instruções criam uma tabela chamada **mobiledata** que é usada pelo fluxo de trabalho.

    Para verificar se a tabela foi criada, use os seguintes comandos:

    ```sql
    SELECT * FROM information_schema.tables
    GO
    ```

    Você vê uma saída semelhante à seguinte:

        TABLE_CATALOG   TABLE_SCHEMA    TABLE_NAME      TABLE_TYPE
        oozietest       dbo     mobiledata      BASE TABLE

4. Para sair do utilitário tsql, insira `exit` no prompt `1>`.

## <a name="create-the-job-definition"></a>Criar a definição de trabalho

A definição de trabalho descreve o local em que o workflow.xml se encontra. Ela também descreve o local em que você pode encontrar outros arquivos usados pelo fluxo de trabalho, como `useooziewf.hql`. Além disso, ele define os valores para as propriedades usadas no fluxo de trabalho e nos arquivos associados.

1. Para obter o endereço completo do armazenamento padrão, use o comando a seguir. Esse endereço é usado no arquivo de configuração que você vai criar na próxima etapa.

    ```bash
    sed -n '/<name>fs.default/,/<\/value>/p' /etc/hadoop/conf/core-site.xml
    ```

    O comando retorna informações semelhantes ao seguinte XML:

    ```xml
    <name>fs.defaultFS</name>
    <value>wasb://mycontainer@mystorageaccount.blob.core.windows.net</value>
    ```

    > [!NOTE]
    > Se o cluster HDInsight usa o armazenamento do Azure como o armazenamento padrão, o conteúdo de elemento `<value>` começa com `wasb://`. Se o Azure Data Lake Store é usado, ele começa com `adl://`.

    Salve o conteúdo do elemento `<value>`, pois ele será usado nas próximas etapas.

2. Para criar a configuração de definição de trabalho do Oozie, use o seguinte comando:

    ```bash
    nano job.xml
    ```

3. Quando o editor nano for aberto, use o seguinte XML como conteúdo do arquivo:

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <configuration>

        <property>
        <name>nameNode</name>
        <value>wasb://mycontainer@mystorageaccount.blob.core.windows.net</value>
        </property>

        <property>
        <name>jobTracker</name>
        <value>headnodehost:8050</value>
        </property>

        <property>
        <name>queueName</name>
        <value>default</value>
        </property>

        <property>
        <name>oozie.use.system.libpath</name>
        <value>true</value>
        </property>

        <property>
        <name>hiveScript</name>
        <value>wasb://mycontainer@mystorageaccount.blob.core.windows.net/tutorials/useoozie/useooziewf.hql</value>
        </property>

        <property>
        <name>hiveTableName</name>
        <value>mobilecount</value>
        </property>

        <property>
        <name>hiveDataFolder</name>
        <value>wasb://mycontainer@mystorageaccount.blob.core.windows.net/tutorials/useoozie/data</value>
        </property>

        <property>
        <name>sqlDatabaseConnectionString</name>
        <value>"jdbc:sqlserver://serverName.database.windows.net;user=adminLogin;password=adminPassword;database=oozietest"</value>
        </property>

        <property>
        <name>sqlDatabaseTableName</name>
        <value>mobiledata</value>
        </property>

        <property>
        <name>user.name</name>
        <value>YourName</value>
        </property>

        <property>
        <name>oozie.wf.application.path</name>
        <value>wasb://mycontainer@mystorageaccount.blob.core.windows.net/tutorials/useoozie</value>
        </property>
    </configuration>
    ```

   * Substitua todas as instâncias de `wasb://mycontainer@mystorageaccount.blob.core.windows.net` pelo valor que você recebeu anteriormente para armazenamento padrão.

     > [!WARNING]
     > Se o caminho for um caminho `wasb`, use o caminho completo. Não o reduza para apenas `wasb:///`.

   * Substitua `YourName` pelo seu nome de logon para o cluster do HDInsight.
   * Substituir `serverName`, `adminLogin` e `adminPassword` pelas informações do Banco de Dados SQL.

     A maioria das informações contidas nesse arquivo é usada para preencher os valores usados nos arquivos workflow.xml ou ooziewf.hql, como `${nameNode}`.

     > [!NOTE]
     > A entrada `oozie.wf.application.path` define onde encontrar o arquivo workflow.xml. Esse arquivo contém o fluxo de trabalho que foi executado pelo trabalho.

5. Para salvar o arquivo, selecione Ctrl+X, digite `Y`e selecione **Enter**.

## <a name="submit-and-manage-the-job"></a>Enviar e gerenciar o trabalho

As etapas a seguir usam o comando Oozie para enviar e gerenciar fluxos de trabalho do Oozie no cluster. O comando do Oozie é uma interface amigável sobre a [API REST do Oozie](https://oozie.apache.org/docs/4.1.0/WebServicesAPI.html).

> [!IMPORTANT]
> Ao usar o comando Oozie, você deverá usar o FQDN para o nó de cabeçalho do HDInsight. Esse FQDN só está acessível no cluster ou, se o cluster estiver em uma rede virtual do Azure, de outros computadores na mesma rede.


1. Para obter a URL para o serviço Oozie, use o seguinte comando:

    ```bash
    sed -n '/<name>oozie.base.url/,/<\/value>/p' /etc/oozie/conf/oozie-site.xml
    ```

    Isso retorna informações semelhantes ao seguinte XML:

    ```xml
    <name>oozie.base.url</name>
    <value>http://hn0-CLUSTERNAME.randomcharacters.cx.internal.cloudapp.net:11000/oozie</value>
    ```

    A parte `http://hn0-CLUSTERNAME.randomcharacters.cx.internal.cloudapp.net:11000/oozie` é a URL a ser usada com o comando do Oozie.

2. Para criar uma variável de ambiente para a URL, use o seguinte para não precisar digitá-la a cada comando:

    ```bash
    export OOZIE_URL=http://HOSTNAMEt:11000/oozie
    ```

    Substitua a URL pela recebida anteriormente.
3. Para enviar o trabalho, use o seguinte:

    ```bash
    oozie job -config job.xml -submit
    ```

    Esse comando carrega as informações do trabalho de `job.xml` e as envia para o Oozie, mas não o executa.

    Depois que o comando é concluído, ele deve retornar a ID do trabalho, por exemplo, `0000005-150622124850154-oozie-oozi-W`. Essa ID será usada para gerenciar o trabalho.

4. Para exibir o status do trabalho, use o seguinte comando:

    ```bash
    oozie job -info <JOBID>
    ```

    > [!NOTE]
    > Substitua `<JOBID>` pela ID retornada na etapa anterior.

    Isso retorna informações semelhantes ao seguinte texto:

        Job ID : 0000005-150622124850154-oozie-oozi-W
        ------------------------------------------------------------------------------------------------------------------------------------
        Workflow Name : useooziewf
        App Path      : wasb:///tutorials/useoozie
        Status        : PREP
        Run           : 0
        User          : USERNAME
        Group         : -
        Created       : 2015-06-22 15:06 GMT
        Started       : -
        Last Modified : 2015-06-22 15:06 GMT
        Ended         : -
        CoordAction ID: -
        ------------------------------------------------------------------------------------------------------------------------------------

    Este trabalho tem o status `PREP`. Este status indica que o trabalho foi criado, mas não foi iniciado.

5. Para iniciar o trabalho, use o seguinte comando:

    ```bash
    oozie job -start JOBID
    ```

    > [!NOTE]
    > Substitua `<JOBID>` pela ID retornada anteriormente.

    Se você verificar o status após o comando, ele estará em um estado de execução e as informações serão retornadas para as ações dentro do trabalho.

6. Depois que a tarefa for concluída com sucesso, você poderá verificar se os dados foram gerados e exportados para a tabela do Banco de Dados SQL usando o seguinte comando:

    ```bash
    TDSVER=8.0 tsql -H <serverName>.database.windows.net -U <adminLogin> -P <adminPassword> -p 1433 -D oozietest
    ```

    No prompt `1>`, insira a seguinte consulta:

    ```sql
    SELECT * FROM mobiledata
    GO
    ```

    As informações retornadas são semelhantes ao seguinte texto:

        deviceplatform  count
        Android 31591
        iPhone OS       22731
        proprietary development 3
        RIM OS  3464
        Unknown 213
        Windows Phone   1791
        (6 rows affected)

Para obter mais informações sobre o comando do Oozie, consulte [Ferramenta de linha de comando do Oozie](https://oozie.apache.org/docs/4.1.0/DG_CommandLineTool.html).

## <a name="oozie-rest-api"></a>API REST do Oozie

Com a API REST do Oozie, você pode criar suas próprias ferramentas que funcionem com Oozie. As informações abaixo são específicas do HDInsight sobre como usar a API REST do Oozie:

* **URI**: você pode acessar a API REST de fora do cluster em `https://CLUSTERNAME.azurehdinsight.net/oozie`.

* **Autenticação**: para fazer a autenticação, use a API com a conta (administrador) e a senha do cluster HTTP. Por exemplo: 

    ```bash
    curl -u admin:PASSWORD https://CLUSTERNAME.azurehdinsight.net/oozie/versions
    ```

Para saber mais sobre como usar a API REST do Oozie, consulte [API de serviços Web do Oozie](https://oozie.apache.org/docs/4.1.0/WebServicesAPI.html).

## <a name="oozie-web-ui"></a>Interface do usuário da Web do Oozie

A IU da Web do Oozie fornece um modo de exibição baseado na web sobre o status dos trabalhos do Oozie no cluster. Com a interface do usuário da Web, você pode exibir as seguintes informações:

   * Status do trabalho
   * Definição de trabalho
   * Configuração
   * Um grafo das ações no trabalho
   * Logs do trabalho

Você também pode exibir detalhes de ações dentro de um trabalho.

Para acessar a interface do usuário do Oozie da Web, conclua as seguintes etapas:

1. Crie um túnel SSH para o cluster HDInsight. Para obter mais informações, consulte [Usar túnel SSH com o HDInsight](hdinsight-linux-ambari-ssh-tunnel.md).

2. Depois de criar um túnel, abra a interface do usuário da Web do Ambari no navegador da Web. O URI para o site Ambari é `https://CLUSTERNAME.azurehdinsight.net`. Substitua `CLUSTERNAME` pelo nome do seu cluster do HDInsight baseado em Linux.

3. No lado esquerdo da página, selecione **Oozie** > **Links Rápidos** > **IU da Web do Oozie**.

    ![Imagem dos menus](./media/hdinsight-use-oozie-linux-mac/ooziewebuisteps.png)

4. A interface do usuário da Web do Oozie assume como padrão a exibição de trabalhos do fluxo de trabalho em execução. Para ver todos os trabalhos de fluxo de trabalho, selecione **Todos os trabalhos**.

    ![Todos os trabalhos exibidos](./media/hdinsight-use-oozie-linux-mac/ooziejobs.png)

5. Selecione um trabalho para exibir mais informações sobre ele.

    ![Informações do Trabalho](./media/hdinsight-use-oozie-linux-mac/jobinfo.png)

6. Na guia **Informações do Trabalho**, veja informações básicas sobre o trabalho, bem como as ações individuais dentro do trabalho. Você pode usar as guias na parte superior para exibir a **Definição de Trabalho**, a **Configuração de Trabalho**, acessar o **Log de Trabalho** ou ver um DAG (grafo direcionado acíclico) do trabalho em **DAG do Trabalho**.

   * **Log de trabalho**: selecione o botão **Obter Logs** para obter todos os logs do trabalho ou use o campo **Inserir Filtro de Pesquisa** para filtrar os logs.

       ![Log de trabalhos](./media/hdinsight-use-oozie-linux-mac/joblog.png)

   * **DAG do Trabalho**: o DAG é uma visão geral gráfica dos caminhos de dados percorridos pelo fluxo de trabalho.

       ![DAG de trabalho](./media/hdinsight-use-oozie-linux-mac/jobdag.png)

7. Se você selecionar uma das ações na guia **Informações do Trabalho**, ela exibirá informações para a ação. Por exemplo, selecione a ação **RunSqoopExport**.

    ![Informações da ação](./media/hdinsight-use-oozie-linux-mac/action.png)

8. Você pode ver detalhes da ação, como um link para a **URL do Console**. Use este link para exibir informações do rastreador de trabalho para o trabalho.

## <a name="schedule-jobs"></a>Agendar trabalhos

Você pode usar o coordenador para especificar um início, um fim e a frequência da ocorrência dos trabalhos. Para definir uma agenda para o fluxo de trabalho, conclusa as seguintes etapas:

1. Use o seguinte comando para criar um arquivo chamado **coordinator.xml**:

    ```bash
    nano coordinator.xml
    ```

    Use o seguinte XML como o conteúdo do arquivo:

    ```xml
    <coordinator-app name="my_coord_app" frequency="${coordFrequency}" start="${coordStart}" end="${coordEnd}" timezone="${coordTimezone}" xmlns="uri:oozie:coordinator:0.4">
        <action>
        <workflow>
            <app-path>${workflowPath}</app-path>
        </workflow>
        </action>
    </coordinator-app>
    ```

    > [!NOTE]
    > As variáveis `${...}` são substituídas por valores na definição de trabalho no tempo de execução. As variáveis são:
    >
    > * `${coordFrequency}`: o tempo entre as instâncias do trabalho em execução.
    > * `${coordStart}`: a hora de início do trabalho.
    > * `${coordEnd}`: a hora de término do trabalho.
    > * `${coordTimezone}`: os trabalhos do coordenador estão em um fuso horário fixo sem horário de verão, geralmente representado com o uso do UTC. Esse fuso horário é chamado de *fuso horário de processamento do Oozie*.
    > * `${wfPath}`: o caminho para workflow.xml.

2. Para salvar o arquivo, selecione Ctrl+X, digite `Y`e selecione **Enter**.

3. Para copiar o arquivo para o diretório de trabalho para este trabalho, use o seguinte comando:

    ```bash
    hadoop fs -put coordinator.xml /tutorials/useoozie/coordinator.xml
    ```

4. Para modificar o arquivo `job.xml`, use o seguinte comando:

    ```
    nano job.xml
    ```

    Faça as seguintes alterações:

   * Para instruir o Oozie a executar o arquivo coordenador em vez do de fluxo de trabalho, altere `<name>oozie.wf.application.path</name>` para `<name>oozie.coord.application.path</name>`.

   * Para definir o `workflowPath` variável usada pelo coordenador, adicione o XML a seguir:

        ```xml
        <property>
            <name>workflowPath</name>
            <value>wasb://mycontainer@mystorageaccount.blob.core.windows.net/tutorials/useoozie</value>
        </property>
        ```

       Substitua o texto `wasb://mycontainer@mystorageaccount.blob.core.windows` pelo valor usado nas outras entradas no arquivo job.xml.

   * Para definir o início, o fim e a frequência a usar para o coordenador, adicione o XML a seguir:

        ```xml
        <property>
            <name>coordStart</name>
            <value>2017-05-10T12:00Z</value>
        </property>

        <property>
            <name>coordEnd</name>
            <value>2017-05-12T12:00Z</value>
        </property>

        <property>
            <name>coordFrequency</name>
            <value>1440</value>
        </property>

        <property>
            <name>coordTimezone</name>
            <value>UTC</value>
        </property>
        ```

       Esses valores definem a hora de início como 12h00 em 10 de maio de 2017 e a hora de término como 12 de maio de 2017. O intervalo para execução desse trabalho é definido como diário. A frequência está em minutos, então 24 horas x 60 minutos = 1440 minutos. Por fim, o fuso horário é definido como UTC.

5. Para salvar o arquivo, selecione Ctrl+X, digite `Y`e selecione **Enter**.

6. Para executar o trabalho, use o seguinte comando:

    ```
    oozie job -config job.xml -run
    ```

    Esse comando envia e inicia o trabalho.

7. Se você acessar a interface do usuário da Web do Oozie e selecionar a guia **Trabalhos do Coordenador**, verá informações semelhantes à seguinte imagem:

    ![Guia Trabalhos do coordenador](./media/hdinsight-use-oozie-linux-mac/coordinatorjob.png)

    A entrada **Próxima Materialização** contém a próxima hora em que o trabalho é executado.

8. De forma semelhante ao fluxo de trabalho anterior, se você selecionar a entrada de trabalho na interface do usuário da Web, ela exibirá informações sobre o trabalho:

    ![Informações de trabalho do coordenador](./media/hdinsight-use-oozie-linux-mac/coordinatorjobinfo.png)

    > [!NOTE]
    > Essa imagem mostra apenas as execuções bem-sucedidas do trabalho, não as ações individuais do fluxo de trabalho agendado. Para ver as ações individuais, selecione uma das entradas em **Ação**.

    ![Informações da ação](./media/hdinsight-use-oozie-linux-mac/coordinatoractionjob.png)

## <a name="troubleshooting"></a>solução de problemas

Com a IU do Oozie, você pode exibir logs do Oozie. A IU do Oozie também contém links para os logs de JobTracker das tarefas de MapReduce que foram iniciadas pelo fluxo de trabalho. O padrão para solução de problemas deve ser:

   1. Exiba o trabalho na interface do usuário da Web do Oozie.

   2. Se houver um erro ou falha de uma ação específica, selecione a ação para ver se o campo **Mensagem de Erro** fornece mais informações sobre a falha.

   3. Se estiver disponível, use a URL da ação para exibir mais detalhes, como logs do JobTracker, sobre ela.

A seguir estão erros específicos que podem ser encontrados e como resolvê-los.

### <a name="ja009-cannot-initialize-cluster"></a>JA009: não é possível inicializar o cluster

**Sintomas**: o status do trabalho é alterado para **SUSPENSO**. Os detalhes do trabalho mostram o status de `RunHiveScript` como **START_MANUAL**. Selecionar a ação exibirá a seguinte mensagem de erro:

    JA009: Cannot initialize Cluster. Please check your configuration for map

**Causa**: os endereços do Armazenamento de Blobs do Azure usados no arquivo **job.xml** não contêm o contêiner de armazenamento ou o nome da conta de armazenamento. O formato de endereço do armazenamento de blobs deve ser `wasb://containername@storageaccountname.blob.core.windows.net`.

**Resolução**: altere os endereços de armazenamento de blobs que o trabalho usa.

### <a name="ja002-oozie-is-not-allowed-to-impersonate-ltuser"></a>JA002: o Oozie não tem permissão para representar o &lt;USER>

**Sintomas**: o status do trabalho é alterado para **SUSPENSO**. Os detalhes do trabalho mostram o status de `RunHiveScript` como **START_MANUAL**. Se você selecionar a ação, a seguinte mensagem de erro será exibida:

    JA002: User: oozie is not allowed to impersonate <USER>

**Causa**: as configurações de permissão atuais não permitem ao Oozie representar a conta de usuário especificada.

**Resolução**: o Oozie pode representar os usuários no grupo **usuários**. Use o `groups USERNAME` para ver os grupos do qual a conta de usuário é membro. Se o usuário não for membro do grupo **usuários** , use o seguinte comando para adicionar o usuário ao grupo:

    sudo adduser USERNAME users

> [!NOTE]
> Pode levar vários minutos antes de o HDInsight reconhecer que o usuário foi adicionado ao grupo.

### <a name="launcher-error-sqoop"></a>ERRO do Iniciador (Sqoop)

**Sintomas**: o status do trabalho é alterado para **ENCERRADO**. Os detalhes do trabalho mostram o status `RunSqoopExport` como **ERRO**. Se você selecionar a ação, a seguinte mensagem de erro será exibida:

    Launcher ERROR, reason: Main class [org.apache.oozie.action.hadoop.SqoopMain], exit code [1]

**Causa**: o Sqoop não conseguiu carregar o driver do banco de dados necessário para acessar o banco de dados.

**Resolução**: ao usar o Sqoop em um trabalho do Oozie, você deve incluir o driver de banco de dados com os outros recursos, como o workflow.xml, usados pelo trabalho. Além disso, referencie o arquivo morto que contém o driver de banco de dados na seção `<sqoop>...</sqoop>` do workflow.xml.

Por exemplo, para o trabalho neste documento, você usaria o seguinte procedimento:

1. Copie o arquivo `sqljdbc4.1.jar` para o diretório **/tutorials/useoozie**:

    ```bash
    hdfs dfs -put /usr/share/java/sqljdbc_4.1/enu/sqljdbc41.jar /tutorials/useoozie/sqljdbc41.jar
    ```

2. Modifique o `workflow.xml` para adicionar o seguinte XML a uma nova linha acima `</sqoop>`:

    ```xml
    <archive>sqljdbc41.jar</archive>
    ```

## <a name="next-steps"></a>Próximas etapas

Neste tutorial, você aprendeu a definir um fluxo de trabalho do Oozie e a executar um trabalho do Oozie. Para saber mais sobre como trabalhar com o HDInsight, consulte os seguintes artigos:

* [Usar o Coordenador do Oozie baseado em tempo com o HDInsight][hdinsight-oozie-coordinator-time]
* [Carregar dados para trabalhos do Hadoop no HDInsight][hdinsight-upload-data]
* [Usar Sqoop com o Hadoop no HDInsight][hdinsight-use-sqoop]
* [Usar o Hive com Hadoop no HDInsight][hdinsight-use-hive]
* [Usar o Pig com Hadoop no HDInsight][hdinsight-use-pig]
* [Desenvolver programas Java MapReduce para HDInsight][hdinsight-develop-mapreduce]

[hdinsight-cmdlets-download]: http://go.microsoft.com/fwlink/?LinkID=325563
[azure-data-factory-pig-hive]: ../data-factory/transform-data.md
[hdinsight-oozie-coordinator-time]: hdinsight-use-oozie-coordinator-time.md
[hdinsight-versions]:  hdinsight-component-versioning.md
[hdinsight-storage]: hdinsight-use-blob-storage.md
[hdinsight-get-started]: hdinsight-get-started.md
[hdinsight-use-sqoop]:hadoop/apache-hadoop-use-sqoop-mac-linux.md
[hdinsight-provision]: hdinsight-hadoop-provision-linux-clusters.md
[hdinsight-upload-data]: hdinsight-upload-data.md
[hdinsight-use-mapreduce]:hadoop/hdinsight-use-mapreduce.md
[hdinsight-use-hive]:hadoop/hdinsight-use-hive.md
[hdinsight-use-pig]:hadoop/hdinsight-use-pig.md
[hdinsight-storage]: hdinsight-use-blob-storage.md
[hdinsight-get-started-emulator]: hdinsight-get-started-emulator.md
[hdinsight-develop-mapreduce]:hadoop/apache-hadoop-develop-deploy-java-mapreduce-linux.md

[sqldatabase-create-configue]: sql-database-create-configure.md
[sqldatabase-get-started]: sql-database-get-started.md

[azure-create-storageaccount]:../storage/common/storage-create-storage-account.md

[apache-hadoop]: http://hadoop.apache.org/
[apache-oozie-400]: http://oozie.apache.org/docs/4.0.0/
[apache-oozie-332]: http://oozie.apache.org/docs/3.3.2/

[powershell-download]: http://azure.microsoft.com/downloads/
[powershell-about-profiles]: http://go.microsoft.com/fwlink/?LinkID=113729
[powershell-install-configure]: /powershell/azureps-cmdlets-docs
[powershell-start]: http://technet.microsoft.com/library/hh847889.aspx
[powershell-script]: https://technet.microsoft.com/en-us/library/ee176961.aspx

[cindygross-hive-tables]: http://blogs.msdn.com/b/cindygross/archive/2013/02/06/hdinsight-hive-internal-and-external-tables-intro.aspx

[img-workflow-diagram]: ./media/hdinsight-use-oozie/HDI.UseOozie.Workflow.Diagram.png
[img-preparation-output]: ./media/hdinsight-use-oozie/HDI.UseOozie.Preparation.Output1.png
[img-runworkflow-output]: ./media/hdinsight-use-oozie/HDI.UseOozie.RunWF.Output.png

[technetwiki-hive-error]: http://social.technet.microsoft.com/wiki/contents/articles/23047.hdinsight-hive-error-unable-to-rename.aspx
