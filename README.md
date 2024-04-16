# breweries_case
O objetivo desse repositório é consumir dados de uma API, transformá-los e mantê-los em um data lake seguindo a arquitetura medallion com três camadas: dados brutos, dados curados particionados por local e uma camada agregada analítica.

Voce vai precisar ter uma conta na Azure e provisionar os recursos abaixo:

-Azure Storage (StorageV2 (Hierarchical namespace enabled)) -> https://learn.microsoft.com/pt-br/azure/storage/common/storage-account-create?tabs=azure-portal

-Azure Data Facory -> https://learn.microsoft.com/pt-br/azure/data-factory/quickstart-get-started

-Azure Key Vault -> https://learn.microsoft.com/pt-br/azure/key-vault/general/quick-create-portal

-Power Shell -> https://learn.microsoft.com/pt-br/powershell/scripting/learn/ps101/01-getting-started?view=powershell-7.4

-Azure Databricks,

1 - Acesse o Portal Azure (https://portal.azure.com).
2 - No menu lateral esquerdo, clique em "Criar um recurso".
3 - Na barra de pesquisa, digite "Databricks" e pressione Enter.
4 - Selecione "Databricks" nos resultados da pesquisa.
5 - Clique em "Criar" para iniciar a configuração do Databricks.
6 - Na página de criação, preencha as informações necessárias, como assinatura, grupo de recursos, nome do workspace, região, plano de preços, etc.
7 - Selecione ou crie um novo grupo de recursos para o workspace do Databricks.
8 - Escolha o plano de preços desejado (Premium ou Standard).
9 - Configure as opções avançadas, se necessário, como redes virtuais, segurança, etc.
10 - Revise e confirme as configurações selecionadas.
11 - Clique em "Criar" para provisionar o workspace do Databricks.
12 - Aguarde a conclusão do processo de criação.
13 - Uma vez criado, acesse o workspace do Databricks no Portal Azure para começar a usar.
14 - Siga esses passos para criar um workspace do Databricks usando o Portal Azure.

Voce tambem vai precisar configurar o acesso entre os serviços

Databricks x KV -> https://learn.microsoft.com/en-us/azure/databricks/security/secrets/secret-scopes 

Databricks x - Data Factory -> https://learn.microsoft.com/en-us/azure/data-factory/transform-data-using-databricks-notebook

Após ter provisionado os serviços e configurado o acesso entre os mesmo voce pode criar os pipeline usando os arquivos .json dentro da pasta Pipelines.

Para o Databricks é só importar o Notebook da pasta Databricks dentro do seu workspace.

É um projeto simples mas eficiente

Iniciamos setando uma data de processamento, depois usamos a atividade copy data para coletar o dados da api e salvar no storage sem nenhuma transformação, criando nossa camada Bronze, que servira como tabela da verdade, caso necessario para comparações futuras. Depois fazemos uma verificação no folder onde o arquivo deveria ser salvo. Caso esteja vazio não iremos iniciar um cluster do Databricks para não processar nada. Se tivermos uma falha no processamento geramos um log de erro atraves de uma SP.

A atividade de copia e do databricks estão setadas para uma tentativa de retry em caso de erro e timeout de 10 min. Não queremos que nosso cluster fique ligado por horas caso alguma coisa de errado.

![image](https://github.com/ricauduro/breweries_case/assets/58055908/018d3d3d-e435-46c4-86c0-d295e7b30b96)


Dentro do Databricks iniciamos pegando a data de processamento que geramos no ADF e criando um mounting point apontando para o nosso datalake

![image](https://github.com/ricauduro/breweries_case/assets/58055908/cd47a011-0fc7-4a6c-8e15-c551fcccf34c)


Depois da leitura inicial dos dados, iserimos uma coluna com a data de processamento, para fazer o controle de dados duplicados. E criamos a segunda camada, Silver, usando o formato Delta

![image](https://github.com/ricauduro/breweries_case/assets/58055908/5ba5f1cb-cf02-45d7-a0a3-52e0a61a6039)

Por fim criamos uma view agregada, onde podemos ver as informações consolidadas, formando nossa camada Gold

![image](https://github.com/ricauduro/breweries_case/assets/58055908/4c46fd19-f1cf-49fc-a3b1-6396edacfd75)
