# API ou Application Programing Interface

API é uma ponte que permite a comunicaçãõ entre dois sistemas (ou dois pedaços de software) de forma padronizada
> O client faz um pedido, a API leva esse pedido ao servidor e depois traz os dados de volta com o código de status corresposdente dessa requisição
> * O client (você) não precisa saber como o servidor funciona e como ele entrega os dados que você pediu, só precisa saber o que pode pedir e como pedir


## Importância
- Separação de responsabilidades: quem consome a API não precisa saber como os dados são gerados
- Reutilização: uma mesma API pode ser usada por diferentes apps (web, mobile, etc)
- Integração: conecta seu sistema com outros serviços (como Google Maps, sistemas de pagamento, clima, etc)
- Escalabilidade: facilita o crescimento de sistemas maiores, com várias partes trabalhando de forma independente

## Contextos de uso
1. Apps de clima puxando dados de temperatura
2. Aplicativos de delivery buscando restaurantes
3. Sistemas de pagamento como o Pix, Stripe ou PayPal
4. Integrações com redes sociais (mostrar feed do instagram do seu site)
5. Integração entre front-end e back-end

## Tipos de API mais comuns

1. **REST**
    * Trabalha com requisições HTTP, como GET, POST, PUT, DELETE
    * Os dados geralmente são trocados em JSON
    
2. SOAP
    * Mais antigo, baseado em XML
    * Uso em sistemas corporativos
    
3. GraphQL
    * Alternativa moderna ao REST
    * Muito flexível, permite consultar exatamente os dados que você deseja

## API Key
Uma API Key é uma chave de acesso única — como uma senha pública — que identifica quem está usando uma determinada API

- Ela geralmente vem nesse formato:
    
    `123abc456def789ghi`
    
    * A chave sempre é enviada juntamente com as requisições para a API
    

## Qual a necessidade de uma api key?
1. Identificação: a api sabe quem está fazendo o pedido
2. Limite de uso (rate limit): controla quantas requisições você pode fazer por minuto, hora ou dia, evitando abusos
3. Segurança: impede que pessoas aleatórias acessem os dados
4. Cobrança: em apis pagas, a key é usada para rastrear o uso e cobrar conforme o plano

* É importante ressaltar que nem toda API pode necessitar de uma API Key para seu uso, e nem toda API pública pode ser usada sem uma API Key. Por isso é sempre bom ler a documentação da API antes de seu uso para garantir as especificações necessárias para seu uso.

