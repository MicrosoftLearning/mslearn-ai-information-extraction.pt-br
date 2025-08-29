---
lab:
  title: Analisar formulários com modelos predefinidos da IA do Azure para Informação de Documentos
  description: Use modelos predefinidos de IA do Azure para Informação de Documentos para processar campos de texto de documentos.
---

# Analisar formulários com modelos predefinidos da IA do Azure para Informação de Documentos

Neste exercício, você configurará um projeto da Fábrica de IA do Azure com todos os recursos necessários para a análise de documentos. Você usará o portal da Fábrica de IA do Azure e o SDK do Python para enviar formulários para esse recurso para análise.

Embora este exercício seja baseado em Python, você pode desenvolver aplicativos semelhantes usando vários SDKs específicos da linguagem, incluindo:

- [Biblioteca de clientes de IA do Azure para Informação de Documentos para Python](https://pypi.org/project/azure-ai-formrecognizer/)
- [Biblioteca de clientes de IA do Azure para Informação de Documentos para Microsoft .NET](https://www.nuget.org/packages/Azure.AI.FormRecognizer)
- [Biblioteca de clientes de IA do Azure para Informação de Documentos para JavaScript](https://www.npmjs.com/package/@azure/ai-form-recognizer)

Este exercício levará aproximadamente **30** minutos.

## Criar um projeto da Fábrica de IA do Azure

Vamos começar criando um projeto da Fábrica de IA do Azure.

1. Em um navegador da Web, abra o [Portal da Fábrica de IA do Azure](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure. Feche todas as dicas ou painéis de início rápido abertos na primeira vez que você entrar e, se necessário, use o logotipo da **Fábrica de IA do Azure** no canto superior esquerdo para navegar até a home page, que é semelhante à imagem a seguir (feche o painel **Ajuda** se estiver aberto):

    ![Captura de tela do portal do Azure AI Foundry.](./media/ai-foundry-home.png)

1. No navegador, navegue até `https://ai.azure.com/managementCenter/allResources` e clique em **Criar**. Em seguida, escolha a opção para criar um novo **Recurso do hub de IA**.
1. No assistente **Criar projeto**, insira um nome válido para o projeto e selecione a opção para criar um novo hub. Em seguida, use o link **Renomear hub** para especificar um nome válido para o novo hub, expanda **Opções avançadas** e especifique as seguintes configurações para o projeto:
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *criar ou selecionar um grupo de recursos*
    - **Região**:  *qualquer região disponível*

    > **Observação**: se você estiver trabalhando em uma assinatura do Azure na qual as políticas são usadas para restringir nomes de recursos permitidos, talvez seja necessário usar o link na parte inferior da caixa de diálogo **Criar um novo projeto** para criar o hub usando o portal do Azure.

    > **Dica**: se o botão **Criar** ainda estiver desativado, certifique-se de renomear o hub com um valor alfanumérico exclusivo.

1. Aguarde até que seu projeto seja criado.
1. Quando o projeto for criado, feche todas as dicas exibidas e examine a página do projeto no Portal da Fábrica de IA do Azure, que deve ser semelhante à imagem a seguir:

    ![Captura de tela dos detalhes de um projeto IA do Azure no Portal da Fábrica de IA do Azure.](./media/ai-foundry-project.png)

## Use o modelo de leitura

Vamos começar usando o portal **Fábrica de IA do Azure** e o modelo de leitura para analisar um documento com vários idiomas:

1. No painel de navegação à esquerda, selecione **Serviços de IA**.
1. Na página **IA de Serviços de IA do Azure**, selecione o bloco **Visão + Documento**.
1. Na página **Visão + Documento**, verifique se a guia **Documento** está selecionada e selecione o bloco **OCR/Leitura**.

    Na página **Leitura**, o recurso Serviços de IA do Azure criado com seu projeto já deve estar conectado.

1. Na lista de documentos à esquerda, selecione **read-german.png**.

    ![Captura de tela mostrando a página de leitura no Estúdio da IA do Azure para Informação de Documentos.](./media/read-german-sample.png#lightbox)

1. Na barra de ferramentas superior, selecione **Analisar opções** e habilite a caixa de seleção **Idioma** (em **Detecção opcional**) no painel **Analisar opções** e selecione **Salvar**. 
1. No canto superior esquerdo, selecione **Executar análise**.
1. Quando a análise é concluída, o texto extraído da imagem é mostrado à direita na guia **Conteúdo**. Examine esse texto e compare-o com o texto na imagem original para ver a precisão.
1. Selecione a guia **Resultado**. Essa guia exibe o código JSON extraído. 

## Preparar-se para desenvolver um aplicativo no Cloud Shell

Agora vamos explorar o aplicativo que usa o SDK do serviço Informação de Documentos do Azure. Você desenvolverá seu aplicativo usando o Cloud Shell. Os arquivos de código do seu aplicativo foram fornecidos em um repositório do GitHub.

Esse é a fatura que o código analisará.

![Captura de tela mostrando um documento de fatura de exemplo.](./media/sample-invoice.png)

1. No Portal da Fábrica de IA do Azure, visualize a página **Visão geral** do seu projeto.
1. Na área **Pontos de extremidade e chaves**, selecione a guia **Serviços IA do Azure** e observe a **Chave de API** e o **ponto de extremidade de Serviços de IA do Azure**. Você usará essas credenciais para se conectar aos serviços IA do Azure em um aplicativo cliente.
1. Abra uma nova guia do navegador (mantendo o portal da Fábrica de IA do Azure aberto na guia existente). Em seguida, na nova guia, navegue até o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com`; efetue login com suas credenciais do Azure, se solicitado.
1. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure selecionando um ambiente do ***PowerShell***. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure.

    > **Observação**: se você já criou um Cloud Shell que usa um ambiente *Bash*, alterne-o para o ***PowerShell***.

1. Na barra de ferramentas do Cloud Shell, no menu **Configurações**, selecione **Ir para a versão clássica** (isso é necessário para usar o editor de código).

    **<font color="red">Verifique se você mudou para a versão clássica do Cloud Shell antes de continuar.</font>**

1. No painel do PowerShell, insira os seguintes comandos para clonar o repositório GitHub para este exercício:

    ```
   rm -r mslearn-ai-info -f
   git clone https://github.com/microsoftlearning/mslearn-ai-information-extraction mslearn-ai-info
    ```

    > **Dica**: conforme você colar comandos no cloudshell, a saída pode ocupar uma grande quantidade do espaço da tela. Você pode limpar a tela digitando o comando `cls` para facilitar o foco em cada tarefa.

    ***Agora siga as etapas para a linguagem de programação escolhida.***

1. Depois que o repositório tiver sido clonado, navegue até a pasta que contém os arquivos de código:

    ```
   cd mslearn-ai-info/Labfiles/prebuilt-doc-intelligence/Python
    ```

1. No painel de linha de comando do Cloud Shell, digite o seguinte comando para instalar as bibliotecas que você usará:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-formrecognizer==3.3.3
    ```

1. Digite o seguinte comando para editar o arquivo de configuração que foi fornecido:

    ```
   code .env
    ```

    O arquivo é aberto em um editor de código.

1. No arquivo de código, substitua os espaços reservados **YOUR_ENDPOINT** e **YOUR_KEY** pelo ponto de extremidade dos serviços IA do Azure e sua chave de API (copiada do portal da Fábrica de IA do Azure).
1. Depois de substituir os espaços reservados, no editor de código, use o comando **CTRL+S** para salvar as alterações e então use **CTRL+Q** para fechar o editor de código, mantendo a linha de comando do Cloud Shell aberta.

## Adicionar código para usar o serviço de Informação de Documentos do Azure

Agora você pode usar o SDK para avaliar o arquivo pdf.

1. Insira o seguinte comando para editar o arquivo de aplicativo fornecido:

    ```
   code document-analysis.py
    ```

    O arquivo é aberto em um editor de código.

1. No arquivo de código, localize o comentário **Importar as bibliotecas necessárias** e adicione o seguinte código:

    ```python
   # Add references
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.formrecognizer import DocumentAnalysisClient
    ```

1. Localize o comentário **Criar o cliente** e adicione o seguinte código (cuidado para manter o nível de recuo correto):

    ```python
   # Create the client
   document_analysis_client = DocumentAnalysisClient(
        endpoint=endpoint, credential=AzureKeyCredential(key)
   )
    ```

1. Localize o comentário **Analisar a fatura** e adicione o seguinte código:

    ```python
   # Analyse the invoice
   poller = document_analysis_client.begin_analyze_document_from_url(
        fileModelId, fileUri, locale=fileLocale
   )
    ```

1. Localize o comentário **Exibir informações da fatura ao usuário** e adicione o seguinte código:

    ```python
   # Display invoice information to the user
   receipts = poller.result()
    
   for idx, receipt in enumerate(receipts.documents):
    
        vendor_name = receipt.fields.get("VendorName")
        if vendor_name:
            print(f"\nVendor Name: {vendor_name.value}, with confidence {vendor_name.confidence}.")

        customer_name = receipt.fields.get("CustomerName")
        if customer_name:
            print(f"Customer Name: '{customer_name.value}, with confidence {customer_name.confidence}.")


        invoice_total = receipt.fields.get("InvoiceTotal")
        if invoice_total:
            print(f"Invoice Total: '{invoice_total.value.symbol}{invoice_total.value.amount}, with confidence {invoice_total.confidence}.")
    ```

1. No editor de código, use o comando **CTRL+S** ou **Clique com o botão direito do mouse > Salvar** para salvar suas alterações. Mantenha o editor de código aberto caso precise corrigir erros no código, mas redimensione os painéis para ver o painel de linha de comando claramente.

1. No painel de linha de comando, insira o comando a seguir para executar o aplicativo.

    ```
    python document-analysis.py
    ```

O programa exibe o nome do fornecedor, o nome do cliente e o total da fatura com os níveis de confiança. Compare os valores relatados com a fatura de exemplo que você abriu no início desta seção.

## Limpeza

Se você tiver terminado de usar o recurso do Azure, exclua o recurso no [portal do Azure](https://portal.azure.com) (`https://portal.azure.com`) para evitar mais cobranças.
