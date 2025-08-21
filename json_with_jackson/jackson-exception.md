# Jackson Exceptions

Ao manipular a biblioteca Jackson no Java, os principais tipos de exceções que você precisa conhecer e, em alguns casos, tratar diretamente, são subclasses de 

`com.fasterxml.jackson.core.JsonProcessingException` (estende `IOEXception`)

* Por isso, é considerada uma unchecked exception: obrigatório tratar!

## Exceptions mais Relevantes

### `JsonProcessingException`

Classe base para a maioria das exceptions lançadas pelo Jackson durante serialização ou desserialização

* **Motivo**: indica que houve algum prolema no processamento do JSON (erro de sintaxe, incompatibilidde de tipos, etc.)

* **Uso comum**: capturar de forma genérica qualquer problema com o JSON

```java
try  {
    objectMapper.readValue(json, MyClass.class);
} catch(JsonProcessingException e) {
    // lida com qualquer problema relacionado ao parsing ou geração de JSON
}
```

### `JsonParseException`

* Subclasse de `JsonProcessingException`

* **Motivo**: JSON inválido ou malformatado (erro de sintaxe)

* **Exemplo**: vírgula extra, chaves sem aspas, colchetes aberto sem fechamento

* **Quando acontece**: no parse inicial, antes mesmo de tentar mapear para o objeto

### `JsonMappingException`

* Subclasse de `JsonProcessingException`

* **Motivo**: problema na hora de mapear o JSON para o POJO (ou vice-versa)

* **Exemplos**:
    
    * Tipo incompatível 

    * Campo obrigatório no Java (ou sem setter) mas ausente no JSON

    * Erros em classes aninhadas durante mapeamento

Essa exception é mais comum no dia a dia, principalmente na desserializção

### `MismatchInputException`

* Subclasse de `JsonMappingException`

* **Motivo**: tipo de dado esperado não corresponde ao encontrado no JSON

```java
objectMapper.readValue("\"string\"", Integer.class); // esperava número mas entregou string
```

### `InvalidDefinitionException`

* Subclasse de `JsonMappingException`

* **Motivo**: configuração incorreta para serialização/desserialização

* **Exemplos**:

    * Classe sem construtor padrão ou sem `@JsonCreator + @JsonProperty`

    * Falta de serializer/desserializer registrado para um tipo específico


### `StremReadException`/`StreamWriteException`

* Motivo: erros de leitura ou escrita em fluxos (`InputStream`/`OutputStream`) durante o processamento do JSON

* São subclasses específicas para tratar erros em streaming 

## Hierarquia

IOException
  └── JsonProcessingException
        ├── JsonParseException
        ├── JsonMappingException
        │       ├── MismatchedInputException
        │       └── InvalidDefinitionException
        ├── StreamReadException
        └── StreamWriteException

