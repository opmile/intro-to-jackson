# Criando uma requisição com `java.net.http`
A API `java.net.http` conta com classes utilitárias que representam o cliente, a requisição e a resposta do servidor.

1. `HttpClient`: cliente http em si
    * Responsável por gerenciar pools de conexões e enviar requisições
2. `HttpRequest`: requisição que o cliente deseja enviar ao servidor (GET, POST, etc.)
3. `HttpResponse`: resposta que recebemos do servidor diante a requisição

```java
public class Request {

    HttpClient client = HttpClient.newHttpClient();

    HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("https://api.exemplo.com"))
                .build();
    
    HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());

}
```

## A instância `HttpClient`
Funciona como um navegador invisível que envia requisições e recebe respostas do servidor

```java
HttpClient client = HttpClient.newHttpClient();
```
* O método estático `newHttpClient()` cria uma instância `client` com suas configurações padrão:
    * Método de requisição `GET`
    * versão HTTP/2
    * NEVER como escolha de redirecionamento
    * Proxy de seleção padrão
    * Padrão de contexto SSL
```java
// custom config
HttpClient client = HttpClient.newBuilder()
							.version(HtppClient.Version.HTTP_2)
							.connectTimeout(Duration.ofSeconds(10))
							.build();
```
Podemos e devemos reaproveitar uma instância de `HttpClient` sempre que possível para múltiplas requisições, já que em um mesmo `client` podemos ter vários pools de conexões

## A instância `HttpRequest`
Represnta uma única requisição http, já que o url da api é o mesmo para a mesma requisição
```java
HttpRequest request = HttpRequest.newBuilder() 
								.uri(URI.create("https://api.exemplo.com")
								.GET() // ou .POST(BodyPublishers.ofString("dados do body"))
								.build();
```
1. O método dentro do builder deve receber no mínimo a URL da API como parâmetro

* `URI.create` cria um objeto do tipo URI a partir de uma string
    
Quando queremos uma busca dinâmica ao coletar dados do usuário com o intuito de inserí-los nas URL, temos que ter cuidado para o código não lançar uma exception de argumento inválido ao garantir a correta formatação desses dados

a. Isso pode ser feito por meio do método de String `replace()`, substituindo os espaços entre as palavras por `+`

```java
String t = "milena oliveira";
String t1 = t.replace(" ". "+"); // milena+oliveira
```

b. Entretanto, a versão mais recomendada sugere o uso do método estático `URLEncoder.encode()` que garante que qualquer string fique válida para uma URL (trata acentos, caracteres especiais)

```java
import java.net.URLEncoder;
import java.nio.charset.StandardCharsets;

//

title = URLEncoder.encode(title, StandardCharsets.UTF_8);
author = URLEncoder.encode(author, StandardCharsets.UTF_8);
```


2. o cabeçalho (header da requisição) pode ser incluso

    * headers são pares `chave: valor` que você envia na requisição para passar metadados, autenticar, informar o tipo de conteúdo, o formato de arquivo aceito como resposta, controlar cache, definir idiomas...

| **Header**          | **O que faz**                              | **Exemplo**                       |
|---------------------|---------------------------------------------|-----------------------------------|
| Content-Type        | Informa o tipo de conteúdo do corpo da requisição | application/json                  |
| Authorization       | Autenticação com token ou credenciais       | Bearer senhaAleatoria123…         |
| Accept              | Diz o formato da resposta que o cliente aceita | application/json              |
| User-Agent          | Identifica o cliente (navegador, app…)      | Java HttpClient/11                |
| Cache-Control       | Define regras de cache                      | no-cache, max-age=3600            |
| Accept-Language     | Idioma que prefere na resposta              | pt-BR, en-US                      |
| Cookie              | Envia cookies já armazenados                | session_id=xyz123                 |
| Referer             | Diz de qual página o pedido está vindo      | https://site.com/pagina-anterior |
| Host                | Define o domínio do servidor                | api.exemplo.com                   |
ex)
```java
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.exemplo.com/dados"))
    .header("Content-Type", "application/json")               // Corpo será JSON
    .header("Accept", "application/json")                     // Espero JSON de volta
    .header("Authorization", "Bearer SEU_TOKEN_AQUI")         // Autenticação com token
    .header("User-Agent", "MinhaAppJava/1.0")                 // Identifica seu app
    .header("Accept-Language", "pt-BR")                       // Pede resposta em português
    .POST(HttpRequest.BodyPublishers.ofString("{...}"))
    .build();
```

3. Os métodos `.GET()`, `.POST(...)`, `.PUT(...)` escolhe o método da requisição HTTP

    a. `.GET()` não envia corpo

    b. `.POST(...)` deve enviar corpo de dados
    * Quando isso acontece, enviamos corpo com dados via `BodyPublisher`

        * `HttpRequest.BodyPublisher`

        É uma classe aninhada a `HttpRequest` e utilitária para definir o corpo da requisição com os dados que queremos enviar ao servidor (JSON, texto, binário...)
            * funciona como uma fábrica de "corpos de dados"
            | **Método**                           | **O que faz**                                                        |
            |--------------------------------------|----------------------------------------------------------------------|
            | ofString(String)                     | Envia uma string como corpo (JSON, XML, etc)                         |
            | ofFile(Path)                         | Envia o conteúdo de um arquivo                                       |
            | ofInputStream(Supplier<InputStream>) | Envia dados vindos de um InputStream, útil para arquivos grandes     |
            | noBody()                             | Indica que a requisição não terá corpo (método GET)                  |


ex) Requisição com método `.GET()`
```java
HttpClient client = newHttpClient();

HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("https://api.exemplo.com"))
            .GET()
            .header("Accept", "application/json")
            .build();
```

ex) Requisição com método `.POST(...)`
```java
String json = """
{
    "nome": "milena",
    "senha": 12345
}
""";

HttpClient client = newHttpClient();

HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("https://api.exemplo.com"))
            .header("Content-Type", "application/json")
            .POST(HttpRequest.BodyPublisers.ofString(json))
            .build();
```

## A instância `HttpResponse`
Depois de enviar a requisição por meio do `client`, o servidor responde com:

* Um status code da requisição (200, 404, 500...)
* Os headers da resposta do servidor
* O corpo da resposta (JSON, XML, etc.)

O `HttpResponse<T>` é o objeto que encapsula essa resposta completa

Uma resposta `HttpResponse` não pe criada diretamente, mas retornada como resultado do envio da requisição 
```java
HttpResponse<String> response = cliend.send(request, BodyHandlers.ofString());
```
1. `HttpResponse<String>` indica que a resposta tem o tipo `String`

2. `.send(...)` método síncrono (espera a resposta do servidor e congela o sistema)

3. `BodyHandlers.ofString()` indica que o corpo da resposta será convertido para String

    * O `BodyHandler` define como o corpo da resposta será tratado
```java
// corpo como String
HttpResponse<String> response = client.send(request, BodyHandlers.ofString());

// corpo como array de bytes
HttpsResponse<byte[]> response = client.send(request, BodyHandlers.ofByteArray());

// corpo como arquivo salvo localmente
Path caminho = Paths.get("resposta.json");
HttpResponse<Path> response = client.send(request, BodyHandlers.ofFile(caminho));

// corpo ignorado
HttpResponse<Void> response = client.send(request, BodyHandlers.discarding());
```

### Extraindo a resposta do servidor
Agora que nossa requisição foi feita, podemos extrair seus dados e os status da requisição
```java
System.out.println("Status Code: " + responde.statusCode());
System.out.println(response.body());
```