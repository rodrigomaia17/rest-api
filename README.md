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

A resposta, em sua linha inicial, indica a **versão do protocolo**, o **status**, e contém uma **mensagem informativa**. O código de status é essencial para o cliente saber se a ação foi devidamente executada no servidor.

A documentação de cada função da API determina o método e o caminho a ser utilizado, e o significado do corpo e de cada status da resposta.

## Exemplo de requisição

```http
GET /documents HTTP/1.1
Host: desk.clicksign.com
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

<!DOCTYPE html>
<html>
  <head>
...
```

- Versão: 1.1
- Status: 200
- Mensagem: OK
- Cabeçalhos: Content-Type, Connection
- Corpo: <!DOCTYPE html>...

# Autenticação

A autenticação é feita através de 2 parâmetros: **api_key** e **api_secret**. O parâmetro `api_key` define qual cliente está fazendo a requisição. O parâmetro `api_secret` define o senha que será utilizada na verificação de acesso à API. Ambos os parâmetros devem ser enviados no **caminho** da requisição. Portanto, toda requisição deverá conter no _path_ `?api_key=string-da-key&api_secret=string-do-secret`.

**Atenção:** Os parâmetros de autenticação devem ser enviados a cada requisição feita pelo cliente. Como esses parâmetros são comuns a todos as funções da API, eles serão omitidos das documentações.


# Criação de usuários corporativos

* **Method:** POST
* **Path:** /registration
* **Cabeçalhos:**
  - **Content-Type:** application/json
  - **Accept**: application/json
* **Corpo:**
  ```json
    {
      "person": {
        "name": {
          "given_name": "John",
          "additional_name": "August",
          "family_name": "Doe",
          "honorific_suffix": "III"
        },

        "documentation": {
          "country": "br",
          "kind": "cpf",
          "value": "999.999.999-99"
        },

        "phone": {
          "country": "br",
          "number": "99-9-9999-9999"
        }
      }
    }
  ```

Para especificar o pré-cadastro de um usuário a requisição json deve seguir o formato especificado acima.
Alguns campos merecem informações extras

<table>
  <thead>
    <tr>
      <th>Campo</th>
      <th>Tipo</th>
      <th>Obrigatório</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>person.name.given_name</td>
      <td>string</td>
      <td>x</td>
    </tr>

    <tr>
      <td>person.name.family_name</td>
      <td>string</td>
      <td>x</td>
    </tr>

    <tr>
      <td></td>
      <td></td>
      <td></td>
    </tr>

    <tr>
      <td></td>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>

O campo `person.name.given_name` é obrigatório

O campo `person.name.family_name` é obrigatório

O campo `person.documentation.country` é obrigatório

O campo `person.documentation.kind` é obrigatório

O campo `person.documentation.value` é obrigatório

O campo `person.phone.country` é obrigatório

O campo `person.phone.number` é obrigatório

O campo `person.documentation.value` é uma `String` contendo o valor do CPF a ser cadastrado. O CPF deve atender
algum dos seguintes formatos:

- Com pontos e traço: `"999.999.999-99"`
- Somente digítos: `"99999999999"`

Em ambos os casos o valor conterá um CPF válido.

O campo `person.phone.number` é uma `String` com seu formato seguindo o seguinte padrão: _ddd_ _número celular_.
`número` um valor entre 8 e 9 digítos. Além disso o valor da string pode conter ou não traços. Os seguintes exemplos são validos:

- Com traços: `"99-9-9999-9999"`
- Somente digítos: `"99999999999"`

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
