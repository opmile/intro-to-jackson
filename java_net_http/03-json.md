# JSON

JSON significa JavaScript Object Notation, mas não se engane, ele não é exclusivo do JavaScript

o JSON é uma formatação leve e padronizada de dados usada para troca de informações entre sistemas, e é totalmente compatível com praticamente todas as linguagens, incluindo Java

Sua estrutura é muito similar a de um dicionário (`Map`)
```json
{
	"nome": "Milena",
	"idade": 19,
	"linguagens": ["Java", "Python", "JavaScript"],
	"status": true
}
```
- Aspas duplas são obrigatórias
- Chaves {} representam um objeto
- Colchetes [] representam uma lista (array)
- Tudo em texto puro — qualquer lingugem consegue ler

## JSON e APIs
Ao realizar uma requisição para uma API, ela geralmente responde com um arquivo em formato JSON
* Ou seja, a API te envia dados já estruturados num formato fácil de ser interpretado

ex) Chamada à API de usuários com:
`GET https://exemplo.com/usuarios`

API devolve a resposta em JSON:
```json
[
	{
		"id": 1,
		"nome": "ana",
		"email": "ana@email.com"
	},
	{
		"id": 2,
		"nome": "bruno",
		"email": "bruno@email.com"
	}
]
```
* Representação de uma lista de objetos, onde cada objeto representa um usuário 

JSON é o formato padrão para respostas de APIs REST

## A Estrutura de Dados do JSON
O arquivo do tipo `.json` é composto, basicamente, por objetos e arrays

`Objeto` ⇒ coleção de pares chave-valor, onde:
- As chaves são as strings
- Os valores podem ser strings, números, booleanos, objetos ou arrays

```json
{
	"nome": "joao",
	"idade": 30,
	"solteiro": false,
	"endereco": {
		"rua": "rua 123",
		"cidade": "sao paulo",
		"estado": "SP"
	},
	"telefones": [
		"1111-1111",
		"2222-2222"
	]
}
```

Enquanto os arrays são uma coleção ordenada de valores, que podem ser strings, números, booleanos, objetos ou outros arrays

```json
[
	{
		"nome": "joao",
		"idade": 30
	},
	{
		"nome": "maria",
		"idade": 25
	}
	{
		"nome": "pedro",
		"idade": 40
	}
]
```

## Parseamento do JSON
Existem dois processos essenciais para quando queremos trabalhar com dados, sem ficar manipulando texto puro

### Serialização
Processo de transformar um objeto Java em texto (json)

* Usado quando queremos transformar os dados de um objeto Java em algo que possamos enviar em uma requisição HTTP, ou salvá-los em um arquivo `.json` para realizar outros procedimentos

### Desserialização
Processo de transformar o JSON (texto) em objeto Java

* Comum quando consumimos dados de uma API que retorna dados em formato `.json` e queremos trabalhar com esses dados de forma manipulável, transformando-os em objeto