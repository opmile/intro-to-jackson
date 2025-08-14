# Type Erasure em Tempo de Execução

O type erasure (ou apagamento de tipos) é um mecanismo interno do Java que está diretamente ligado a como a linguagem implementa generics (tipos genéricos)

## O problema que o Java quis resolver

Antes do Java 5, não existia generics, então você usava coleções assim:

```java
List lista = new ArrayList();
list.add("texto");
list.add(42);
```
Isso era perigoso porque você só descobria erros de tipo em tempo de execução, e não em compilação
```java
String s = (String) list.get(1); // ClassCastException
```

## A chegada dos *generics*

No Java 5, foram introduzidos os generics para permitir checagem de tipos em tempo de compilação

```java
List<String> list = new ArrayList<>();
list.add("texto");
list.add(42); // ERRO de compilação
```
* Agora o compilador sabe que só `String` pode ser colocado na lista, porque analisa o tipo genérico informado para essa lista 

## O detalhe: por que o Java não guardou essa informação em tempo de execução?

Aí entra o type erasure.

O Java quis manter compatibilidade binária com todo o código legado pré-Java 5. Isso significa que classes compiladas antes de generics precisariam continuar funcionando com o código novo.

Para isso, os generics existem só no tempo de compilação

Depois de compilado, as informações de tipo genérico são "apagadas" (erased) e substituídas pelo tipo mais genérico possível (normalmente `Object`, ou o upper bound se houver)

```java
List<String> lista = new ArrayList<>();
```
No bytecode:
```java
List lista = new ArrayList();
```

Em compile-time, Java usa os tipos genéricos para verificar compatibilidade de tipos e daí remove os tipos genéricos, substituindo por:

* `Object`, se não houver bound

* o `bound` expecificado, se houver (`<T extends Number>` se torna `Number`)

Por isso, quando recebemos um JSON e queremos parseá-lo em uma classe Java, com:

```java
List<String> lista = mapper.readValue(json, List.class);
```

* O Jackson verá isso somente como `List<T>`, como não reconhece qual é esse tipo genérico `T`, isso porque, em tempo de execução, o método só recebe `List.class`

Ao usar uma classe genérica abstrata `TypeReference`:

```java
List<String> lista = mapper.readValue(json, new TypeReference<List<String>>() {});
```

O truque aqui é

* Quando você cria uma subclasse anônima, o tipo genérico é armazenado no objeto da classe (via reflexão, em `getGenericSuperclass()`)

* O Jackson consegue então extrair essa informação antes que o compilador apague (porque no momento da criação da instância anônima, a informação ainda está no bytecode da subclasse)