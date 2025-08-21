# Introdução ao `ObjectMapper`

Imagine que você esteja lidando com APIs REST ou arquivos de configuraçaõ em JSON. Como fazer **a ponte entre dados JSON e objetos Java**? Entra em cena o **Jackson**, uma poderosa biblioteca *data-binding*, com seu personagem principal: o **`ObjectMapper`**

---

## O que é o `ObjectMapper`?

O `ObjectMapper` é a classe principal da biblioteca Jackson, responsável por:

* **Serializar**: conversão de objetos Java em JSON

* **Desserializar/Parsear**: conversão de arquivos JSON em objetos Java

Ou seja, é uma ferramenta de mapeamento de campos do mundo Java e o mundo JSON

Vamos analisar a seguir um exemplo completo e amplo do `ObjectMapper` para o proceso de serialização e desserialização simples, cobrindo primeiro sem annotations, para depois analisar a necessidade das mesmas

Para isso precisamos criar nossa classe modelo, comumente chamada de **POJO** (Plain Old Java Object). Esse é um termo usado para descrever um objeto Java simples, que não depende de bibliotecas, frameworks ou convenções especiais

* Ele é puro: só tem atributos, construtores, getters/setters e, às vezes, `toString`, `equals` e `hashCode`

* Não precisa estender classes específicas ou implementar interfaces obrigatórias para funcionar

* Surge com a ideia de manter o objeto desacoplado e independente, de modo que qualquer ferramente ou framework conseguem trabalhar com ele, justamente porque segue o contrato básico de um JavaBean (construtor sem atumentos + getters/setters)

1. Criando a classe modelo (POJO)

```java
package model;

public class Pessoa {
    private String nome;
    private int idade;

    public Pessoa() {} // construtor vazio essencial para o Jackson

    public String getNome() { return nome; }
    public void setNome(String nome) { this.nome = nome; }

    public int getIdade() { return idade; }
    public void setIdade(int idade) { this.idade = idade; }
}
```

2. Classe para a requisição HTTP

```java
public clas ClientHttp {
    private final HttpClient client = HttpClient.newHttpClient();
    private final ObjectMapper mapper = new ObjectMapper();

    public Pessoa getPessoa(String query) throws Exception {
        String url = URLEncoder.encode(query, StandardCharsets.UTF_8);

        HttpRequest request = HttpRequest.newBuilder()
                    .uri(URI.create(url))
                    .GET()
                    .header("Accept", "application/json")
                    .build();

        HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());

        if (reponse.statusCode() != 200) {
            throw new RuntimeException("Erro na requisição: " response.statusCode());
        }

        // desserializa json para object
        return mapper.readValue(response.body(), Pessoa.class);
    }

    public void postPessoa(String query, Pessoa pessoa) throws Exception {
        mapper.enable(SerializationFeature.IDENT_OUTPUT); // pretty print
        String url = URLEncoder.encode(query, StandardCharsets.UTF_8);

        String json = mapper.writeValue(pessoa); // serializa Pessoa para JSON

        HttpRequest request = HttpRequest.newBuilder()
                    .uri(URI.create(url))
                    .POST(HttpRequest.BodyPublishers.ofString(json));
                    .header("Content-Type", "application/json")
                    .header("Accept", "application/json")
                    .build();
        
        HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());

        if (reponse.statusCode() != 201 && reponse.statusCode() != 200) {
            throw new RuntimeException("Erro ao postar: " + reponse.statusCode());
        }

        System.out.println("Resposta do server: " + response.body());
    }
}
```
    
---

## Funcionamento interno

O `ObjectMapper` usa reflexão (API `java.lang.reflect`) para inspecionar as propriedades públicas, getters/setters, e até annotations das classes.

### Um primeiro mergulho no parseamento de JSON

Reflexão (Reflection) é uma técnica que permite ao programa examinar e modificar sua própria estrutura e comportamento em tempo de execução. 

Em outras palavras...
> Java permite que você "olhe" e até "mexa" nas classes, atributos e métodos de objetos mesmo sem saber quem eles são em tempo de compilação

*(ex)* Extraindo uma classe em tempo de execução
```java
Class<?> clazz = Class.forName("model.Pessoa"); // carrega a classe pelo nome
Object obj = clazz.getDeclaredConstructor().newInstance(); // cria uma instância a partir do construtor
```
* Aqui, você não chamou `new Pessoa()` diretamente. Na verdade, o código criou um objeto da classe `Pessoa` sem saber que era essa classe em tempo de compilação. Isso é reflexão!

