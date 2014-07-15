# Índice

- [Introdução](#introducao)
- [Funcionamento geral](#funcionamento-geral)
- [Autenticação](#autenticacao)
- [Versão](#versao)
- [Upload de documentos](#upload-de-documentos)
- [Criação de lista de assinatura](#criacao-de-lista-de-assinatura)
- [Hooks](#hooks)
- [Exemplos](#exemplos)

# <a name="introducao"></a>Introdução

A Clicksign é uma solução online para enviar, guardar e assinar documentos, com validade jurídica. Foi criada para facilitar, reduzir custo e aumentar a segurança e compliance do processo de assinatura e _workflow_ de documentos. 

A Clicksign pode ser acessada em https://desk.clicksign.com. 

O propósito desta **REST API** é prover meios para que nossos clientes adequem a Clicksign aos seus processos e sistemas p. ex. automatizar tarefas, desenhar fluxos de assinatura, e definir _workflow_. 

Qualquer linguagem de programação compativel com requisições **HTTP / JSON** cumpre os requisitos necessários para consumir os serviços desta API. Assim, com pouco esforço de programação é possível integrar desde scripts shell até sistemas de ERP.

# <a name="funcionamento-geral"></a>Funcionamento geral

Uma _REST API_ é composta, basicamente, por dois elementos: um **cliente** e um **servidor**. O cliente sempre inicia a comunicação mediante requisição HTTP. O servidor sempre finaliza a comunicação respondendo à requisição.

<p align="center">
  <img src="https://raw.github.com/clicksign/rest-api-v2/master/images/client-server.png" />
</p>

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
- Corpo: [{...


# <a name="autenticacao"></a>Autenticação

A Clicksign utiliza duplo fator de autenticação para aumentar a segurança de suas transações. Autenticações que utilizam duplo fator geralmente são baseadas em algo que o cliente _conhece_ e algo que o cliente _possui_. No caso da API os fatores são:

1. Conhecer uma _string_ de **identificação**
1. Possuir um endereço **IP** específico

A autenticação é feito através do parâmetro **access_token** que automaticamente determina um usuário e realiza sua autenticação. O parâmetro deve ser enviado no **caminho** da requisição. Portanto, toda requisição deverá conter no _path_ `?access_token=string-do-token`._

**Atenção:** O parâmetro de autenticação deve ser enviado a cada requisição feita pelo cliente. Como esse parâmetro é comum a todas as funções da API, ele será omitido das documentações.

O segundo fator da autenticação é realizado automaticamente pelo servidor da Clicksign, que verifica se o **IP** de origem da requisição está dentro de uma lista de endereços previamente cadastrados para determinado cliente. Este fator de autenticação é **opcional**.


# <a name="versao"></a>Versão

Para possibilitar a expansão contínua da API, a Clicksign implementa um sistema de versões. Dessa forma é necessário que a requisição contenha qual versão da API está sendo utilizada. Isto é feito através do cabeçalho `Accept` que deverá possuir o valor `application/vnd.clicksign.v1`. Caso haja mais de um valor para o cabeçalho `Accept`, eles deverão ser concatenados utilizando `;`, p.e.: `Accept: application/vnd.clicksign.v1; application/json`.

Atualmente a Clicksign possui apenas este cabeçalho para versões, mas a medida que outras versões forem implementadas, outros valores serão possíveis.


# <a name="upload-de-documentos"></a>Upload de documentos

O processo de envio de um documento para a Clicksign contempla a criação de um arquivo de **log** contendo informações de _upload_, usuário, etc, anexado a uma cópia do documento "carimbada" com um **número de série**. Ao final do processo haverá 2 arquivos na Clicksign: documento original e arquivo de log. Enquanto o arquivo é processado a requisição *não fica bloqueada*. O _status_ do documento será _working_ enquanto o processo ocorre. Após concluído, o _status_ será _open_.

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

É possível criar uma lista de assinatura e enviá-la a outras pessoas em uma única ação. Para isso, é necessário que estejam presentes os campos que especificam o documento, os signatários, e a mensagem.

* **Method:** POST
* **Path:** /documents/:key/list
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

    "message": "Hi guys, please sign this document."
  }
  ```

Para criar uma lista de assinatura, adicionar signatários ao documento e iniciar o processo de assinatura automaticamente, deve-se adicionar um campo `signers` ao JSON. Caso não haja o campo `signers` ou ele seja `null`, o documento não possuirá lista de assinatura definida.

O campo `signers` deverá ser um `Array` contendo os signatários. Cada signatário é especificado através de e-mail e ação, sendo os respectivos campos `email` e `act`.

Os possíveis campos de `act` são:
- sign
- approve
- acknowledge
- witness
- intervenient
- party
- receipt

A mensagem a ser enviada aos signatários é definida pelo campo `message`.


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
* **Path:** /documents/:key/hooks
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
export TOKEN=put-your-token-here
export DOCUMENT=/home/joe/document-de-exemplo.pdf

# Obter os documentos de uma determinada conta
curl -X GET -H "Accept: application/vnd.clicksign.v1" https://api.clicksign.com/documents/?access_token=$TOKEN

# Realizar upload de um documento
curl -X POST -H "Accept: application/vnd.clicksign.v1" -F "document[archive][original]=@$DOCUMENT" https://api.clicksign.com/documents?access_token=$TOKEN

export KET=ver-key-retornada no JSON

# Criar uma lista de assinatura
curl -X POST -H "Accept: application/vnd.clicksign.v1" -H "Accept: application/json" -H "Content-type: application/json" -d '{"signers": [{ "email": "joe@example.com", "act": "sign" }]}' https://api.clicksign.com/documents/$KEY/list?access_token=$TOKEN
```
