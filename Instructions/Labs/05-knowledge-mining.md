---
lab:
  title: Criar uma solução de mineração de conhecimento
  description: Use a Pesquisa de IA do Azure para extrair informações importantes de documentos e facilitar a pesquisa e a análise.
---

# Criar uma solução de mineração de conhecimento

Neste exercício, você usará a Pesquisa de IA para indexar um conjunto de documentos mantidos pela Margie's Travel, uma agência de viagens fictícia. O processo de indexação envolve o uso de habilidades de IA para extrair informações importantes para torná-las pesquisáveis e gerar um repositório de conhecimento que contém ativos de dados para análise posterior.

Embora este exercício seja baseado em Python, você pode desenvolver aplicativos semelhantes usando vários SDKs específicos da linguagem, incluindo:

- [Biblioteca de clientes de Pesquisa de IA do Azure para Python](https://pypi.org/project/azure-search-documents/)
- [Biblioteca de clientes de Pesquisa de IA do Azure para Microsoft .NET](https://www.nuget.org/packages/Azure.Search.Documents)
- [Biblioteca de clientes de Pesquisa de IA do Azure para JavaScript](https://www.npmjs.com/package/@azure/search-documents)

Este exercício leva aproximadamente **40** minutos.

## Criar recursos do Azure

A solução que você criará para a Margie's Travel requer vários recursos em sua assinatura do Azure. Neste exercício, você os criará diretamente no portal do Azure. Você também pode criar usando um script ou um modelo ARM ou BICEP; ou criar uma Fábrica de IA do Azure que inclui um recurso de Pesquisa de IA do Azure.

> **Importante**: seus recursos do Azure devem ser criados no mesmo local!

### Criar um recurso do Azure AI Search

1. Em um navegador da Web, abra o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com` e entre usando suas credenciais do Azure.
1. Selecione o botão **&#65291;Criar um recurso**, pesquise `Azure AI Search` e crie um recurso de **Pesquisa de IA do Azure** com as seguintes configurações:
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *criar ou selecionar um grupo de recursos*
    - **Nome do serviço**: *um nome válido para o recurso de pesquisa*
    - **Localização**: *Qualquer localização disponível*
    - **Tipo de preço**: Gratuito

1. Aguarde a conclusão da implantação e acesse o recurso implantado.
1. Examine a página **Visão geral** no painel do recurso de Pesquisa de IA do Azure no portal do Azure. Aqui, você pode usar uma interface visual para criar, testar, gerenciar e monitorar os vários componentes de uma solução de pesquisa; incluindo fontes de dados, índices, indexadores e conjuntos de habilidades.

### Criar uma conta de armazenamento

1. Retorne à home page e crie um recurso de **Conta de armazenamento** com as seguintes configurações:
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *o mesmo grupo de recursos que seus recursos da Pesquisa de IA do Azure e dos Serviços de IA do Azure*
    - **Nome da conta de armazenamento**: *um nome válido para o recurso de armazenamento*
    - **Região**: *a mesma região que o recurso de Pesquisa de IA do Azure*
    - **Serviço primário**: Armazenamento de Blobs do Azure ou Azure Data Lake Storage Gen2
    - **Desempenho**: padrão
    - **Redundância**: LRS (armazenamento com redundância local)

1. Aguarde a conclusão da implantação e acesse o recurso implantado.

    > **Dica**: mantenha a página do portal da conta de armazenamento aberta, você a usará no próximo procedimento.

## Carregar documentos no Armazenamento do Azure

sua solução de mineração de conhecimento extrairá informações de documentos de folheto de viagem em um contêiner de blobs do Armazenamento do Azure.

1. Em uma nova guia do navegador, baixe [documents.zip](https://github.com/microsoftlearning/mslearn-ai-information-extraction/raw/main/Labfiles/knowledge/documents.zip) de `https://github.com/microsoftlearning/mslearn-ai-information-extraction/raw/main/Labfiles/knowledge/documents.zip` e salve-o em uma pasta local.
1. Extraia o arquivo *documents.zip* baixado e exiba os arquivos de folheto de viagem que ele contém. Você extrairá e indexará informações desses arquivos.
1. Na guia do navegador que contém a portal do Azure da sua conta de armazenamento, no painel de navegação à esquerda, selecione **Navegador de armazenamento**.
1. No navegador de armazenamento, selecione **Contêineres de Blob**.

    Atualmente, sua conta de armazenamento deve ter apenas o contêiner **$logs** padrão.

1. Na barra de ferramentas, selecione **+ Contêiner** e crie um novo contêiner com as seguintes configurações:
    - **Nome**: `documents`
    - **Nível de acesso anônimo**: Privado (sem acesso anônimo)\*

    > **Observação**: \*a menos que você tenha habilitado a opção para permitir o acesso de contêiner anônimo ao criar sua conta de armazenamento, não poderá selecionar nenhuma outra configuração!

1. selecione o contêiner de **documentos** para abri-lo e use o botão da barra de ferramentas **Upload** para carregar os arquivos .pdf extraídos de **documents.zip** antes para a raiz do contêiner, conforme mostrado aqui:

    ![Captura de tela do navegador de armazenamento do Azure com o contêiner de documentos e conteúdos de arquivo.](./media/blob-containers.png)

## Criar e executar um indexador

Agora que você tem os documentos, pode criar um indexador para extrair informações deles.

1. No portal do Azure, procure pelo seu recurso da Pesquisa de IA do Azure. Em seguida, na página **Visão geral**, selecione **Importar dados**.
1. Na página **Conectar-se aos seus dados**, na lista **Fonte de Dados**, escolha **Armazenamento de Blobs do Azure**. Em seguida, preencha os detalhes do armazenamento de dados com os seguintes valores:
    - **Fonte de dados**: Armazenamento de Blobs do Azure
    - **Nome da fonte de dados**: `margies-documents`
    - **Dados para extração**: Conteúdo e metadados
    - **Modo de análise**: Padrão
    - **Assinatura**: *sua assinatura do Azure*
    - **Cadeia de conexão:** 
        - Selecione **Escolher uma conexão existente**
        - Selecione sua conta de armazenamento
        - Selecionar o contêiner de **documentos**
    - **Autenticação da identidade gerenciada**: Nenhuma
    - **Nome do contêiner**: documentos
    - **Pasta do blob**: *mantenha essa opção em branco*
    - **Descrição**: `Travel brochures`
1. Prossiga para a próxima etapa (**Adicionar habilidades cognitivas**), que tem três seções expansíveis a serem concluídas.
1. Na seção **Anexar Serviços de IA do Azure**, selecione **Gratuito (enriquecimentos limitados**)\*.

    > **Observação**: \*o recurso de Serviços de IA do Azure gratuito para Pesquisa de IA do Azure pode ser usado para indexar até 20 documentos. Em uma solução real, você deve criar um recurso IA do Azure Services em sua assinatura para habilitar o enriquecimento de IA para um número maior de documentos.

1. Na seção **Adicionar enriquecimentos**:
    - Altere o **nome do Conjunto de Habilidades** para `margies-skillset`.
    - Selecione a opção **Habilitar OCR e mesclar todo o texto no campo merged_content**.
    - Verifique se o **campo Dados de origem** está definido **como merged_content**.
    - Deixe o **nível de granularidade do Enriquecimento** como ** campo Origem**, que é definido por todo o conteúdo do documento que está sendo indexado, mas observe que você pode alterar isso para extrair informações em níveis mais granulares, como páginas ou frases.
    - Selecione os seguintes campos enriquecidos:

        | Habilidade cognitiva | Parâmetro | Nome do campo |
        | --------------- | ---------- | ---------- |
        | **Habilidades Cognitivas de Texto** | |  |
        | Extrair nomes de pessoas | | people |
        | Extrair nomes de localização | | Locais |
        | Extrair frases-chave | | keyphrases |
        | **Habilidades Cognitivas de Imagem** | |  |
        | Gerar marcas com base em imagens | | imageTags |
        | Gerar legendas com base em imagens | | imageCaption |

        Verifique as seleções mais uma vez (pode ser difícil alterá-las mais tarde).

1. na seção **Salvar aprimoramentos em um repositório de conhecimento**:
    - Marque apenas as seguintes caixas de seleção (um <font color="red">erro</font> será exibido, você resolverá isso em breve):
        - **Projeções de arquivo do Azure**:
            - Projeções de imagem
        - **Projeções de tabela do Azure**:
            - Documentos
                - Frases-chave
        - **Projeções de blob do Azure**:
            - Documento
    - Em **Cadeia de conexão da conta de armazenamento** (abaixo das <font color="red">mensagens de erro</font>):
        - Selecione **Escolher uma conexão existente**
        - Selecione sua conta de armazenamento
        - Selecione o contêiner de **documentos** (*isso só é necessário para selecionar a conta de armazenamento na interface de navegação – você especificará um nome de contêiner diferente para os ativos de conhecimento extraídos!*)
    - Altere o **Nome do contêiner** para `knowledge-store`.
1. Prossiga para a próxima etapa (**Personalizar índice de destino**), em que você especificará os campos para o índice. 
1. Altere o **Nome do índice** para `margies-index`.
1. Verifique se a **Chave** está definida como **metadata_storage_path**, deixe o **Nome do sugeridor** em branco e verifique se o **Modo de pesquisa** é **analyzingInfixMatching**.
1. Faça as seguintes alterações nos campos de índice, deixando todos os outros campos com suas configurações padrão (**IMPORTANTE**: talvez seja necessário rolar para a direita para ver a tabela inteira):

    | Nome do campo | Recuperável | Filtrável | Classificável | Com faceta | Pesquisável |
    | ---------- | ----------- | ---------- | -------- | --------- | ---------- |
    | metadata_storage_size | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | metadata_storage_last_modified | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | metadata_storage_name | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | Locais | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | people | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | keyphrases | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |

    Verifique novamente suas seleções, prestando atenção específica para garantir que as opções **Recuperável**, **Filtrável**, **Classificável**, **Facetável** e **Pesquisável** corretas estejam selecionadas para cada campo (pode ser difícil alterá-las posteriormente).

1. Prossiga para a próxima etapa (**Criar um indexador**), em que você criará e agendará o indexador.
1. Altere o **Nome do indexador** para `margies-indexer`.
1. Mantenha o **Agendamento** definido como **Uma vez**.
1. Escolha **Enviar** para criar a fonte de dados, o conjunto de habilidades, o índice e o indexador. O indexador é executado automaticamente e executa o pipeline de indexação, que:
    - Extrai os campos de metadados do documento e o conteúdo da fonte de dados
    - Executa o conjunto de habilidades cognitivas para gerar campos enriquecidos adicionais
    - Mapeia os campos extraídos para o índice.
    - Salva os ativos de dados extraídos no repositório de conhecimento.
1. No painel de navegação à esquerda, em **Gerenciamento de pesquisa** exiba a página **Indexadores**, que deve mostrar o **margies-indexer** recém-criado. Aguarde alguns minutos e clique em **&#8635; Atualizar** até que o **Status** indique **Êxito**.

## Pesquisar o índice

Agora que você tem um índice, você pode pesquisá-lo.

1. Retorne à página **Visão geral** do recurso Pesquisa de IA do Azure e, na barra de ferramentas, selecione **Gerenciador de pesquisa**.
1. No Gerenciador de pesquisa, na caixa **Cadeia de consulta**, digite `*` (um único asterisco) e selecione **Pesquisar**.

    Esta consulta recupera todos os documentos no índice no formato JSON. Examine os resultados e observe os campos de cada documento, que contêm conteúdo do documento, metadados e dados enriquecidos extraídos pelas habilidades cognitivas selecionadas.

1. No menu **Exibição**, selecione o **modo de exibição JSON** e observe que a solicitação JSON para a pesquisa é mostrada, da seguinte maneira:

    ```json
    {
      "search": "*",
      "count": true
    }
    ```

1. Os resultados incluem um campo **@odata.count** na parte superior dos resultados que indica o número de documentos que a pesquisa retornou.

1. Modifique a solicitação JSON para incluir o parâmetro **select**, conforme mostrado aqui:

    ```json
    {
      "search": "*",
      "count": true,
      "select": "metadata_storage_name,locations"
    }
    ```

    Desta vez, os resultados incluem apenas o nome do arquivo e os locais mencionados no conteúdo do documento. O nome do arquivo está no campo **metadata_storage_name**, que foi extraído do documento de origem. O campo **locais** foi gerado por uma habilidade de IA.

1. Agora tente a seguinte cadeia de caracteres de consulta:

    ```json
    {
      "search": "New York",
      "count": true,
      "select": "metadata_storage_name,keyphrases"
    }
    ```

    Esta pesquisa localiza documentos que mencionam "New York" em qualquer um dos campos pesquisáveis e retorna o nome do arquivo e as frases principais no documento.

1. Vamos tentar mais uma consulta:

    ```json
    {
        "search": "New York",
        "count": true,
        "select": "metadata_storage_name,keyphrases",
        "filter": "metadata_storage_size lt 380000"
    }
    ```

    Essa consulta retorna o nome do arquivo e as frases-chave para todos os documentos que mencionam "Nova York" com menos de 380 mil bytes.

## Criar um aplicativo cliente de pesquisa

Agora que você tem um índice útil, você pode usá-lo a partir de um aplicativo cliente. Você pode fazer isso consumindo a interface REST, enviando solicitações e recebendo respostas no formato JSON sobre HTTP; ou você pode usar o SDK (kit de desenvolvimento de software) para sua linguagem de programação preferida. Neste exercício, usaremos o SDK.

> **Observação**: você pode optar por usar o SDK para **C#** ou **Python**. Nas etapas abaixo, execute as ações apropriadas para a linguagem de sua preferência.

### Obter o ponto de extremidade e chaves para seu recurso de pesquisa

1. Na página portal do Azure, feche a página do gerenciador de pesquisa e retorne à página **Visão geral** do recurso de Pesquisa de IA do Azure.

    Observe o valor da **URL**, que deve ser semelhante a **https://*your_resource_name*.search.windows.net**. Este é o ponto de extremidade para seu recurso de pesquisa.

1. No painel de navegação à esquerda, expanda **Configurações** e exiba a página **Chaves**.

    Observe que há duas chaves de **administrador** e uma única chave de **consulta**. Uma chave de *administrador* é usada para criar e gerenciar recursos de pesquisa, uma chave de *consulta* é usada por aplicativos cliente que só precisam executar consultas de pesquisa.

    *Você precisará do ponto de extremidade e da chave de **consulta** para seu aplicativo cliente.*

### Preparar-se para usar o SDK de Pesquisa de IA do Azure

1. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior do portal do Azure para criar um Cloud Shell no portal do Azure selecionando um ambiente do ***PowerShell*** sem armazenamento em sua assinatura.

    O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure. Você pode redimensionar ou maximizar esse painel para facilitar o trabalho. Inicialmente, você precisará ver o Cloud Shell e o portal do Azure (para localizar e copiar o ponto de extremidade e a chave de que precisará).

1. Na barra de ferramentas do Cloud Shell, no menu **Configurações**, selecione **Ir para a versão clássica** (isso é necessário para usar o editor de código).

    **<font color="red">Verifique se você mudou para a versão clássica do Cloud Shell antes de continuar.</font>**

1. No painel do Cloud Shell, insira os seguintes comandos para clonar o repositório GitHub que contém os arquivos de código para este exercício (digite o comando ou copie-o para a área de transferência e clique com o botão direito do mouse na linha de comando e cole como texto sem formatação):

    ```
   rm -r mslearn-ai-info -f
   git clone https://github.com/microsoftlearning/mslearn-ai-information-extraction mslearn-ai-info
    ```

    > **Dica**: ao colar comandos no Cloud Shell, a saída poderá ocupar uma grande quantidade do buffer da tela. Você pode limpar a tela digitando o comando `cls` para facilitar o foco em cada tarefa.

1. Após o repositório ser clonado, navegue até a pasta que contém os arquivos de código do aplicativo:

    ```
   cd mslearn-ai-info/Labfiles/knowledge/python
   ls -a -l
    ```

1. Instale o SDK de Pesquisa de IA do Azure e os pacotes de identidade do Azure executando os seguintes comandos:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-identity azure-search-documents==11.5.1
    ```

1. Execute o seguinte comando para editar o arquivo de configuração do aplicativo:

    ```
   code .env
    ```

    O arquivo de configuração é aberto em um editor de código.

1. Edite o arquivo de configuração para substituir os seguintes valores de espaço reservado:

    - **your_search_endpoint** (*substitua pelo ponto de extremidade para seu recurso de Pesquisa de IA do Azure*)
    - **your_query_key** *(substitua pela chave de consulta para seu recurso de Pesquisa de IA do Azure*)
    - **your_index_name** (*substitua pelo nome do índice, que deve ser `margies-index`*)

1. Quando você tiver atualizado os espaços reservados, use o comando **CTRL+S** para salvar o arquivo e use o comando **CTRL+Q** para fechar o arquivo.

    > **Dica**: agora que você copiou o ponto de extremidade e a chave do portal do Azure, maximize o painel do Cloud Shell para facilitar o trabalho.

1. Execute o seguinte comando para abrir o arquivo de código para seu aplicativo:

    ```
   code search-app.py
    ```

    O arquivo de código é aberto em um editor de código.

1. Examine o código e observe que ele executa as seguintes ações:

    - Recupera as definições de configuração para o recurso Pesquisa de IA do Azure e o índice do arquivo de configuração que você editou.
    - Cria um **SearchClient** com o ponto de extremidade, a chave e o nome do índice para se conectar ao serviço de pesquisa.
    - Solicita ao usuário uma consulta de pesquisa (até que ele insira "sair")
    - Pesquisa o índice usando a consulta, retornando os seguintes campos (ordenados por metadata_storage_name):
        - metadata_storage_name
        - Locais
        - people
        - keyphrases
    - Analisa os resultados da pesquisa retornados para exibir os campos retornados para cada documento no conjunto de resultados.

1. Feche o painel do editor de código (*CTRL+Q*), mantendo o painel de console da linha de comando do Cloud Shell aberto
1. Insira o seguinte comando para executar o aplicativo:

    ```
   python search-app.py
    ```

1. Quando solicitado, insira uma consulta como `London` e exiba os resultados.
1. Tente outra consulta, como `flights`.
1. Ao terminar de testar o aplicativo, digite `quit` para fechá-lo.
1. Feche o Cloud Shell, retornando ao portal do Azure.

## Exibir o repositório de conhecimento

Depois de executar um indexador que usa um conjunto de habilidades para criar um repositório de conhecimento, os dados enriquecidos extraídos pelo processo de indexação são persistidos nas projeções do repositório de conhecimento.

### Exibir projeções de objeto

As projeções de *objeto* definidas no conjunto de habilidades da Margie's Travel consistem em um arquivo JSON para cada documento indexado. Esses arquivos são armazenados em um contêiner de blob na conta de Armazenamento do Azure especificada na definição do conjunto de habilidades.

1. No portal do Azure, exiba a conta de armazenamento do Azure criada anteriormente.
1. Selecione a guia do **navegador de armazenamento** (no painel à esquerda) para ver a conta de armazenamento na interface do gerenciador de armazenamento no portal do Azure.
1. Expanda **contêineres de blob** para ver os contêineres na conta de armazenamento. Além do contêiner de **documentos** em que os dados de origem são armazenados, deve haver dois novos contêineres: **knowledge-store** e **margies-skillset-image-projection**. Eles foram criados pelo processo de indexação.
1. Selecione o contêiner **knowledge-store**. Ele deve conter uma pasta para cada documento indexado.
1. Abra qualquer uma das pastas e selecione o arquivo **objectprojection.json** que ele contém e use o botão **Baixar** na barra de ferramentas para baixá-lo e abri-lo. Cada arquivo JSON contém uma representação de um documento indexado, incluindo os dados enriquecidos extraídos pelo conjunto de habilidades, conforme mostrado aqui (formatado para facilitar a leitura).

    ```json
    {
        "metadata_storage_content_type": "application/pdf",
        "metadata_storage_size": 388622,
        "<more_metadata_fields>": "...",
        "key_phrases":[
            "Margie’s Travel",
            "Margie's Travel",
            "best travel experts",
            "world-leading travel agency",
            "international reach"
            ],
        "locations":[
            "Dubai",
            "Las Vegas",
            "London",
            "New York",
            "San Francisco"
            ],
        "image_tags":[
            "outdoor",
            "tree",
            "plant",
            "palm"
            ],
        "more fields": "..."
    }
    ```

A capacidade de criar projeções de *objeto* como essa permite gerar objetos de dados enriquecidos que podem ser incorporados em uma solução de análise de dados corporativos.

### Exibir projeções de arquivo

As projeções de *arquivo* definidas no conjunto de habilidades criam arquivos JPEG para cada imagem que foi extraída dos documentos durante o processo de indexação.

1. Na interface do *Navegador de armazenamento* no portal do Azure, selecione contêiner de blob **margies-skillset-image-projection**. Ele contém uma pasta para cada documento que continha imagens.
2. Abra qualquer uma das pastas e veja o conteúdo – cada pasta contém pelo menos um arquivo \*. jpg.
3. Abra qualquer um dos arquivos de imagem e baixe-os e exiba-os para ver a imagem.

A capacidade de gerar projeções de *arquivo* faz da indexação uma forma eficiente de extrair imagens incorporadas de um grande volume de documentos.

### Exibir projeções de tabela

As projeções de *tabela* definidas no conjunto de habilidades formam um esquema relacional de dados enriquecidos.

1. Na interface do *Navegador de armazenamento* no portal do Azure, expanda **Tabelas**.
2. Selecione a tabela **margiesSkillsetDocument** para exibir dados. Esta tabela contém uma linha para cada documento indexado:
3. Exiba a tabela **margiesSkillsetKeyPhrases**, que contém uma linha para cada frase-chave extraída dos documentos.

A capacidade de criar projeções de *tabela* permite criar soluções analíticas e de relatório que consultam um esquema relacional. As colunas de chave geradas automaticamente podem ser usadas para unir as tabelas em consultas, por exemplo, para retornar todas as frases-chave extraídas de um documento específico.

## Limpar

Agora que você concluiu o exercício, exclua todos os recursos de que não precisa mais. Exclua os recursos do Azure:

1. No **portal do Azure**, selecione Grupos de recursos.
1. Selecione o grupo de recursos que você não precisa e, em seguida, selecione **Excluir grupo de recursos**.

## Mais informações

Para saber mais sobre o serviço de Pesquisa IA do Azure, confira a [documentação da Pesquisa de IA do Azure](https://docs.microsoft.com/azure/search/search-what-is-azure-search).
