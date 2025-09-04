---
lab:
  title: Analisar formulários com modelos personalizados da IA do Azure para Informação de Documentos
  description: Crie um modelo personalizado de Inteligência de Documento para extrair dados específicos de documentos.
---

# Analisar formulários com modelos personalizados da IA do Azure para Informação de Documentos

Suponha que uma empresa atualmente exija que os funcionários comprem manualmente folhas de pedidos e insiram os dados em um banco de dados. Eles querem que você utilize os serviços de IA para melhorar o processo de entrada de dados. Você decide criar um modelo de machine learning que lerá o formulário e produzirá dados estruturados que podem ser usados para atualizar automaticamente um banco de dados.

**IA do Azure para Informação de Documentos** é um serviço de IA do Azure que permite aos usuários criar um software de processamento de documento automatizado. Este software pode extrair texto, pares de chave/valor e tabelas de documentos de formulário usando OCR (reconhecimento óptico de caracteres). A IA do Azure para Informação de Documentos tem modelos pré-criados para reconhecer faturas, recibos e cartões de visita. O serviço também fornece a capacidade de treinar modelos personalizados. Neste exercício, nos concentraremos na criação de modelos personalizados.

Embora este exercício seja baseado em Python, você pode desenvolver aplicativos semelhantes usando vários SDKs específicos da linguagem, incluindo:

