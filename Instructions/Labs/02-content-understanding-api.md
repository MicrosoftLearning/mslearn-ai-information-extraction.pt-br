---
lab:
  title: Desenvolver um aplicativo cliente de Compreensão de Conteúdo
  description: Use a API REST de Compreensão de Conteúdo da IA do Azure para desenvolver um aplicativo cliente para um analisador.
---

# Desenvolver um aplicativo cliente de Compreensão de Conteúdo

Neste exercício, você usará a Compreensão de Conteúdo da IA do Azure para criar um analisador que extrai informações de cartões de visita. Em seguida, você desenvolverá um aplicativo cliente que usa o analisador para extrair detalhes de contato de cartões de visita digitalizados.

Este exercício levará aproximadamente **30** minutos.

## Crie um hub e projeto da Fábrica de IA do Azure

Os recursos da Fábrica de IA do Azure que usaremos neste exercício requerem um projeto baseado em um recurso de *hub* da Fábrica de IA do Azure.

1. Em um navegador da Web, abra o [Portal da Fábrica de IA do Azure](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure. Feche todas as dicas ou painéis de início rápido abertos na primeira vez que você entrar e, se necessário, use o logotipo da **Fábrica de IA do Azure** no canto superior esquerdo para navegar até a home page, que é semelhante à imagem a seguir (feche o painel **Ajuda** se estiver aberto):

    ![Captura de tela do portal do Azure AI Foundry.](./media/ai-foundry-home.png)

1. No navegador, navegue até `https://ai.azure.com/managementCenter/allResources` e clique em **Criar**. Em seguida, escolha a opção para criar um novo **Recurso do hub de IA**.
1. No assistente **Criar projeto**, insira um nome válido para o projeto e selecione a opção para criar um novo hub. Em seguida, use o link **Renomear hub** para especificar um nome válido para o novo hub, expanda **Opções avançadas** e especifique as seguintes configurações para o projeto:
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *criar ou selecionar um grupo de recursos*
    - **Nome do hub**: um nome válido para o hub
    - **Localização**: escolha um dos seguintes locais:\*
        - Leste da Austrália
        - Suécia Central
        - Oeste dos EUA

    > \*No momento da redação, a compreensão de Conteúdo da IA do Azure só estava disponível nessas regiões.

    > **Dica**: se o botão **Criar** ainda estiver desativado, certifique-se de renomear o hub com um valor alfanumérico exclusivo.

1. Aguarde até que seu projeto seja criado e navegue até a página de visão geral do projeto.

## Usar a API REST para criar um analisador de Compreensão de Conteúdo

Você usará a API REST para criar um analisador que pode extrair informações de imagens de cartões de visita.

1. Abra uma nova guia do navegador (mantendo o portal da Fábrica de IA do Azure aberto na guia existente). Em seguida, na nova guia, navegue até o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com`; efetue login com suas credenciais do Azure, se solicitado.

    Feche todas as notificações de boas-vindas para ver a home page do portal do Azure.

1. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure selecionando um ambiente do ***PowerShell*** sem armazenamento em sua assinatura.

    O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure. Você pode redimensionar ou maximizar esse painel para facilitar o trabalho.

    > **Dica**: redimensione o painel para que você possa trabalhar principalmente no Cloud Shell, mas ainda ver as chaves e o ponto de extremidade na página portal do Azure. Você precisará copiá-las para seu código.

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
   cd mslearn-ai-info/Labfiles/content-app
   ls -a -l
    ```

    A pasta contém duas imagens de cartão de visita digitalizadas, bem como os arquivos de código Python necessários para compilar seu aplicativo.

1. No painel de linha de comando do Cloud Shell, digite o seguinte comando para instalar as bibliotecas que você usará:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt
    ```

1. Digite o seguinte comando para editar o arquivo de configuração que foi fornecido:

    ```
   code .env
    ```

    O arquivo é aberto em um editor de código.

1. No arquivo de código, substitua os espaços reservados **YOUR_ENDPOINT** e **YOUR_KEY** pelo ponto de extremidade dos serviços IA do Azure e qualquer uma das respectivas chaves (copiadas do portal do Azure) e garanta que **ANALYZER_NAME** esteja definido como `business-card-analyzer`.
1. Depois de substituir os espaços reservados, no editor de código, use o comando **CTRL+S** para salvar as alterações e então use **CTRL+Q** para fechar o editor de código, mantendo a linha de comando do Cloud Shell aberta.

    > **Dica**: Você pode maximizar o painel do Cloud Shell agora.

1. Na linha de comando do Cloud Shell, digite o seguinte comando para exibir o arquivo JSON **biz-card.json** fornecido:

    ```
   cat biz-card.json
    ```

    Role o painel do Cloud Shell para exibir o JSON no arquivo, que define um esquema de analisador para um cartão de visita.

1. Quando você tiver exibido o arquivo JSON para o analisador, insira o seguinte comando para editar o arquivo de código Python **create-analyzer.py** fornecido:

    ```
   code create-analyzer.py
    ```

    O arquivo de código Python é aberto em um editor de código.

1. Revise o código, que:
    - Carrega o esquema do analisador do arquivo **biz-card.json**.
    - Recupera o ponto de extremidade, a chave e o nome do analisador do arquivo de configuração de ambiente.
    - Chama uma função **create_analyzer**, que não está implementada

1. Na função **create_analyzer**, localize o comentário **Criar um analisador de Compreensão de Conteúdo** e adicione o seguinte código (cuidado para manter o recuo correto):

    ```python
   # Create a Content Understanding analyzer
   print (f"Creating {analyzer}")

   # Set the API version
   CU_VERSION = "2025-05-01-preview"

   # initiate the analyzer creation operation
   headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/json"}

   url = f"{endpoint}/contentunderstanding/analyzers/{analyzer}?api-version={CU_VERSION}"

   # Delete the analyzer if it already exists
   response = requests.delete(url, headers=headers)
   print(response.status_code)
   time.sleep(1)

   # Now create it
   response = requests.put(url, headers=headers, data=(schema))
   print(response.status_code)

   # Get the response and extract the callback URL
   callback_url = response.headers["Operation-Location"]

   # Check the status of the operation
   time.sleep(1)
   result_response = requests.get(callback_url, headers=headers)

   # Keep polling until the operation is no longer running
   status = result_response.json().get("status")
   while status == "Running":
        time.sleep(1)
        result_response = requests.get(callback_url, headers=headers)
        status = result_response.json().get("status")

   result = result_response.json().get("status")
   print(result)
   if result == "Succeeded":
        print(f"Analyzer '{analyzer}' created successfully.")
   else:
        print("Analyzer creation failed.")
        print(result_response.json())
    ```

1. Examine o código que você adicionou, que:
    - Cria os cabeçalhos apropriados para as solicitações REST
    - Envia uma solicitação HTTP *DELETE* para excluir o analisador se ele já existir.
    - Envia uma solicitação HTTP *PUT* para iniciar a criação do analisador.
    - Verifica a resposta para recuperar a URL de retorno de chamada *Operation-Location*.
    - Envia repetidamente uma solicitação HTTP *GET* para a URL de retorno de chamada para verificar o status da operação até que ela não esteja mais em execução.
    - Confirma o êxito (ou a falha) da operação para o usuário.

    > **Observação**: o código inclui alguns atrasos de tempo deliberados para evitar exceder o limite de taxa de solicitação para o serviço.

1. Use o comando **CTRL+S** para salvar as alterações de código, mas mantenha o painel do editor de código aberto caso precise corrigir erros no código. Redimensione os painéis para ver claramente o painel da linha de comando.
1. No painel de linha de comando do Cloud Shell, insira o seguinte comando para executar o código Python:

    ```
   python create-analyzer.py
    ```

1. Examine a saída do programa, que deve indicar que o analisador foi criado.

## Usar a API REST para analisar o conteúdo

Agora que você criou um analisador, pode consumi-lo em um aplicativo cliente por meio da API REST de Compreensão de conteúdo.

1. Na linha de comando do Cloud Shell, digite o seguinte comando para editar o arquivo de código Python **read-card.py** fornecido:

    ```
   code read-card.py
    ```

    O arquivo de código Python é aberto em um editor de código:

1. Revise o código, que:
    - Identifica o arquivo de imagem a ser analisado, com um padrão de **biz-card-1.png**.
    - Recupera o ponto de extremidade e a chave do recurso Serviços de IA do Azure do projeto (usando as credenciais do Azure da sessão atual do Cloud Shell para autenticar).
    - Chama uma função **analyze_card**, que não está implementada

1. Na função **analyze_card**, localize o comentário **Usar a Compreensão de Conteúdo** para analisar a imagem e adicionar o seguinte código (cuidado para manter o recuo correto):

    ```python
   # Use Content Understanding to analyze the image
   print (f"Analyzing {image_file}")

   # Set the API version
   CU_VERSION = "2025-05-01-preview"

   # Read the image data
   with open(image_file, "rb") as file:
        image_data = file.read()
    
   ## Use a POST request to submit the image data to the analyzer
   print("Submitting request...")
   headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/octet-stream"}
   url = f'{endpoint}/contentunderstanding/analyzers/{analyzer}:analyze?api-version={CU_VERSION}'
   response = requests.post(url, headers=headers, data=image_data)

   # Get the response and extract the ID assigned to the analysis operation
   print(response.status_code)
   response_json = response.json()
   id_value = response_json.get("id")

   # Use a GET request to check the status of the analysis operation
   print ('Getting results...')
   time.sleep(1)
   result_url = f'{endpoint}/contentunderstanding/analyzerResults/{id_value}?api-version={CU_VERSION}'
   result_response = requests.get(result_url, headers=headers)
   print(result_response.status_code)

   # Keep polling until the analysis is complete
   status = result_response.json().get("status")
   while status == "Running":
        time.sleep(1)
        result_response = requests.get(result_url, headers=headers)
        status = result_response.json().get("status")

   # Process the analysis results
   if status == "Succeeded":
        print("Analysis succeeded:\n")
        result_json = result_response.json()
        output_file = "results.json"
        with open(output_file, "w") as json_file:
            json.dump(result_json, json_file, indent=4)
            print(f"Response saved in {output_file}\n")

        # Iterate through the fields and extract the names and type-specific values
        contents = result_json["result"]["contents"]
        for content in contents:
            if "fields" in content:
                fields = content["fields"]
                for field_name, field_data in fields.items():
                    if field_data['type'] == "string":
                        print(f"{field_name}: {field_data['valueString']}")
                    elif field_data['type'] == "number":
                        print(f"{field_name}: {field_data['valueNumber']}")
                    elif field_data['type'] == "integer":
                        print(f"{field_name}: {field_data['valueInteger']}")
                    elif field_data['type'] == "date":
                        print(f"{field_name}: {field_data['valueDate']}")
                    elif field_data['type'] == "time":
                        print(f"{field_name}: {field_data['valueTime']}")
                    elif field_data['type'] == "array":
                        print(f"{field_name}: {field_data['valueArray']}")
    ```

1. Examine o código que você adicionou, que:
    - Lê o conteúdo do arquivo de imagem
    - Define a versão da API REST de Compreensão de Conteúdo ser usada
    - Envia uma solicitação HTTP *POST* para o ponto de extremidade de Compreensão de Conteúdo, instruindo-o a analisar a imagem.
    - Verifica a resposta da operação para recuperar uma ID para a operação de análise.
    - Envia repetidamente uma solicitação HTTP *GET* ao ponto de extremidade de Compreensão de conteúdo para verificar o status da operação até que ela não esteja mais em execução.
    - Se a operação tiver sido bem-sucedida, salvará a resposta JSON e, em seguida, analisará o JSON e exibirá os valores recuperados para cada campo específico do tipo.

    > **Observação**: em nosso esquema de cartão de visita simples, todos os campos são cadeias de caracteres. O código aqui ilustra a necessidade de verificar o tipo de cada campo para que você possa extrair valores de diferentes tipos de um esquema mais complexo.

1. Use o comando **CTRL+S** para salvar as alterações de código, mas mantenha o painel do editor de código aberto caso precise corrigir erros no código. Redimensione os painéis para ver claramente o painel da linha de comando.
1. No painel de linha de comando do Cloud Shell, insira o seguinte comando para executar o código Python:

    ```
   python read-card.py biz-card-1.png
    ```

1. Examine a saída do programa, que deve mostrar os valores dos campos no seguinte cartão de visita:

    ![Um cartão de visita para Roberto Tamburello, um funcionário da Adventure Works Cycles.](./media/biz-card-1.png)

1. Use o seguinte comando para executar o programa com um cartão de visita diferente:

    ```
   python read-card.py biz-card-2.png
    ```

1. Examine os resultados, que devem refletir os valores neste cartão de visita:

    ![Um cartão de visita para Mary Duartes, uma funcionário da Contoso.](./media/biz-card-2.png)

1. No painel de linha de comando do Cloud Shell, use o seguinte comando para exibir a resposta JSON completa retornada:

    ```
   cat results.json
    ```

    Role para exibir o JSON.

## Limpar

Se tiver terminado de usar o serviço Compreensão de conteúdo, deverá excluir os recursos que criou neste exercício para evitar incorrer em custos desnecessários do Azure.

1. No portal do Azure, exclua os recursos criados neste exercício.