Ao usar o `ObjectMapper` para mapear os campos de uma classe a partir de um JSON (parsear), usamos o método `readValue(json, .class)`
```java
Pessoa pessoa = mapper.readValue(json, Pessoa.class);
```
Com isso, estamos dizendo:
> "Aceite esse JSON aqui, e me devolva um objeto dessa classe informada"

Mas o Jackson é uma biblioteca genérica. Ele pode estar lidando com `Pessoa`, `Carro`, `Pedido`, `Cliente`... Qualquer coisa em essência, o que leva a não ter um código fixo que instancia qualquer coisa, o que leva ao uso de reflexões. 

Então se ele não pode instanciar diretamente com `new Pessoa()`, ele faz:
```java
Class<?> clazz = Pessoa.class;
Constructor<?> ctor = clazz.getDeclaredConstructor(); // acha o construtor padrão
Object obj = ctor.newInstance(); // instancia o objeto dinamicamente
```

É importante ressaltar que esse processo só é possível se na classe entidade existir um construtor público ou protegido sem argumentos (construtor padrão). Caso contrário, Jackson vai lançar uma exception do tipo `InvalidDefinitionException`
```java
com.fasterxml.jackson.databind.exc.InvalidDefinitionException:
Cannot construct instance of `Pessoa` (no Creators, like default constructor, exist):
```
Outro ponto importante é garantir a compatibilidade entre os tipos de dados recebidos do JSON e os campos do objeto Java. Caso contrário, será lançada uma exception do tipo `MismatchInputException`

Você pode se perguntar agora como isso pode funcionar sem um construtor com argumentos para preencher os atributos da instância criada, mas:
```java
Pessoa p = mapper.readValue(json, Pessoa.class);
```
Voltamos ao funcionamento do método `readValue()`:

1. Lê o JSON como string, byte, ou stream

2. Cria uma nova instância da classe informada `Pessoa` usando reflexão

3. A partir dos dados das chaves vindos do JSON, preenche todos os campos da instância usando os setters, quando os atributos são privados, ou acessando diretamente quando os atributos são públicos

Mesmo assim, você pode ter notado que usar um construtor padrão, sem parâmetros, é pouco flexível e, de fato, não é uma boa prática analisando pela ótica da orientação a objetos. Por isso, surgem as annotations do Jackson

#### O Problema da Correspondência de Campos

Quando você desserializa um JSON com `ObjectMapper.readValue(json, Classe.class)`, o Jackson segue este prcesso:

1. Lê o JSON como texto (string) ou stream

2. Analisa cada chave do JSON (`"nome"`, `"idade"`, etc.)

3. Procura um campo correspondente na classe Java:

    * Primeiro por métodos (setters) ou campos com acesso público

    * Se anotados, busca por construtores com parâmetros nomeados. Caso contrário, procura por construtor padrão público sem parâmetros

4. Converte os valores do JSON para o tipo esperado (String, int, etc.)

5. Atribui os valores aos campos do objeto, via setters ou acesso direto

*(ex)* Correspondência automática por nome:

* Por padrão, o Jackson tenta casar os nomes dos campos:
```json
{
    "nome": "Milena",
    "idade": 19
}
```
```java
public class Pessoa {
    private String nome;
    private int idade;

    // getters e setters
}
```
Jackson vai encontrar o setter `setNome(String)` e `setIdade(int)` e atribuir valores automaticamente

É importante ressaltar a sensibilidade a maiúsculas/minúsculas. Isso pode ser configurado via annotation usando `@JsonSetter`, por exemplo

**E se os campos não baterem?**
```json
{
    "full_name": "milena",
    "years_old": 19
}
```
O JSON acima não vai funcionar com a definição da nossa classe `Pessoa`, porque as chaves `full_name` e `years_old` não correspondem a nenhum campo nem setter

##### Solução: Annotations do Jackson

##### `@JsonProperty`

Essa é a annotation mais importante e usada no mapeamento de nomes e serve para mapear um campo JSON para um atributo Java, mesmo que os nomes sejam diferentes

* Dentro da annotation, você leva como argumento a correspondência da chave JSON para o objeto Java

