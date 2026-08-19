# CQRS Design Pattern in .NET 10 Microservices (Without MediatR)

## What is CQRS?
CQRS (Command Query Responsibility Segregation) separates:
- Commands -> Change data
- Queries -> Read data

## Why Use CQRS?
- Better separation of concerns
- Easier maintenance
- Independent read/write scaling
- Complex business logic isolation
- Better fit for microservices

## Architecture

Controller
    ↓
Application Service
    ↓
Command / Query
    ↓
Handler
    ↓
Repository
    ↓
EF Core
    ↓
SQL Server

## Solution Creation

```bash
mkdir CQRS-Microservices
cd CQRS-Microservices

dotnet new sln -n CQRS.Microservices
dotnet new webapi -n OrderService --framework net10.0
dotnet sln add OrderService/OrderService.csproj
```

## Install EF Core

```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.EntityFrameworkCore.Tools
```

## Install EF CLI

```bash
dotnet tool install --global dotnet-ef
```

## Create Migration

```bash
dotnet ef migrations add InitialCreate
```

## Update Database

```bash
dotnet ef database update
```

## Folder Structure

```text
Controllers
Application
  Commands
  Queries
  Services
Domain
Infrastructure
  Data
  Repositories
DTOs
Migrations
```

## CQRS Flow

POST /api/orders

Controller
→ Service
→ CreateOrderCommand
→ CreateOrderCommandHandler
→ Repository
→ EF Core
→ SQL Server

GET /api/orders/1

Controller
→ Service
→ GetOrderByIdQuery
→ GetOrderByIdQueryHandler
→ Repository
→ EF Core
→ SQL Server

## Swagger

```csharp
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
```

## Interview Questions

1. What is CQRS?
2. CQRS vs CRUD?
3. CQRS vs Event Sourcing?
4. Does CQRS require two databases?
5. Benefits of CQRS in Microservices?
