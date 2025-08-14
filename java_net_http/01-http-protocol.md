# Requisições HTTP

Requisições HTTP são a base de como a web funciona, sendo usadas para comunicação entre cliente e servidor. Vamos explorar os principais conceitos e entender o ciclo de uma requisição HTTP, detalhando o processo.

### 1. **O que é HTTP?**

HTTP (HyperText Transfer Protocol) é um protocolo de comunicação que define como os dados são solicitados e transferidos entre clientes (normalmente navegadores) e servidores na web. Ele permite a transferência de textos, imagens, vídeos, dados em JSON, etc.

### 2. **Componentes de uma Requisição HTTP**

Quando você faz uma requisição HTTP, ela contém os seguintes componentes:

#### a) **Método HTTP**

O método indica a ação que o cliente deseja realizar no servidor. Os mais comuns são:

- **GET**: Usado para solicitar a recuperação de dados de um servidor. É o método mais comum, utilizado para buscar páginas da web, imagens, etc.
- **POST**: Usado para enviar dados ao servidor, como ao preencher um formulário ou criar um novo recurso.
- **PUT**: Usado para atualizar completamente um recurso existente no servidor.
- **PATCH**: Usado para atualizar parcialmente um recurso existente.
- **DELETE**: Usado para remover um recurso do servidor.

#### b) **URL (Uniform Resource Locator)**

A URL é o endereço do recurso que você está solicitando. Exemplo:

```
<https://www.exemplo.com/artigos/1>
```

Essa URL pode conter parâmetros de consulta (query params), como:

```
<https://www.exemplo.com/artigos?categoria=tecnologia>

```

#### c) **Cabeçalhos (Headers)**

Os cabeçalhos fornecem informações adicionais sobre a requisição ou a resposta. Eles podem incluir:

- **Content-Type**: Especifica o tipo de dado que está sendo enviado ou solicitado (ex: `application/json`, `text/html`).
- **Authorization**: Usado para passar tokens de autenticação.
- **User-Agent**: Informa o navegador que está sendo utilizado.
- **Accept**: Indica o formato de resposta esperado (ex: JSON, XML).

#### d) **Corpo da Requisição (Body)**

Nem toda requisição tem um corpo. Geralmente, ele é presente em métodos como POST e PUT, que envolvem o envio de dados ao servidor. O corpo pode conter dados em diferentes formatos, como JSON, XML ou até mesmo um formulário.

### 3. **Ciclo de uma Requisição HTTP**

O processo completo pode ser dividido em várias etapas:

#### a) **Cliente envia uma requisição**:

1. O cliente (navegador ou aplicativo) faz uma requisição usando um método (GET, POST, etc.).
2. Essa requisição contém a URL, os cabeçalhos e, dependendo do método, um corpo com dados.

#### b) **Servidor processa a requisição**:

1. O servidor recebe a requisição, analisa o método e os dados enviados.
2. O servidor executa a ação correspondente, como buscar dados de um banco de dados ou salvar algo.

#### c) **Servidor envia uma resposta**:

1. O servidor devolve uma resposta ao cliente com um **código de status HTTP** que indica o sucesso ou falha da operação.
2. A resposta também contém cabeçalhos e, geralmente, um corpo com dados (por exemplo, um documento HTML, JSON com dados de uma API, etc.).

### 4. **Códigos de Status HTTP**

Quando o servidor responde, ele envia um código de status que indica se a requisição foi bem-sucedida ou não. Os códigos mais comuns são:

- **200 (OK)**: Requisição bem-sucedida.
- **201 (Created)**: Um recurso foi criado com sucesso (normalmente em resposta a um POST).
- **400 (Bad Request)**: A requisição foi inválida, como dados malformados.
- **401 (Unauthorized)**: Acesso não autorizado (necessário autenticação).
- **403 (Forbidden)**: Acesso proibido, mesmo autenticado.
- **404 (Not Found)**: O recurso solicitado não foi encontrado.
- **500 (Internal Server Error)**: Ocorreu um erro no servidor.

### 5. **Exemplo de Requisição HTTP**

Aqui está um exemplo de requisição HTTP para ilustrar:

### Exemplo 1: Requisição GET

```
GET /api/usuarios HTTP/1.1
Host: www.exemplo.com
Accept: application/json
User-Agent: Mozilla/5.0

```

Essa requisição GET está pedindo a lista de usuários da API. O servidor responderá com um corpo JSON ou uma mensagem de erro, dependendo do resultado.

### Exemplo 2: Requisição POST

```
POST /api/usuarios HTTP/1.1
Host: www.exemplo.com
Content-Type: application/json
Content-Length: 56

{
   "nome": "João",
   "email": "joao@exemplo.com"
}

```

Aqui, a requisição POST está enviando um JSON no corpo para criar um novo usuário com nome e e-mail. A resposta pode ser um código `201 (Created)` com os dados do novo usuário.

### 6. **HTTPS**

O HTTPS é a versão segura do HTTP. Ele utiliza criptografia (SSL/TLS) para proteger os dados transmitidos entre o cliente e o servidor, garantindo confidencialidade e integridade das informações.

### Resumo:

- **HTTP** é o protocolo que governa a comunicação entre cliente e servidor na web.
- As requisições HTTP têm métodos (GET, POST, etc.), cabeçalhos, URLs e, às vezes, um corpo.
- O servidor responde com códigos de status e, geralmente, dados no corpo.
- **HTTPS** é a versão segura do protocolo, com criptografia.
