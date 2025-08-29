---
lab:
  title: Extrair informações do conteúdo multimodal
  description: 'Use a Compreensão de Conteúdo da IA do Azure para extrair insights de documentos, imagens, gravações de áudio e vídeos.'
---

# Extrair informações do conteúdo multimodal

Neste exercício, você usará o Reconhecimento de Conteúdo do Azure para extrair informações de uma variedade de tipos de conteúdo; incluindo uma fatura, imagens de um slide contendo gráficos, uma gravação de áudio de mensagens de voz e uma gravação de vídeo de uma chamada em conferência.

Este exercício leva aproximadamente **40** minutos.

## Crie um hub e projeto da Fábrica de IA do Azure

Os recursos da Fábrica de IA do Azure que usaremos neste exercício requerem um projeto baseado em um recurso de *hub* da Fábrica de IA do Azure.

1. Em um navegador da Web, abra o [Portal da Fábrica de IA do Azure](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure. Feche todas as dicas ou painéis de início rápido abertos na primeira vez que você entrar e, se necessário, use o logotipo da **Fábrica de IA do Azure** no canto superior esquerdo para navegar até a home page, que é semelhante à imagem a seguir (feche o painel **Ajuda** se estiver aberto):

    ![Captura de tela do portal do Azure AI Foundry.](./media/ai-foundry-home.png)

1. No navegador, navegue até `https://ai.azure.com/managementCenter/allResources` e clique em **Criar**. Em seguida, escolha a opção para criar um novo **Recurso do hub de IA**.
1. No assistente **Criar projeto**, insira um nome válido para o projeto e selecione a opção para criar um novo hub. Em seguida, use o link **Renomear hub** para especificar um nome válido para o novo hub, expanda **Opções avançadas** e especifique as seguintes configurações para o projeto:
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *criar ou selecionar um grupo de recursos*
    - **Região**:  Selecione um dos seguintes locais (*no momento da redação, a compreensão de Conteúdo da IA do Azure estava disponível apenas nestas regiões*):
        - Leste da Austrália
        - Suécia Central
        - Oeste dos EUA

    > **Observação**: se você estiver trabalhando em uma assinatura do Azure na qual as políticas são usadas para restringir nomes de recursos permitidos, talvez seja necessário usar o link na parte inferior da caixa de diálogo **Criar um novo projeto** para criar o hub usando o portal do Azure.

    > **Dica**: se o botão **Criar** ainda estiver desativado, certifique-se de renomear o hub com um valor alfanumérico exclusivo.

1. Aguarde até que seu projeto seja criado.

## Baixar conteúdo de 

O conteúdo que você vai analisar está em um arquivo .zip. Baixe-o e extraia-o em uma pasta local.

1. Em uma nova guia do navegador, baixe [content.zip](https://github.com/microsoftlearning/mslearn-ai-information-extraction/raw/main/Labfiles/content/content.zip) e `https://github.com/microsoftlearning/mslearn-ai-information-extraction/raw/main/Labfiles/content/content.zip` salve-o em uma pasta local.
1. Extraia o arquivo *content.zip* baixado e exiba os arquivos que ele contém. Você usará esses arquivos para criar vários analisadores de Compreensão de Conteúdo neste exercício.

> **Observação**: Se você estiver interessado apenas em explorar a análise de uma modalidade específica (documentos, imagens, vídeo ou áudio), poderá pular para a tarefa relevante abaixo. Para a melhor experiência, percorra cada tarefa para saber como extrair informações de diferentes tipos de conteúdo.

## Extrair informações de documentos da fatura

Você criará um analisador de Compreensão de Conteúdo da IA do Azure que pode extrair informações de faturas. Você começará definindo um esquema com base em uma fatura de exemplo.

### Definir um esquema para análise de fatura

1. Na guia do navegador que contém a home page para seu projeto da Fábrica de IA do Azure; no painel de navegação à esquerda, selecione **Compreensão de Conteúdo**.
1. Na página **Compreensão de Conteúdo**, selecione a guia **Tarefa personalizada** na parte superior.
1. Na página de tarefa personalizada de Compreensão de Conteúdo, selecione **+ Criar** e crie uma tarefa com as seguintes configurações:
    - **Nome da tarefa:**: `Invoice analysis`
    - **Descrição**: `Extract data from an invoice`
    - **Análise de conteúdo de arquivo único**: *Selecionado*
    - **Configurações avançadas**:
        - **Conexão de serviços de IA do Azure**: *o recurso Serviços de IA do Azure no hub da Fábrica de IA do Azure*
        - **Conta de Armazenamento de Blobs do Azure**: *a conta de armazenamento padrão no hub da Fábrica de IA do Azure*
1. Aguarde pela tarefa a ser criada.

    > **Dica**: se ocorrer um erro ao acessar o armazenamento, aguarde um minuto e tente novamente. A propagação das permissões para um novo hub pode levar alguns minutos.

1. Na página **Definir esquema**, carregue o arquivo **invoice-1234.pdf** da pasta em que você extraiu arquivos de conteúdo. Esse arquivo contém a seguinte fatura:

    ![Imagem de um número de fatura 1234.](./media/invoice-1234.png)

1. Na página **Definir esquema**, depois de carregar o arquivo de fatura, selecione o modelo de **Extração de dados da fatura** e selecione **Criar**.

    O modelo de *Análise de fatura* inclui campos comuns encontrados em faturas. É possível usar o editor de esquema para excluir qualquer um dos campos sugeridos que você não precisa e adicionar os campos personalizados necessários.

1. Na lista de campos sugeridos, selecione **BillingAddress**. Este campo não é necessário para o formato de fatura que você carregou, portanto, use o ícone **Excluir campo** (**&#128465;**) que aparece na linha de campo selecionada para excluí-lo.
1. Agora, exclua os seguintes campos sugeridos, que não são necessários para seu esquema de fatura:
    - BillingAddressRecipient
    - CustomerAddressRecipient
    - CustomerId
    - CustomerTaxId
    - DueDate
    - InvoiceTotal
    - PaymentTerm
    - PreviousUnpaidBalance
    - PurchaseOrder
    - RemittanceAddress
    - RemittanceAddressRecipient
    - ServiceAddress
    - ServiceAddressRecipient
    - ShippingAddress
    - ShippingAddressRecipient
    - TotalDiscount
    - VendorAddressRecipient
    - VendorTaxId
    - TaxDetails
1. Use o botão **+ Adicionar novo campo** para adicionar os seguintes campos, selecionando **Salvar alterações** (**&#10003;**) para cada novo campo:

    | Nome do campo | Descrição do campo | Tipo de valor | Método |
    |--|--|--|--|
    | `VendorPhone` | `Vendor telephone number` | String | Extração |
    | `ShippingFee` | `Fee for shipping` | Número | Extração |

1. Na linha do campo **Itens**, observe que esse campo é uma *tabela* (ele contém a coleção de itens na fatura). Selecione o ícone **Editar** (&#9638;) para abrir uma nova página com seus subcampo.
1. Remova os seguintes subcampo da tabela **Itens**:
    - Data
    - ProductCode
    - Unidade
    - TaxAmount
    - TaxRate
1. Use o botão **OK** para confirmar as alterações e retornar ao nível superior do esquema da fatura.

1. Verifique se o esquema concluído tem esta aparência e selecione **Salvar**.

    ![Captura de tela de um esquema para uma fatura.](./media/invoice-schema.png)

1. Na página **Analisador de Teste**, se a análise não começar automaticamente, selecione **Executar análise**. Aguarde a conclusão da análise.

1. Revise os resultados da análise, que devem ser semelhantes a estes:

    ![Captura de tela dos resultados do teste de análise de fatura.](./media/invoice-analysis.png)

1. Exiba os detalhes dos campos identificados no painel **Campos**.

### Criar e testar um analisador para faturas

Agora que você treinou um modelo para extrair campos de faturas, pode criar um analisador para usar com documentos semelhantes.

1. Selecione a página **Lista do analisador** e selecione **+ Criar analisador** e crie um analisador com as seguintes propriedades (digitadas exatamente como mostrado aqui):
    - **Nome**: `invoice-analyzer`
    - **Descrição**: `Invoice analyzer`
1. Aguarde até que o novo analisador esteja pronto (use o botão **Atualizar** para verificar).
1. Quando o analisador tiver sido criado, selecione o link **invoice-analyzer**. Os campos definidos no esquema do analisador serão exibidos.
1. Na página **invoice-analyzer**, selecione a guia **Teste**.
1. Use o botão **+ Carregar arquivos de teste** para carregar **invoice-1235.pdf** da pasta em que você extraiu os arquivos de conteúdo e clique em **Executar análise** para extrair dados de campo da fatura.

    A fatura que está sendo analisada tem esta aparência:

    ![Imagem de um número de fatura 1235.](./media/invoice-1235.png)

1. Examine o painel **Campos** e verifique se o analisador extraiu os campos corretos da fatura de teste.
1. Examine o painel **Resultados** para ver a resposta JSON que o analisador retornaria a um aplicativo cliente.
1. Na guia **Exemplo de código**, exiba o código de exemplo que você pode usar para desenvolver um aplicativo cliente que usa a interface REST de Compreensão de conteúdo para chamar o analisador.
1. Feche a página **invoice-analyzer**.

## Extrair informações de uma imagem de slide

Você criará um analisador de Compreensão de Conteúdo da IA do Azure que pode extrair informações de um slide que contém gráficos.

### Definir um esquema para análise de imagem

1. Na guia do navegador que contém a home page para seu projeto da Fábrica de IA do Azure; no painel de navegação à esquerda, selecione **Compreensão de Conteúdo**.
1. Na página **Compreensão de Conteúdo**, selecione a guia **Tarefa personalizada** na parte superior.
1. Na página de tarefa personalizada de Compreensão de Conteúdo, selecione **+ Criar** e crie uma tarefa com as seguintes configurações:
    - **Nome da tarefa:**: `Slide analysis`
    - **Descrição**: `Extract data from an image of a slide`
    - **Análise de conteúdo de arquivo único**: *selecionado*
    - **Configurações avançadas**:
        - **Conexão de serviços de IA do Azure**: *o recurso Serviços de IA do Azure no hub da Fábrica de IA do Azure*
        - **Conta de Armazenamento de Blobs do Azure**: *a conta de armazenamento padrão no hub da Fábrica de IA do Azure*
1. Aguarde pela tarefa a ser criada.

    > **Dica**: se ocorrer um erro ao acessar o armazenamento, aguarde um minuto e tente novamente. A propagação das permissões para um novo hub pode levar alguns minutos.

1. Na página **Definir esquema**, carregue o arquivo **slide-1.jpg** da pasta na qual você extraiu arquivos de conteúdo. Selecione o modelo de **Análise de imagem** e selecione **Criar**.

    O modelo de *Análise de imagem* não inclui campos predefinidos. Defina campos para descrever as informações que você quer extrair.

1. Use o botão **+ Adicionar novo campo** para adicionar os seguintes campos, selecionando **Salvar alterações** (**&#10003;**) para cada novo campo:

    | Nome do campo | Descrição do campo | Tipo de valor | Método |
    |--|--|--|--|
    | `Title` | `Slide title` | String | Generate |
    | `Summary` | `Summary of the slide` | String | Generate |
    | `Charts` | `Number of charts on the slide` | Inteiro | Generate |

1. Use o botão **+ Adicionar novo botão** para adicionar um novo campo chamado `QuarterlyRevenue` com a descrição `Revenue per quarter` com o tipo de valor **Tabela** e salve o novo campo (**&#10003;**). Então, na nova página para os subcampo de tabela que é aberta, adicione os seguintes subcampos:

    | Nome do campo | Descrição do campo | Tipo de valor | Método |
    |--|--|--|--|
    | `Quarter` | `Which quarter?` | String | Generate |
    | `Revenue` | `Revenue for the quarter` | Número | Generate |

1. Selecione **Voltar** (o ícone de seta perto do botão **Adicionar novo subcampo**) ou **&#10003; OK** para retornar ao nível superior do esquema e use o botão **= Adicionar campo** para adicionar um campo chamado `ProductCategories` com a descrição `Product categories` com o tipo de valor **Tabela **, e salve o novo campo (**&#10003;**). Então, na nova página para os subcampo de tabela que é aberta, adicione os seguintes subcampos:

    | Nome do campo | Descrição do campo | Tipo de valor | Método |
    |--|--|--|--|
    | `ProductCategory` | `Product category name` | String | Generate |
    | `RevenuePercentage` | `Percentage of revenue` | Número | Generate |

1. Selecione **Voltar** (o ícone de seta perto do botão **Adicionar subcampo**) ou **&#10003; OK** para retornar ao nível superior do esquema e verificar se ele tem esta aparência. Em seguida, selecione **Salvar**.

    ![Captura de tela de um esquema para uma imagem de slide.](./media/slide-schema.png)

1. Na página **Analisador de Teste**, se a análise não começar automaticamente, selecione **Executar análise**. Aguarde a conclusão da análise.

    O slide que está sendo analisado tem esta aparência:

    ![Imagem de um slide que contém dois gráficos para dados de receita do Adventure Works Cycles.](./media/slide-1.jpg)

1. Revise os resultados da análise, que devem ser semelhantes a estes:

    ![Captura de tela dos resultados do teste de análise de imagem.](./media/slide-analysis.png)

1. Exiba os detalhes dos campos identificados no painel **Campos**, expandindo os campos **QuarterlyRevenue** e **ProductCategories** para ver os valores de subcampo.

### Criar e testar um analisador

Agora que você treinou um modelo para extrair campos de slides, pode criar um analisador para usar com imagens de slide semelhantes.

1. Selecione a página **Lista do analisador** e selecione **+ Criar analisador** e crie um analisador com as seguintes propriedades (digitadas exatamente como mostrado aqui):
    - **Nome**: `slide-analyzer`
    - **Descrição**: `Slide image analyzer`
1. Aguarde até que o novo analisador esteja pronto (use o botão **Atualizar** para verificar).
1. Quando o analisador tiver sido criado, selecione o link **slide-analyzer**. Os campos definidos no esquema do analisador serão exibidos.
1. Na página **slide-analyzer**, selecione a guia **Teste**.
1. Use o botão **+ Carregar arquivos de teste** para carregar **slide-2.jpg** da pasta na qual você extraiu os arquivos de conteúdo e clique em **Executar análise** para extrair dados de campo da imagem.

    O slide que está sendo analisado tem esta aparência:

    ![Imagem de um slide que contém um gráfico para dados de receita da Contoso Ltd.](./media/slide-2.jpg)

1. Examine o painel **Campos** e verifique se o analisador extraiu os campos corretos da imagem do slide.

    > **Observação**: O slide 2 não inclui um detalhamento pela categoria de produto, portanto, os dados de receita da categoria de produto não são encontrados.

1. Examine o painel **Resultados** para ver a resposta JSON que o analisador retornaria a um aplicativo cliente.
1. Na guia **Exemplo de código**, exiba o código de exemplo que você pode usar para desenvolver um aplicativo cliente que usa a interface REST de Compreensão de Conteúdo para chamar o analisador.
1. Feche a página **slide-analyzer**.

## Extrair informações de uma gravação de áudio da caixa postal

Você criará um analisador de Compreensão de Conteúdo da IA do Azure que pode extrair informações de uma gravação de áudio de uma mensagem de caixa postal.

### Definir um esquema para análise de áudio

1. Na guia do navegador que contém a home page para seu projeto da Fábrica de IA do Azure; no painel de navegação à esquerda, selecione **Compreensão de Conteúdo**.
1. Na página **Compreensão de Conteúdo**, selecione a guia **Tarefa personalizada** na parte superior.
1. Na página de tarefa personalizada de Compreensão de Conteúdo, selecione **+ Criar** e crie uma tarefa com as seguintes configurações:
    - **Nome da tarefa:**: `Voicemail analysis`
    - **Descrição**: `Extract data from a voicemail recording`
    - **Análise de conteúdo de arquivo único**: *selecionado*
    - **Configurações avançadas**:
        - **Conexão de serviços de IA do Azure**: *o recurso Serviços de IA do Azure no hub da Fábrica de IA do Azure*
        - **Conta de Armazenamento de Blobs do Azure**: *a conta de armazenamento padrão no hub da Fábrica de IA do Azure*
1. Aguarde pela tarefa a ser criada.

    > **Dica**: se ocorrer um erro ao acessar o armazenamento, aguarde um minuto e tente novamente. A propagação das permissões para um novo hub pode levar alguns minutos.

1. Na página **Definir esquema**, carregue o arquivo **call-1.mp3** da pasta na qual você extraiu arquivos de conteúdo. Selecione o modelo de **Análise de transcrição de fala** e selecione **Criar**.
1. No painel **Conteúdo** à direita, selecione **Obter visualização de transcrição** para ver uma transcrição da mensagem gravada.

    O modelo de *Análise de transcrição de fala* não inclui campos predefinidos. Defina campos para descrever as informações que você quer extrair.

1. Use o botão **+ Adicionar novo campo** para adicionar os seguintes campos, selecionando **Salvar alterações** (**&#10003;**) para cada novo campo:

    | Nome do campo | Descrição do campo | Tipo de valor | Método |
    |--|--|--|--|
    | `Caller` | `Person who left the message` | String | Generate |
    | `Summary` | `Summary of the message` | String | Generate |
    | `Actions` | `Requested actions` | String | Generate |
    | `CallbackNumber` | `Telephone number to return the call` | String | Generate |
    | `AlternativeContacts` | `Alternative contact details` | Lista de cadeias de caracteres | Generate |

1. Verifique se o esquema tem esta aparência. Em seguida, selecione **Salvar**.

    ![Captura de tela de um esquema para uma gravação de caixa postal.](./media/voicemail-schema.png)

1. Na página **Analisador de Teste**, se a análise não começar automaticamente, selecione **Executar análise**. Aguarde a conclusão da análise.

    A análise de áudio pode levar algum tempo. Enquanto aguarda, você pode reproduzir o arquivo de áudio abaixo:

    <video controls src="https://github.com/MicrosoftLearning/mslearn-ai-information-extraction/raw/refs/heads/main/Instructions/Labs/media/call-1.mp4" title="Chamada 1" width="300">
        <track src="https://github.com/MicrosoftLearning/mslearn-ai-information-extraction/raw/refs/heads/main/Instructions/Labs/media/call-1.vtt" kind="captions" srclang="en" label="English">
    </video>

    **Observação**: Esse áudio foi gerado usando IA.

1. Revise os resultados da análise, que devem ser semelhantes a estes:

    ![Captura de tela dos resultados do teste de análise de imagem.](./media/voicemail-analysis.png)

1. Exiba os detalhes dos campos que foram identificados no painel **Campos**, expandindo o campo **AlternativeContacts** para ver os valores listados.

### Criar e testar um analisador

Agora que você treinou um modelo para extrair campos de mensagens de voz, pode criar um analisador para usar com gravações de áudio semelhantes.

1. Selecione a página **Lista do analisador** e selecione **+ Criar analisador** e crie um analisador com as seguintes propriedades (digitadas exatamente como mostrado aqui):
    - **Nome**: `voicemail-analyzer`
    - **Descrição**: `Voicemail audio analyzer`
1. Aguarde até que o novo analisador esteja pronto (use o botão **Atualizar** para verificar).
1. Quando o analisador tiver sido criado, selecione o link **voicemail-analyzer**. Os campos definidos no esquema do analisador serão exibidos.
1. Na página **voicemail-analyzer**, selecione a guia **Teste**.
1. Use o botão **+ Carregar arquivos de teste** para carregar **call-2.mp3** da pasta na qual você extraiu os arquivos de conteúdo e clique em **Executar análise** para extrair dados de campo do arquivo de áudio.

    A análise de áudio pode levar algum tempo. Enquanto aguarda, você pode reproduzir o arquivo de áudio abaixo:

    <video controls src="https://github.com/MicrosoftLearning/mslearn-ai-information-extraction/raw/refs/heads/main/Instructions/Labs/media/call-2.mp4" title="Chamada 2" width="300">
        <track src="https://github.com/MicrosoftLearning/mslearn-ai-information-extraction/raw/refs/heads/main/Instructions/Labs/media/call-2.vtt" kind="captions" srclang="en" label="English">
    </video>

    **Observação**: Esse áudio foi gerado usando IA.

1. Examine o painel **Campos** e verifique se o analisador extraiu os campos corretos da mensagem de voz.
1. Examine o painel **Resultados** para ver a resposta JSON que o analisador retornaria a um aplicativo cliente.
1. Na guia **Exemplo de código**, exiba o código de exemplo que você pode usar para desenvolver um aplicativo cliente que usa a interface REST de Compreensão de Conteúdo para chamar o analisador.
1. Feche a página **voicemail-analyzer**.

## Extrair informações de uma gravação de chamada em conferência

Você criará um analisador Compreensão de Conteúdo da IA do Azure que pode extrair informações de uma gravação de vídeo de uma chamada em conferência.

### Definir um esquema para análise de vídeo

1. Na guia do navegador que contém a home page para seu projeto da Fábrica de IA do Azure; no painel de navegação à esquerda, selecione **Compreensão de Conteúdo**.
1. Na página **Compreensão de Conteúdo**, selecione a guia **Tarefa personalizada** na parte superior.
1. Na página de tarefa personalizada de Compreensão de Conteúdo, selecione **+ Criar** e crie uma tarefa com as seguintes configurações:
    - **Nome da tarefa:**: `Conference call video analysis`
    - **Descrição**: `Extract data from a video conference recording`
    - **Análise de conteúdo de arquivo único**: *selecionado*
    - **Configurações avançadas**:
        - **Conexão de serviços de IA do Azure**: *o recurso Serviços de IA do Azure no hub da Fábrica de IA do Azure*
        - **Conta de Armazenamento de Blobs do Azure**: *a conta de armazenamento padrão no hub da Fábrica de IA do Azure*
1. Aguarde pela tarefa a ser criada.

    > **Dica**: se ocorrer um erro ao acessar o armazenamento, aguarde um minuto e tente novamente. A propagação das permissões para um novo hub pode levar alguns minutos.

1. Na página **Definir esquema**, carregue o arquivo **meeting-1.mp4** da pasta na qual você extraiu arquivos de conteúdo. Selecione o modelo **Análise de vídeo** e selecione **Criar**.
1. No painel **Conteúdo** à direita, selecione **Obter visualização de transcrição** para ver uma transcrição da mensagem gravada.

    O modelo *Análise de vídeo* extrai dados para o vídeo. Ele não inclui campos predefinidos. Defina campos para descrever as informações que você quer extrair.

1. Use o botão **+ Adicionar novo campo** para adicionar os seguintes campos, selecionando **Salvar alterações** (**&#10003;**) para cada novo campo:

    | Nome do campo | Descrição do campo | Tipo de valor | Método |
    |--|--|--|--|
    | `Summary` | `Summary of the discussion` | String | Generate |
    | `Participants` | `Count of meeting participants` | Inteiro | Generate |
    | `ParticipantNames` | `Names of meeting participants` | Lista de cadeias de caracteres | Generate |
    | `SharedSlides` | `Descriptions of any PowerPoint slides presented` | Lista de cadeias de caracteres | Generate |
    | `AssignedActions` | `Tasks assigned to participants` | Tabela |  |

1. Quando você insere o campo **AssignedActions**, na tabela de subcampo que aparece, crie os seguintes subcampo:

    | Nome do campo | Descrição do campo | Tipo de valor | Método |
    |--|--|--|--|
    | `Task` | `Description of the task` | String | Generate |
    | `AssignedTo` | `Who the task is assigned to` | String | Generate |

1. Selecione **Voltar** (o ícone de seta perto do botão **Adicionar subcampo**) ou **&#10003; OK** para retornar ao nível superior do esquema e verificar se ele tem esta aparência. Em seguida, selecione **Salvar**.

1. Verifique se o esquema tem esta aparência. Em seguida, selecione **Salvar**.

    ![Captura de tela de um esquema para uma gravação de vídeo de chamada em conferência.](./media/meeting-schema.png)

1. Na página **Analisador de Teste**, se a análise não começar automaticamente, selecione **Executar análise**. Aguarde a conclusão da análise.

    A análise de vídeo pode levar algum tempo. Enquanto aguarda, você pode exibir o vídeo abaixo:

    <video controls src="https://github.com/MicrosoftLearning/mslearn-ai-information-extraction/raw/refs/heads/main/Instructions/Labs/media/meeting-1.mp4" title="Reunião 1" width="480">
        <track src="https://github.com/MicrosoftLearning/mslearn-ai-information-extraction/raw/refs/heads/main/Instructions/Labs/media/meeting-1.vtt" kind="captions" srclang="en" label="English">
    </video>

    **Observação**: Este vídeo foi gerado usando IA.

1. Quando a análise estiver concluída, examine os resultados, que devem ser semelhantes a este:

    ![Captura de tela dos resultados do teste de análise de imagem.](./media/meeting-analysis.png)

1. No painel **Campos**, exiba os dados extraídos do vídeo, incluindo os campos que você adicionou. Exiba os valores de campo gerados, expandindo os campos de lista e tabela conforme necessário.

### Criar e testar um analisador

Agora que você treinou um modelo para extrair campos de gravações de chamada em conferência, pode criar um analisador para usar com vídeos semelhantes.

1. Selecione a página **Lista do analisador** e selecione **+ Criar analisador** e crie um analisador com as seguintes propriedades (digitadas exatamente como mostrado aqui):
    - **Nome**: `conference-call-analyzer`
    - **Descrição**: `Conference call video analyzer`
1. Aguarde até que o novo analisador esteja pronto (use o botão **Atualizar** para verificar).
1. Quando o analisador tiver sido criado, selecione o link **conference-call-analyzer**. Os campos definidos no esquema do analisador serão exibidos.
1. Na página **conference-call-analyzer**, selecione a guia **Teste**.
1. Use o botão **Carregar arquivos de teste** para carregar **meeting-2.mp4** da pasta em que você extraiu os arquivos de conteúdo e execute a análise para extrair dados de campo do arquivo de áudio.

    A análise de vídeo pode levar algum tempo. Enquanto aguarda, você pode exibir o vídeo abaixo:

    <video controls src="https://github.com/MicrosoftLearning/mslearn-ai-information-extraction/raw/refs/heads/main/Instructions/Labs/media/meeting-2.mp4" title="Reunião 2" width="480">
        <track src="https://github.com/MicrosoftLearning/mslearn-ai-information-extraction/raw/refs/heads/main/Instructions/Labs/media/meeting-2.vtt" kind="captions" srclang="en" label="English">
    </video>

    **Observação**: Este vídeo foi gerado usando IA.

1. Examine o painel **Campos** e exiba os campos que o analisador extraiu para o vídeo de chamada em conferência.
1. Examine o painel **Resultados** para ver a resposta JSON que o analisador retornaria a um aplicativo cliente.
1. Feche a página **conference-call-analyzer**.

## Limpar

Se tiver terminado de usar o serviço Compreensão de conteúdo, deverá excluir os recursos que criou neste exercício para evitar incorrer em custos desnecessários do Azure.

1. No portal da Fábrica de IA do Azure, navegue até o hub na página de visão geral, selecione seu projeto e exclua-o.
1. No portal do Azure, exclua o grupo de recursos criado para este exercício.
