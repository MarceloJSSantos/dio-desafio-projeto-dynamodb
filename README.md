# dio-desafio-projeto-dynamodb
Repositório do desafio de projeto "Boas práticas com DynamoDB" do Bootcamp Banco "PAN Java Developer"

## Arquitetura de exemplo
![](img/arquitetura-bd.png "arquitetura-bd.png")
**nota**: entidades e atributos traduzidos para pt_br

### Serviços/ferramentas utilizadas
- AWS DynamoDB
- AWS CLI (client) para execução em linha de comando
  - Configurando:
    - Baixar em https://aws.amazon.com/pt/cli/ e instalar
    - ``$ aws config`` passando as credenciais da AWS
    
### Desenvolvimento do desafio

- Criar tabela/coleção 'musica'
``` 
aws dynamodb create-table \
    --table-name Musica \
    --attribute-definitions \
        AttributeName=Artista,AttributeType=S \
        AttributeName=TituloMusica,AttributeType=S \
    --key-schema \
        AttributeName=Artista,KeyType=HASH \
        AttributeName=TituloMusica,KeyType=RANGE \
    --provisioned-throughput \
        ReadCapacityUnits=10,WriteCapacityUnits=5
```

- Inserir 1 item com dados de origem de um arquivo .json

```
aws dynamodb put-item \
    --table-name Musica \
    --item file://itemMusica.json \
```

- Inserir múltiplos itens de um arquico .json

```
aws dynamodb batch-write-item \
    --request-items file://batchMusica.json
```

- Pesquisar item por artista (índice primário)

```
aws dynamodb query \
    --table-name Musica \
    --key-condition-expression "Artista = :artista" \
    --expression-attribute-values  '{":artista":{"S":"Iron Maiden"}}'
```

- Criar index global secundário baeado no título do álbum

```
aws dynamodb update-table \
    --table-name Musica \
    --attribute-definitions AttributeName=TituloAlbum,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"TituloAlbum-index\",\"KeySchema\":[{\"AttributeName\":\"TituloAlbum\",\"KeyType\":\"HASH\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

- Pesquisa pelo index secundário baseado no título do álbum (criado a cima)

```
aws dynamodb query \
    --table-name Musica \
    --index-name TituloAlbum-index \
    --key-condition-expression "TituloAlbum = :nome" \
    --expression-attribute-values  '{":nome":{"S":"Fear of the Dark"}}'
```

- Criar index global secundário baseado no 'nome do artista' e no 'título do álbum' (múltiplos parâmetros)

```
aws dynamodb update-table \
    --table-name Musica \
    --attribute-definitions\
        AttributeName=Artista,AttributeType=S \
        AttributeName=TituloAlbum,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"ArtistaTituloAlbum-index\",\"KeySchema\":[{\"AttributeName\":\"Artista\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"TituloAlbum\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5},\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

- Pesquisa pelo index secundário baseado no 'nome do artista' e no 'título do álbum'

```
aws dynamodb query \
    --table-name Musica \
    --index-name ArtistaTituloAlbum-index \
    --key-condition-expression "Artista = :v_artista and TituloAlbum = :v_tituloAlbum" \
    --expression-attribute-values  '{":v_artista":{"S":"Iron Maiden"},":v_tituloAlbum":{"S":"Fear of the Dark"} }'
```

- Criar index global secundário baseado no 'título da música' e no 'ano'

```
aws dynamodb update-table \
    --table-name Musica \
    --attribute-definitions\
        AttributeName=TituloMusica,AttributeType=S \
        AttributeName=AnoMusica,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"TituloAnoMusica-index\",\"KeySchema\":[{\"AttributeName\":\"TituloMusica\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"AnoMusica\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5},\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

- Pesquisa pelo index secundário baseado no 'título da música' e no 'ano'

```
aws dynamodb query \
    --table-name Musica \
    --index-name TituloAnoMusica-index \
    --key-condition-expression "TituloMusica = :v_titulo_musica and AnoMusica = :v_ano_musica" \
    --expression-attribute-values  '{":v_titulo_musica":{"S":"Wasting Love"},":v_ano_musica":{"S":"1992"} }'
```

- Pesquisar item por 'artista' e 'título da música' (índice primário) com uso de arquivo com 

```
aws dynamodb query \
    --table-name Musica \
    --key-condition-expression "Artista = :artista and TituloMusica = :tituloMusica" \
    --expression-attribute-values file://chavesCondicoes.json
```