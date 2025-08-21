# Persistencia de Dados Simples com Jackson

**Por padrão, JSON não é considerado *appendable* por natureza**

Isso significa que não podemos abrir o arquivo de modo `append` (como em `FileWriter(..., true)`) e escrever um novo objeto na "próxima linha", porque isso gera um JSON inválido!

Acabamos com algo parecido com:

```
{...}{...}{...}
```
ou
```
[ {...} ]
{...} -- fora do array
```

**Como proceder?**

1. Ler o conteúdo atual do arquivo e desserializá-lo para uma estrutura Java (geralmente `List<ClasseQualquer>`)
    * Lembre-se do uso de `TypeReference` para driblar o apagamento de tipos (*type erasure*)

2. Adicionar o novo objeto nessa lista

3. Serializar a lista inteira novamente e sobrescrever o arquivo

```java
public static void persistUser(User newUser, File filepath) throws IOException {
    ObjectMapper mapper = new ObjectMapper();
    List<User> users = new ArrayList<>();

    if (file.exists() && file.length > 0) {
        users = mapper.readValue(filepath, new TypeReference<List<User>>() {});
    }

    users.add(newUser);

    mapper.writerWithDefaultPrettyPrinter().writeValue(filepath, users);
}
```