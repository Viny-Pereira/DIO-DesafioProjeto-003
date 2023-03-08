# Dio Desafio Amazon DynamoDB

## Introdução

- Criação de banco de dados em Amazon DynamoDB da lista dos 10 filmes mais bem avaliados da história

### Tipos de bancos

#### SQL

- Tem como foco o relacionamento entre as entidades dentro do banco, se baseando no modelo relaciona.
- É necessário determinar uma extrutura previamente, composta de chaves primárias e chaves estrangeiras que ligam as tabelas entre si.
- Exemplos de bancos
  - Microsoft SQL Server, POstgreSQL, Oracle DB e MySQL

#### NoSQL - Não relacional

- Não há relação entre tabelas/entidades, neles os dados são armazenados no modelo par chave/valor.
- Mais otimizado, maiores volumes de dados

- Exemplos de bancos:
  - MongoDB, DynamoDB, Redis

### Amazon DynamoDB introdução

- Serviço NoSQL gerenciado pela AWS, não necessitando de aporte em equipamentos, e cobrança por demanda
- Baixa laténcia e grande volume de dados.

#### Componetes

- Tabela
- Itens
- Atributos
- Chave primária:
  - Partition key
  - Sort key

#### Boas práticas

- Regras de negócio bem definidas
- Pre-dimensionamento adequado
- Minimizar o número de indices secundários
- Access patterns
- Indices secundários
  - Global
  - Local

#### Tipos de Dados

    - Escalares
    - Documentos
    - Conjunto

#### Modo de leitura e gravação

- Sob demanda
- Provisionado

### AWS

- [Cadastro AWS](https://portal.aws.amazon.com/billing/signup#/start/email)
- [AWS Command Line Interface](https://aws.amazon.com/pt/cli/)

## Estrutura base do banco de dados

- Partition key
- Sort Key
- Attributes

### Comandos para execução do experimento:

#### Configuração AWS

- Inserir credenciais:
  `asw configure`

#### Criação do Banco de dados

- Criar uma tabela
  - Nome: Films
  - Atributos: Director(HashKey), Title (SortKey)
  - Critérios de transferência do banco: Capacidade de leitura (10un), Capacidade de Escrita (5un) ;

```
aws dynamodb create-table \
    --table-name Films \
    --attribute-definitions \
        AttributeName=Director,AttributeType=S \
        AttributeName=Title,AttributeType=S \
    --key-schema \
        AttributeName=Director,KeyType=HASH \
        AttributeName=Title,KeyType=RANGE \
    --provisioned-throughput \
        ReadCapacityUnits=10,WriteCapacityUnits=5
```

- Inserir um item a partir de um JSON
- Inserir na Tabela: Films
- Dados contidos no JSON itemmusic.json, contido no mesmo repositório que abriu o AWS

```
aws dynamodb put-item \
    --table-name Films \
    --item file://itemfilm.json \
```

- Inserir múltiplos itens a partir de um arquivo JSON
- Dados contidos no JSON batchmusic.json, contido no mesmo repositório que abriu o AWS

```
aws dynamodb batch-write-item \
    --request-items file://batchfilm.json
```

- Criar um index global secundário baeado no título do álbum
  - Atualizar as informações da tabela
  - Adicionar os Atributos: Gender
  - Configurar o index secundário:
    - Gender como HashKey com 10un de leitura e 5un de escrita

```
aws dynamodb update-table \
    --table-name Films \
    --attribute-definitions AttributeName=Gender,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"Gender-index\",\"KeySchema\":[{\"AttributeName\":\"Gender\",\"KeyType\":\"HASH\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5},\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

- Criar um index global secundário baseado no nome do artista e no título do álbum
  - Atualizar as informações da tabela
  - Adicionar os Atributos: Director, Gender
  - Configurar o index secundário:
    - Gender como HashKey com 10un de leitura e 5un de escrita
    - Director como HashKey, Gender como SortKey
    - Com 10un de leitura e 5un de escrita

```
aws dynamodb update-table \
    --table-name Films \
    --attribute-definitions\
        AttributeName=Director,AttributeType=S \
        AttributeName=Gender,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"DirectorGender-index\",\"KeySchema\":[{\"AttributeName\":\"Director\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"Gender\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5},\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

- Criar um index global secundário baseado no título da música e no ano
  - Atualizar as informações da tabela presente no banco de dado dynamodb
  - Adicionar os Atributos: Title, FilmYear
  - Configurar o index secundário:
    - Title como HashKey, FilmYear como SortKey
    - Com 10un de leitura e 5un de escrita

```
aws dynamodb update-table \
    --table-name Films \
    --attribute-definitions\
        AttributeName=Title,AttributeType=S \
        AttributeName=FilmYear,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"TitleYear-index\",\"KeySchema\":[{\"AttributeName\":\"Title\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"FilmYear\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

- Pesquisar item por artista
  - Buscar na tabela: Films
  - Buscar Director: Francis Ford Coppola

```
aws dynamodb query \
    --table-name Films \
    --key-condition-expression "Director = :director" \
    --expression-attribute-values  '{":director":{"S":"Francis Ford Coppola"}}'
```

- Pesquisar item por artista e título da música
  - Pesquisar na Tabela: Films
  - Por Director e Title contidos no aquivo keyconditions.json

```
aws dynamodb query \
    --table-name Films \
    --key-condition-expression "Director = :director and Title = :title" \
    --expression-attribute-values file://keyconditions.json
```

- Pesquisa pelo index secundário baseado no título do álbum
  - Pesquisar na tabela: Films
  - Por Gender: Drama

```
aws dynamodb query \
    --table-name Films \
    --index-name Gender-index \
    --key-condition-expression "Gender = :name" \
    --expression-attribute-values  '{":name":{"S":"Drama"}}'
```

- Pesquisa pelo index secundário baseado no nome do artista e no título do álbum
  - Pesquisar na tabela: Films
  - Por Gender: Drama e Director: Francis Ford Coppola

```
aws dynamodb query \
    --table-name Films \
    --index-name DirectorGender-index \
    --key-condition-expression "Director = :v_director and Gender = :v_title" \
    --expression-attribute-values  '{":v_director":{"S":"Francis Ford Coppola"},":v_title":{"S":"Drama"} }'
```

- Pesquisa pelo index secundário baseado no título da música e no ano
  - Pesquisar na tabela: Films
  - Por Title: The Dark Knight e FilmYear: 2008

```
aws dynamodb query \
    --table-name Films \
    --index-name FilmsYear-index \
    --key-condition-expression "Title = :v_song and FilmYear = :v_year" \
    --expression-attribute-values  '{":v_song":{"S":"The Dark Knight"},":v_year":{"S":"2008"} }'
```
