# Annotations Jackson

Segue um guia unificado com as principais annotations do Jackson, separadas por uso em **desserialização**, **serialização** e **gerais** (que funcionam para ambos).

---

## Annotations para Desserialização

**@JsonAlias**
Define nomes alternativos para um campo na leitura do JSON.
Não afeta a serialização.

```java
@JsonAlias({"nome_completo", "nome_usuario"})
private String nome;
```

**@JsonCreator**
Indica qual construtor ou método de fábrica deve ser usado para criar a instância na desserialização.

```java
@JsonCreator
public Pessoa(@JsonProperty("nome") String nome) {
    this.nome = nome;
}
```

**@JsonSetter**
Define o nome do campo esperado no JSON para um setter específico.

```java
@JsonSetter("funcao")
public void setCargo(String cargo) { ... }
```

**@JsonDeserialize**
Usa um desserializador customizado para o campo.

```java
@JsonDeserialize(using = DataCustomDeserializer.class)
private LocalDate data;
```

**@JsonAnySetter**
Captura campos não mapeados no JSON e os armazena em um `Map`.

```java
@JsonAnySetter
public void setAtributoExtra(String key, Object value) { ... }
```

---

## Annotations para Serialização

**@JsonGetter**
Define o nome da propriedade no JSON para um método getter.

```java
@JsonGetter("nome_formatado")
public String getNomeFormatado() { ... }
```

**@JsonValue**
Indica que o valor retornado pelo método será usado como representação JSON do objeto.

```java
@JsonValue
public String getValor() { ... }
```

**@JsonAnyGetter**
Serializa o conteúdo de um `Map` como pares de chave/valor diretamente no JSON.

```java
@JsonAnyGetter
public Map<String, Object> getAtributosExtras() { ... }
```

**@JsonRawValue**
Serializa o valor já como JSON, sem escapar caracteres.

```java
@JsonRawValue
private String jsonPronto;
```

**@JsonSerialize**
Usa um serializador customizado para o campo.

```java
@JsonSerialize(using = NomeUppercaseSerializer.class)
private String nome;
```

**@JsonPropertyOrder**
Define a ordem das propriedades no JSON.

```java
@JsonPropertyOrder({ "nome", "idade" })
```

**@JsonRootName**
Define o nome do nó raiz do JSON quando usado com `WRAP_ROOT_VALUE`.

```java
@JsonRootName("usuario")
```

---

## Annotations Gerais (desserialização e serialização)

**@JsonProperty**
Define o nome da propriedade no JSON (usado na leitura e escrita).

```java
@JsonProperty("nome_completo")
private String nome;
```

**@JsonIgnore**
Ignora o campo tanto na leitura quanto na escrita.

```java
@JsonIgnore
private String senha;
```

**@JsonIgnoreProperties**
Ignora uma lista de propriedades ou todas as desconhecidas.

```java
@JsonIgnoreProperties({"senha", "token"})
@JsonIgnoreProperties(ignoreUnknown = true)
```

**@JsonInclude**
Controla quando incluir o campo no JSON (ex.: apenas se não for null).

```java
@JsonInclude(JsonInclude.Include.NON_NULL)
```

**@JsonFormat**
Controla o formato de datas, números e enums.

```java
@JsonFormat(pattern = "dd-MM-yyyy")
private LocalDate data;
```

**@JsonUnwrapped**
Desagrupa as propriedades de um objeto aninhado para o nível atual do JSON.

```java
@JsonUnwrapped
private Endereco endereco;
```

---

