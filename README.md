# Boas práticas com DynamoDB - desafio

Repositório para o desafio sobre o Amazon DynamoDB:

- Na pasta **projeto-music** estão os arquivos do projeto desenvolvido no curso pelo especialista [Cassiano Peres](https://github.com/cassianobrexbit) 
- E na pasta **desafio-menu** estão os arquivos do projeto desenvolvido por mim. Tendo como referência o repositório [dio-live-dynamodb](https://github.com/cassianobrexbit/dio-live-dynamodb)


## Comandos para execução do experimento:
- Criar uma tabela

```
aws dynamodb create-table \
    --table-name Menu \
    --attribute-definitions \
        AttributeName=Identifier,AttributeType=S \
        AttributeName=Price,AttributeType=N \
    --key-schema \
        AttributeName=Identifier,KeyType=HASH \
        AttributeName=Price,KeyType=RANGE \
    --provisioned-throughput \
        ReadCapacityUnits=10,WriteCapacityUnits=5
```

- Inserir um item

```
aws dynamodb put-item \
    --table-name Menu \
    --item file://itemmenu.json \
```

- Inserir múltiplos itens

```
aws dynamodb batch-write-item \
    --request-items file://batchmenu.json
```	

- Pesquisar item pelo identificador (comida)

```
aws dynamodb query \
    --table-name Menu \
    --key-condition-expression "Identifier = :food" \
    --expression-attribute-values  '{":food":{"S":"Food"}}'
```

- Pesquisar item pelo identificador (comida) e preço (lasanha)

```
aws dynamodb query \
    --table-name Menu \
    --key-condition-expression "Identifier = :food and Price = :price" \
    --expression-attribute-values file://keyconditions.json
```

- Criar um index global secundário baeado no título

```
aws dynamodb update-table \
    --table-name Menu \
    --attribute-definitions AttributeName=Title,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"Title-index\",\"KeySchema\":[{\"AttributeName\":\"Title\",\"KeyType\":\"HASH\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

- Pesquisa pelo index secundário baseado no título

```
aws dynamodb query \
    --table-name Menu \
    --index-name Title-index \
    --key-condition-expression "Title = :title" \
    --expression-attribute-values  '{":title":{"S":"Soda"}}'		
```

- Criar um index global secundário baseado no identificador e no preço

```
aws dynamodb update-table \
    --table-name Menu \
    --attribute-definitions\
        AttributeName=Identifier,AttributeType=S \
        AttributeName=Price,AttributeType=N \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"IdentifierPrice-index\",\"KeySchema\":[{\"AttributeName\":\"Identifier\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"Price\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```		
		
- Pesquisa pelo index secundário baseado no identificador e no preço

```
aws dynamodb query \
    --table-name Menu \
    --index-name IdentifierPrice-index \
    --key-condition-expression "Identifier = :identifier and Price = :price" \
    --expression-attribute-values  '{":identifier":{"S":"Drink"},":price":{"N":"3"} }'
```