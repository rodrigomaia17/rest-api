# Índice

- [Introdução](#introducao)
- [Funcionamento geral](#funcionamento-geral)
- [Autenticação](#autenticacao)
- [Criação de usuários corporativos](#criacao-de-usuarios-corporativos)
- [Download de um documento](#download-de-um-documento)
- [Hooks](#hooks)

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

**Atenção:** Toda a comunição cliente/servidor é feita através de HTTP sobre SSL/TLS (HTTPS). Requisições em HTTP simples resultam em redirecionamentos (304) para o protocolo HTTPS. Por questão de simplicidade, os exemplos foram construídos utilizando o protocolo HTTP.

Todas as requisições da _REST API_ são feitas para `api.clicksign.com`.

## Exemplo de requisição

```http
GET /documents HTTP/1.1
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
```

- Versão: 1.1
- Status: 200
- Mensagem: OK
- Cabeçalhos: Content-Type, Connection
- Corpo: [{...

# <a name="autenticacao"></a>Autenticação

A Clicksign utiliza duplo fator de autenticação para aumentar a segurança de suas transações. Autenticações que utilizam duplo fator geralmente são baseadas em algo que o parte cliente _conhece_ e algo que a parte cliente _possui_. No caso da API os fatores são:

1. Conhecer um par **identificação** e **segredo**
1. Possuir um endereço **IP** específico

O primeior fator da autenticação é feito através de 2 parâmetros: **api_id** e **api_secret**. O parâmetro `api_id` define qual cliente está fazendo a requisição. O parâmetro `api_secret` define o senha que será utilizada na verificação de acesso à API. Ambos os parâmetros devem ser enviados no **caminho** da requisição. Portanto, toda requisição deverá conter no _path_ `?api_id=string-da-key&api_secret=string-do-secret`.

**Atenção:** Os parâmetros de autenticação devem ser enviados a cada requisição feita pelo cliente. Como esses parâmetros são comuns a todos as funções da API, eles serão omitidos das documentações.

O segundo fator da autenticação é realizado automaticamente pelo servidor da Clicksign, que verifica se o **IP** de origem da requisição está dentro de uma lista de endereços previamente cadastrados para determinado cliente.


# <a name="criacao-de-usuarios-corporativos"></a>Criação de usuários corporativos

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

Para especificar a criação de um usuário corporativo a requisição json deve seguir o formato especificado acima.

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
      <td>String com até 50 caracteres</td>
      <td>x</td>
    </tr>

    <tr>
      <td>person.name.additional_name</td>
      <td>String com até 50 caracteres</td>
      <td></td>
    </tr>

    <tr>
      <td>person.name.family_name</td>
      <td>String com até 50 caracteres. A quantidade total de sobrenomes (additional_name + family_name + honorific_suffix, separados por espaço) não deve ultrapassar a quantidade de <b>6 nomes</b></td>
      <td>x</td>
    </tr>

    <tr>
      <td>person.name.honorific_suffix</td>
      <td>String com até 50 caracteres</td>
      <td></td>
    </tr>

    <tr>
      <td>person.documentation.country</td>
      <td>"br"</td>
      <td>x</td>
    </tr>

    <tr>
      <td>person.documentation.kind</td>
      <td>"cpf"</td>
      <td>x</td>
    </tr>

    <tr>
      <td>person.documentation.value</td>
      <td>String com 11 digítos com ou sem pontuação, exemplos válidos: "99999999999", "999.999.999-99"</td>
      <td>x</td>
    </tr>

    <tr>
      <td>person.phone.country</td>
      <td>"br"</td>
      <td>x</td>
    </tr>

    <tr>
      <td>person.phone.number</td>
      <td>String com 10 ou 11 digítos com ou sem pontuação, onde os dois primeiros representam o **ddd** e os últimos 8 ou 9 digítos representam ou número celular</td, exemplos válidos: "99-9999-9999", "99-9-9999-9999", "99999999999", "9999999999"</td>
      <td>x</td>
    </tr>
  </tbody>
</table>


## Resposta 200

Caso não ocorra nenhuma falha na requisição, o corpo da resposta será um _JSON_ contendo as informações do cadastro de um usuário corporativo.

* **Cabeçalhos**:
  - **Content-Type:** application/json
* **Corpo:**

  ```json
  {
    "registration": {
      "url": "https://desk.clicksign.com/registration?uuid=....",
      "uid": "..."
    }
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


# <a name="download-de-um-documento"></a>Download de um documento

Retorna um arquivo _ZIP_ contendo os 2 arquivos resultantes do processamento: arquivo original, log concatenado a uma cópia carimbada do arquivo.

* **Method:** GET
* **Path:** /documents/:id
* **Cabeçalhos:**
  - **Accept**: application/zip
* **Corpo:** _vazio_

## Resposta 200

Caso não ocorra nenhuma falha na requisição, o corpo da resposta será o arquivo _ZIP_.t 

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


# <a name="hooks"></a>Hooks

É possível que a Clicksign notifique outras aplições à respeito de determinados eventos, p. ex., documento completamente assinado.

Para isso, a Clicksign dispõe de um sistema de **hooks** que realizam chamadas HTTP para outras aplicações. As _hooks_ são definidas por usuário, portanto cada usuário deve configurar as _hooks_ com os parâmetros que deseja.

Os parâmetros possíveis são:

- **URL**: caminho completo, incluíndo protocolo
- **Método**: GET, POST, PUT, DELETE, PATCH
- **Content-Type**: application/json

É anexado ao corpo da requisição uma representação em _JSON_ do evento que a disparou, p. ex., em evento de documento completamente assinado é anexado um _JSON_ do documento, da lista de assinatura e das assinaturas do documento.
