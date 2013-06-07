# Clicksign API REST

- [Introdução](#introduo)
- [Funcionamento geral](#funcionamento-geral)
- [Autenticação](#autenticao)
- [Listagem de documentos](#listagem-de-documentos)
- [Super envio](#super-envio)

# Introdução

A Clicksign é uma solução online para enviar, guardar e assinar documentos, com validade jurídica. Foi criada para facilitar, reduzir custo e aumentar a segurança e compliance do processo de assinatura e workflow de documentos. Os documentos podem ser carregados, enviados e assinados pelo nosso site www.clicksign.com.

Apesar disso, sabemos que muitos de nossos clientes possuem fluxos próprios de assinatura e/ou desejam automatizar determinadas tarefas. A Clicksign possui uma **API REST**, o que significa que qualquer linguagem de programação que possa realizar requisições HTTP cumpre os requisitos necessários para consumir os serviços da API. Desde aplicações scripts shell até sistemas de ERP podem integrar com esforço mínimo de programação.

Os exemplos construídos nessa documentação utilizam **bash** e **curl**. O `bash` é o _shell_ padrão da grande maioria das distribuições _Linux_. O programa `curl` é amplamente disponível em ambientes _Unix_ e sua principal utilidade é realizar requisições HTTP. Os exemplos que envolvem programação utilizam a linguagem **Javascript** pelo fato de ser amplamente conhecida, além de ilustrar a versatilidade da API.

# Funcionamento geral

Uma API REST é elementarmente composta de dois elementos: um **cliente** e um **servidor**. O cliente sempre inicializa a comunicação através de uma requisição HTTP e o servidor sempre a finaliza respondendo.

Há dois tipos de mensagens HTTP: requisições e respostas, que estarão associadas respectivamente ao cliente e ao servidor. As mensagens HTTP são compostas de uma linha inicial, um conjunto de cabeçalhos e um corpo. As requisições, em sua linha inicial, indicam o **verbo**, o **caminho** e a versão do protocolo. O verbo e o caminho são essenciais em uma API REST, pois o conjunto dos dois é que indicará a ação a ser executada no servidor. As respostas, em sua linha inicial, indicam a versão do protocolo, o **código de status** e uma mensagem informativa. O código de status da resposta é essencial para o cliente saber se a ação foi devidamente executada no servidor.

![requisição/resposta HTTP](https://raw.github.com/clicksign/rest-api/master/images/request_response.png)

A documentação de cada método da API determina o caminho e o verbo a ser utilizado, assim como, o que significa cada código de status da resposta. É interessante notar que algumas requisições podem contar com parâmetros codificados no caminho da requisição ou estarem presentes no corpo da requisição, o que estará especificado na documentação de cada método.

# Autenticação

O protocolo HTTP é um protocolo _stateless_, o que significa que o protocolo não oferece armazenamento do estado entre as requisições, conhecido como **sessão**. Aplicações Web geralmente utilizam _cookies_ para fazer o controle de sessão, porém isto pode trazer complicações no uso de uma API REST entre servidores, por esse motivo foi definido que não haverá abertura de sessão nem utilização de cookies, ao invés disso, a autenticação será feita por parâmetros no _path_.

**Atenção:** os parâmetros de autenticação devem ser enviados a cada requisição feita pelo cliente.

A autenticação é feita através de 2 parâmentros: **api_key** e **api_token**. O parâmetro `api_key` define qual o cliente que está fazendo a requisição e o parâmetro `api_token` define o token que será utilizado na verificação de acesso à API. Ambos parâmetros devem ser enviados no caminho da requisição. Portanto toda requisição deverá conter no caminho `?api_key=valor-da-key&api_token=valor-do-token`.

**Atenção:** Como esses parâmetros são comuns a todos os métodos da API, eles serão omitidos de suas documentações.

# Listagem de documentos

Retorna os documentos. Podem ser aplicados três tipos de filtros simultaneamente: data de criação antes de determinada data, data de criação após determinada data, estado do documento. Os dados dos documentos serão retornados em _JSON_ sendo o elemento _root_ um `Array`.

## Requisição

* **Método:** GET
* **Caminho:** /documents
* **Parâmetros opcionais**
  - **status:** open, locked, running, closed
  - **before:** _data_
  - **after:** _data_
* **Cabeçalhos:**
  - **Accept**: application/json
* **Corpo:** _vazio_

## Resposta 200

Caso não ocorra nenhuma falha na requisição, o corpo da resposta será um _JSON_ contendo os documentos que atendem os critérios dos filtros.

* **Cabeçalhos**:
  - **Content-Type:** application/json
* **Corpo:**
  ```json
  [
    {
      "document_id": "4d3ed089fb60ab534684b7e9",
      "created_at": "2013-04-11T13:04:32.542Z",
      "status": "running",
    },

    {
      "document_id": "4baa56f1230048567300485c",
      "created_at": "2013-04-22T09:01:18.312Z",
      "status": "running",
    },

    {
      "document_id": "51b1d97e25dc552297f95b97",
      "created_at": "2013-04-20T11:04:32.072Z",
      "status": "open",
    },
   ]
  ```


## Resposta 4XX

Caso o cliente utilize parâmetros inválidos, o corpo da resposta será um _JSON_ contendo uma mensagem erro.

* **Cabeçalhos**:
  - **Content-Type:** application/json
  - **Content-Type:** application/json
* **Corpo:**
  ```json
    {
      "message": "Parâmetros inválidos."
    }
  ```

# Super envio

É possível criar uma lista de assinatura e envia-la a outros pessoas em uma única ação. Para isso, é necessário que estejam presentes os seguintes campos que especificam o documento, os signatários e a mensagem.

## Documento

O documento a ser enviado é determinado pelo campo `document\_id`. Cabe ressaltar que o `document\_id` é diferente para cada usuário que possui uma cópia do arquivo, portanto um mesmo arquivo possui múltiplos `document\_id`, sendo um para cada usuário que tem acesso a ele.

```json
{
  "document_id": "4d3ed089fb60ab534684b7e9"
}
```

## Signatários

Caso seja deseje-se criar uma lista de assinatura, adicionar signatários ao documento e iniciar o processo de assinatura automaticamente, deve-se adicionar um campo `signers` ao JSON. Caso não haja o campo `signers` ou ele seja `null`, o documento não possuirá lista de assinatura definida.

O campo `signers` que deverá ser um `Array` contendo os signatários. Cada signatário é especificados através de e-mail e ação, sendo os respectivos campos `email` e `action`.

Os possíveis campos de `action` são:
- sign
- approve
- sign_as_party
- sign_as_witness
- sign_as_intervenient

```json
{
  "signers": [
    { "email": "foo@example.com", "action": "sign" },
    { "email": "bar@example.com", "action": "sign_as_witness" }
  ]
}
```


## Mensagem

Para especificar a mensagem a ser enviada são necessários dois campos: `recipients` e `body`.

O campo `recipients` é um `Array` obrigatório com tamanho mínimo de `1`. Nenhuma mensagem será enviada caso o campo `recipients` não exista, seja `null` ou tenha tamanho igual a `0`.

O campo `body` especifica o corpo da mensagem, é opcional e caso presente deve ser do tipo `String`.

```json
{
  "message": {
    "recipients": [ "foo@example.com", "bar@example.com" ],
    "body": "Hi guys, please sign this documents."
  }
}
```

## Exemplos

```HTTP
POST /foo HTTP/1.1
Host: api.clicksign.com
Content-Type: application/json
Accept: application/json

{
  "document_id": "4d3ed089fb60ab534684b7e9",

  "signers": [
    { "email": "foo@example.com", "action": "sign" },
    { "email": "bar@example.com", "action": "sign_as_witness" },
  ],

  "message": {
    "recipients": [ "foo@example.com", "bar@example.com" ],
    "body": "Hi guys, please sign this documents."
  }
}
```
