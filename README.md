
# Projeto Clean Architecture

Este projeto é um serviço de pedidos (Order Service) desenvolvido em Go, que expõe APIs REST, GraphQL e gRPC para manipulação de dados de pedidos em um banco de dados PostgreSQL. A arquitetura segue princípios limpos e separação de responsabilidades entre camadas como handler, service, repository e model.

## Estrutura de Diretórios

```
clean-architecture/
├── internal/
│   ├── handler/         # Endpoints REST, GraphQL e gRPC
│   ├── model/           # Modelos de dados
│   ├── repository/      # Acesso ao banco de dados
│   └── service/         # Regras de negócio
├── migrations/          # Scripts SQL para criação das tabelas
├── proto/               # Definições gRPC
├── Dockerfile
├── docker-compose.yml
├── go.mod
├── go.sum
└── README.md
```

## Requisitos

- Go 1.21 ou superior
- Docker e Docker Compose
- PostgreSQL (rodando localmente ou via container)
- Ferramenta de migração [golang-migrate](https://github.com/golang-migrate/migrate)
- Ferramenta para acessar gRPC via terminal [grpcurl](https://github.com/fullstorydev/grpcurl), ou
- Ferramenta para acessar gRPC via web-interface [grpcui](https://github.com/fullstorydev/grpcui)

## Instalação

1. Clone o repositório:

```bash
git clone <url-do-repositorio>
cd clean-architecture
```

2. Baixe as dependências Go:

```bash
go mod tidy
```

## Execução com Docker Compose

1. Configure as variáveis de ambiente (se necessário):

```env
POSTGRES_USER=user
POSTGRES_PASSWORD=password
POSTGRES_DB=orders
```

2. Suba os containers:

```bash
docker compose up --build
```

Isso irá iniciar o banco de dados PostgreSQL e o serviço Go com suporte a REST, GraphQL e gRPC.

## Migrações de Banco de Dados

1. Instale o CLI do [golang-migrate](https://github.com/golang-migrate/migrate/releases).

2. Execute a migração com:

```bash
migrate -path migrations -database "postgres://user:password@localhost:5432/orders?sslmode=disable" up
```

> Substitua `user`, `password` e `orders` conforme o seu ambiente Docker.

## Testando o Projeto

### Verificar se o banco de dados foi criado

Dentro do container do Postgres:

```bash
docker exec -it clean-architecture-postgres-1 psql -U user -d orders
```

Saída em caso de sucesso:

```bash
psql (15.13)
Type "help" for help.
orders=#
```

### Inserir dados para teste

Ainda dentro do Postgres:

```sql
INSERT INTO orders (product_name, quantity, created_at) VALUES ('Notebook', 2, NOW());
```

### Testar API REST

Endpoint disponível na porta `8080`.
Teste via navegador acessando a url: [http://localhost:8080/order](http://localhost:8080/order)

Ou via terminal com o comando "**curl**":

```bash
curl http://localhost:8080/order
```

Saída impressa:

```bash
[{"id":1,"product_name":"Produto A","quantity":5,"created_at":"2025-05-21T14:27:04.752741Z"},{"id":2,"product_name":"Produto B","quantity":10,"created_at":"2025-05-21T14:27:17.325009Z"}]
```


### Testar GraphQL

GraphQL disponível na porta `8081`.
Teste via navegador acessando a url: [http://localhost:8081](http://localhost:8081)
Utilize a seguinte quwey para teste:

```graphql
query {
  orders {
    id
    productName
    quantity
    createdAt
  }
}
```

Ou via terminal com o comando "**curl**":

```bash
curl -X POST http://localhost:8081/graphql \
  -H "Content-Type: application/json" \
  -d '{"query":"query { orders { id productName quantity createdAt } }"}'
```

Repare que a saída será o conteúdo do arquivo html apresentando pelo GraphQL

### Testar gRPC

gRPC disponível na porta `50051`.

Acessando com **grpcurl**:

Entrada no terminal:

```bash
grpcurl -plaintext localhost:50051 list
```

Saída no terminal:

```bash
grpc.reflection.v1.ServerReflection
grpc.reflection.v1alpha.ServerReflection
order.OrderService
```

Entrada no terminal:

```bash
grpcurl -plaintext localhost:50051 order.OrderService/ListOrders
```

Saída no terminal:

```bash
{
  "orders": [
    {
      "id": 1,
      "productName": "Produto A",
      "quantity": 5,
      "createdAt": "2025-05-21T14:27:04Z"
    },
    {
      "id": 2,
      "productName": "Produto B",
      "quantity": 10,
      "createdAt": "2025-05-21T14:27:17Z"
    }
  ]
}
```

Acessando com **grpcui**:

Entrada no terminal:

```bash
grpcui -plaintext localhost:50051
```

Será iniciado um servidor web local para acessar o serviço gRPC via navagador.

## Possíveis Erros e Soluções

- **Erro de autenticação ao conectar no Postgres:**
  Certifique-se de usar o usuário e senha corretos definidos no `docker-compose.yml`.

- **gRPC não responde ou dá erro de reflexão:**
  Verifique se a reflexão está habilitada no servidor gRPC (`reflection.Register(server)` foi chamado).

- **Erro 404 ao acessar endpoints REST ou GraphQL:**
  Certifique-se de que os containers estão rodando (`docker ps`) e que está acessando a porta correta.

- **Erro: relation "orders" does not exist**  
  Solução: Certifique-se de que a migração foi executada com sucesso e que a tabela `orders` foi criada.

- **Erro: password authentication failed for user "postgres"**  
  Solução: Verifique se o usuário/postgres está correto nas variáveis de ambiente do Docker.

- **Erro ao rodar `migrate`**  
  Solução: Confirme que o CLI `migrate` é compatível com a sua arquitetura. Baixe a versão correta e torne o binário executável.

## Considerações Finais

Este projeto demonstra a integração de REST, GraphQL e gRPC em um único serviço com persistência em PostgreSQL. É uma base sólida para construir microsserviços robustos com suporte a múltiplas interfaces de comunicação.
