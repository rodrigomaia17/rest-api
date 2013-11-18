# Índice

- [Introdução](#introduo)
- [Funcionamento geral](#funcionamento-geral)
- [Autenticação](#autenticao)
- [Upload de documento](#upload-de-documento)
- [Listagem de documentos](#listagem-de-documentos)
- [Dados de um documento](#dados-de-um-documento)
- [Download de um documento](#download-de-um-documento)
- [Criação de lista de assinatura](#criao-de-lista-de-assinatura)

# Introdução

A Clicksign é uma solução online para enviar, guardar e assinar documentos, com validade jurídica. Foi criada para facilitar, reduzir custo e aumentar a segurança e compliance do processo de assinatura e _workflow_ de documentos. 

A Clicksign pode ser acessada em desk.clicksign.com. 

O propósito desta **REST API** é prover meios para que nossos clientes adequem a Clicksign aos seus processos e sistemas p. ex. automatizar tarefas, desenhar fluxos de assinatura, e definir _workflow_. 

Qualquer linguagem de programação compativel com requisições **HTTP / JSON** cumpre os requisitos necessários para consumir os serviços desta API. Assim, com pouco esforço de programação é possível integrar desde scripts shell até sistemas de ERP.

# Funcionamento geral

Uma _REST API_ é composta, basicamente, por dois elementos: um **cliente** e um **servidor**. O cliente sempre inicia a comunicação mediante requisição HTTP. O servidor sempre finaliza a comunicação respondendo à requisição.

As mensagens HTTP são compostas por uma linha inicial, um conjunto de cabeçalhos e um corpo. A linha inicial difere nas requisições e nas respostas, o cabeçalho compartilha parâmetros em comum e parâmetros específicos, e o corpo é completamente dependente de cada mensagem, podendo até ser nulo.

A requisição, em sua linha inicial, indica o **método**, o **caminho**, e a **versão do protocolo**. O método e o caminho são essenciais em uma _REST API_ uma vez que ambos indicam a ação a ser executada no servidor.

A resposta, em sua linha inicial, indica a **versão do protocolo**, o **código de status**, e contém uma mensagem informativa. O código de status é essencial para o cliente saber se a ação foi devidamente executada no servidor.

Exemplo de requisição
```http
GET /documents HTTP/1.1
Host: desk.clicksign.com
Accept: application/json
```

Exemplo de resposta
```http
HTTP/1.1 200 OK
Content-Type:application/json
Connection: Keep-Alive

<!DOCTYPE html>
<html>
  <head>
...
```

A documentação de cada função da API determina o _method_ e o _path_ e a ser utilizado, e o significado de cada código de status da resposta.


# Autenticação

A autenticação é feita através de 2 parâmetros: **api_key** e **api_token**. O parâmetro `api_key` define qual cliente está fazendo a requisição. O parâmetro `api_token` define o token que será utilizado na verificação de acesso à API. Ambos os parâmetros devem ser enviados no _path_ da requisição. Portanto, toda requisição deverá conter no _path_ `?api_key=valor-da-key&api_token=valor-do-token`.

**Atenção:** Os parâmetros de autenticação devem ser enviados a cada requisição feita pelo cliente. Como esses parâmetros são comuns a todos as funções da API, eles serão omitidos das documentações.


# Upload de documento

O processo de envio de um documento para o servidor contempla (i) a criação de uma cópia do documento, "carimbada" com um **número de série**, e (ii) a geração de um **log** contendo informações de _upload_, usuário, etc. Ao final do processo haverá 3 arquivos nos servidores da Clicksign: documento original, cópia carimbada do documento, e arquivo de log. A requisição *não fica bloqueada* enquanto o documento é processado. O _status_ do documento será _working_ enquanto o processo ocorre. Após concluído, o _status_ será _open_.

* **Method:** POST
* **Path:** /documents
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


# Listagem de documentos

Podem ser aplicados, simultaneamente, três tipos de filtros à listagem de documentos: data de criação antes de determinada data; data de criação após determinada data; estado do documento.

* **Method:** GET
* **Path:** /documents
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


# Dados de um documento

Informações detalhadas sobre um documento. O `document_id` é necessário no _path_ da requisição.

* **Method:** GET
* **Path:** /documents/:id
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
      ]
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


# Download de um documento

Retorna um arquivo _ZIP_ contendo os 3 arquivos resultantes do processamento: arquivo original, cópia carimbada do arquivo e log.

**Atenção**: A única diferença entre esta função e função [dados de um documento](#dados-de-um-documento) é o cabeçalho **Accept**, que neste caso é _application/zip_.

* **Method:** GET
* **Path:** /documents/:id
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
    { "message": "Server error." }
  ```


# Criação de lista de assinatura

É possível criar uma lista de assinatura e enviá-la a outras pessoas em uma única ação. Para isso, é necessário que estejam presentes os campos que especificam o documento, os signatários, e a mensagem.

* **Method:** POST
* **Path:** /documents/:id/signature_list
* **Cabeçalhos:**
  - **Content-Type:** application/json
  - **Accept**: application/json
* **Corpo:**
  ```json
    {
      "signers": [
        { "email": "foo@example.com", "action": "sign" },
        { "email": "bar@example.com", "action": "sign_as_witness" }
      ],

      "message": {
        "recipients": [ "foo@example.com", "bar@example.com" ],
        "body": "Hi guys, please sign this document."
      }
    }
  ```

## Documento

O documento a ser enviado é determinado pelo campo `document_id`. Cabe ressaltar que o `document_id` é diferente para cada usuário que possui uma cópia do arquivo, portanto um mesmo arquivo possui múltiplos `document_id`, sendo um para cada usuário que tem acesso.

```json
  { "document_id": "4d3ed089fb60ab534684b7e9" }
```

## Signatários

Para criar uma lista de assinatura, adicionar signatários ao documento e iniciar o processo de assinatura automaticamente, deve-se adicionar um campo `signers` ao JSON. Caso não haja o campo `signers` ou ele seja `null`, o documento não possuirá lista de assinatura definida.

O campo `signers` deverá ser um `Array` contendo os signatários. Cada signatário é especificado através de e-mail e ação, sendo os respectivos campos `email` e `action`.

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
      "body": "Hi guys, please sign this document."
    }
  }
```
