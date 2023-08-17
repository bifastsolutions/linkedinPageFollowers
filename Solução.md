# Desafio

Uma empresa possui a página de sua empresa no linkedin e gostaria de acompnhar o acrescimo e decrescimo de seguidores de sua página fora das limitações e do que é oferecido pelo linkedin,
ou seja, deseja ter mais liberdade na utilização dos dados de seguidores de linkedin, realizando cruzamento com dados de outras das suas redes sociais como o facebook, twitter e instagram,
entender o comportamento desses dados junto com outras ações que a empresa toma (será que isso está influenciando o número de seguidores aumentar ou diminuir?) e poder trabalhar com agragações, metas e projeções.

# Quais a principais perguntas que devem ser respondidas (qual é a dor?)

## Benefícios(valor) esperado pelo cliente

- Serei capaz de tomar decisões mais rápidas e em tempo hábil?
- Essa solução irá automatizar o processo evitando a intervenção humano para coletar, tratar e visualisar esses dados?
- Os meus dados ficaram seguros e disponíveis com qualidade?
- Irei economizar em relação a custos com infraestrutura?
- Irei conseguir alcançar meus objetivos comerciais?
- Essa solução irá me ajudar a melhorar a experiência do meu cliente?

# Pipeline de dados

![Pipeline linkedinFollowers](https://github.com/bifastsolutions/linkedinPageFollowers/assets/134235178/c67c6eee-64f5-45f1-9a50-ee154e267146)



## Serviços da AWS utilizados

- S3
- EC2
- Lambda
- eventBridget
- Glue(Crawler e Catalog)
- Athena

# Desenvolvimento da solução

## Configuração da API do linkedin

Segue explicação resumida dos passos necessários para configurar e acessar a API do LinkedIn em um aplicativo.
- Primeiro, após criar o aplicativo, você deve verificar o mesmo através da geração de uma URL na seção "Settings".
- Em seguida, você será redirecionado para uma tela de confirmação.
- Na seção "products", são apresentadas as opções de uso da API do LinkedIn, cada uma com sua própria documentação e permissões.
- Para obter acesso, é preciso gerar um token de verificação na seção "Auth", selecionando OAuth 2.0 tools e criando o token com os escopos desejados.
- Uma vez concluído, o token permite solicitações de acordo com os produtos selecionados.
- Para acessar dados de seguidores, é necessário ativar o advertising API, preencher um formulário do LinkedIn e aguardar a aprovação, que não é automática.

![Linkedin API](https://github.com/bifastsolutions/linkedinPageFollowers/assets/134235178/a2755211-f9f5-4fab-83c2-a680c729cc70)


# Solução


Como se trata de algo muito simples, não vi a necessidade de separar os dados da API em zonas em uma Data Lake, usei apenas um código que realiza a extração, tratamento e carga no S3, sendo nesse caso um ETL e não um ELT, o código não demora nem 10s para rodar e carregar os dados que são feitos diariamente(agendado), o código foi todo feito em OOP pois achei melhor a divisão das class e suas funções como blocos onde cada um executa uma determinada ação, deixando assim mais organizado, estarei explicando tudo com mais detalhes abaixo mostrando o código.

# Desenvolvimento do código em Python

## Bibliotecas

```python
import requests
import pandas as pd
import datetime
import smtplib
from email.mime.text import MIMEText
import boto3
import os
from dotenv import load_dotenv
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

os:

A biblioteca os é uma biblioteca padrão do Python que fornece funções para interagir com o sistema operacional, incluindo a manipulação de caminhos de arquivos, variáveis de ambiente, entre outros.
No contexto do código, a biblioteca os não está sendo usada diretamente no código que você forneceu. No entanto, é comum utilizá-la para manipular caminhos de arquivos, checar a existência de arquivos ou diretórios, entre outras operações relacionadas ao sistema operacional.

dotenv:

A biblioteca dotenv não faz parte da biblioteca padrão do Python. É uma biblioteca de terceiros que permite carregar variáveis de ambiente a partir de um arquivo .env.
No contexto deste código, a função principal do dotenv é carregar variáveis de ambiente a partir de um arquivo .env (caso exista) e torná-las disponíveis para o programa.
Variáveis de ambiente podem conter informações sensíveis, como chaves de API, tokens de acesso, senhas etc. Carregar essas informações de um arquivo .env em vez de mantê-las diretamente no código-fonte ajuda a proteger essas informações sensíveis.




### Class LinkedInAPI

```python
class LinkedInAPI:
    def __init__(self, access_token, api_url):
        self.access_token = access_token
        self.api_url = api_url
        self.headers = {
            'Authorization': 'Bearer ' + self.access_token,
            'Content-Type': 'application/json',
            'Accept-Language': 'en-US'
        }

    def fetch_data(self, params):
        response = requests.get(self.api_url, params=params, headers=self.headers)
        return response.json() if response.status_code == 200 else None
```

Esta classe é responsável por encapsular a interação com a API do LinkedIn.
O construtor __init__ recebe um token de acesso (access_token) e a URL da API (api_url), e configura os cabeçalhos necessários para as requisições.
O método fetch_data recebe parâmetros, faz uma requisição GET à API usando o token e os cabeçalhos configurados, e retorna os dados da resposta no formato JSON se a resposta for bem-sucedida (status code 200) ou None caso contrário.

Documentação da API para extração de dados de seguidores do LinkedIn: https://learn.microsoft.com/en-us/linkedin/marketing/integrations/community-management/organizations/follower-statistics?view=li-lms-unversioned&tabs=curl Essa documentação é importante para preenchimento dos cabeçalhos necessários e outras informações do restante do código.

### Class DataProcessor

```python
class DataProcessor:
    def process_data(self, data, inicio_intervalo, fim_intervalo):
        try:
            df = pd.json_normalize(data, record_path='elements')

            df['timeRange.start'] = pd.to_datetime(df['timeRange.start'], unit='ms')
            df['timeRange.end'] = pd.to_datetime(df['timeRange.end'], unit='ms')

            df = df.rename(columns={
                'followerGains.organicFollowerGain': 'Ganhos de seguidores orgânicos',
                'followerGains.paidFollowerGain': 'Ganhos de seguidores patrocinados',
                'timeRange.start': 'Data',
                'timeRange.end': 'Data final'
            })

            df = df[['organizationalEntity', 'Data', 'Ganhos de seguidores orgânicos', 'Ganhos de seguidores patrocinados']]

            nome_arquivo = "follower_stats_" + fim_intervalo.strftime("%Y%m%d") + ".csv"

            # Salvar o DataFrame no Amazon S3
            self.save_to_s3(df, nome_arquivo)

            return True
        except KeyError:
            return False

    def save_to_s3(self, df, nome_arquivo):
        BUCKET_NAME = 'SEU_BUCKET'
        CAMINHO_NO_BUCKET = 'CAMINHO_BUCKET'
        session = boto3.Session(profile_name='default')  # 'default' é o perfil no arquivo credentials da sua máquina (.aws)
        s3 = session.client('s3')
    
        csv_buffer = df.to_csv(index=False)
        s3.put_object(Body=csv_buffer, Bucket=BUCKET_NAME, Key=CAMINHO_NO_BUCKET + nome_arquivo)
```

Esta classe é responsável por processar os dados obtidos da API do LinkedIn.
O método process_data recebe os dados brutos da API, além de datas de início e fim de intervalo(essas datas vem em milesegundos(ms)).
Ele transforma os dados brutos em um DataFrame do pandas usando pd.json_normalize.
Realiza transformações nas colunas de datas e renomeia colunas relevantes.
Seleciona as colunas desejadas e salva o DataFrame em um arquivo CSV no Amazon S3 usando o método save_to_s3.
Caso ocorra uma exceção do tipo KeyError, retorna False.
"session = boto3.Session(profile_name='default')  # 'default' é o perfil no arquivo credentials da sua máquina (.aws)" essa parte
do código é muito importante, pois evita que você passe as credenciais da AWS diretamente no código, deixando assim mais seguro.

Os arquivos são salvos no S3 com o nome padronizado em D-3 pois a API não disponibiliza os dados antes disso.



![S3 Bucket](https://github.com/bifastsolutions/linkedinPageFollowers/assets/134235178/bc115133-d0a2-468b-9539-f676f6682e09)


### Class EmailSender:

```python
class EmailSender:
    def send_email(self, mensagem):
        load_dotenv("SEU_ARQUIVO.env")  # Carregar variáveis de ambiente do arquivo .env
        remetente = os.environ.get('EMAIL_REMETENTE')
        destinatario = os.environ.get('EMAIL_DESTINATARIO')
        assunto = 'Alerta: Não há dados disponíveis'
        corpo_email = f"""\
            <html>
                <body>
                    <h3>{mensagem}</h3>
                </body>
            </html>
            """
        mime_text = MIMEText(corpo_email, 'html')
        mime_text['From'] = remetente
        mime_text['To'] = destinatario
        mime_text['Subject'] = assunto
        servidor_smtp = 'smtp-mail.outlook.com'
        porta_smtp = 587
        usuario_smtp = os.environ.get('SMTP_USUARIO')
        senha_smtp = os.environ.get('SMTP_SENHA')

        with smtplib.SMTP(servidor_smtp, porta_smtp) as servidor:
            servidor.starttls()
            servidor.login(usuario_smtp, senha_smtp)
            servidor.send_message(mime_text)

        print("E-mail enviado com sucesso!")
```

Esta classe lida com o envio de e-mails de alerta.
O método send_email recebe uma mensagem como entrada.
Ele configura informações como remetente, destinatário, assunto e conteúdo do e-mail.
As informações de email e senha ficam em um arquivo separado .env por segurança para não ficar exposto direto no código.
Utiliza a biblioteca smtplib para se conectar a um servidor SMTP (neste caso, o servidor do Outlook) e envia o e-mail.
Imprime uma mensagem quando o e-mail é enviado com sucesso.

### Função Main:


```python
def main():
    ACCESS_TOKEN = 'SEU_TOKEN'
    API_URL = 'https://api.linkedin.com/v2/organizationalEntityFollowerStatistics'

    inicio_intervalo = datetime.date.today() - datetime.timedelta(days=3)
    inicio_intervalo = datetime.datetime.combine(inicio_intervalo, datetime.time.min)

    fim_intervalo = datetime.date.today() - datetime.timedelta(days=3)
    fim_intervalo = datetime.datetime.combine(fim_intervalo, datetime.time.max)

    params = {
        'q': 'organizationalEntity',
        'organizationalEntity': 'urn:li:organization:SEU_ORGANIZATION',
        'timeIntervals.timeGranularityType': 'DAY',
        'timeIntervals.timeRange.start': str(int(inicio_intervalo.timestamp() * 1000)),
        'timeIntervals.timeRange.end': str(int(fim_intervalo.timestamp() * 1000))
    }

    linkedin_api = LinkedInAPI(ACCESS_TOKEN, API_URL)
    data = linkedin_api.fetch_data(params)

    if data:
        data_processor = DataProcessor()
        if data_processor.process_data(data, inicio_intervalo, fim_intervalo):
            print("Dados processados e salvos no Amazon S3 com sucesso!")
        else:
            email_sender = EmailSender()
            email_sender.send_email("Não há dados para o intervalo de tempo especificado.")

if __name__ == "__main__":
    main()
```

A função main é onde o fluxo principal do programa é orquestrado.
Configura os parâmetros para a chamada à API do LinkedIn, definindo o intervalo de tempo, organização alvo, etc.
Usa a classe LinkedInAPI para buscar os dados da API.
Se os dados forem obtidos, instancia a classe DataProcessor e tenta processar os dados.
Se o processamento for bem-sucedido, imprime uma mensagem de sucesso. Caso contrário, chama a classe EmailSender para enviar um e-mail de alerta.

# Serviços na AWS

### EC2

A VM que que foi criada no EC2 possui 2 funções muito importante, uma delas é ter o código na VM que irá rodar através de um arquivo .bat agendado pelo agendador do windows, isso economizará recurso de ETL como
uso de um serviço como o GLUE, e a outra função é de ter instalado o gateway do Power BI para rodas as atualizações dos daods conforme o agendamento do Power BI online e tambem a configuração do drive ODBC do Athena deve ser configurado com suas credenciais, por conta disso deve-se criar uma imagem do Windows Server com uma t2.micro que possui free tier em x quantidade de horas mês por 1 ano, evitando mais um custo desnecessário.

![VM no EC2](https://github.com/bifastsolutions/linkedinPageFollowers/assets/134235178/9fb2b2f4-74c8-439b-96bc-9f8d1b45c633)


### LAMBDA e EVENTBRIDGE

Pensando no futuro após 1 ano e evitar que a VM fique ligada 24h por dia e tendo cobrança alta, através de scripts python no lambda para ligar e desligar a VM juntamento com o scheduler no eventbridge a VM ficará ligada apenas 1 hora por dia, tempo suficiente para rodar o código e atualizar os dados através do gateway do Power BI.


![Funções Lambda](https://github.com/bifastsolutions/linkedinPageFollowers/assets/134235178/93c72ae9-e8a6-4121-a6cd-99dd1c98bf7e)


![Exemplo schedule EVENTBRIDGE](https://github.com/bifastsolutions/linkedinPageFollowers/assets/134235178/d6167853-434e-41fc-abef-71c98a838d54)


### CRAWLER, CATALOG e ATHENA

Através do dados salvos no Bucket do S3, eles serão enviados em forma de tabela para o Athena, esses dados quando transformaos em tabela criam um catalogo de dados sobre as colunas da
tabela enviada, esse catalogo de dados além de você poder colocar a descrição manualmente do significado da coluna, traz tambem a possibilidade de alteração do tipo de dados, caso o ETL
não tenha conseguido fixar de forma correta e persistente. O Athena servirá como um "Data Warehouse", sendo possível realizar consultas me sql que são cobradas apenas quando realizadas.


![Data Catalog](https://github.com/bifastsolutions/linkedinPageFollowers/assets/134235178/ab77382f-224a-4943-b935-aa9ae780a035)



![Athena](https://github.com/bifastsolutions/linkedinPageFollowers/assets/134235178/9a94a7a9-68d8-402e-b633-d3d9ae55ef87)



### POWER BI

E o Power BI consome os dados do Athena que é alimnentado e atualiado diariamente, abaixo um exemplo básico e simples de uso apenas para validação.


![Exemplo Power BI](https://github.com/bifastsolutions/linkedinPageFollowers/assets/134235178/70fb0a61-a643-44be-94a3-3d6903bff694)


# Entrega de valor

 A solução permitiu aos gestores acessar as informações de seguidores do linkedin em um ambiente com visão integrada e dinâmica (power BI), com custo quase zero de infraestrutura, com economia através do S3,
VM no EC2 com liga e desliga automatico por apenas 1 hora, tendo a segurança de uso na Cloud unindo recursos simples com a utilização de agendamento do próprio windows, a solução trouxe economia e velocidade
no desenvolvimento, pois além de não haver a necessidade de comprar um servidor fisico a construção da arquietura de dados em cloud e rapidamente levantado, como tambem caso o cliente desista é facilmente
destruído com total segurança. Os dados poderão ser cruzados com outras fontes de informações e poderão ser utilizados com agragações e etc. Esse projeto também tem a intenção de mostrar que mesmos projetos
simples podem tirar um bom proveito da computação em nuvem.



