# Warehouse Payments PHP

Starter em **PHP 8.2** para um sistema de gerenciamento de **warehouse/depósitos**, **estoque** e **pagamentos online**.

O objetivo é oferecer uma base pequena e clara para evoluir para um WMS (Warehouse Management System) real, mantendo o domínio de estoque separado da integração com provedores de pagamento.

## Funcionalidades

- Cadastro e listagem de produtos.
- Cadastro e listagem de depósitos/warehouses.
- Controle de saldo por produto e depósito.
- Entrada, saída e transferência de estoque.
- Histórico de movimentações no banco.
- Criação e acompanhamento de pagamentos.
- Gateway de pagamento desacoplado por interface.
- Gateway `fake` para desenvolvimento e testes locais.
- Webhook com validação simples de assinatura no ambiente de demonstração.
- API JSON e banco SQLite.
- GitHub Actions para validar sintaxe PHP.

## Requisitos

- PHP 8.2 ou superior.
- Composer.
- Extensão PDO SQLite habilitada.

## Instalação

```bash
git clone https://github.com/KeltonAlmeida/warehouse-payments-php
cd warehouse-payments-php
composer install
cp .env.example .env
composer serve
```

A API ficará disponível em:

```text
http://localhost:8000
```

O banco SQLite é criado automaticamente em `storage/warehouse.sqlite` na primeira requisição.

## Rotas

| Método | Rota | Função |
|---|---|---|
| GET | `/health` | Health check |
| GET | `/api/products` | Lista produtos |
| POST | `/api/products` | Cria produto |
| GET | `/api/warehouses` | Lista depósitos |
| POST | `/api/warehouses` | Cria depósito |
| GET | `/api/stocks` | Exibe saldos de estoque |
| POST | `/api/stocks/move` | Entrada, saída ou transferência |
| GET | `/api/payments` | Lista pagamentos |
| POST | `/api/payments/checkout` | Cria checkout |
| POST | `/api/payments/webhook` | Processa atualização do gateway |

Veja exemplos prontos em [`examples/curl.md`](examples/curl.md).

## Estrutura do projeto

```text
warehouse-payments-php/
├── .github/workflows/php.yml
├── database/schema.sql
├── examples/curl.md
├── public/index.php
├── src/
│   ├── Config.php
│   ├── Database.php
│   ├── Http.php
│   ├── Payments/
│   │   ├── FakePaymentGateway.php
│   │   └── PaymentGateway.php
│   ├── Repositories/
│   │   ├── ProductRepository.php
│   │   └── WarehouseRepository.php
│   └── Services/
│       ├── InventoryService.php
│       └── PaymentService.php
├── storage/.gitkeep
├── .env.example
├── .gitignore
├── composer.json
└── README.md
```

## Modelo de estoque

A tabela `stock` guarda o saldo atual de cada produto por depósito. A tabela `stock_movements` registra a trilha de auditoria para entradas, saídas e transferências.

As operações de estoque usam transações de banco para evitar que uma transferência atualize apenas um lado da movimentação.

## Pagamentos online

O projeto usa `PaymentGateway` como contrato. O `FakePaymentGateway` permite testar a aplicação sem usar dinheiro real ou chaves de API.

Para integrar **Mercado Pago**, **Stripe**, **PagSeguro** ou outro provedor, crie uma nova classe que implemente:

```php
interface PaymentGateway
{
    public function createCheckout(string $orderReference, int $amountCents, string $currency): array;
    public function parseWebhook(array $payload, string $signature): array;
    public function name(): string;
}
```

Depois troque a criação do gateway em `public/index.php`. Em produção, armazene tokens e secrets somente em variáveis de ambiente ou no secret manager da infraestrutura.

## Segurança para produção

Este repositório é um starter. Antes de produção, adicione autenticação e autorização por perfil, validação robusta dos payloads, rate limiting, logs estruturados, idempotência nos webhooks, assinatura oficial do provedor de pagamento, proteção de secrets e testes automatizados.

Também é recomendável migrar o banco para PostgreSQL ou MySQL, adicionar migrations e implementar locking/controle de concorrência para movimentações de estoque em alto volume.

## Próximos módulos sugeridos

- Usuários, perfis e permissões.
- Fornecedores e clientes.
- Pedidos de compra e venda.
- Picking, packing e expedição.
- Lotes, validade, serial number e localização/bin.
- Reserva de estoque.
- Inventário cíclico.
- Devoluções.
- Dashboard de indicadores.
- Integração com ERP/e-commerce.
- Conciliação e estorno de pagamentos.

## Licença

MIT.
