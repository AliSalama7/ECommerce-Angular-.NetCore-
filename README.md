# ECommerce Backend API

A layered, production-style e-commerce backend built with **ASP.NET Core (.NET 8)**, following clean architecture principles (API / Core / Infrastructure). It provides authentication, product catalog, basket, order, and payment functionality for a full e-commerce storefront.

## Features

- **Authentication & Identity** — Register/login with ASP.NET Core Identity, JWT bearer tokens, and user address management
- **Product Catalog** — Browse products with filtering, sorting, pagination, brands, and types (via the Specification pattern)
- **Shopping Basket** — Redis-backed basket for fast, session-independent cart storage
- **Orders** — Create orders, view order history, retrieve delivery methods
- **Payments** — Stripe integration for payment intents and webhook handling
- **Clean Architecture** — Separation of concerns across API, Core (domain/business logic), and Infrastructure (data access/external services)
- **Generic Repository + Unit of Work** pattern for data access
- **Global exception handling middleware** with consistent API error responses
- **Swagger/OpenAPI** documentation with JWT auth support built in

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | ASP.NET Core 8 (.NET 8) |
| Database | SQLite (Entity Framework Core) |
| Caching / Basket store | Redis |
| Auth | ASP.NET Core Identity + JWT |
| Payments | Stripe |
| Object Mapping | AutoMapper |
| API Docs | Swagger / Swashbuckle |
| Containerization | Docker Compose (Redis + Redis Commander) |

## Project Structure

```
API/
├── api/                    # Presentation layer (ASP.NET Core Web API)
│   ├── Controllers/        # Account, Product, Basket, Orders, Payments
│   ├── DTOS/                # Data transfer objects
│   ├── Errors/              # API error response models
│   ├── Extensions/          # Extension methods (e.g. ClaimsPrincipal helpers)
│   ├── Middlewares/         # Global exception handling
│   └── Helpers/
├── core/                   # Domain layer
│   ├── Models/              # Product, ProductBrand, ProductType, Basket, OrderAggregate
│   ├── Identity/             # AppUser, Address
│   ├── Interfaces/           # Repository & service contracts
│   └── Specifications/       # Query specification pattern (filtering/sorting/paging)
├── infrastructure/         # Data access & external services
│   ├── Data/                 # DbContext, Migrations, Repositories, Seed data
│   ├── Identity/              # Identity DbContext & migrations
│   └── Services/              # TokenService, OrderService, PaymentService
└── docker-compose.yml       # Redis + Redis Commander
```

## Architecture Overview

The project follows a **3-layer architecture**:

- **Core** — Domain entities and interfaces, with no dependencies on other layers
- **Infrastructure** — Implements Core interfaces: EF Core repositories, Identity, Stripe payment service, JWT token service
- **API** — Controllers, DTOs, and presentation concerns; depends on Core and Infrastructure

Data access uses a **Generic Repository** combined with the **Specification pattern**, allowing flexible, reusable queries (e.g. filtering products by brand/type, sorting, pagination) without leaking query logic into controllers. A **Unit of Work** coordinates repository operations and commits changes as a single transaction.

## Getting Started

### Prerequisites

- [.NET 8 SDK](https://dotnet.microsoft.com/download)
- [Docker](https://www.docker.com/) (for Redis)
- A [Stripe](https://stripe.com/) account (test mode keys) for payment functionality

### 1. Clone the repository

```bash
git clone https://github.com/AliSalama7/ECommerceBackEnd.git
cd ECommerceBackEnd/API
```

### 2. Start Redis

```bash
docker-compose up -d
```

This spins up:
- **Redis** on port `6379`
- **Redis Commander** (UI) on port `8081`

### 3. Configure application settings

Create/update `api/appsettings.Development.json` (do **not** commit real secrets to `appsettings.json`):

```json
{
  "ConnectionStrings": {
    "StoreConnection": "Data source = Ecommerce.db",
    "IdentityConnection": "Data source = Identity.db",
    "Redis": "localhost"
  },
  "Token": {
    "Key": "<your-jwt-signing-key>",
    "Issuer": "https://localhost:7184/"
  }
}
```

### 4. Apply database migrations

```bash
cd api
dotnet ef database update --context StoreContext
dotnet ef database update --context AppIdentityDbContext
```

### 5. Run the API

```bash
dotnet run --project api
```

The API will be available at `https://localhost:7184` (or the port configured in `launchSettings.json`), with Swagger UI at `/swagger`.

## API Endpoints

### Account (`/api/account`)
| Method | Endpoint | Description |
|---|---|---|
| POST | `/login` | Log in and receive a JWT |
| POST | `/register` | Register a new user |
| GET | `/` | Get current authenticated user |
| GET | `/IsEmailExisted?email=` | Check if an email is already registered |
| GET | `/address` | Get current user's address |
| PUT | `/address` | Update current user's address *(requires auth)* |

### Products (`/api/product`)
| Method | Endpoint | Description |
|---|---|---|
| GET | `/` | Get all products (supports filtering, sorting, pagination) |
| GET | `/{id}` | Get a single product by id |
| GET | `/Brands` | Get all product brands |
| GET | `/Types` | Get all product types |

### Basket (`/api/basket`)
| Method | Endpoint | Description |
|---|---|---|
| GET | `/?id=` | Get basket by id |
| POST | `/` | Create/update basket |
| DELETE | `/?id=` | Delete basket |

### Orders (`/api/orders`)
| Method | Endpoint | Description |
|---|---|---|
| POST | `/` | Create a new order |
| GET | `/` | Get orders for the current user |
| GET | `/{id}` | Get a specific order for the current user |
| GET | `/deliveryMethod` | Get available delivery methods |

### Payments (`/api/payments`)
| Method | Endpoint | Description |
|---|---|---|
| POST | `/{basketId}` | Create or update a Stripe payment intent for a basket |
| POST | `/webhook` | Stripe webhook endpoint for payment events |

## Error Handling

The API uses a global `ExceptionMiddleware` to catch unhandled exceptions and return a consistent JSON error response (`ApiException` / `ApiResponse`), instead of leaking raw stack traces to clients.

## Roadmap / Possible Improvements

- Move secrets out of `appsettings.json` into environment variables or a secrets manager
- Add automated tests (unit/integration)
- Add role-based authorization (e.g. Admin endpoints for managing products)
- Add CI/CD pipeline (build, test, containerize)
- Migrate from SQLite to SQL Server/PostgreSQL for production use
