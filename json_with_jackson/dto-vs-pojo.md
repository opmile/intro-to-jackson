# Usar um POJO ou DTO ao Consumir Dados de uma API?

## POJOs

POJO (Pain Old Java Object) é um termo usado para descrever um objeto Java simples, que não depende de bibliotecas, frameworks ou convenções especiais.

```java
public class Pessoa {
    private String nome;
    private int idade;

    public Pessoa() {} // construtor padrão

    public Pessoa(String nome, int idade) {
        this.nome = nome;
        this.idade = idade;
    }

    public String getNome() { return nome; }
    public void setNome(String nome) { this.nome = nome; }

    public int getIdade() { return idade; }
    public void setIdade(int idade) { this.idade = idade; }
}
```
* Esse objeto não depende de nada. Jackson, Hibernate, Spring... Todos conseguem trabalhar com ele justamente porque segue o "contrato" básico de um JavaBean 

---

### JavaBean

(*obs*) Um JavaBean é um padrão de escrita de calsses Java definido nos anos 90 pela Sun Microsystems para padronizar como objetos simples devem ser estruturados, justamente para que, quando usufruindo qualquer framework ou biblioteca, essa ferramenta seja capaz de criar e manipular objetos automaticamente. Ele é basicamente um POJO, mas deve seguir as seguintes regras:

1. Construtor público sem argumentos
    * Necessário porque muitas ferramentas (Jackson, Hibernate, etc.) devem conseguir criar instâncias via reflexão

2. Atributos privados (encapsulamento): campos não podem ser expostos diretamente
    * Apesar de que muitas ferramentas (como o Jackson) podem popular objetos com seus campos públicos, entretanto, encapsulamento é além de boa prática

3. Getters e setter públicos seguindo convenção
    * Para cada atributo `x`, deve existir `getX()` e `setX(valor)`
    * Para booleanos, o getter por ser do padão `isX()`

---

## DTOs

Um DTO (Data Transfer Object) é um padrão de projeto usado para transportar dados entre camadas ou sistemas

* Na prática um DTO também é um POJO, só que com propósito específico: carregar dados para tráfego (ex.: requisições/respostas de API, serializa;'ao para JSON, etc.)

* A diferença não é técnica, é conceitual: todo DTO é um POJO, mas nem todo POJO é um DTO

```java
public class PessoaDTO {
    private String nome;
    private int idade;

    public PessoaDTO() {}

    public PessoaDTO(String nome, int idade) {
        this.nome = nome;
        this.idade = idade;
    }

    public String getNome() { return nome; }
    public void setNome(String nome) { this.nome = nome; }

    public int getIdade() { return idade; }
    public void setIdade(int idade) { this.idade = idade; }
}
```
* Igual ao POJO na estrutura, mas sua função é transportar dados, não necessariamente representar uma entidade do banco


## Jackson

Jackson não liga se o objeto é uma entidaqde de negócio ou um DTO: ele só precisa que seja um POJO válido. Porém, na prática, quando trabalhamos com APIs REST (Spring, por exemplo), quase sempre usamos DTOs para receber e enviar JSON, porque:

* Isolam a representação externa da lógica interna

* Evitam expor atributos sensíveis

* Permitem transformar dados antes/depois da serialização

Diante disso, caimos em dois cenários, em que cabe uma decisão de arquitetura

Se lembre que...
* **POJO** → qualquer objeto Java simples (pode ser uma entidade, um modelo de domínio, um DTO, etc.).
* **DTO** → um POJO que serve especificamente para transportar dados entre sistemas/camadas

* **Entidade de domínio** → representa o estado real da aplicação, ligado à lógica de negócio e/ou banco de dados

Imagine que estamos consumindo dados de uma API REST com Jackson a partir de uma requisição com método GET, e o JSON recebido seja do seguinte formato:
```json
{
  "full_name": "Ada Lovelace",
  "birth_year": 1815,
  "country": "United Kingdom"
}
```

**Opção 1:** Usar só um POJO

Aqui você cria uma classe que corresponde diretamente ao JSON e deixa o Jackson fazer o biding

* Uso de annotations @JsonProperty para correspondência de campos:
```java
public class Person {
    @JsonProperty("full_name")
    private String name;

    @JsonProperty("birth_year")
    private int yearOfBirth;

    private String country;

    // getters e setters
}
```

**Vantagens**:

* Simples e rápido
* Menos classes para manter

**Desvantagens**:

* Acoplamento do modelo interno ao formato externo do JSON
* Se o formato da API mudar, você tem que mudar a mesma classe que talvz já seja usada em outras partes da aplicação


**Opção 2:** Usar DTOs dedicados

Você cria uma classe só para receber dados da API (DTO), separando o modelo interno da estrutura externa

```java
public class PersonDTO {
    @JsonProperty("full_name")
    private String name;

    @JsonProperty("birth_year")
    private int yearOfBirth;

    private String country;

    // getters e setters
}
```

Depois você só converte para sua entidade de domínio

```java
public class Person {
    private String name;
    private LocalDate birthDate;
    private String country;
    // construtores, getters e lógica de negócio
}
```
* Aqui você também poderia incluir um construtor que aceita um dto como parâmetro para realizar a correspondência de campos entre o DTO recebido o objeto criado a partir da classe de domínio


Mapeando um para o outro com o `ObjectMapper`

```java
PersonDTO dto = mapper.readValue(json, PersonDTO.class);
Person person = new Person(dto.getName(),
                        LocalDate.of(dto.getYearOfBirth(), 1, 1),
                        dto.getCoubntry());
```


**Vantagens**:

* Desacoplamento total ente o formato da API e seu modelo interno
* Facilita evoluir seu domínio sem depender de mudanças externas
* Você pode fazer validações e transformações no meio do caminho

**Desvantagens**:

* Mais código (classe extra + mapeamento)



