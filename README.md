# Pipeline de dados do Telegram
![000_1433zq-654x450](https://github.com/luisfernandogbraga/Pipeline-de-dados-do-Telegram/assets/134460985/166cdb89-e429-4b40-8316-d4475c19f982)

# Apresentação e desenvolvimento do projeto de Pipeline de dados do Telegram, desde sua criação até automação com bot no Telegram e coleta/tratativa de dados
Vou construir um Pipeline completo de Dados, ou seja vou construir uma solução na AWS que permitar automatizar automaticamente: extrair, manipular, armazenar e processar um grande volume de dados. Isso permitirá que seja gerado diáriamente insghts novos sobre esse tipo de informação. Portamdo estarei dividindo o projeto em dois: insfraestrutura e extrair / estoritelin na nuvem.

# Contexto
Um chatbot é um tipo de software que interage com usuários através de conversas automatizadas em plataformas de mensagens. Uma aplicação comum de chatbots é o seu uso no atendimento ao cliente, onde, de maneira geral, ajudam clientes a resolver problemas ou esclarecer dúvidas recorrentes antes mesmo que um atendente humano seja acionado.

Telegram é uma plataforma de mensagens instantâneas freeware (distribuído gratuitamente) e, em sua maioria, open source. É muito popular entre desenvolvedores por ser pioneiro na implantação da funcionalidade de criação de chatbots, que, por sua vez, permitem a criação de diversas automações.

# Arquitetura

Uma atividade analítica de interesse é a de realizar a análise exploratória de dados enviadas a um chatbot para responder perguntas como:

1. Qual o horário que os usuários mais acionam o bot?
2. Qual o problema ou dúvida mais frequente?
3. O bot está conseguindo resolver os problemas ou esclarecer as dúvidas?
4. Etc.

Portanto, vamos construir um Pipeline de Dados que ingira, processe, armazene e exponha mensagens de um grupo do Telegram para que profissionais de dados possam realizar análises. A arquitetura proposta é dividida em duas: transacional, no Telegram, onde os dados são produzidos, e analítica, na Amazon Web Services (AWS), onde os dados são analisados.


Telegram . O Telegram representa a fonte de dados transacionais. Mensagens enviadas por usuários em um grupo são capturadas por um bot e redirecionadas via webhook do backend do aplicativo para um endpoint (endereço web que aceita requisições HTTP) exposto pelo AWS API Gateway. As mensagens trafegam no corpo ou payload da requisição.

AWS | Ingestão . Uma requisição HTTP com o conteúdo da mensagem em seu payload é recebia pelo AWS API Gateway que, por sua vez, as redireciona para o AWS Lambda, servindo assim como seu gatilho. Já o AWS Lambda recebe o payload da requisição em seu parâmetro event, salva o conteúdo em um arquivo no formato JSON (original, mesmo que o payload) e o armazena no AWS S3 particionado por dia.

AWS | ETL . Uma vez ao dia, o AWS Event Bridge aciona o AWS Lambda que processa todas as mensagens do dia anterior (atraso de um dia ou D-1), denormaliza o dado semi-estruturado típico de arquivos no formato JSON, salva o conteúdo processado em um arquivo no formato Apache Parquet e o armazena no AWS S3 particionado por dia.

AWS | Apresentação . Por fim, uma tabela do AWS Athena é apontada para o bucket do AWS S3 que armazena o dado processado: denormalizado, particionado e orientado a coluna. Profissionais de dados podem então executar consultas analíticas (agregações, ordenações, etc.) na tabela utilizando o SQL para a extração de insights.

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

`Nota` = Processo feito no arquivo bot_api