```java
public class Pessoa {
    @JsonProperty("full_name")
    private String name;

    @JsonProperty("years_old")
    private int idade;
}
```
Agora o Jackson casa:

* `"full_name"` - campo `nome`
* `"years_old"` - campo `idade`

Essa anotação pode ser colocada:

* Campo
* Métodos (inclusive getters/setters)
* Parâmetros do construtor

##### `@JsonCreator` + `@JsonProperty` (em construtores)

Essa combinação permite usar um construtor customizado para desserialização

```java
public class Pessoa {
    private final String nome;
    private final int idade;

    @JsonCreator
    public Pessoa(@JsonProperty("full_name") String nome,
                  @JsonProperty("years_old") int idade) {
        this.nome = nome;
        this.idade = idade;
                } 
}
```

Sem usar essa abordagem, o método de desserialização `readValue()` Jackson usa o construtor padrão vazio para instanciar o objeto e, para cada chave do JSON, procura um setter (ou acessa o campo diretamente, quando público), usando reflexão para atribuir valores aos campos após a instanciação.

O problema da abordagem padrão é que os setters são obrigatórios! Ou seja, se não tiver setters públicos, ou campos mutáveis, ou a definição de um construtor padrão, o Jackson não consegue desserializar.

Na realidade, nos nossos projetos Java orientados a objetos, não queremos permitir a mudança dos campos, e muitas vezes só temos getters na mesa. Além disso, precisaríamos de no mínimo dois construtores para cada entidade = código a mais (sem necessidade).

E aí entra o `@JsonCreator` com construtor parametrizado, uma anotação que diz explicitamente do Jackson:
> "Use este construtor para criar o objeto, em vez de tentar usar o construtor vazio + setters"
Cada parâmetro do construtor deve então ser ligado a uma chave do JSON com `@JsonProperty`, uma vez que, sem essa annotation, o Jackson não sabe que o primeiro argumento do construtor corresponde ao primeiro campo do JSON, você deve então fazer isso manualmente

O que o Jackson agora faz com essas annotations:
* Lê o JSON
* Encontra `nome` e `idade`
* Vê que existe um construtor anotato com `@JsonCreator`
* Associa `"nome"` ao parâmetro `nome` do construtor
* Associa `"idade"` ao parâmetro `idade` do construtor
* Constroí o objeto diretamente com o construtor, sem precisar de setters

##### `@JsonIgnoreProperties` e `@JsonIgnore`

Impede que um campo seja serializado ou desserializado. Muito útil quando queremos ignorar campos do JSON que não têm correspondência na classe. 

* Usar essa annotation é o que impede do lançamento da exception `UnrecognizedPropertyException` quando existem campos no JSON que não existem na classe Java

**`@JsonIgnore`**: Ignora um campo 
```java
@JsonIgnore
private String senha;
```

**`@JsonIgnoreProperties`**: Permite ignorar vários campos
```java
@JsonIgnoreProperties({"senha", "token", "ultimo_login"})
public class Usuario {
    private String nome;
}
```

Se você não quer listar campos um por um:
```java
@JsonProperties(ignoreUnknown = true)
```
* Isso faz com que quaisquer campos desconhecidos no JSON sejam ignorados silenciosamente

##### `@JsonAlias`

Permite que um atributo aceite múltiplos nomes alternativos do JSON. Isso se torna útil em APIs que mudam e podem apresentar diferentes campos no JSON
```java
@JsonAlias({"nome_completo", "full_name", "nomeCompleto"})
private String nome;
```
* Se qualquer uma dessas chaves estiver presente no JSON, ela será associada ao campo `nome`

**Diferenças entre `@JsonAlias` e `@JsonProperty`**

**`@JsonProperty`**: Serve para definir o nome exato que será usado tanto na serialização quanto na desserialização

* Funciona para atributor, métodos (setters e getters) e parâmetros de construtor (quando associado com a annotation `@JsonCreator`)
* Substitui o nome da chave no JSON (de entrada e de saída)

```java
public class Pessoa {
    @JsonProperty("nome_completo")
    private String nome;
}
```
* No JSON, sempre será `"nome_completo"` para ler e escrever

**`@JsonAlias`**: Serve para aceitar nomes alternativos na desserialização, mas não altera a serialização