- [Biblioteca de clientes de IA do Azure para Informação de Documentos para Python](https://pypi.org/project/azure-ai-formrecognizer/)
- [Biblioteca de clientes de IA do Azure para Informação de Documentos para Microsoft .NET](https://www.nuget.org/packages/Azure.AI.FormRecognizer)
- [Biblioteca de clientes de IA do Azure para Informação de Documentos para JavaScript](https://www.npmjs.com/package/@azure/ai-form-recognizer)

Este exercício levará aproximadamente **30** minutos.

## Criar um recurso da IA do Azure para Informação de Documentos

Para usar o serviço IA do Azure para Informação de Documentos, você precisa de um recurso da IA do Azure para Informação de Documentos ou de Serviços de IA do Azure em sua assinatura do Azure. Você usará o portal do Azure para criar um recurso.

1. Em uma guia do navegador, abra o portal do Azure em `https://portal.azure.com` e entre usando a conta Microsoft associada à sua assinatura do Azure.
1. Na página inicial do portal do Azure, navegue até a caixa de pesquisa superior e digite **Informação de Documentos** e pressione **Enter**.
1. Na página **Informação de Documentos**, selecione **Criar**.
1. Na página **Criar Inteligência de Documento**, crie um recurso com as seguintes configurações:
    - **Assinatura**: Sua assinatura do Azure.
    - **Grupo de recursos**: crie ou selecione um grupo de recursos
    - **Região**: Qualquer região disponível
    - **Nome**: um nome válido para o recurso de Inteligência de Documento
    - **Tipo de preço**: F0 Gratuita (*se você não tiver uma camada gratuita disponível, selecione* S0 Padrão).
1. Depois que a implantação for concluída, selecione **Ir para o recurso** para exibir a página **Visão geral** do recurso.

## Preparar-se para desenvolver um aplicativo no Cloud Shell

Você desenvolverá seu aplicativo de tradução de texto usando o Cloud Shell. Os arquivos de código do seu aplicativo foram fornecidos em um repositório do GitHub.

1. No Portal do Azure, use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure selecionando um ambiente do ***PowerShell***. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure.

    > **Observação**: se você já criou um Cloud Shell que usa um ambiente *Bash*, alterne-o para o ***PowerShell***.

1. Dimensione o painel do Cloud Shell para ver o console de linha de comando e o portal do Azure. Você precisará usar a barra de divisão para alternar conforme alterna entre os dois painéis.

1. Na barra de ferramentas do Cloud Shell, no menu **Configurações**, selecione **Ir para a versão clássica** (isso é necessário para usar o editor de código).

    **<font color="red">Verifique se você mudou para a versão clássica do Cloud Shell antes de continuar.</font>**

1. No painel do PowerShell, insira os seguintes comandos para clonar o repositório GitHub para este exercício:

    ```
   rm -r mslearn-ai-info -f
   git clone https://github.com/microsoftlearning/mslearn-ai-information-extraction mslearn-ai-info
    ```

    > **Dica**: conforme você colar comandos no cloudshell, a saída pode ocupar uma grande quantidade do espaço da tela. Você pode limpar a tela digitando o comando `cls` para facilitar o foco em cada tarefa.

1. Após o repositório ser clonado, navegue até a pasta que contém os arquivos de código do aplicativo:  

    ```
   cd mslearn-ai-info/Labfiles/custom-doc-intelligence
    ```

## Reunir documentos para treinamento

Você usará os formulários de exemplo como este para treinar um modelo de teste: 

![Uma imagem de uma fatura usada neste projeto.](./media/Form_1.jpg)

1. Na linha de comando, execute `ls ./sample-forms` para listar o conteúdo na pasta **sample-forms**. Observe que há arquivos terminados em **.json** e **.jpg** na pasta.

    Você usará os arquivos **.jpg** para treinar seu modelo.  

    Os arquivos **.json** foram gerados para você e contêm informações de rótulo. Os arquivos serão carregados em seu contêiner de armazenamento de blobs ao lado dos formulários.

1. No **portal do Azure** e navegue até a página **Visão geral** do recurso, se você ainda não estiver lá. Na seção *Essentials*, observe o **Grupo de recursos**, a **ID da assinatura** e o **Local**. Você precisará desses valores em etapas subsequentes.
1. Execute o comando para `code setup.sh` para abrir **setup.sh** em um editor de código. Você usará esse script para executar os comandos da CLI (interface de linha de comando) do Azure necessários para criar os outros recursos do Azure necessários.

1. No script **setup.sh**, examine os comandos. O programa irá:
    - Criar uma conta de armazenamento no seu grupo de recursos do Azure
    - Carregar arquivos da pasta *sampleforms* local para um contêiner chamado *sampleforms* na conta de armazenamento
    - Imprimir um URI de assinatura de acesso compartilhado

1. Modifique as declarações de variáveis **subscription_id**, **resource_group** e **location** com os valores apropriados para a assinatura, o grupo de recursos e o nome do local onde você implantou o recurso Informação de Documentos.

    > **Importante**: para sua cadeia de caracteres de **localização**, use a versão de código da sua localização. Por exemplo, se sua localização for "Leste dos EUA", a cadeia de caracteres no script deverá ser `eastus`. Você pode ver que a versão é o botão **Exibição JSON** no lado direito da guia **Essentials** do grupo de recursos no portal do Azure.

    Se a variável **expiry_date** estiver no passado, atualize-a para uma data futura. Essa variável é usada ao gerar o URI de SAS (Assinatura de Acesso Compartilhado). Na prática, você vai querer definir uma data de validade apropriada para a sua SAS. Você pode aprender mais sobre SAS [aqui](https://docs.microsoft.com/azure/storage/common/storage-sas-overview#how-a-shared-access-signature-works).  

1. Depois de substituir os espaços reservados, use o comando **CTRL+S** ou **botão direito do mouse > Salvar** para salvar as suas alterações e, em seguida, use o comando **CTRL+Q** ou **botão direito do mouse > Sair** para fechar o editor de código, mantendo a linha de comando do Cloud Shell aberta.

1. Insira os seguintes comandos para tornar o script executável e executá-lo:

    ```PowerShell
   chmod +x ./setup.sh
   ./setup.sh
    ```

1. Quando o script for concluído, revise a saída exibida.

1. No portal do Azure, atualize seu grupo de recursos e verifique se ele contém a conta de Armazenamento do Azure recém-criada. Abra a conta de armazenamento e, no painel à esquerda, selecione **Navegador de armazenamento**. Em seguida, no Navegador de Armazenamento, expanda **Contêineres de blob** e selecione o contêiner **sampleforms** para verificar se os arquivos foram carregados da pasta local **custom-doc-intelligence/sample-forms**.

## Utilize o Estúdio de Informação de Documentos para treinar o modelo

Agora você treinará o modelo usando os arquivos carregados na conta de armazenamento.

1. Abra uma nova guia do navegador e navegue até o Document Intelligence Studio em `https://documentintelligence.ai.azure.com/studio`. 
1. Role para baixo até a seção **Modelos personalizados** e selecione o bloco de **Modelo de extração personalizado**.
1. Se solicitado, entre com suas credenciais do Azure.
1. Se o prompt perguntar qual recurso da IA do Azure para Informação de Documentos você quer usar, selecione a assinatura e o nome do recurso que você usou ao criar o recurso da IA do Azure para Informação de Documentos.
1. Em **Meus Projetos**, crie um novo projeto com a seguinte configuração:

    - **Insira os detalhes do projeto**:
        - **Nome do projeto**: um nome válido para seu projeto
    - **Configurar recurso de serviço**:
        - **Assinatura:** sua assinatura do Azure
        - **Grupo de recursos**: o grupo de recursos em que você implantou o recurso de Inteligência de Documento
        - **Recurso de inteligência de documento** Seu recurso de Inteligência de Documento (selecione a opção *Definir* como padrão e use a versão da API padrão)
    - **Conectar fonte de dados de treinamento**:
        - **Assinatura:** sua assinatura do Azure
        - **Grupo de recursos**: o grupo de recursos em que você implantou o recurso de Inteligência de Documento
        - **Conta de armazenamento**: a conta de armazenamento criada pelo script de instalação (selecione a opção *Definir como padrão*, selecione o contêiner de blob `sampleforms` e deixe o caminho da pasta em branco)

1. Quando o projeto for criado, no canto superior direito da página, selecione **Treinar** para treinar seu modelo. Use as seguintes configurações:
    - **ID do Modelo**: um nome válido para seu modelo (*você precisará do nome da ID do modelo na próxima etapa*)
    - **Modo de criação**: modelo.
1. Selecione **Ir para modelos**.
1. O treinamento pode levar algum tempo. Aguarde até que o status seja **bem-sucedido**.

## Testar seu modelo personalizado de Informação de documentos

1. Retorne à guia do navegador que contém o Portal do Azure e o Cloud Shell. Na linha de comando, execute o seguinte comando para alterar para a pasta que contém os arquivos de código do aplicativo:

    ```
    cd Python
    ```

1. Instale o pacote de Inteligência de Documento executando o seguinte comando:

    ```
    python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-formrecognizer==3.3.3
    ```

1. Digite o seguinte comando para editar o arquivo de configuração que foi fornecido:

    ```
   code .env
    ```

1. No painel que contém o portal do Azure, na página **Visão geral** do recurso de Inteligência de Documento, selecione **Clique aqui para gerenciar chaves** para ver o ponto de extremidade e as chaves do recurso. Edite o arquivo de configuração com os seguintes valores:
    - Seu ponto de extremidade de Inteligência de Documentos
    - Sua chave da Inteligência de Documentos
    - A ID do Modelo que você especificou ao treinar seu modelo

1. Depois de substituir os espaços reservados, no editor de código, use o comando **CTRL+S** para salvar as alterações e então use **CTRL+Q** para fechar o editor de código, mantendo a linha de comando do Cloud Shell aberta.

1. Abra o arquivo de código para seu aplicativo cliente (`code Program.cs` para C#, `code test-model.py` para Python) e examine o código que ele contém, especialmente que a imagem na URL se refere ao arquivo neste repositório GitHub na Web. Feche o arquivo sem fazer alterações.

1. Na linha de comando e insira o seguinte comando para executar o programa:

    ```
   python test-model.py
    ```

1. Exiba a saída e observe como a saída para o modelo fornece nomes de campo como `Merchant` e `CompanyPhoneNumber`.

## Limpeza

Quando terminar de usar o recurso do Azure, lembre-se de excluir o recurso no [portal do Azure](https://portal.azure.com/?azure-portal=true) para evitar mais cobranças.

## Mais informações

Para obter mais informações sobre o serviço Informação de Documentos, consulte a [documentação de Informação de Documentos](https://learn.microsoft.com/azure/ai-services/document-intelligence/?azure-portal=true).
