# Clicksign API REST

A Clicksign é uma solução online para enviar, guardar e assinar documentos, com validade jurídica. Foi criada para facilitar, reduzir custo e aumentar a segurança e compliance do processo de assinatura e workflow de documentos. Os documentos podem ser carregados, enviados e assinados pelo nosso site www.clicksign.com.

Apesar disso, sabemos que muitos de nossos clientes possuem fluxos próprios de assinatura e/ou desejam automatizar determinadas tarefas. A Clicksign possui uma **API REST**, o que significa que qualquer linguagem de programação que possa realizar requisições HTTP cumpre os requisitos necessários para consumir os serviços da API. Desde aplicações scripts shell até sistemas de ERP podem integrar com esforço mínimo de programação.

Os exemplos construídos nessa documentação utilizam **curl**. O programa curl é amplamente disponível em ambientes **Unix** e sua principal utilidade é realizar requisições HTTP. Os exemplos que envolvem programação utilizam a linguagem **Javascript** pelo fato de ser amplamente conhecida, além de ilustrar a versatilidade da API.

É possível criar uma lista de assinatura e envia-la a outros pessoas em uma única ação. Para isso, é necessário que estejam presentes os seguintes campos que especificam o documento, os signatários e a mensagem.

## Documento

O documento a ser enviado é determinado pelo campo `document_id`. Cabe ressaltar que o `document_id` é diferente para cada usuário que possui uma cópia do arquivo, portanto um mesmo arquivo possui múltiplos `document_id`, sendo um para cada usuário que tem acesso a ele.

### Exemplo

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

### Exemplo

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

## Exemplo

```json
{
  "message": {
    "recipients": [ "foo@example.com", "bar@example.com" ],
    "body": "Hi guys, please sign this documents."
  }
}
```

## Exemplos

```
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