* Funciona somente para atributos e métodos setters
* O Jackson vai aceitar qualquer um dos aliases ao ler o JSON, mas vai escrever usando o nome real do campo Java (ou quando estiver em `@JsonProperty`, se houver)

```java
public class Pessoa {
    @JsonAlias("nome_completo", "nomeCliente", "nome_user")
    private String nome;
}
```
* Se chegar qualquer um dessas chaves no JSON, o Jackson vai preencher o atributo `nome`. Mas se você serializar esse mesmo objeto (de Java para JSON), o nome da chave será `"nome"` (ou `"nome_completo"`, somente se `@JsonProperty` estiver junto)

**Cuidado!**
> Você não pode usar `@JsonAlias` diretamente em parâmetros do construtor como fazia com o `@JsonProperty`
Jackson ignora `@JsonAlias` em parâmetros e só considera `@JsonProperty` nesses casos

Isso acontece porque quando o Jackson instancia objetos via construtor (com `@JsonCreator`), ele precisa saber exatamente qual nome da chave JSON corresponde a cada um dos parâmetros.

Por isso o `@JsonAlias` funciona apenas quando Jackson já instanciou o objeto via construtor padrão e está setando os campos depois, ou seja:
* Via setters
* Via campos públicos (se configurado)

```java
public class Pessoa {
    private String nome;

    @JsonAlias({"nome_completo", "nome_usuario"})
    public void setNome(String nome) {
        this.nome = nome;
    }

    public Pessoa() {}

    public Pessoa(String nome) {
        this.nome = nome;
    }
}
```

### Serialização de objetos com Jackson

O que o Jackson faz:

1. Acessa os campos com getters públicos (ou diretamente, se configurado)
2. Constrói uma estrutura em árvore (`JsonNode`) ou uma `String` JSON diretamente
3. Aplica as configurações de visibilidade, formatações e anotações, se houver

```java
Pessoa pessoa = new Pessoa("Milena", 19);
String json = mapper.writeValueAsString(pessoa);
```
Esperado:
```json
{
    "nome": "Milena",
    "idade": 19
}
```

O método `writeValue()` é usado para serializar objetos Java em JSON, e enviá-los para diferentes tipos de saída. Pense nele na seguinte sintaxe:
```java
mapper.writeValue(destino, objetoJava);
```
Onde o `destino` pode ser quase qualquer coisa: arquivo, string, stream, etc.

Da mesma forma, você também pode serializar qualquer objeto que o Jackson reconheça (POJO, List, Map, arrays, etc.)

#### Variações principais do `writeValue()`

1. `writeValue(File file, Object value)`

Salva o JSON diretamente em um arquivo

```java
mapper.writeValue(new File("pessoas.json"), pessoa);
```
* `new File(...)` não significa recriar o arquivo físico ou sobrescrevê-lo sem controle. Na verdade, ele representa apenas uma referência ao caminho do arquivo, não cria ou escreve nada ainda
* Entretanto, o método `writeValue(File, Object)` sobrescreve o conteúdo do arquivo, não acrescentando os dados ao final do JSON! Ou seja, chamar esse mesmo método repetidas vezes significa perder os dados anteriores a cada chamada, porque ele escreverá um novo JSON inteiro do zero, substituindo o conteúdo

Esse método se torna útil para persistir dados em um arquivo JSON. Veremos esse tópico mais à frente

2. `writeValue(OutputStream out, Object value)`

Escreve para qualquer tipo de stream (ex: `System.out`, `FileOutputStream`, `Socket.getOutputStream`, etc.)

```java
mapper.writeValue(System.out, pessoa); // imprime no console

// ou

OutputStream out = new FileOutpuStream("saida.json");
mapper.writeValue(out, pessoa);
```

3. `writeValueAsString(Object value)`

Serializa e retorna o JSON como uma String Java
```java
String json = mapper.writeValueAsString(pessoa);
System.out.println(json);
```
* Essa é a versão mais comum em aplicações web, onde você precisa serializar para enviar no corpo de uma resposta HTTP, salvar em banco, etc.

4. `writeValueAsBytes(Object value)`

Serializa e retorna o JSON como um array de bytes (`byte[]`)

```java
byte[] bytes = mapper.writeValueAsBytes(pessoa);
```
* Útil para enviar JSON por rede, salvar em arquivos binários ou usar com `OutputStream`

