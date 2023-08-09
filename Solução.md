# Desafio

Uma empresa possui a página de sua empresa no linkedin e gostaria de acompnhar o acrescimo e decrescimo de seguidores de sua página fora das limitações e do que é oferecido pelo linkedin,
ou seja, deseja ter mais liberdade na utilização dos dados de seguidores de linkedin, realizando cruzamento com dados de outras das suas redes sociais como o facebook, twitter e instagram,
entender o comportamento desses dados junto com outras ações que a empresa toma (sera que isso está influenciando o numero de seguidores aumentar ou diminuir?) e poder trabalhar com agragações,
metas e projeções.

# Quais a principais perguntas que devem ser respondidas (qual é a dor?)

## Benefícios(valor) esperado pelo cliente

- Serei capaz de tomar decisões mais rápidas e em tempo hábil?
- Essa solução irá automatizar o processo evitando a intervenção humano para coletar, tratar e visualisar esses dados?
- Os meus dados ficaram seguros e disponíveis com qualidade?
- Irei economizar em relação a custos com infraestrutura?
- Irei conseguir alcançar meus objetivos comerciais?
- Essa solução irá me ajudar a melhorar a experiência do meu cliente?

# Pipeline de dados

![Pipeline linkedinFollowers](https://github.com/bifastsolutions/linkedinPageFollowers/assets/134235178/9af205f5-e287-4584-852a-3acc37bafd0d)

## Serviços da AWS utilizados

- S3
- EC2
- Lambda
- eventBridget
- Glue(Crawler e Catalog)
- Athena

# Desenvolvimento da solução

## Configuração da API do linkedin

●	Entre em https://developer.linkedin.com/ 
●	Realize o acesso utilizando suas credenciais, navegue até a seção "My Apps" e clique em "Create app". Nesse local, será necessário preencher os dados solicitados, tais como o título da aplicação, o endereço da página de sua empresa no LinkedIn e o logotipo.

![Create an app](https://github.com/bifastsolutions/linkedinPageFollowers/assets/134235178/fc2fff91-920f-4221-a5fb-cf9d4e56b79c)


●	Após concluir a criação do aplicativo, acesse a seção de "Settings" e proceda com a verificação do app. Gere a URL na etapa seguinte, copie-a e cole-a na barra de endereços do navegador. Em seguida, você será redirecionado para uma tela de confirmação indicando que a verificação foi realizada com sucesso.

![Settings](https://github.com/bifastsolutions/linkedinPageFollowers/assets/134235178/98b14d4b-f5e4-497b-a18a-af113ff42c6b)


●	Na seção de "products" terá as opções do que é possível fazer com a API do LinkedIn junto com a documentação para cada caso, onde mostra como você deve utilizar ao fazer os "requests". Libere o acesso pelo "Request access".

![Products](https://github.com/bifastsolutions/linkedinPageFollowers/assets/134235178/ece6bf2b-f11b-4b41-8426-a0a0f2ca6101)


● Com o intuito de gerar o token de verificação para acesso aos itens disponíveis, é necessário acessar a seção de Auth e selecionar a alternativa localizada ao lado direito denominada  OAuth 2.0 tools.

![OAuth 2.0 tools](https://github.com/bifastsolutions/linkedinPageFollowers/assets/134235178/d158c4dc-a6a6-4311-9386-7312a535336f)


●	Entre em "create token", escolha os scopes(permissões) desejados e crie seu token para utilização.


![create token](https://github.com/bifastsolutions/linkedinPageFollowers/assets/134235178/51da68c1-ce94-438e-8cc8-a05b26df78ed)


![Token](https://github.com/bifastsolutions/linkedinPageFollowers/assets/134235178/79836aae-ad85-4d1e-b965-d20109c0ab1f)

● Após concluir essas etapas, você obterá o seu token de acesso, permitindo que você faça solicitações de acordo com os products que foram selecionados. É importante lembrar que cada product possui sua própria documentação exclusiva para realizar as solicitações.

● Para obter as permissões necessárias para extrair os dados de seguidores, é fundamental ativar o advertising API e completar um formulário fornecido pelo LinkedIn. O formulário incluirá algumas perguntas sobre a sua empresa e os motivos pelos quais deseja habilitar esse produto. É importante destacar que o LinkedIn enfatiza que a aprovação não é automática, podendo levar alguns dias até que você receba uma resposta.

## Permissões sem ativar advertising API

![Permissões sem ativar advertising API](https://github.com/bifastsolutions/linkedinPageFollowers/assets/134235178/1ffbb1b0-ee36-4958-ae66-819a44a33ab8)

## Permissões depois de ativar advertising API

![Permissões depois de ativar advertising API](https://github.com/bifastsolutions/linkedinPageFollowers/assets/134235178/b26bfa33-264c-48f1-9490-0023d5bf045f)


# Solução

Como se trata de algo muito simples, não vi a necessidade de separar os dados da API em zonas em uma Data Lake, usei apenas um código que realiza a extração, tratamento e carga no S3, sendo nesse caso um ETL e não um ELT, o código não demora nem 10s para rodar e carregar os dados que são feitos diariamente, o código foi todo feito em OOP pois achei melhor a divisão das class e suas funções como blocos onde cada um executa uma determinada ação, deixando assim mais organizado, estarei explicando tudo com mais detalhes abaixo mostrando o código.

## Desenvolvimento do código em Python

### Class LinkedInAPI

```python
import requests
import pandas as pd
import datetime
import smtplib
from email.mime.text import MIMEText
import boto3
```

requests:

A biblioteca requests é usada para fazer requisições HTTP. No contexto desse código, ela é utilizada para fazer requisições GET à API do LinkedIn para obter os dados das estatísticas de seguidores da organização.

pandas (importado como pd):

A biblioteca pandas é uma poderosa biblioteca para análise e manipulação de dados em Python.
Ela é usada para realizar operações como normalização de dados JSON, transformações em DataFrames, manipulação de datas e horas, seleção de colunas, e, finalmente, para salvar um DataFrame em um arquivo CSV.
O DataFrame é uma estrutura de dados tabular semelhante a uma tabela de banco de dados e é amplamente usado para trabalhar com dados estruturados.

datetime:

A biblioteca datetime é usada para trabalhar com datas e horas em Python.
Ela é utilizada neste código para calcular as datas de início e fim do intervalo desejado e para manipular informações temporais, como a criação de objetos de data e hora.

smtplib:

A biblioteca smtplib faz parte da biblioteca padrão do Python e é usada para enviar e-mails utilizando o protocolo SMTP (Simple Mail Transfer Protocol).
No contexto do código, a classe EmailSender utiliza a smtplib para se conectar a um servidor SMTP (neste caso, o servidor do Outlook) e enviar um e-mail de alerta.

email.mime.text (importado como MIMEText):

Esta é uma submódulo da biblioteca email padrão do Python, que fornece classes para criar e manipular mensagens de e-mail.
A classe MIMEText é usada para criar uma parte de conteúdo de e-mail em formato de texto, que pode ser usado para criar o corpo de um e-mail com suporte a HTML.

boto3:

A biblioteca boto3 é usada para interagir com os serviços da AWS (Amazon Web Services).
No contexto deste código, é usada para interagir com o serviço Amazon S3 (Simple Storage Service) para salvar o DataFrame como um arquivo CSV.
É necessário ter as credenciais corretamente configuradas para usar o boto3 e acessar os recursos da AWS.
Essas bibliotecas são componentes essenciais que permitem ao código realizar tarefas específicas, como requisições HTTP, manipulação de dados, envio de e-mails e interação com serviços da AWS. Cada biblioteca traz funcionalidades especializadas que são utilizadas para atender às necessidades do programa.





