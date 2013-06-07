# Clicksign API REST

- [Introdução](#introduo)
- [Funcionamento geral](#funcionamento-geral)
- [Autenticação](#autenticao)
- [Upload de documento](#upload-de-documento)
- [Listagem de documentos](#listagem-de-documentos)
- [Dados de um documento](#dados-de-um-documento)
- [Download de um documento](#download-de-um-documento)
- [Super envio](#super-envio)

# Introdução

A Clicksign é uma solução online para enviar, guardar e assinar documentos, com validade jurídica. Foi criada para facilitar, reduzir custo e aumentar a segurança e compliance do processo de assinatura e workflow de documentos. 

A Clicksign pode ser acessada em www.clicksign.com. 

O propósito desta **API REST** é prover meios para que nossos clientes adequem a Clicksign aos seus processos e sistemas p.ex. automatizar determinadas tarefas, desenhar fluxos de assinatura, e definir workflow. 

Qualquer linguagem de programação compativel com requisições **HTTP / JSON** cumpre os requisitos necessários para consumir os serviços desta API. Assim, com pouco esforço de programação é possível integrar desde aplicações scripts shell até sistemas de ERP.

Os exemplos contidos nessa documentação utilizam **bash** e **curl**, bem como **Javascript**. O `bash` é o _shell_ padrão de grande maioria das distribuições _Linux_. O programa `curl` é amplamente disponível em ambientes _Unix_ e sua principal utilidade é realizar requisições HTTP. A linguagem **Javascript** é utilizada nos exemplos que envolvem programação.

# Funcionamento geral

Uma API REST é composta, basicamente, por dois elementos: um **cliente** e um **servidor**. O cliente sempre inicia a comunicação mediante uma requisição HTTP. O servidor sempre a finaliza a comunicação respondendo a requisição.

As mensagens HTTP são compostas por uma linha inicial, um conjunto de cabeçalhos e um corpo. A requisições, na linha inicial, indica o **caminho**, o **verbo**, e a versão do protocolo. O caminho e o verbo são essenciais em uma API REST uma vez que ambos indicam a ação a ser executada no servidor. 


DANIEL PAREI AQUI: A resposta, em sua linha inicial, indicam a versão do protocolo, o **código de status** e uma mensagem informativa. O código de status da resposta é essencial para o cliente saber se a ação foi devidamente executada no servidor.

![requisição/resposta HTTP](https://raw.github.com/clicksign/rest-api/master/images/request_response.png)

A documentação de cada método da API determina o caminho e o verbo a ser utilizado, assim como, o que significa cada código de status da resposta. É interessante notar que algumas requisições podem contar com parâmetros codificados no caminho da requisição ou estarem presentes no corpo da requisição, o que estará especificado na documentação de cada método.


# Autenticação

A autenticação é feita através de 2 parâmentros: **api_key** e **api_token**. O parâmetro `api_key` define qual cliente está fazendo a requisição e o parâmetro `api_token` define o token que será utilizado na verificação de acesso à API. Ambos os parâmetros devem ser enviados no caminho da requisição. Portanto toda requisição deverá conter no caminho `?api_key=valor-da-key&api_token=valor-do-token`.

**Atenção:** Os parâmetros de autenticação devem ser enviados a cada requisição feita pelo cliente. Como esses parâmetros são comuns a todos os métodos da API, eles serão omitidos de suas documentações.


# Upload de documento

Envia um documento para o servidor. É feita uma cópia do documento e adicionada um **número de série** à cópia. Um **log** é gerado contendo as informações de data de _upload_, usuário e outras informações. Ao final do processo haverá 3 arquivos nos servidores da Clicksign: documento original, cópia do documento e arquivo de log. A requisição *não fica bloqueada* enquanto o documento é processado. O documento apenas estará com _status working_ enquanto o processo ocorre. Após concluído, o _status_ será _open_.

* **Método:** POST
* **Caminho:** /documents
  - **Content-Type:** multipart/mixed; boundary=frontier
  - **Accept**: application/json
* **Corpo:**
  - **Content-Type:** application/octet-stream
  - **Content-Transfer-Encoding:** base64

   ```
   --frontier--
   PGh0bWw+CiAgPGhlYWQ+CiAgPC9oZWFkPgogIDxib2R5PgogICAgPHA+VGhpcyBpcyB0aGUg
    Ym9keSBvZiB0aGUgbWVzc2FnZS48L3A+CiAgPC9ib2R5Pgo8L2h0bWw+Cg==
   --frontier--
   ```

## Resposta 200

Caso não ocorra nenhuma falha na requisição, o corpo da resposta será um _JSON_ contendo os documentos que atendem os critérios dos filtros. Os dados dos documentos serão retornados em _JSON_ sendo o elemento _root_ um `Array`.

* **Cabeçalhos**:
  - **Content-Type:** application/json
* **Corpo:**

  ```json
  {
    "document_id": "51b1eceb438ef026f91758b1",
    "created_at": "2013-04-11T13:04:32.542Z",
    "status": "working"
  }
  ```

## Resposta 4XX

Caso o cliente utilize parâmetros inválidos, o corpo da resposta será um _JSON_ contendo uma mensagem de erro.

* **Cabeçalhos**:
  - **Content-Type:** application/json
* **Corpo:**

  ```json
  { "message": "Parâmetros inválidos." }
  ```

## Resposta 5XX

Caso ocorra qualquer tipo de falha no servidor, o corpo da resposta será um _JSON_ contendo uma mensagem de erro.

* **Cabeçalhos**:
  - **Content-Type:** application/json
* **Corpo:**

  ```json
  { "message": "Ocorreu um erro no servidor" }
  ```


# Listagem de documentos

Retorna os documentos. Podem ser aplicados três tipos de filtros simultaneamente: data de criação antes de determinada data, data de criação após determinada data, estado do documento.

* **Método:** GET
* **Caminho:** /documents
* **Parâmetros opcionais**
  - **status:** working, open, locked, running, closed
  - **before:** _data_
  - **after:** _data_
* **Cabeçalhos:**
  - **Accept**: application/json
* **Corpo:** _vazio_

## Resposta 200

Caso não ocorra nenhuma falha na requisição, o corpo da resposta será um _JSON_ contendo os documentos que atendem os critérios dos filtros. Os dados dos documentos serão retornados em _JSON_ sendo o elemento _root_ um `Array`.

* **Cabeçalhos**:
  - **Content-Type:** application/json
* **Corpo:**

  ```json
  [
    {
      "document_id": "4d3ed089fb60ab534684b7e9",
      "created_at": "2013-04-11T13:04:32.542Z",
      "status": "running"
    },

    {
      "document_id": "4baa56f1230048567300485c",
      "created_at": "2013-04-22T09:01:18.312Z",
      "status": "running"
    },

    {
      "document_id": "51b1d97e25dc552297f95b97",
      "created_at": "2013-04-20T11:04:32.072Z",
      "status": "open"
    },
   ]
  ```

## Resposta 4XX

Caso o cliente utilize parâmetros inválidos, o corpo da resposta será um _JSON_ contendo uma mensagem de erro.

* **Cabeçalhos**:
  - **Content-Type:** application/json
* **Corpo:**

  ```json
  { "message": "Parâmetros inválidos." }
  ```

## Resposta 5XX

Caso ocorra qualquer tipo de falha no servidor, o corpo da resposta será um _JSON_ contendo uma mensagem de erro.

* **Cabeçalhos**:
  - **Content-Type:** application/json
* **Corpo:**

  ```json
  { "message": "Ocorreu um erro no servidor" }
  ```


# Dados de um documento

Retorna os dados de um documento. Deve ser adicionado o `document\_id` ao _path_ da requisição.

* **Método:** GET
* **Caminho:** /documents/:id
* **Cabeçalhos:**
  - **Accept**: application/json
* **Corpo:** _vazio_

## Resposta 200

Caso não ocorra nenhuma falha na requisição, o corpo da resposta será um _JSON_ contendo as informações do documento, incluindo dados da sua lista de assinatura.

* **Cabeçalhos**:
  - **Content-Type:** application/json
* **Corpo:**

  ```json
  {
    "document_id": "4d3ed089fb60ab534684b7e9",
    "created_at": "2013-05-02T14:24:38.447Z",
    "user_id": "51b1efd1438ef026f91758b2",
    "status": "running",

    "signers": [
      { "email": "foo@example.com", "action": "sign", "signed": "approved", "signed_at": "2013-05-07T14:34:36.447Z" },
      { "email": "bar@example.com", "action": "sign_as_witness", "signed": "waiting", "signed_at": null }
    ],
  }
  ```

## Resposta 4XX

Caso o cliente utilize parâmetros inválidos, o corpo da resposta será um _JSON_ contendo uma mensagem de erro.

* **Cabeçalhos**:
  - **Content-Type:** application/json
* **Corpo:**

  ```json
  { "message": "Parâmetros inválidos." }
  ```

## Resposta 5XX

Caso ocorra qualquer tipo de falha no servidor, o corpo da resposta será um _JSON_ contendo uma mensagem de erro.

* **Cabeçalhos**:
  - **Content-Type:** application/json
* **Corpo:**

  ```json
  { "message": "Ocorreu um erro no servidor" }
  ```


# Download de um documento

Retorna um arquivo _ZIP_ contendo os 3 arquivos resultantes do processamento: arquivo original, cópia do arquivo e log.

**Atenção**: A única diferença entre este método e obter os [dados de um documento](#dados-de-um-documento) é o cabeçalho **Accept**, que neste caso é _application/zip_.

* **Método:** GET
* **Caminho:** /documents/:id
* **Cabeçalhos:**
  - **Accept**: application/zip
* **Corpo:** _vazio_

## Resposta 200

Caso não ocorra nenhuma falha na requisição, o corpo da resposta será um _JSON_ contendo as informações do documento, incluindo dados da sua lista de assinatura.

* **Cabeçalhos**:
  - **Content-Type:** application/zip
* **Corpo:**

  ```
   PGh0bWw+CiAgPGhlYWQ+CiAgPC9oZWFkPgogIDxib2R5PgogICAgPHA+VGhpcyBpcyB0aGUg
   ...
   Ym9keSBvZiB0aGUgbWVzc2FnZS48L3A+CiAgPC9ib2R5Pgo8L2h0bWw+Cg==
  ```

## Resposta 4XX

Caso o cliente utilize parâmetros inválidos, o corpo da resposta será um _JSON_ contendo uma mensagem de erro.

* **Cabeçalhos**:
  - **Content-Type:** application/json
* **Corpo:**

  ```json
  { "message": "Parâmetros inválidos." }
  ```

## Resposta 5XX

Caso ocorra qualquer tipo de falha no servidor, o corpo da resposta será um _JSON_ contendo uma mensagem de erro.

* **Cabeçalhos**:
  - **Content-Type:** application/json
* **Corpo:**

  ```json
  { "message": "Ocorreu um erro no servidor" }
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


# Exemplos

```HTTP
POST /foo HTTP/1.1
Host: api.clicksign.com
Content-Type: application/json
Accept: application/json

{
  "document_id": "4d3ed089fb60ab534684b7e9",

  "signers": [
    { "email": "foo@example.com", "action": "sign" },
    { "email": "bar@example.com", "action": "sign_as_witness" }
  ],

  "message": {
    "recipients": [ "foo@example.com", "bar@example.com" ],
    "body": "Hi guys, please sign this documents."
  }
}
```
