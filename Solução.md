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

![Pipeline linkedinFollowers](https://github.com/bifastsolutions/linkedinPageFollowers/assets/134235178/cc40f3e2-5884-4c65-bcad-3a3301a53d34)

# Desenvolvimento da solução

## Configuração da API do linkedin

●	Entre em https://developer.linkedin.com/ 
●	Realize o acesso utilizando suas credenciais, navegue até a seção "My Apps" e clique em "Create app". Nesse local, será necessário preencher os dados solicitados, tais como o título da aplicação, o endereço da página de sua empresa no LinkedIn e o logotipo.


![Create an app](https://github.com/bifastsolutions/linkedinPageFollowers/assets/134235178/18325286-d3ff-4d65-af3e-7c28acdb5004)

●	Após concluir a criação do aplicativo, acesse a seção de "Settings" e proceda com a verificação do app. Gere a URL na etapa seguinte, copie-a e cole-a na barra de endereços do navegador. Em seguida, você será redirecionado para uma tela de confirmação indicando que a verificação foi realizada com sucesso.

![Settings](https://github.com/bifastsolutions/linkedinPageFollowers/assets/134235178/97ceae70-5c7a-470d-9d27-dd1fb6a72c9d)

●	Na seção de "products" terá as opções do que é possível fazer com a API do LinkedIn junto com a documentação para cada caso, onde mostra como você deve utilizar ao fazer os "requests". Libere o acesso pelo "Request access".

![Products](https://github.com/bifastsolutions/linkedinPageFollowers/assets/134235178/c1ce133d-c2e1-433a-8ec6-af18be2e3979)

● Com o intuito de gerar o token de verificação para acesso aos itens disponíveis, é necessário acessar a seção de Auth e selecionar a alternativa localizada ao lado direito denominada  OAuth 2.0 tools.

![Page 4](https://github.com/bifastsolutions/linkedinPageFollowers/assets/134235178/b5be28dd-673b-4f8b-b6bc-e3d4e6fd93bf)

●	Entre em "create token", escolha os scopes(permissões) desejados e crie seu token para utilização.

![Oauth 2.0 tools](https://github.com/bifastsolutions/linkedinPageFollowers/assets/134235178/5c974624-579d-4da3-97f8-d306ef5237b7)

![Token](https://github.com/bifastsolutions/linkedinPageFollowers/assets/134235178/a5c13e9d-0860-425e-bd18-75ea73916a60)

● Após concluir essas etapas, você obterá o seu token de acesso, permitindo que você faça solicitações de acordo com os products que foram selecionados. É importante lembrar que cada product possui sua própria documentação exclusiva para realizar as solicitações.

● Para obter as permissões necessárias para extrair os dados de seguidores, é fundamental ativar o advertising API e completar um formulário fornecido pelo LinkedIn. O formulário incluirá algumas perguntas sobre a sua empresa e os motivos pelos quais deseja habilitar esse produto. É importante destacar que o LinkedIn enfatiza que a aprovação não é automática, podendo levar alguns dias até que você receba uma resposta.

## Permissões sem ativar advertising API

![Permissões sem ativar advertising API](https://github.com/bifastsolutions/linkedinPageFollowers/assets/134235178/1ffbb1b0-ee36-4958-ae66-819a44a33ab8)

## Permissões depois de ativar advertising API

![Permissões depois de ativar advertising API](https://github.com/bifastsolutions/linkedinPageFollowers/assets/134235178/b26bfa33-264c-48f1-9490-0023d5bf045f)







