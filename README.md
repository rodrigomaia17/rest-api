# TL;DR

Estão disponíveis bibliotecas para as seguintes plataformas:

- [.Net](https://github.com/clicksign/clicksign-dotnet)
- [Ruby](https://github.com/clicksign/clicksign-ruby)
- [PHP](https://github.com/clicksign/clicksign-php)

Particularidades da API da Clicksign:

- a versão da API está contida no caminho da requisição;
- a chave da API é enviada como um dos parâmetros;
- a chave da API é enviada em todas requisições;
- a chave da API identifica e autentica o usuário;

Fluxo de um documento:

1. Upload do documento
2. Criação de hooks em um documento
3. Criação de uma lista de assinatura

Cada documento possui uma chave única dentro da Clicksign.  Através desta chave
é feita toda referência a um documento.  É necessário criar uma lista de
documento para que um documento seja assinado.

# Índice

- [Introdução](#introducao)
- [Funcionamento geral](#funcionamento-geral)
- [Autenticação](#autenticacao)
- [Versão](#versao)
- [Formatos](#formatos)
- [Listagem de documentos](#listagem-de-documentos)
- [Visualizacão de Documento](#visualizacao-de-documento)
- [Upload de documentos](#upload-de-documentos)
- [Criação de lista de assinatura](#criacao-de-lista-de-assinatura)
- [Upload e criação de lista de assinatura](#upload-e-criacao-de-lista-de-assinatura)
- [Download de documento](#download-de-documento)
- [Criação de pacote de documentos](#criacao-de-pacote-de-documentos)
- [Hooks](#hooks)
- [Exemplos](#exemplos)

# <a name="introducao"></a>Introdução

A Clicksign é uma solução online para enviar, guardar e assinar documentos, com validade jurídica. Foi criada para facilitar, reduzir custo e aumentar a segurança e compliance do processo de assinatura e _workflow_ de documentos. 

A Clicksign pode ser acessada em https://desk.clicksign.com. 

O propósito desta **REST API** é prover meios para que nossos clientes adequem a Clicksign aos seus processos e sistemas p. ex. automatizar tarefas, desenhar fluxos de assinatura, e definir _workflow_. 

Qualquer linguagem de programação capaz de realizar requisições **HTTP / JSON** cumpre os requisitos necessários para consumir os serviços desta API. Assim, com pouco esforço de programação é possível integrar desde scripts shell até sistemas de ERP.

# <a name="funcionamento-geral"></a>Funcionamento geral

Uma _REST API_ é composta, basicamente, por dois elementos: um **cliente** e um **servidor**. O cliente sempre inicia a comunicação mediante requisição HTTP. O servidor sempre finaliza a comunicação respondendo à requisição.

![Cliente-Servidor](https://raw.github.com/clicksign/rest-api/master/images/client-server.png)

As mensagens HTTP são compostas por uma linha inicial, um conjunto de cabeçalhos e um corpo. A linha inicial difere nas requisições e nas respostas, o cabeçalho compartilha parâmetros em comum e parâmetros específicos, e o corpo é completamente dependente de cada mensagem, podendo até ser nulo.

A requisição, em sua linha inicial, indica o **método**, o **caminho**, e a **versão do protocolo**. O método e o caminho são essenciais em uma _REST API_ uma vez que ambos indicam a ação a ser executada no servidor.

A resposta, em sua linha inicial, indica a **versão do protocolo**, o **status**, e contém uma **mensagem informativa**. O código de status é essencial para o cliente saber se a ação foi devidamente executada no servidor.

A documentação de cada função da API determina o método e o caminho a ser utilizado, e o significado do corpo e de cada status da resposta.

**Atenção:** Toda a comunição cliente/servidor é feita através de HTTP sobre SSL/TLS (HTTPS). Requisições em HTTP simples resultam em redirecionamentos (301) para o protocolo HTTPS.

Todas as requisições da _REST API_ são feitas para `api.clicksign.com`.

## Exemplo de requisição

```http
GET /documents HTTPS/1.1
Host: api.clicksign.com
Accept: application/json
```

- Método: GET
- Caminho: /documents
- Versão: 1.1
- Cabeçalhos: Host, Accept
- Corpo: vazio

## Exemplo de resposta

```http
HTTP/1.1 200 OK
Content-Type:application/json
Connection: Keep-Alive
```

```json
[
  {
    "id": "abcd..."
  },

  {
    "id": "..."
  }
]
```

- Versão: 1.1
- Status: 200
- Mensagem: OK
- Cabeçalhos: Content-Type, Connection
- Corpo: [{...}]


# <a name="autenticacao"></a>Autenticação

A Clicksign utiliza duplo fator de autenticação para aumentar a segurança de suas transações. Autenticações que utilizam duplo fator geralmente são baseadas em algo que o cliente _conhece_ e algo que o cliente _possui_. No caso da API os fatores são:

1. Conhecer uma _string_ de **identificação**
1. Possuir um endereço **IP** específico

A autenticação é feito através do parâmetro **access_token** que automaticamente determina um usuário e realiza sua autenticação. O parâmetro deve ser enviado no **caminho** da requisição. Portanto, toda requisição deverá conter no _path_ `?access_token=string-do-token`.

**Atenção:** O parâmetro de autenticação deve ser enviado a cada requisição feita pelo cliente. Como esse parâmetro é comum a todas as funções da API, ele será omitido das documentações.

O segundo fator da autenticação é realizado automaticamente pelo servidor da Clicksign, que verifica se o **IP** de origem da requisição está dentro de uma lista de endereços previamente cadastrados para determinado cliente. Este fator de autenticação é **opcional**.


# <a name="versao"></a>Versão da API

Para possibilitar a expansão contínua da API, a Clicksign implementa um sistema
de versões. Dessa forma é necessário que a requisição contenha qual versão da
API está sendo utilizada. Isto **é feito através do path** da requisição.

A Clicksign manterá, além da versão atual, a versão anterior da sua API em
funcionamento. Quando uma nova versão é lançada, a versão anterior deixa de ser
suportada, p.e.: supondo que a versão atual seja 4, a versão 3 também será
suportada, no lançamento da versão 5, as versões suportadas serão 5 e 4 e a
versão 3 deixará de ser suportada.

Um nova versão é lançada **apenas quando há quebra de funcionalidade**. Ou seja,
melhorias, novas funcionalidades e correções de _bugs_, desde que não alterem o
comportamento esperado, não implicam no lançamento de uma nova versão.

As melhorias, noas funcionalidades e correções, assim como o lançamento de novas
versões da API, serão comunicados através de e-mail para todos os usuários que
possuem chave da API configurada nos ambientes de _production_ e _demo_.

# <a name="formatos"></a> Formatos

A API da Clicksign atualmente aceita dois formatos distintos como formato de
*resposta* da requisição, sendo eles: `JSON` e `XML`. O formato deve ser
passado através do `HTTP HEADER` `Accept` seguido do **mime type** referente ao
formato. Por exemplo, para que o formato de sua resposta seja em `JSON`, será
necessário prover um header com o seguinte formato:

```http
Accept: application/json
```

Caso deseje receber o formato da resposta em `XML` basta passar dessa forma:

```http
Accept: application/xml
```

Nos exemplos restantes dessa documentação o formato da resposta serão mostrados
em JSON, porém todos os exemplos também podem ter as respostas no formato `XML`.

# <a name="listagem-de-documentos"></a>Listagem de documentos

Você pode obter uma listagem de todos os documentos da conta além de informações extras pertinentes ao andamento da lista de assinatura. A listagem retornarár todos os documentos na conta, sem a necessidade de parâmetros de paginação ou busca.

Os atributos do documento:

- **key**: chave única dentro do sistema
- **original_name**: nome do arquivo
- **status**: estado do documento
- **archive_id**: parâmetro interno
- **created_at**: data de criação do documento
- **updated_at**: data de última alteração no documento
- **user_key**: chave do usuário que criou o documento
- **list**: lista de assinatura

O _status_ de um documento varia conforme a geração do _preview_, o estado da
lista de assinatura e as próprias assinaturas.  Segue abaixo a descrição do
_status_ e suas transições possíveis:

- **working**: gerando _preview_ do documento
- **editing**: lista de assinatura aberta para edição
- **pending**: documento aguardando assinatura do usuário
- **running**: documento aguardando assinatura
- **completed**: documento completamente assinado
- **declined**: documento assinado com aos menos uma declinação

![Diagrama de estados](https://raw.github.com/clicksign/rest-api/master/images/state-diagram.png)

Atributos da lista de assinatura:

- **locked**: parâmetro interno
- **started_at**: data de início da lista de assinatura
- **created_at**: data de criação da lista de assinatura
- **updated_at**: data de última alteração na lista de assinatura
- **user_key**: chave do usuário que criou a lista de assinatura
- **signatures**: vetor de assinaturas

Atributos da assinatura:

- **display_name**: nome do signatário
- **title**: cargo do signatário
- **company_name**: empresa do signatário
- **key**: chave do signatário
- **act**: ação da assinatura
- **decision**: estado da assinatura
- **address**: ip do signatário
- **email**: e-mail do signatário
- **signed_at**: data da assinatura
- **created_at**: data de criação da assinatura
- **updated_at**: data de última alteração na assinatura

Os possíveis valores para **decision** são:

- **agreed**: assinatura aceita
- **declined**: assinatura declinada
- **null**: assinatura não realizada

Segue abaixo a requisição para uma listagem de documentos.

* **Method:** GET
* **Path:** /v1/documents
  - **Accept**: application/json

## Exemplo de resposta

```http
HTTP/1.1 200 OK
Content-Type:application/json
Connection: Keep-Alive
```

```json
[
  {
    "document": {
      "key": "1123-4567-89ab-cdef",
      "original_name": "document-2.pdf"
      "status": "completed",
      "archive_id": 2,
      "created_at": "2014-06-18T09:55:16.873-03:00",
      "updated_at": "2014-06-18T10:02:03.056-03:00",
      "user_key":"A0BF-848B-0A42-916C",
      "list": {
        "locked":null,
        "started_at":null,
        "created_at":"2014-06-18T09:57:14.434-03:00",
        "updated_at":"2014-06-18T09:57:14.434-03:00",
        "user_key":"A0BF-848B-0A42-916C",
        "signatures": []
      }
    }
  },
  {
    "document": {
      "key": "0123-4567-89ab-cdef",
      "original_name": "document-1.pdf"
      "status": "completed",
      "archive_id": 1,
      "created_at": "2014-06-18T09:55:16.873-03:00",
      "updated_at": "2014-06-18T10:02:03.056-03:00",
      "user_key":"A0BF-848B-0A42-916C",
      "list": {
        "locked":null,
        "started_at":"2014-06-18T09:58:46.450-03:00",
        "created_at":"2014-06-18T09:57:14.434-03:00",
        "updated_at":"2014-06-18T09:58:46.452-03:00",
        "user_key":"A0BF-848B-0A42-916C",
        "signatures": [
          {
            "display_name":"Daniel Gaboardi Libanori",
            "title":null,
            "company_name":null,
            "key":"A0BF-848B-0A42-916C",
            "act":"receipt",
            "decision":"agreed",
            "address":"201.81.113.174",
            "email":"daniel.libanori@clicksign.com",
            "signed_at":"2014-06-18T09:59:58.713-03:00",
            "created_at":"2014-06-18T09:57:24.191-03:00",
            "updated_at":"2014-06-18T09:59:58.836-03:00"
          },
          {
            "display_name":"Daniel Gaboardi Libanori",
            "title":null,
            "company_name":null,
            "key":"A0BF-848B-0A42-916C",
            "act":"intervening",
            "decision":"agreed",
            "address":"201.81.113.174",
            "email":"daniel.libanori@clicksign.com",
            "signed_at":"2014-06-18T10:01:18.327-03:00",
            "created_at":"2014-06-18T09:57:31.114-03:00",
            "updated_at":"2014-06-18T10:01:18.389-03:00"
          }
        ]
      }
    }
  }
]
```

## Resposta 4XX

Caso o cliente utilize parâmetros inválidos, o corpo da resposta será um _JSON_ contendo uma mensagem de erro.

* **Cabeçalhos**:
  - **Content-Type:** application/json
* **Corpo:**

```json
{ "message": "Invalid parameters." }
```

## Resposta 5XX

Caso ocorra qualquer tipo de falha no servidor, o corpo da resposta será um _JSON_ contendo uma mensagem de erro.

* **Cabeçalhos**:
  - **Content-Type:** application/json
* **Corpo:**

```json
{ "message": "Server error." }
```

# <a name="visualizacao-de-documento"></a>Visualização de documentos

Caso seja necessário obter detalhes sobre um documento em específico, existe um endpoint onde é possível
obter essas informações. A visualização de um documento funciona de forma
semelhante as demais chamadas da api, sendo necessário apenas passar a **key**
do documento.

* **Method:** GET
* **Path:** /v1/documents/:key
  - **Accept**: application/json

## Exemplo de resposta

```http
HTTP/1.1 200 OK
Content-Type:application/json
Connection: Keep-Alive
```

```json
{
  "document": {
    "key": "1123-4567-89ab-cdef",
    original_name: "document-2.pdf"
    "status": "completed",
    "archive_id": 2,
    "created_at": "2014-06-18T09:55:16.873-03:00",
    "updated_at": "2014-06-18T10:02:03.056-03:00",
    "user_key":"A0BF-848B-0A42-916C",
    "list": {
      "locked":null,
      "started_at":null,
      "created_at":"2014-06-18T09:57:14.434-03:00",
      "updated_at":"2014-06-18T09:57:14.434-03:00",
      "user_key":"A0BF-848B-0A42-916C",
      "signatures": []
    }
  }
}
```

## Resposta 4XX

Caso o cliente utilize parâmetros inválidos, o corpo da resposta será um _JSON_ contendo uma mensagem de erro.

* **Cabeçalhos**:
  - **Content-Type:** application/json
* **Corpo:**

```json
{ "message": "Invalid parameters." }
```

## Resposta 5XX

Caso ocorra qualquer tipo de falha no servidor, o corpo da resposta será um _JSON_ contendo uma mensagem de erro.

* **Cabeçalhos**:
  - **Content-Type:** application/json
* **Corpo:**

```json
{ "message": "Server error." }
```

# <a name="upload-de-documentos"></a>Upload de documentos

O processo de envio de um documento para a Clicksign contempla a criação de um arquivo de **log** contendo informações de _upload_, usuário, etc, anexado a uma cópia do documento "carimbada" com um **número de série**. Ao final do processo haverá 2 arquivos na Clicksign: documento original e arquivo de log. Enquanto o arquivo é processado a requisição *não fica bloqueada*. O _status_ do documento será _working_ enquanto o processo ocorre. Após concluído, o _status_ será _open_.

O nome do parâmetro do documento a ser enviado é **document[archive][original]**. O parâmetro geralmente vai dentro do corpo (_payload_) devido a requisição ser _multipart_.

* **Method:** POST
* **Path:** /v1/documents
  - **Accept**: application/json
  - **Content-Type:** multipart/form-data; boundary=----WebKitFormBoundaryjm7rLhiPSO6cEjWs
* **Corpo:**
  ```
  ------WebKitFormBoundaryjm7rLhiPSO6cEjWs
  Content-Disposition: form-data; name="document[archive][original]"; filename="06pages.pdf"
  Content-Type: application/pdf

  ------WebKitFormBoundaryjm7rLhiPSO6cEjWs--
  ```


## Exemplo de resposta

```http
HTTP/1.1 200 OK
Content-Type:application/json
Connection: Keep-Alive
```

```json
{
  "key": "0123-4567-89ab-cdef",
  "status": "working"
}
```

## Resposta 4XX

Caso o cliente utilize parâmetros inválidos, o corpo da resposta será um _JSON_ contendo uma mensagem de erro.

* **Cabeçalhos**:
  - **Content-Type:** application/json
* **Corpo:**

```json
{ "message": "Invalid parameters." }
```

## Resposta 5XX

Caso ocorra qualquer tipo de falha no servidor, o corpo da resposta será um _JSON_ contendo uma mensagem de erro.

* **Cabeçalhos**:
  - **Content-Type:** application/json
* **Corpo:**

```json
{ "message": "Server error." }
```


# <a name="criacao-de-lista-de-assinatura"></a>Criação de lista de assinatura

É possível criar uma lista de assinatura e enviá-la a outras pessoas em uma única ação.  Para isso, é necessário que estejam presentes os campos que especificam o documento, os signatários, a mensagem e opcionalmente se deseja enviar e-mail para os signatários ou não.

* **Method:** POST
* **Path:** /v1/documents/:key/list
* **Cabeçalhos:**
  - **Content-Type:** application/json
  - **Accept**: application/json
* **Corpo:**

  ```json
  {
    "signers": [
      { "email": "foo@example.com", "act": "sign" },
      { "email": "bar@example.com", "act": "witness" }
    ],

    "message": "Hi guys, please sign this document.",
    "skip_email": false
  }
  ```

Para criar uma lista de assinatura, adicionar signatários ao documento e iniciar o processo de assinatura automaticamente, deve-se adicionar um campo `signers` ao JSON.  Caso não haja o campo `signers` ou ele seja `null`, o documento não possuirá lista de assinatura definida.

O campo `signers` deverá ser um `Array` contendo os signatários. Cada signatário é especificado através de e-mail e ação, sendo os respectivos campos `email` e `act`.

Os possíveis valores de `act` são:
- sign
- approve
- acknowledge
- witness
- intervenient
- party
- receipt

A mensagem a ser enviada aos signatários é definida pelo campo `message`.

Além disso também é possível especificar se deseja ao iniciar a lista de assinatura que seja enviado um e-mail para os signatários ou não, isso é feito através do parametro ```skip_email``` que recebe um boolean.

Os possíveis valores de ```skip_email``` são:
- false (padrão)
- true

Caso o parametro seja passado como **true** ao criar a lista de assinatura não será enviado nenhum e-mail para os signatários. É importante notar
que caso seja fornecido o parametro ```skip_email```` como *true*, o parametro `message` se torna desnecessário dado que não será enviado nenhum e-mail. Também
é importante observar que como o valor padrão desse parametro é `false` ele pode ser omitido do json que é enviado para o servidor caso você deseje enviar os e-mails normalmente.


# <a name="upload-e-criacao-de-lista-de-assinatura"></a>Upload e criação de lista de assinatura

É possível realizar o _upload_ e a criação da lista de assinatura em uma única ação.  Basta enviar, além do documento, os parâmetros da lista de assinatura.  Todas as opções válidas em ambas chamadas continuam válidas aqui.  O retorno será conforme a chamada de _upload_ de documento.

* **Method:** POST
* **Path:** /v1/documents
  - **Accept**: application/json
  - **Content-Type:** multipart/form-data; boundary=700481
* **Corpo:**
  ```
  --700481
  Content-Disposition: form-data; name="document[archive][original]"; filename="06pages.pdf"
  Content-Type: application/pdf
  ...

  --700481
  Content-Disposition: form-data; name="signers[][email]"

  elefanta@example.org
  --700481
  Content-Disposition: form-data; name="signers[][act]"

  sign
  --700481
  Content-Disposition: form-data; name="signers[][email]"

  elefante@example.org
  --700481
  Content-Disposition: form-data; name="signers[][act]"

  sign
  --700481
  Content-Disposition: form-data; name="message"

  hello world
  --700481
  Content-Disposition: form-data; name="access_token"

  d76a22b82a7d0807d406fa64231fae97
  --700481--
  ```


# <a name="download-de-documento"></a>Download de documento

Você pode realizar download de um arquivo.  O arquivo original empacotado com um arquivo de log e enviado em **formato ZIP**.  O empacotamento é realizado sob demanda e em _background_, para isso, o servidor não irá aguardar a conclusão do empacotamento para retornar a requisição, ao invés disso, ele irá retorna uma respota com _status_ 202 indicando o início do empacotamento.  Enquanto ocorre o empacotamento o servidor irá retornar a mesma resposta 202.  Quando o processo for concluído, a resposta será código 200 contendo em seu corpo o ZIP.

Você deverá realizar uma espera ativa, efetuando diversas requisições e obtendo respostas 202 até obter um código 200.  O empacotamento ocorre em 2 ou 3 segundos no máximo, por isso, **recomendamos realizar uma requisição por segundo** até você obter um código 200, dessa forma em 2 ou 3 requisições você obterá o ZIP.

* **Method:** GET
* **Path:** /v1/documents/:key/download
* **Cabeçalhos:**
  - **Accept**: application/json


# <a name="criacao-de-pacote-de-documentos"></a>Criação de pacote de documentos

Você pode criar um pacote de documentos caso você deseja possibilitar ao signatário que ele assine todos os documentos em uma única etapa (_feature_ atualmente disponível apenas via _widget_).  Para isto, é necessário que os documentos sejam _empacotados_ em uma estrutura.  A criação de um pacote é extremamente simples, basta fazer uma requisição com o _array_ de documentos que devam compor o pacote.

* **Method:** POST
* **Path:** /v1/batches
* **Cabeçalhos:**
  - **Content-Type:** application/json
  - **Accept**: application/json
* **Corpo:**

  ```json
  {
    "keys": [
      "0b2ec469-6b49-42bb-809f-d978279baeeb",
      "db75a09b-6b30-48b9-b9f0-873951a050ed",
      "42334730-5a93-48ae-b5e4-e8e86d62610a"
    ]
  }
  ```


# <a name="hooks"></a>Hooks

É possível que a Clicksign notifique outras aplições à respeito da alteração de estado de um determinado documento. O estado de um documento é alterado quando um dos seguintes eventos ocorrem:

- o documento é processado
- alguém se recusa a assinar o documento
- o documento é completamente assinado

Para isso, a Clicksign dispõe de um sistema de **hooks** que realizam chamadas HTTP para outras aplicações. As _hooks_ são definidas por documento, portanto cada documento deve configurar as _hooks_ com os parâmetros que deseja.

Quando o documento alterar o seu estado, a Clicksign irá realizar um POST para a `url` que foi configurada na _hook_ do documento. No corpo da requisição irá em anexo a chave do documento em formato JSON. Portanto você pode determinar para cada documento um endereço específico a ser notificado ou colocar em todos os documentos o mesmo endereço a ser notificado e o servidor que atender a requisição inspecionar o JSON e determinar o que fazer com cada documento em particular.

- **URL**: caminho completo, incluíndo protocolo
- **Content-Type**: application/json

## Configuração de hooks

Cadastra um _hook_ para um determinado usuário.

* **Method:** POST
* **Path:** /v1/documents/:key/hooks
* **Corpo:**
  ```json
  {
    "url": "https://example.com/signed/123",
  }
  ```

## Resposta 200

Caso não ocorra nenhuma falha na requisição, a resposta será status 200 e seu corpo será um JSON contendo os dados da _hook_ recém criada.

```http
HTTP/1.1 200 OK
Content-Type:application/json
```

```json
{
  "id":1,
  "url":"http://example.com",
  "document_id": 1,
  "created_at":"2014-07-08T12:35:55.777-03:00",
  "updated_at":"2014-07-08T12:35:55.777-03:00"
}
```

## Resposta 4XX

Caso o cliente utilize parâmetros inválidos, o corpo da resposta será um _JSON_ contendo uma mensagem de erro.

* **Cabeçalhos**:
  - **Content-Type:** application/json
* **Corpo:**

  ```json
  {
    "message": "Parâmetros inválidos."
  }
  ```

## Resposta 5XX

Caso ocorra qualquer tipo de falha no servidor, o corpo da resposta será um _JSON_ contendo uma mensagem de erro.

* **Cabeçalhos**:
  - **Content-Type:** application/json
* **Corpo:**

  ```json
  {
    "message": "Server error."
  }
  ```

# <a name="exemplos"></a>Exemplos

Os exemplos abaixo podem ser executados direto da linha de comando utilizando o cURL.

```bash
export DOMAIN=api.clicksign.com
export TOKEN=put-your-token-here
export DOCUMENT=/home/joe/document-de-exemplo.pdf

# Obter os documentos de uma determinada conta
curl -X GET -H "Accept: application/json" https://$DOMAIN/v1/documents/?access_token=$TOKEN

# Realizar upload de um documento
curl -X POST -H "Accept: application/json" -F "document[archive][original]=@$DOCUMENT" https://$DOMAIN/v1/documents?access_token=$TOKEN

export KEY=ver-key-retornada-no-JSON

# Criar uma lista de assinatura
curl -X POST -H "Accept: application/json" -H "Content-type: application/json" -d '{"signers": [{ "email": "joe@example.com", "act": "sign" }]}' https://$DOMAIN/v1/documents/$KEY/list?access_token=$TOKEN
```