##### Personalização da saída

Você pode aplicar diversas configurações antes de chamar `writeValue(...)`

1. Formatar com identação (Pretty Print)
```java
mapper.enable(SerializationFeature.IDENT_OUTPUT)
```
Saída esperada:
```java
{
  "nome" : "Milena",
  "idade" : 19
}
```
Ou ainda, você pode optar por:
```java
mapper.writerWithDefaultPrettyPrinter().writeValue(filepath, users);
```


2. Ignorar campos nulos

Isso pode ser feito com a annotation `@JsonInclude`, que controla quando um campo deve aparecer na serialização
```java
@JsonInclude(JsonInclude.Include.NON_NULL)
public class Produto {
    private String name;
    private String descricao; //  se for null, não aparece
}
```
Ou no próprio `ObjectMapper`
```java
mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
```

Outros campos possível para o `@JsonInclude`
* `ALWAYS` (padrão)
* `NON_NULL`
* `NON_EMPTY`
* `NON_DEFAULT`

3. Serializar datas com formato customizado

```java
ObjectMapper mapper = new ObjectMapper();
mapper.registerModule(new JavaTimeModule());
mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
```

Ou melhor ainda, usar a annotation `@JsonFormat` direto no campo da classe, que define a formatação padrão desejada para datas ou números, por exemplo:

```java
public class Evento {
    @JsonFormat(pattern = "dd-MM-yyyy")
    private LocalDate data;
}

public class Pessoa {
    @JsonFormat(pattern = "dd/MM/yyyy")
    private LocalDate dataNascimento;
}
```

### Criando uma Estrutura Java a partir de Dados do JSON

Se queremos serializar ou parsear um JSON na forma de array para uma estrutura de dados Java, devemos usar `TypeReference`.

No Java, quando se usa generics (como `List<Usuario>`), o compilador sabe o tipo dos elementos só em tempo de compilação. No tempo de execução, graças ao *type erasure* (apagamento de tipos), o Java não sabe mais que essa lista era de `Usuario` - na verdade, ele só vê `List<T>`

Na hora de parsear, se você fizer:
```java
List<Usuario> usuarios = mapper.readValue(json, List.class);
```
O Jackson não sabe que o conteúdo deve ser `Usuario`, então ele vai criar uma lista de `LinkedHashMap` representando os campos, não instâncias de `Usuario`

O `TypeReference<T>` é uma classe genérica que o Jackson usa para capturar informações de tipo em tempo de compilação e repassá-las para o tempo de execução, permitindo reconstruir exatamente o tipo que você quer usar.

Ele funciona assim:
```java
List<Usuario> usuarios = mapper.readValue(json,
    new TypeReference<List<Usuario>>() {}
);
```
* Esse `{}` no final serva para criar uma (sub-)classe anônima que preserva o tipo genérico (`List<Usuario>`) internamente para que o Jackson possa usá-lo

*(ex)* array JSON -> Lista de objetos Java
```json
[
    {
        "nome": "Milena",
        "idade": 19
    },
    {
        "nome": "Lucas",
        "idade": 20
    }
]
```
```java
ObjectMapper mapper = new ObjectMapper();

List<Usuario> users = mapper.readValue(
    json,
    new TypeReference<List<Usuario>>() {}
);

for (Usuario u : users) {
    System.out.println(u.getName);
}
```
1. O Jackson captura o tipo de referência em tempo de execução, vendo que é uma `List` do tipo `Usuario`
2. Em seguida, cria uma lista (`ArrayList`) vazia
3. Para cada elemento do array JSON:  
    * Constrói um `Usuario` usando o mapeamento normal (via setters, campos públicos ou construtor com `@JsonCreator`)
4. Retorna a lista com todos os objetos preenchidos

*(ex)* objeto JSON -> `Map<String, Usuario>`
```json
{
    "admin": {
        "nome": "Milena",
        "idade": 19
    },
    "guest": {
        "nome": "Lucas",
        "idade": 20
    }
}
```
```java
Map<String, Usuario> usuarios = mapper.readValue(
    json,
    new TypeReference<Map<String, Usuario>>() {}
);

Usuario admin = usarios.get("admin")
System.out.println(admin.getNome());
```
* A chave do JSON vira String
* O valor é convertido para um `Usuario`