# Pipeline de dados do Telegram
![000_1433zq-654x450](https://github.com/luisfernandogbraga/Pipeline-de-dados-do-Telegram/assets/134460985/166cdb89-e429-4b40-8316-d4475c19f982)

Este é um projeto de pipeline de dados para coletar mensagens de um determinado grupo do Telegram e realizar análises usando ferramentas da AWS.

Objetivo
O objetivo deste projeto é criar um pipeline de dados que permita coletar mensagens de um grupo do Telegram, armazená-las em um local seguro na AWS, realizar transformações e análises utilizando SQL ou Python e, finalmente, apresentar os resultados de forma acessível.

Etapas do Projeto
1. Criação do Grupo no Telegram
Antes de começar, é necessário criar um grupo no Telegram do qual você deseja coletar as mensagens. Certifique-se de que o bot que será utilizado para coletar as mensagens tenha acesso a esse grupo.

2. Configuração na AWS
Ingestão de Dados
- API Gateway: Configurar um ponto de extremidade na API Gateway para receber as mensagens do Telegram.
- Lambda: Criar uma função Lambda para processar as mensagens recebidas pela API Gateway.
- S3: Armazenar as mensagens processadas em um bucket do S3.

ETL (Extract, Transform, Load)
- Event Bridge: Configurar regras no Event Bridge para acionar a função Lambda de processamento sempre que novas mensagens forem recebidas.
- Lambda: Implementar a lógica de transformação das mensagens, se necessário, e carregar os dados transformados no S3.

3. Apresentação dos Dados
- Athena: Utilizar o Athena para executar consultas SQL sobre os dados armazenados no S3 e obter insights a partir das análises realizadas.

![Profissao Analista de dados M42 Material de apoio arch](https://github.com/luisfernandogbraga/Pipeline-de-dados-do-Telegram/assets/134460985/5552f083-ade9-4b4b-ac56-8a2366747cbb)

# Telegram
O Telegram representa a fonte transacional de dados do nosso pipeline de dados. Nesta etapa, vamos criar um grupo, criar um bot e adiciona-lo ao grupo recém criado. O bot então captará todas as mensagens enviadas no grupo. As mensagens pode ser acessadas através da API (application programming interface) de bots dos Telegram (documentação neste [link](https://core.telegram.org/bots/api)).

# Conta
Para criar uma conta no **Telegram**, basta fazer o download do aplicativo na loja de aplicativos do seu *smartphone*. Uma vez criada, acesse sua conta através da versão *web* da plataforma de mensagens neste [link](https://web.telegram.org).

#Bot
Para criar um bot:

1. Abra o *chat* com o `BotFather`;
2. Digite `/newbot`;
3. Digite o nome do *bot*;
4. Digite o nome de usuário do *bot* (precisa terminar com sufixo `_bot`);
5. Salve o `token` de acesso a API HTTP em um <font color='red'>local seguro</font>.

Para conferir o token novamente:

1. Abra o *chat* com o `BotFather`;
2. Digite `/mybots`;
3. Selecione o *bot* pelo seu nome de usuário;
4. Selecione `API Token`.

Por fim, precisamos ativiar o bot.
1. Abra o *chat* com o *bot*;
2. Selecione `start`.

# Grupo
Para criar um novo grupo.
1. Aperte o botão com o ícone de um lápis;
2. Selecione `New Group`;
3. Busque e selecione o *bot* recém criado pelo seu nome;
4. Aperte o botão com o ícone de uma seta;
5. Digite o nome do grupo.

Com o grupo criado, vamos adicionar o bot como administrador para que ele possa receber todas as mensagens do grupo. Uma outra opção seria desabilitar o seu modo de privacidade.
1. Abra o *chat* do grupo recém criado;
2. Abra o perfil do grupo;
3. Aperte o botão com o ícone de um lápis;
4. No campo de descrição do grupo escreva: **Atenção, todas as mensagens são armazenadas pelo *bot* do grupo**;
5. Selecione Administrators;
6. Aperte o botão com o ícone de um usuário;
7. Selecione o *bot*.
8. Aperte o botão com o ícone de um *check*.

Por fim, vamos configurar o bot para que ele não possa ser adicionado a outros grupos.
1. Abra o *chat* com o `BotFather`;
2. Digite `/mybots`;
3. Selecione o *bot* pelo seu nome de usuário;
4. Selecione `Bot Settings`;
5. Selecione `Allow Groups?`;
6. Selecione `Turn groups off`.

Com tudo pronto, envie algumas mensagens no grupo.

Bot API
As mensagens captadas por um *bot* podem ser acessadas via API. A única informação necessária é o `token` de acesso fornecido pelo `BotFather` na criação do *bot*.
Nota: A documentação completa da API pode ser encontrada neste [link](https://core.telegram.org/bots/api)

`Nota` = Processo feito no arquivo bot_api.ipynb

## Dados
Antes de avançar para etapa analítica, vamos trabalhar na manipulação dos dados de mensagens do Telegram.

### **Mensagem** 
Uma mensagem recuperada via API é um dado semi-estruturado no formato JSON com algumas chaves mandatórias e diversas chaves opcionais, estas últimas presentes (ou não) dependendo do tipo da mensagem. Por exemplo, mensagens de texto apresentam a chave `text` enquanto mensagens de áudio apresentam a chave `audio`. Neste projeto vamos focar em mensagens do tipo texto, ou seja, vamos ingerir as chaves mandatórias e a chave `text`.

> **Nota**: A lista completa das chaves disponíveis pode ser encontrada na documentação neste [link](https://core.telegram.org/bots/api#message).
 - Exemplo:
   %%writefile telegram.json
{
    "update_id": 123,
    "message": {
        "message_id": 1,
        "from": {
            "id": 321,
            "is_bot": false,
            "first_name": "Andre"
        },
        "chat": {
            "id": -789,
            "type": "group"
        },
        "date": 1640995200,
        "text": "Ola, mundo!"
    }
}

- Descrição:
| chave | tipo valor | opcional | descrição |
| -- | -- | -- | -- |
| updated_id | int | não | id da mensagem enviada ao **bot** |
| message_id | int | não | id da mensagem enviada ao grupo |
| from_id | int | sim | id do usuário que enviou a mensagem |
| from_is_bot | bool | sim | se o usuário que enviou a mensagem é um **bot** |
| from_first_name | str | sim | primeiro nome do usário que enviou a mensagem |
| chat_id | int | não | id do *chat* em que a mensagem foi enviada |
| chat_type | str | não | tipo do *chat*: private, group, supergroup ou channel |
| date | int | não | data de envio da mensagem no formato unix |
| text | str | sim | texto da mensagem |

## . Contexto

- **Arquitetura**

**Telegram**
As mensagens captadas por um *bot* podem ser acessadas via API. A única informação necessária é o `token` de acesso fornecido pelo `BotFather` na criação do *bot*.
> **Nota:** A documentação completa da API pode ser encontrada neste [link](https://core.telegram.org/bots/api)

Insira o token aqui:
from getpass import getpass
token = getpass()

A `url` base é comum a todos os métodos da API.
import json
base_url = f'https://api.telegram.org/bot{token}'

- **getMe**
O método `getMe` retorna informações sobre o *bot*.
import requests
response = requests.get(url=f'{base_url}/getMe')
print(json.dumps(json.loads(response.text), indent=2))

- **getUpdates**
O método `getMe` retorna as mensagens captadas pelo *bot*.
response = requests.get(url=f'{base_url}/getUpdates')
print(json.dumps(json.loads(response.text), indent=2))

![Captura de tela 2024-05-10 191828](https://github.com/luisfernandogbraga/Pipeline-de-dados-do-Telegram/assets/134460985/24cdc6c9-764c-46cd-8b28-580df15d405d)

OBS: Nunca disponibilize o id do chat.

## . Ingestão
O dado ingerido é persistido no formato mais próximo do original, ou seja, nenhuma transformação é realizada em seu conteúdo ou estrutura (*schema*). Como exemplo, dados de uma API *web* que segue o formato REST (*representational state transfer*) são entregues, logo, persistidos, no formato JSON.

No projeto, as mensagens capturadas pelo *bot* podem ser ingeridas através da API *web* de *bots* do **Telegram**, portanto são fornecidos no formato JSON. Como o **Telegram** retem mensagens por apenas 24h em seus servidores, a ingestão via **streaming** é a mais indicada. Para que seja possível esse tipo de **ingestão** seja possível, vamos utilizar um *webhook* (gancho *web*), ou seja, vamos redirecionar as mensagens automaticamente para outra API *web*.

Sendo assim, precisamos de um serviço da AWS que forneça um API *web* para receber os dados redirecionados, o `AWS API Gateway` (documentação neste [link](https://docs.aws.amazon.com/pt_br/apigateway/latest/developerguide/welcome.html)). Dentre suas diversas funcionalidades, o `AWS API Gateway` permite o redirecionamento do dado recebido para outros serviços da AWS. Logo, vamos conecta-lo ao `AWS Lambda`, que pode sua vez, irá armazenar o dado em seu formato original (JSON) em um *bucket* do `AWS S3`.

> Sistemas que reagem a eventos são conhecidos como *event-driven*.

Portanto, precisamos:
 - Criar um *bucket* no `AWS S3`;
 - Criar uma função no `AWS Lambda`;
 - Criar uma API *web* no `AWS API Gateway`;
 - Configurar o *webhook* da API de *bots* do **Telegram**.

### **. AWS S3**
Na etapa de **ingestão**, o `AWS S3` tem a função de passivamente armazenar as mensagens captadas pelo *bot* do **Telegram** no seu formato original: JSON. Para tanto, basta a criação de um *bucket*. Como padrão, vamos adicionar o sufixo `-raw` ao seu nome (vamos seguir esse padrão para todos os serviços desta camada).

> **Nota**: um `data lake` é o nome dado a um repositório de um grande volume dados. É organizado em zonas que armazenam replicadas dos dados em diferentes níveis de processamento. A nomenclatura das zonas varia, contudo, as mais comuns são: *raw* e *enriched* ou *bronze*, *silver* e *gold*.

Bucket criado
![2](https://github.com/luisfernandogbraga/Pipeline-de-dados-do-Telegram/assets/134460985/4cd873cd-253f-4ca9-b11c-a35ade85d856)

### **. AWS Lambda**
Na etapa de **ingestão**, o `AWS Lambda` tem a função de ativamente persistir as mensagens captadas pelo *bot* do **Telegram** em um *bucket* do `AWS S3`. Para tanto vamos criar uma função que opera da seguinte forma:
- Recebe a mensagem no parâmetro `event`;
 - Verifica se a mensagem tem origem no grupo do **Telegram** correto;
 - Persiste a mensagem no formato JSON no *bucket* do `AWS S3`;
 - Retorna uma mensagem de sucesso (código de retorno HTTP igual a 200) a API de *bots* do **Telegram**.
 - 
> **Nota**: No **Telegram**, restringimos a opção de adicionar o *bot* a grupos, contudo, ainda é possível iniciar uma conversa em um *chat* privado.

Função lambda
OBS: codigo no arquivo lambda.ipynb

TESTE

![4](https://github.com/luisfernandogbraga/Pipeline-de-dados-do-Telegram/assets/134460985/da0239db-c003-43a2-909d-5b0c78d92b0e)

CODIGO

![3](https://github.com/luisfernandogbraga/Pipeline-de-dados-do-Telegram/assets/134460985/4586f6a0-639f-49d0-82ac-25a137e6bc50)

VARIAVEIS DE AMBIENTE

Note que o código exige a configuração de duas variáveis de ambiente: `AWS_S3_BUCKET` com o nome do *bucket* do `AWS S3` e `TELEGRAM_CHAT_ID` com o id do *chat* do grupo do **Telegram**. Para adicionar variáveis de ambiente em uma função do `AWS Lambda`, basta acessar configurações -> variáveis de ambiente no console da função.

> **Nota**: Variáveis de ambiente são excelentes formas de armazenar informações sensíveis.

![5](https://github.com/luisfernandogbraga/Pipeline-de-dados-do-Telegram/assets/134460985/d01f1939-0e58-4425-8541-eda40b99a061)

PERMISSÕES

Por fim, precisamos adicionar a permissão de escrita no *bucket* do `AWS S3` para a função do `AWS Lambda` no `AWS IAM`.
### **AWS API Gateway**

Na etapa de **ingestão**, o `AWS API Gateway` tem a função de receber as mensagens captadas pelo *bot* do **Telegram**, enviadas via *webhook*, e iniciar uma função do `AWS Lambda`, passando o conteúdo da mensagem no seu parâmetro *event*. Para tanto vamos criar uma API e configurá-la como gatilho da função do `AWS Lambda`:

- Acesse o serviço e selecione: *Create API* -> *REST API*;
- Insira um nome;
- Selecione: *Actions* -> *Create Method* -> *POST*;
![7](https://github.com/luisfernandogbraga/Pipeline-de-dados-do-Telegram/assets/134460985/9f12835d-0fdc-4400-8377-d09eebc89839)

- Na tela de *setup*: 
- Selecione *Integration type* igual a *Lambda Function*;
- Habilite o *Use Lambda Proxy integration*;
- Busque pelo nome a função do `AWS Lambda`.

Podemos testar a integração com o `AWS Lambda` através da ferramenta de testes do serviço. Por fim, vamos fazer a implantação da API e obter o seu endereço *web*.
- Selecione: *Actions* -> *Deploy API*;
- Selecione: *New Stage* para *Deployment stage*;
- Adicione *dev* como `Stage name`.

![6](https://github.com/luisfernandogbraga/Pipeline-de-dados-do-Telegram/assets/134460985/b911a300-e089-4794-a54c-8f930b4dd478)

Copie o a `url` gerada na variável `aws_api_gateway_url`.

from getpass import getpass
aws_api_gateway_url = getpass()

### **. Telegram**

Vamos configurar o *webhook* para redirecionar as mensagens para a `url` do `AWS API Gateway`.

 - **setWebhook**
O método `setWebhook` configura o redirecionamento das mensagens captadas pelo *bot* para o endereço *web* do paramametro `url`.

> **Nota**: os métodos `getUpdates` e `setWebhook` são mutualmente exclusivos, ou seja, enquanto o *webhook* estiver ativo, o método `getUpdates` não funcionará. Para desativar o *webhook*, basta utilizar o método `deleteWebhook`.

response = requests.get(url=f'{base_url}/setWebhook?url={aws_api_gateway_url}')
print(json.dumps(json.loads(response.text), indent=2))

![8](https://github.com/luisfernandogbraga/Pipeline-de-dados-do-Telegram/assets/134460985/bbb2d32f-fb07-4da6-a958-83e246f94c4b)


## 2\. ETL

A etapa de **extração, transformação e carregamento** (do inglês *extraction, transformation and load* ou **ETL**) é uma etapa abrangente responsável pela manipulação dos dados ingeridos de sistemas transacionais, ou seja, já persistidos em camadas cruas ou *raw* de sistemas analíticos. Os processos conduzidos nesta etapa variam bastante de acordo com a área da empresa, do volume/variedade/velocidade do dado consumido, etc. Contudo, em geral, o dado cru ingerido passa por um processo recorrente de *data wrangling* onde o dado é limpo, deduplicado, etc. e persistido com técnicas de particionamento, orientação a coluna e compressão. Por fim, o dado processado está pronto para ser analisado por profissionais de dados.

No projeto, as mensagens de um único dia, persistidas na camada cru, serão compactas em um único arquivo, orientado a coluna e comprimido, que será persistido em uma camada enriquecida. Além disso, durante este processo, o dado também passará por etapas de *data wrangling*.

### **. AWS S3** 
Na etapa de **ETL**, o `AWS S3` tem a função de passivamente armazenar as mensagens processadas de um dia em um único arquivo no formato Parquet. Para tanto, basta a criação de um *bucket*. Como padrão, vamos adicionar o sufixo `-enriched` ao seu nome (vamos seguir esse padrão para todos os serviços desta camada).

> **Nota**: um `data lake` é o nome dado a um repositório de um grande volume dados. É organizado em zonas que armazenam replicadas dos dados em diferentes níveis de processamento. A nomenclatura das zonas varia, contudo, as mais comuns são: *raw* e *enriched* ou *bronze*, *silver* e *gold*.

### **. AWS Lambda** 

Na etapa de **ETL**, o `AWS Lambda` tem a função de ativamente processar as mensagens captadas pelo *bot* do **Telegram**, persistidas na camada cru no *bucket* do `AWS S3`, e persisti-las na camada enriquecida, também em um *bucket* do `AWS S3`. Logo, vamos criar uma função que opera da seguinte forma:

- Lista todos os arquivos JSON de uma única participação da camada crua de um *bucket* do `AWS S3`;
- Para cada arquivo listado:
- Faz o *download* do arquivo e carrega o conteúdo da mensagem;
- Executa uma função de *data wrangling*;
- Cria uma tabela do PyArrow e a contatena com as demais.
- Persiste a tabela no formato Parquet na camada enriquecida em um *bucket* do `AWS S3`.

> **Nota**: O fato de utilizarmos duas camadas de armazenamento e processamento, permite que possamos reprocessar os dados crus de diversas maneiras, quantas vezes forem preciso.

> **Nota**: Atente-se ao fato de que a função processa as mensagens do dia anterior (D-1).

OBS: arqueivo com o codigo enriched.ipynb

### **. AWS Event Bridge** 
Na etapa de **ETL**, o `AWS Event Bridge` tem a função de ativar diariamente a função de **ETL** do `AWS Lambda`, funcionando assim como um *scheduler*.
> **Nota**: Atente-se ao fato de que a função processa as mensagens do dia anterior (D-1).

## . Apresentação
A etapa de **apresentação** é reponsável por entregar o dado para os usuários (analistas, cientistas, etc.) e sistemas (dashboards, motores de consultas, etc.), idealmente através de uma interface de fácil uso, como o SQL, logo, essa é a única etapa que a maioria dos usuários terá acesso. Além disso, é importante que as ferramentas da etapa entregem dados armazenados em camadas refinadas, pois assim as consultas são mais baratas e o dados mais consistentes.

### **. AWS Athena** 
Na etapa de **apresentação**, o `AWS Athena` tem função de entregar o dados através de uma interface SQL para os usuários do sistema analítico. Para criar a interface, basta criar uma tabela externa sobre o dado armazenado na camada mais refinada da arquitetura, a camada enriquecida.

```sql
CREATE EXTERNAL TABLE `telegram`(
  `message_id` bigint, 
  `user_id` bigint, 
  `user_is_bot` boolean, 
  `user_first_name` string, 
  `chat_id` bigint, 
  `chat_type` string, 
  `text` string, 
  `date` bigint)
PARTITIONED BY ( 
  `context_date` date)
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe' 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat'
LOCATION
  's3://<bucket-enriquecido>/'
```
Por fim, adicione as partições disponíveis.

> **Importante**: Toda vez que uma nova partição é adicionada ao repositório de dados, é necessário informar o `AWS Athena` para que a ela esteja disponível via SQL. Para isso, use o comando SQL `MSCK REPAIR TABLE <nome-tabela>` para todas as partições (mais caro) ou `ALTER TABLE <nome-tabela> ADD PARTITION <coluna-partição> = <valor-partição>` para uma única partição (mais barato), documentação neste [link](https://docs.aws.amazon.com/athena/latest/ug/alter-table-add-partition.html)).




