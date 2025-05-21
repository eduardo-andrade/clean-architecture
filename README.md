
# Projeto Clean Architecture

Este projeto é um serviço de pedidos (Order Service) desenvolvido em Go, que expõe APIs REST, GraphQL e gRPC para manipulação de dados de pedidos em um banco de dados PostgreSQL. A arquitetura segue princípios limpos e separação de responsabilidades entre camadas como handler, service, repository e model.

## Estrutura de Diretórios

```
order-service/
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

## Instalação

1. Clone o repositório:

```bash
git clone <url-do-repositorio>
cd order-service
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

### REST

- Endpoint: `GET http://localhost:8080/orders`
- Use ferramentas como Postman ou curl:

```bash
curl http://localhost:8080/orders
```

### GraphQL

- Playground: `http://localhost:8080`
- Exemplo de query:

```graphql
query {
  listOrders {
    id
    productName
    quantity
    createdAt
  }
}
```

### gRPC

- Utilize ferramentas como [grpcurl](https://github.com/fullstorydev/grpcurl)
- Endpoint: `localhost:50051`
- Exemplo:

```bash
grpcurl -plaintext localhost:50051 list
```

```bash
grpcurl -plaintext localhost:50051 order.OrderService/ListOrders
```

## Inserção Manual de Dados

Você pode acessar o banco de dados PostgreSQL rodando via Docker:

```bash
docker exec -it <container_id_postgres> psql -U user -d orders
```

Inserir exemplo:

```sql
INSERT INTO orders (product_name, quantity, created_at) VALUES ('Produto Teste', 10, NOW());
```

## Possíveis Erros e Soluções

- **Erro: relation "orders" does not exist**  
  Solução: Certifique-se de que a migração foi executada com sucesso e que a tabela `orders` foi criada.

- **Erro: password authentication failed for user "postgres"**  
  Solução: Verifique se o usuário/postgres está correto nas variáveis de ambiente do Docker.

- **Erro ao rodar `migrate`**  
  Solução: Confirme que o CLI `migrate` é compatível com a sua arquitetura. Baixe a versão correta e torne o binário executável.

## Considerações Finais

Este projeto demonstra a integração de REST, GraphQL e gRPC em um único serviço com persistência em PostgreSQL. É uma base sólida para construir microsserviços robustos com suporte a múltiplas interfaces de comunicação.
