# CQRS Design Pattern in .NET 10 — Complete End-to-End Guide

> A production-style CQRS implementation in ASP.NET Core / .NET 10 using **C#**, **EF Core**, **SQL Server**, **Swagger/OpenAPI**, and a clean enterprise-style **Service → Command/Query → Handler → Repository → EF Core** flow.
>
> This implementation intentionally **does not use MediatR**. The goal is to understand the CQRS pattern itself rather than hiding the flow behind a mediator library.

---

## 1. What is CQRS?

CQRS stands for:

**Command Query Responsibility Segregation**

The main idea is to separate:

- **Commands** — operations that change state/data.
- **Queries** — operations that read data.

Instead of putting create, update, delete and read logic into one large service, CQRS organizes the application around the intent of the operation.

```text
                    CQRS
                      |
          +-----------+-----------+
          |                       |
       COMMAND                  QUERY
          |                       |
      Change Data             Read Data
          |                       |
   Create / Update / Delete    Get / Search / List
```

---

# 2. Why Do We Need CQRS?

A traditional CRUD service often looks like:

```text
Controller
    |
OrderService
    |
Repository
    |
Database
```

The service may eventually contain:

```text
CreateOrder()
UpdateOrder()
DeleteOrder()
GetOrder()
GetOrders()
SearchOrders()
GetOrderSummary()
GetCustomerOrders()
GetOrderStatistics()
...
```

As the application grows, the service can become large and difficult to maintain.

CQRS separates the responsibilities:

```text
                    Controller
                        |
              +---------+---------+
              |                   |
          Command Service     Query Service
              |                   |
          Command             Query
              |                   |
           Handler             Handler
              |                   |
        Write Repository    Read Repository
              |                   |
             EF Core / SQL Server
```

---

# 3. What Problems Does CQRS Solve?

CQRS can help with:

- Separation of read and write responsibilities
- Complex business rules
- Large enterprise applications
- Independent optimization of read/write operations
- Independent scaling of read/write workloads
- Cleaner application services
- Better organization of use cases
- Easier testing
- Read models / projections
- Event-driven architectures
- Microservices

CQRS is especially useful when the read model and write model have significantly different requirements.

---

# 4. Does CQRS Always Require Two Databases?

**No.**

CQRS is fundamentally about separating the responsibilities of commands and queries.

You can start with:

```text
Command -> SQL Server
Query   -> SQL Server
```

using the same database.

Later, the architecture can evolve into:

```text
Command Side
     |
Write Database
     |
Events
     |
Message Broker
     |
Read Model
     |
Read Database
```

For this tutorial we intentionally start with **one SQL Server database** so the CQRS concept is easy to understand.

---

# 5. CQRS vs CRUD

| CRUD | CQRS |
|---|---|
| Create, Read, Update, Delete are commonly handled together | Commands and Queries are separated |
| Usually one service handles everything | Separate command/query responsibilities |
| Simple applications | Useful for complex applications |
| Easier to start | More architectural structure |
| Usually one model | Can have separate write/read models |
| Good for simple business logic | Good for complex business workflows |

CQRS is **not automatically better than CRUD**.

For a simple application, CRUD may be the better choice.

---

# 6. Real-World Example

We will build an **Order Management API**.

The API will support:

```text
POST   /api/orders
GET    /api/orders/{id}
GET    /api/orders
PUT    /api/orders/{id}
DELETE /api/orders/{id}
```

The architecture will be:

```text
HTTP Request
     |
     v
Controller
     |
     v
Application Service
     |
     +--------------------------+
     |                          |
     v                          v
Command                     Query
     |                          |
     v                          v
Command Handler            Query Handler
     |                          |
     v                          v
Write Repository           Read Repository
     |                          |
     +------------+-------------+
                  |
                  v
               EF Core
                  |
                  v
             SQL Server
```

---

# 7. Technology Stack

This example uses:

- .NET 10
- ASP.NET Core Web API
- C#
- Entity Framework Core
- SQL Server
- Swagger / OpenAPI
- Dependency Injection
- Repository Pattern
- CQRS
- Async/Await
- Data Annotations / validation
- No MediatR

---

# 8. Prerequisites

Install:

1. .NET 10 SDK
2. SQL Server
3. SQL Server Management Studio (SSMS) or another SQL client
4. Visual Studio / VS Code
5. Git

Verify .NET:

```bash
dotnet --version
```

---

# 9. Create the Solution

Create a folder:

```bash
mkdir CQRS-Microservices
cd CQRS-Microservices
```

Create the solution:

```bash
dotnet new sln -n CQRS.Microservices
```

Create the Web API:

```bash
dotnet new webapi -n OrderService --framework net10.0
```

Add the project to the solution:

```bash
dotnet sln add OrderService/OrderService.csproj
```

Open the project:

```bash
cd OrderService
```

---

# 10. Install Required NuGet Packages

Install EF Core SQL Server:

```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
```

Install EF Core Design:

```bash
dotnet add package Microsoft.EntityFrameworkCore.Design
```

Install EF Core Tools:

```bash
dotnet add package Microsoft.EntityFrameworkCore.Tools
```

If Swagger/OpenAPI packages are not already present in the generated project, install the appropriate OpenAPI/Swagger package for your project template.

Install EF CLI globally if required:

```bash
dotnet tool install --global dotnet-ef
```

Verify:

```bash
dotnet ef --version
```

---

# 11. Recommended Enterprise Folder Structure

Create the following structure:

```text
CQRS.Microservices
│
├── CQRS.Microservices.sln
│
└── OrderService
    │
    ├── Controllers
    │   └── OrdersController.cs
    │
    ├── Application
    │   │
    │   ├── Commands
    │   │   ├── CreateOrder
    │   │   │   ├── CreateOrderCommand.cs
    │   │   │   └── CreateOrderCommandHandler.cs
    │   │   │
    │   │   ├── UpdateOrder
    │   │   │   ├── UpdateOrderCommand.cs
    │   │   │   └── UpdateOrderCommandHandler.cs
    │   │   │
    │   │   └── DeleteOrder
    │   │       ├── DeleteOrderCommand.cs
    │   │       └── DeleteOrderCommandHandler.cs
    │   │
    │   ├── Queries
    │   │   ├── GetOrderById
    │   │   │   ├── GetOrderByIdQuery.cs
    │   │   │   └── GetOrderByIdQueryHandler.cs
    │   │   │
    │   │   └── GetOrders
    │   │       ├── GetOrdersQuery.cs
    │   │       └── GetOrdersQueryHandler.cs
    │   │
    │   ├── Services
    │   │   ├── IOrderCommandService.cs
    │   │   ├── OrderCommandService.cs
    │   │   ├── IOrderQueryService.cs
    │   │   └── OrderQueryService.cs
    │   │
    │   └── DTOs
    │       ├── CreateOrderRequest.cs
    │       ├── UpdateOrderRequest.cs
    │       └── OrderResponse.cs
    │
    ├── Domain
    │   └── Entities
    │       └── Order.cs
    │
    ├── Infrastructure
    │   │
    │   ├── Data
    │   │   └── ApplicationDbContext.cs
    │   │
    │   └── Repositories
    │       ├── IOrderCommandRepository.cs
    │       ├── OrderCommandRepository.cs
    │       ├── IOrderQueryRepository.cs
    │       └── OrderQueryRepository.cs
    │
    ├── Migrations
    │
    ├── Program.cs
    ├── appsettings.json
    └── OrderService.csproj
```

---

# 12. Why This Structure?

The important flow is:

```text
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
```

The controller should not know how a command handler works.

The controller should deal mainly with HTTP concerns.

The application service coordinates the application use case.

The handler contains the command/query execution logic.

The repository deals with persistence.

---

# 13. Create the Domain Entity

Create:

```text
Domain/Entities/Order.cs
```

```csharp
namespace OrderService.Domain.Entities;

public class Order
{
    public int Id { get; set; }

    public string CustomerName { get; set; } = string.Empty;

    public string ProductName { get; set; } = string.Empty;

    public int Quantity { get; set; }

    public decimal UnitPrice { get; set; }

    public decimal TotalAmount { get; set; }

    public DateTime CreatedAt { get; set; }

    public DateTime? UpdatedAt { get; set; }
}
```

---

# 14. Create DTOs

## CreateOrderRequest.cs

```text
Application/DTOs/CreateOrderRequest.cs
```

```csharp
namespace OrderService.Application.DTOs;

public class CreateOrderRequest
{
    public string CustomerName { get; set; } = string.Empty;

    public string ProductName { get; set; } = string.Empty;

    public int Quantity { get; set; }

    public decimal UnitPrice { get; set; }
}
```

## UpdateOrderRequest.cs

```text
Application/DTOs/UpdateOrderRequest.cs
```

```csharp
namespace OrderService.Application.DTOs;

public class UpdateOrderRequest
{
    public string CustomerName { get; set; } = string.Empty;

    public string ProductName { get; set; } = string.Empty;

    public int Quantity { get; set; }

    public decimal UnitPrice { get; set; }
}
```

## OrderResponse.cs

```text
Application/DTOs/OrderResponse.cs
```

```csharp
namespace OrderService.Application.DTOs;

public class OrderResponse
{
    public int Id { get; set; }

    public string CustomerName { get; set; } = string.Empty;

    public string ProductName { get; set; } = string.Empty;

    public int Quantity { get; set; }

    public decimal UnitPrice { get; set; }

    public decimal TotalAmount { get; set; }

    public DateTime CreatedAt { get; set; }

    public DateTime? UpdatedAt { get; set; }
}
```

---

# 15. Create EF Core DbContext

Create:

```text
Infrastructure/Data/ApplicationDbContext.cs
```

```csharp
using Microsoft.EntityFrameworkCore;
using OrderService.Domain.Entities;

namespace OrderService.Infrastructure.Data;

public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(
        DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    public DbSet<Order> Orders => Set<Order>();
}
```

---

# 16. Add Connection String

Open:

```text
appsettings.json
```

Add:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=CQRSOrderDb;Trusted_Connection=True;TrustServerCertificate=True;MultipleActiveResultSets=true"
  }
}
```

For SQL authentication:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=CQRSOrderDb;User Id=sa;Password=YourPassword;TrustServerCertificate=True"
  }
}
```

Do not commit real production passwords to GitHub.

Use environment variables, Azure Key Vault, user secrets, or another secret-management solution in real applications.

---

# 17. Register EF Core in Program.cs

```csharp
using Microsoft.EntityFrameworkCore;
using OrderService.Infrastructure.Data;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

builder.Services.AddDbContext<ApplicationDbContext>(options =>
{
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection"));
});

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

app.MapControllers();

app.Run();
```

---

# 18. Create the Database Using EF Core Migrations

Make sure you are inside the project directory:

```bash
cd OrderService
```

Create migration:

```bash
dotnet ef migrations add InitialCreate
```

This creates the `Migrations` folder.

Then create/update the database:

```bash
dotnet ef database update
```

EF Core will create:

```text
CQRSOrderDb
```

with an `Orders` table.

---

# 19. Verify the Database

Open SQL Server Management Studio.

Connect to SQL Server.

Find:

```text
Databases
    |
    └── CQRSOrderDb
          |
          └── Tables
                |
                └── dbo.Orders
```

The database is now created through EF Core migrations.

---

# 20. Create Command Repository

Create:

```text
Infrastructure/Repositories/IOrderCommandRepository.cs
```

```csharp
using OrderService.Domain.Entities;

namespace OrderService.Infrastructure.Repositories;

public interface IOrderCommandRepository
{
    Task AddAsync(Order order);

    Task<Order?> GetByIdAsync(int id);

    Task UpdateAsync(Order order);

    Task DeleteAsync(Order order);

    Task SaveChangesAsync();
}
```

---

# 21. Implement Command Repository

Create:

```text
Infrastructure/Repositories/OrderCommandRepository.cs
```

```csharp
using Microsoft.EntityFrameworkCore;
using OrderService.Domain.Entities;
using OrderService.Infrastructure.Data;

namespace OrderService.Infrastructure.Repositories;

public class OrderCommandRepository : IOrderCommandRepository
{
    private readonly ApplicationDbContext _context;

    public OrderCommandRepository(ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task AddAsync(Order order)
    {
        await _context.Orders.AddAsync(order);
    }

    public async Task<Order?> GetByIdAsync(int id)
    {
        return await _context.Orders
            .FirstOrDefaultAsync(x => x.Id == id);
    }

    public Task UpdateAsync(Order order)
    {
        _context.Orders.Update(order);

        return Task.CompletedTask;
    }

    public Task DeleteAsync(Order order)
    {
        _context.Orders.Remove(order);

        return Task.CompletedTask;
    }

    public async Task SaveChangesAsync()
    {
        await _context.SaveChangesAsync();
    }
}
```

---

# 22. Create Query Repository

Create:

```text
Infrastructure/Repositories/IOrderQueryRepository.cs
```

```csharp
using OrderService.Domain.Entities;

namespace OrderService.Infrastructure.Repositories;

public interface IOrderQueryRepository
{
    Task<Order?> GetByIdAsync(int id);

    Task<List<Order>> GetAllAsync();
}
```

---

# 23. Implement Query Repository

Create:

```text
Infrastructure/Repositories/OrderQueryRepository.cs
```

```csharp
using Microsoft.EntityFrameworkCore;
using OrderService.Domain.Entities;
using OrderService.Infrastructure.Data;

namespace OrderService.Infrastructure.Repositories;

public class OrderQueryRepository : IOrderQueryRepository
{
    private readonly ApplicationDbContext _context;

    public OrderQueryRepository(ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task<Order?> GetByIdAsync(int id)
    {
        return await _context.Orders
            .AsNoTracking()
            .FirstOrDefaultAsync(x => x.Id == id);
    }

    public async Task<List<Order>> GetAllAsync()
    {
        return await _context.Orders
            .AsNoTracking()
            .OrderByDescending(x => x.Id)
            .ToListAsync();
    }
}
```

Notice the query side uses:

```csharp
AsNoTracking()
```

because the query is only reading data.

---

# 24. Create CreateOrder Command

Create:

```text
Application/Commands/CreateOrder/CreateOrderCommand.cs
```

```csharp
namespace OrderService.Application.Commands.CreateOrder;

public class CreateOrderCommand
{
    public string CustomerName { get; set; } = string.Empty;

    public string ProductName { get; set; } = string.Empty;

    public int Quantity { get; set; }

    public decimal UnitPrice { get; set; }
}
```

A command represents an intention to change data.

Example:

```text
CreateOrderCommand
```

means:

> "I want to create an order."

---

# 25. Create CreateOrder Command Handler

Create:

```text
Application/Commands/CreateOrder/CreateOrderCommandHandler.cs
```

```csharp
using OrderService.Domain.Entities;
using OrderService.Infrastructure.Repositories;

namespace OrderService.Application.Commands.CreateOrder;

public class CreateOrderCommandHandler
{
    private readonly IOrderCommandRepository _repository;

    public CreateOrderCommandHandler(
        IOrderCommandRepository repository)
    {
        _repository = repository;
    }

    public async Task<int> HandleAsync(
        CreateOrderCommand command)
    {
        var order = new Order
        {
            CustomerName = command.CustomerName,
            ProductName = command.ProductName,
            Quantity = command.Quantity,
            UnitPrice = command.UnitPrice,
            TotalAmount = command.Quantity * command.UnitPrice,
            CreatedAt = DateTime.UtcNow
        };

        await _repository.AddAsync(order);

        await _repository.SaveChangesAsync();

        return order.Id;
    }
}
```

The handler contains the command execution logic.

---

# 26. Create UpdateOrder Command

Create:

```text
Application/Commands/UpdateOrder/UpdateOrderCommand.cs
```

```csharp
namespace OrderService.Application.Commands.UpdateOrder;

public class UpdateOrderCommand
{
    public int Id { get; set; }

    public string CustomerName { get; set; } = string.Empty;

    public string ProductName { get; set; } = string.Empty;

    public int Quantity { get; set; }

    public decimal UnitPrice { get; set; }
}
```

---

# 27. Create UpdateOrder Handler

Create:

```text
Application/Commands/UpdateOrder/UpdateOrderCommandHandler.cs
```

```csharp
using OrderService.Infrastructure.Repositories;

namespace OrderService.Application.Commands.UpdateOrder;

public class UpdateOrderCommandHandler
{
    private readonly IOrderCommandRepository _repository;

    public UpdateOrderCommandHandler(
        IOrderCommandRepository repository)
    {
        _repository = repository;
    }

    public async Task<bool> HandleAsync(
        UpdateOrderCommand command)
    {
        var order = await _repository.GetByIdAsync(command.Id);

        if (order == null)
        {
            return false;
        }

        order.CustomerName = command.CustomerName;
        order.ProductName = command.ProductName;
        order.Quantity = command.Quantity;
        order.UnitPrice = command.UnitPrice;
        order.TotalAmount = command.Quantity * command.UnitPrice;
        order.UpdatedAt = DateTime.UtcNow;

        await _repository.UpdateAsync(order);

        await _repository.SaveChangesAsync();

        return true;
    }
}
```

---

# 28. Create DeleteOrder Command

Create:

```text
Application/Commands/DeleteOrder/DeleteOrderCommand.cs
```

```csharp
namespace OrderService.Application.Commands.DeleteOrder;

public class DeleteOrderCommand
{
    public int Id { get; set; }
}
```

---

# 29. Create DeleteOrder Handler

Create:

```text
Application/Commands/DeleteOrder/DeleteOrderCommandHandler.cs
```

```csharp
using OrderService.Infrastructure.Repositories;

namespace OrderService.Application.Commands.DeleteOrder;

public class DeleteOrderCommandHandler
{
    private readonly IOrderCommandRepository _repository;

    public DeleteOrderCommandHandler(
        IOrderCommandRepository repository)
    {
        _repository = repository;
    }

    public async Task<bool> HandleAsync(
        DeleteOrderCommand command)
    {
        var order = await _repository.GetByIdAsync(command.Id);

        if (order == null)
        {
            return false;
        }

        await _repository.DeleteAsync(order);

        await _repository.SaveChangesAsync();

        return true;
    }
}
```

---

# 30. Create GetOrderById Query

Create:

```text
Application/Queries/GetOrderById/GetOrderByIdQuery.cs
```

```csharp
namespace OrderService.Application.Queries.GetOrderById;

public class GetOrderByIdQuery
{
    public int Id { get; set; }
}
```

A query represents an intention to read data.

---

# 31. Create GetOrderById Query Handler

Create:

```text
Application/Queries/GetOrderById/GetOrderByIdQueryHandler.cs
```

```csharp
using OrderService.Application.DTOs;
using OrderService.Infrastructure.Repositories;

namespace OrderService.Application.Queries.GetOrderById;

public class GetOrderByIdQueryHandler
{
    private readonly IOrderQueryRepository _repository;

    public GetOrderByIdQueryHandler(
        IOrderQueryRepository repository)
    {
        _repository = repository;
    }

    public async Task<OrderResponse?> HandleAsync(
        GetOrderByIdQuery query)
    {
        var order = await _repository.GetByIdAsync(query.Id);

        if (order == null)
        {
            return null;
        }

        return new OrderResponse
        {
            Id = order.Id,
            CustomerName = order.CustomerName,
            ProductName = order.ProductName,
            Quantity = order.Quantity,
            UnitPrice = order.UnitPrice,
            TotalAmount = order.TotalAmount,
            CreatedAt = order.CreatedAt,
            UpdatedAt = order.UpdatedAt
        };
    }
}
```

---

# 32. Create GetOrders Query

Create:

```text
Application/Queries/GetOrders/GetOrdersQuery.cs
```

```csharp
namespace OrderService.Application.Queries.GetOrders;

public class GetOrdersQuery
{
}
```

---

# 33. Create GetOrders Query Handler

Create:

```text
Application/Queries/GetOrders/GetOrdersQueryHandler.cs
```

```csharp
using OrderService.Application.DTOs;
using OrderService.Infrastructure.Repositories;

namespace OrderService.Application.Queries.GetOrders;

public class GetOrdersQueryHandler
{
    private readonly IOrderQueryRepository _repository;

    public GetOrdersQueryHandler(
        IOrderQueryRepository repository)
    {
        _repository = repository;
    }

    public async Task<List<OrderResponse>> HandleAsync(
        GetOrdersQuery query)
    {
        var orders = await _repository.GetAllAsync();

        return orders.Select(order => new OrderResponse
        {
            Id = order.Id,
            CustomerName = order.CustomerName,
            ProductName = order.ProductName,
            Quantity = order.Quantity,
            UnitPrice = order.UnitPrice,
            TotalAmount = order.TotalAmount,
            CreatedAt = order.CreatedAt,
            UpdatedAt = order.UpdatedAt
        }).ToList();
    }
}
```

---

# 34. Why Do We Need an Application Service?

A common mistake when learning CQRS is:

```text
Controller
    ↓
Command Handler
```

This works, but in a larger enterprise application we may want the controller to remain focused on HTTP concerns.

Therefore we introduce:

```text
Controller
    ↓
Application Service
    ↓
Command / Query
    ↓
Handler
```

The service becomes the application-level entry point.

---

# 35. Create Command Service Interface

Create:

```text
Application/Services/IOrderCommandService.cs
```

```csharp
using OrderService.Application.Commands.CreateOrder;
using OrderService.Application.Commands.DeleteOrder;
using OrderService.Application.Commands.UpdateOrder;

namespace OrderService.Application.Services;

public interface IOrderCommandService
{
    Task<int> CreateAsync(CreateOrderCommand command);

    Task<bool> UpdateAsync(UpdateOrderCommand command);

    Task<bool> DeleteAsync(DeleteOrderCommand command);
}
```

---

# 36. Implement Command Service

Create:

```text
Application/Services/OrderCommandService.cs
```

```csharp
using OrderService.Application.Commands.CreateOrder;
using OrderService.Application.Commands.DeleteOrder;
using OrderService.Application.Commands.UpdateOrder;

namespace OrderService.Application.Services;

public class OrderCommandService : IOrderCommandService
{
    private readonly CreateOrderCommandHandler _createHandler;
    private readonly UpdateOrderCommandHandler _updateHandler;
    private readonly DeleteOrderCommandHandler _deleteHandler;

    public OrderCommandService(
        CreateOrderCommandHandler createHandler,
        UpdateOrderCommandHandler updateHandler,
        DeleteOrderCommandHandler deleteHandler)
    {
        _createHandler = createHandler;
        _updateHandler = updateHandler;
        _deleteHandler = deleteHandler;
    }

    public async Task<int> CreateAsync(
        CreateOrderCommand command)
    {
        return await _createHandler.HandleAsync(command);
    }

    public async Task<bool> UpdateAsync(
        UpdateOrderCommand command)
    {
        return await _updateHandler.HandleAsync(command);
    }

    public async Task<bool> DeleteAsync(
        DeleteOrderCommand command)
    {
        return await _deleteHandler.HandleAsync(command);
    }
}
```

---

# 37. Create Query Service Interface

Create:

```text
Application/Services/IOrderQueryService.cs
```

```csharp
using OrderService.Application.DTOs;
using OrderService.Application.Queries.GetOrderById;
using OrderService.Application.Queries.GetOrders;

namespace OrderService.Application.Services;

public interface IOrderQueryService
{
    Task<OrderResponse?> GetByIdAsync(
        GetOrderByIdQuery query);

    Task<List<OrderResponse>> GetAllAsync(
        GetOrdersQuery query);
}
```

---

# 38. Implement Query Service

Create:

```text
Application/Services/OrderQueryService.cs
```

```csharp
using OrderService.Application.DTOs;
using OrderService.Application.Queries.GetOrderById;
using OrderService.Application.Queries.GetOrders;

namespace OrderService.Application.Services;

public class OrderQueryService : IOrderQueryService
{
    private readonly GetOrderByIdQueryHandler _getByIdHandler;
    private readonly GetOrdersQueryHandler _getAllHandler;

    public OrderQueryService(
        GetOrderByIdQueryHandler getByIdHandler,
        GetOrdersQueryHandler getAllHandler)
    {
        _getByIdHandler = getByIdHandler;
        _getAllHandler = getAllHandler;
    }

    public async Task<OrderResponse?> GetByIdAsync(
        GetOrderByIdQuery query)
    {
        return await _getByIdHandler.HandleAsync(query);
    }

    public async Task<List<OrderResponse>> GetAllAsync(
        GetOrdersQuery query)
    {
        return await _getAllHandler.HandleAsync(query);
    }
}
```

---

# 39. Dependency Injection Registration

Update `Program.cs`:

```csharp
using Microsoft.EntityFrameworkCore;
using OrderService.Application.Commands.CreateOrder;
using OrderService.Application.Commands.DeleteOrder;
using OrderService.Application.Commands.UpdateOrder;
using OrderService.Application.Queries.GetOrderById;
using OrderService.Application.Queries.GetOrders;
using OrderService.Application.Services;
using OrderService.Infrastructure.Data;
using OrderService.Infrastructure.Repositories;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

builder.Services.AddDbContext<ApplicationDbContext>(options =>
{
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection"));
});

// Repositories
builder.Services.AddScoped<IOrderCommandRepository,
    OrderCommandRepository>();

builder.Services.AddScoped<IOrderQueryRepository,
    OrderQueryRepository>();

// Command Handlers
builder.Services.AddScoped<CreateOrderCommandHandler>();
builder.Services.AddScoped<UpdateOrderCommandHandler>();
builder.Services.AddScoped<DeleteOrderCommandHandler>();

// Query Handlers
builder.Services.AddScoped<GetOrderByIdQueryHandler>();
builder.Services.AddScoped<GetOrdersQueryHandler>();

// Application Services
builder.Services.AddScoped<IOrderCommandService,
    OrderCommandService>();

builder.Services.AddScoped<IOrderQueryService,
    OrderQueryService>();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

app.MapControllers();

app.Run();
```

---

# 40. Create Orders Controller

Create:

```text
Controllers/OrdersController.cs
```

```csharp
using Microsoft.AspNetCore.Mvc;
using OrderService.Application.Commands.CreateOrder;
using OrderService.Application.Commands.DeleteOrder;
using OrderService.Application.Commands.UpdateOrder;
using OrderService.Application.Queries.GetOrderById;
using OrderService.Application.Queries.GetOrders;
using OrderService.Application.Services;
using OrderService.Application.DTOs;

namespace OrderService.Controllers;

[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    private readonly IOrderCommandService _commandService;
    private readonly IOrderQueryService _queryService;

    public OrdersController(
        IOrderCommandService commandService,
        IOrderQueryService queryService)
    {
        _commandService = commandService;
        _queryService = queryService;
    }

    [HttpPost]
    public async Task<IActionResult> Create(
        CreateOrderRequest request)
    {
        var command = new CreateOrderCommand
        {
            CustomerName = request.CustomerName,
            ProductName = request.ProductName,
            Quantity = request.Quantity,
            UnitPrice = request.UnitPrice
        };

        var id = await _commandService.CreateAsync(command);

        return CreatedAtAction(
            nameof(GetById),
            new { id },
            new { id });
    }

    [HttpGet("{id:int}")]
    public async Task<IActionResult> GetById(int id)
    {
        var query = new GetOrderByIdQuery
        {
            Id = id
        };

        var result = await _queryService.GetByIdAsync(query);

        if (result == null)
        {
            return NotFound();
        }

        return Ok(result);
    }

    [HttpGet]
    public async Task<IActionResult> GetAll()
    {
        var query = new GetOrdersQuery();

        var result = await _queryService.GetAllAsync(query);

        return Ok(result);
    }

    [HttpPut("{id:int}")]
    public async Task<IActionResult> Update(
        int id,
        UpdateOrderRequest request)
    {
        var command = new UpdateOrderCommand
        {
            Id = id,
            CustomerName = request.CustomerName,
            ProductName = request.ProductName,
            Quantity = request.Quantity,
            UnitPrice = request.UnitPrice
        };

        var updated = await _commandService.UpdateAsync(command);

        if (!updated)
        {
            return NotFound();
        }

        return NoContent();
    }

    [HttpDelete("{id:int}")]
    public async Task<IActionResult> Delete(int id)
    {
        var command = new DeleteOrderCommand
        {
            Id = id
        };

        var deleted = await _commandService.DeleteAsync(command);

        if (!deleted)
        {
            return NotFound();
        }

        return NoContent();
    }
}
```

---

# 41. Final End-to-End Architecture

The complete application now looks like:

```text
                           CLIENT
                             |
                             | HTTP
                             v
                     OrdersController
                             |
                +------------+------------+
                |                         |
                v                         v
        OrderCommandService        OrderQueryService
                |                         |
                |                         |
        +-------+-------+          +------+------+
        |       |       |          |             |
        v       v       v          v             v
      Create  Update  Delete    GetById        GetAll
      Command Command Command    Query          Query
        |       |       |          |             |
        v       v       v          v             v
     Handler Handler Handler     Handler       Handler
        |       |       |          |             |
        +-------+-------+          +------+------+
                |                         |
                v                         v
       Command Repository        Query Repository
                |                         |
                +------------+------------+
                             |
                             v
                         EF Core
                             |
                             v
                        SQL Server
```

---

# 42. Complete POST Request Flow

Request:

```http
POST /api/orders
```

Body:

```json
{
  "customerName": "John",
  "productName": "Laptop",
  "quantity": 2,
  "unitPrice": 50000
}
```

Flow:

```text
POST /api/orders
       |
       v
OrdersController.Create()
       |
       v
CreateOrderRequest
       |
       v
CreateOrderCommand
       |
       v
OrderCommandService.CreateAsync()
       |
       v
CreateOrderCommandHandler.HandleAsync()
       |
       v
Order entity created
       |
       v
IOrderCommandRepository.AddAsync()
       |
       v
EF Core
       |
       v
SQL Server
       |
       v
Orders table
```

The database record will contain approximately:

```text
CustomerName = John
ProductName  = Laptop
Quantity     = 2
UnitPrice    = 50000
TotalAmount  = 100000
```

---

# 43. Complete GET Request Flow

Request:

```http
GET /api/orders/1
```

Flow:

```text
GET /api/orders/1
       |
       v
OrdersController.GetById()
       |
       v
GetOrderByIdQuery
       |
       v
OrderQueryService.GetByIdAsync()
       |
       v
GetOrderByIdQueryHandler
       |
       v
IOrderQueryRepository
       |
       v
EF Core
       |
       v
SQL Server
       |
       v
Order entity
       |
       v
OrderResponse
       |
       v
HTTP 200
```

---

# 44. Command Side vs Query Side

## Command Side

Used for:

```text
Create
Update
Delete
```

Flow:

```text
Controller
    ↓
Command Service
    ↓
Command
    ↓
Command Handler
    ↓
Command Repository
    ↓
EF Core
    ↓
Database
```

## Query Side

Used for:

```text
Get By Id
Get All
Search
Reports
Read Models
```

Flow:

```text
Controller
    ↓
Query Service
    ↓
Query
    ↓
Query Handler
    ↓
Query Repository
    ↓
EF Core
    ↓
Database
```

---

# 45. Why Separate Command and Query Repositories?

This is an important architectural decision.

Instead of:

```text
IOrderRepository
    |
    +-- Add
    +-- Update
    +-- Delete
    +-- Get
    +-- Search
    +-- GetReport
```

we have:

```text
IOrderCommandRepository
    |
    +-- Add
    +-- Update
    +-- Delete

IOrderQueryRepository
    |
    +-- Get
    +-- Search
    +-- List
```

This makes the responsibility explicit.

---

# 46. Why Use AsNoTracking() for Queries?

For read-only operations:

```csharp
_context.Orders
    .AsNoTracking()
    .ToListAsync();
```

EF Core does not need to track entities when we do not intend to modify them.

This can reduce unnecessary change-tracking overhead for read-heavy workloads.

---

# 47. Swagger Testing

Run:

```bash
dotnet run
```

The console will show the application URL.

Open Swagger, for example:

```text
https://localhost:<port>/swagger
```

You should see:

```text
Orders

POST   /api/Orders
GET    /api/Orders
GET    /api/Orders/{id}
PUT    /api/Orders/{id}
DELETE /api/Orders/{id}
```

---

# 48. Test Create Order

Use:

```http
POST /api/Orders
```

Request:

```json
{
  "customerName": "Nikhil",
  "productName": "MacBook",
  "quantity": 1,
  "unitPrice": 120000
}
```

Expected response:

```json
{
  "id": 1
}
```

---

# 49. Test Get Order

```http
GET /api/Orders/1
```

Expected response:

```json
{
  "id": 1,
  "customerName": "Nikhil",
  "productName": "MacBook",
  "quantity": 1,
  "unitPrice": 120000,
  "totalAmount": 120000,
  "createdAt": "2026-08-19T...",
  "updatedAt": null
}
```

---

# 50. Test Get All Orders

```http
GET /api/Orders
```

Expected response:

```json
[
  {
    "id": 1,
    "customerName": "Nikhil",
    "productName": "MacBook",
    "quantity": 1,
    "unitPrice": 120000,
    "totalAmount": 120000
  }
]
```

---

# 51. Test Update

```http
PUT /api/Orders/1
```

Body:

```json
{
  "customerName": "Nikhil",
  "productName": "MacBook Pro",
  "quantity": 2,
  "unitPrice": 150000
}
```

The handler recalculates:

```text
TotalAmount = Quantity × UnitPrice

            = 2 × 150000

            = 300000
```

---

# 52. Test Delete

```http
DELETE /api/Orders/1
```

Expected response:

```text
204 No Content
```

---

# 53. EF Core Migration Workflow

Whenever the entity model changes:

Example:

```csharp
public string Status { get; set; }
```

Create a migration:

```bash
dotnet ef migrations add AddOrderStatus
```

Apply it:

```bash
dotnet ef database update
```

Complete workflow:

```text
Modify Entity
     ↓
dotnet ef migrations add MigrationName
     ↓
Migration Generated
     ↓
dotnet ef database update
     ↓
SQL Server Updated
```

---

# 54. Useful EF Core Commands

Create migration:

```bash
dotnet ef migrations add InitialCreate
```

Update database:

```bash
dotnet ef database update
```

List migrations:

```bash
dotnet ef migrations list
```

Remove last migration:

```bash
dotnet ef migrations remove
```

Generate SQL:

```bash
dotnet ef migrations script
```

---

# 55. CQRS Without MediatR

MediatR is commonly used to implement mediator-based CQRS.

For example:

```text
Controller
    ↓
Mediator.Send(command)
    ↓
Handler
```

In this tutorial:

```text
Controller
    ↓
Application Service
    ↓
Command
    ↓
Handler
```

This makes the underlying CQRS flow explicit.

You can later introduce MediatR if your project benefits from it.

---

# 56. CQRS Is Not the Same as MediatR

This is an important interview point.

```text
CQRS = Architectural Pattern

MediatR = Library / Mediator Implementation
```

CQRS does not require MediatR.

You can implement CQRS manually.

---

# 57. CQRS Is Not Event Sourcing

These are different concepts.

## CQRS

Separates:

```text
Commands
Queries
```

## Event Sourcing

Stores state changes as events.

Example:

```text
OrderCreated
OrderUpdated
OrderCancelled
```

Event sourcing can be combined with CQRS, but it is not required.

---

# 58. CQRS and Event-Driven Microservices

In a more advanced architecture:

```text
Order Service
     |
     | Command
     v
Command Handler
     |
     v
Write Database
     |
     v
OrderCreated Event
     |
     v
Message Broker
     |
     +-------------------+
     |                   |
     v                   v
Inventory Service    Notification Service
     |                   |
     v                   v
Inventory DB         Email/SMS
```

Possible technologies:

- Azure Service Bus
- RabbitMQ
- Kafka
- MassTransit
- Amazon SQS/SNS

---

# 59. CQRS in Microservices

A realistic microservice architecture could be:

```text
                    API Gateway
                         |
       +-----------------+-----------------+
       |                 |                 |
       v                 v                 v
 Order Service     Inventory Service   Payment Service
       |                 |                 |
       v                 v                 v
 Order DB          Inventory DB        Payment DB
```

Each service owns its database.

Inside each microservice:

```text
Controller
    ↓
Application Service
    ↓
CQRS
    ↓
Repository
    ↓
Database
```

---

# 60. Scaling Read and Write Operations

One of the major benefits of CQRS appears when read and write workloads differ.

Example:

```text
100,000 GET requests
10,000 POST/PUT requests
```

The read side may require more resources.

With CQRS and separate read/write infrastructure:

```text
                Load Balancer
                     |
          +----------+----------+
          |                     |
          v                     v
     Read Service          Command Service
          |                     |
          v                     v
     Read Database         Write Database
```

The exact architecture depends on business requirements.

---

# 61. Advanced CQRS Read Model

For reporting-heavy systems, the query side can use a specialized read model.

Example:

```text
Orders
Customers
Products
Payments
```

might normally require multiple joins.

Instead, create:

```text
OrderSummaryReadModel
```

Example:

```text
OrderId
CustomerName
ProductName
Quantity
TotalAmount
PaymentStatus
ShippingStatus
```

The read model is optimized for the UI/report.

---

# 62. CQRS with Separate Databases

Advanced architecture:

```text
                 Command
                    |
                    v
              Command Handler
                    |
                    v
              Write Database
                    |
                    v
                  Event
                    |
                    v
              Message Broker
                    |
                    v
              Query Projection
                    |
                    v
               Read Database
                    |
                    v
                  Query
```

This allows the read side to be optimized independently.

But it introduces:

- Eventual consistency
- Synchronization complexity
- Operational complexity
- Duplicate infrastructure
- More monitoring requirements

Do not introduce this architecture unless the business actually needs it.

---

# 63. CQRS and Eventual Consistency

When separate read/write databases are used:

```text
Write DB
   |
   | Event
   v
Message Broker
   |
   v
Read Model
```

The read database may not be updated immediately.

For a short period:

```text
Write DB = Updated
Read DB  = Old
```

This is called:

**Eventual Consistency**

This is one of the most important trade-offs of advanced CQRS.

---

# 64. CQRS and SAGA

CQRS can also be combined with the SAGA pattern.

Example:

```text
Create Order
    |
    v
Reserve Inventory
    |
    v
Process Payment
    |
    v
Confirm Order
```

If payment fails:

```text
Payment Failed
     |
     v
Release Inventory
     |
     v
Cancel Order
```

CQRS handles command/query separation.

SAGA handles distributed business transactions.

They solve different problems.

---

# 65. CQRS vs SAGA

| CQRS | SAGA |
|---|---|
| Separates reads and writes | Manages distributed transactions |
| Command/Query responsibility | Business transaction orchestration |
| Improves application organization | Handles failures across services |
| Can use one database | Usually relevant to distributed systems |
| Does not require messaging | Commonly uses messaging/events |

---

# 66. CQRS Benefits

### 1. Separation of Concerns

Read and write operations are separated.

### 2. Maintainability

Each use case has a focused handler.

### 3. Scalability

Read and write workloads can be optimized independently.

### 4. Complex Business Logic

Commands can encapsulate complex business rules.

### 5. Better Read Models

Queries can use optimized projections.

### 6. Microservices Friendly

CQRS works well with event-driven architectures.

### 7. Testability

Handlers can be tested independently.

---

# 67. CQRS Disadvantages

CQRS introduces additional complexity.

Potential disadvantages:

- More classes
- More files
- More abstractions
- More code
- Higher learning curve
- Debugging can involve multiple layers
- Separate read/write databases increase operational complexity
- Eventual consistency can be difficult
- Over-engineering risk

For a simple CRUD application, CQRS may be unnecessary.

---

# 68. When Should You Use CQRS?

Good candidates:

- Large enterprise systems
- Complex business workflows
- High read/write differences
- Complex domain models
- Reporting-heavy applications
- Event-driven microservices
- Systems requiring independent read/write scaling
- Systems where different models are needed for reads and writes

---

# 69. When Should You NOT Use CQRS?

Avoid CQRS when:

- Application is very small
- CRUD is straightforward
- Business rules are simple
- Team does not need the additional architecture
- Read/write requirements are almost identical
- You are adding CQRS only because it is popular

A simple CRUD service is often the correct architecture for a simple application.

---

# 70. Important Interview Questions

## 1. What is CQRS?

CQRS stands for Command Query Responsibility Segregation and separates operations that modify data from operations that read data.

---

## 2. Does CQRS require two databases?

No.

CQRS can be implemented with one database or separate read/write databases.

---

## 3. Does CQRS require MediatR?

No.

MediatR is an implementation option, not a requirement.

---

## 4. What is a Command?

A command represents an intention to change state.

Examples:

```text
CreateOrder
UpdateOrder
DeleteOrder
```

---

## 5. What is a Query?

A query represents an intention to retrieve data.

Examples:

```text
GetOrderById
GetOrders
SearchOrders
```

---

## 6. Can a Query modify the database?

Normally, a query should not modify application state.

A query is intended to read data.

---

## 7. Can a Command return data?

Yes.

For example:

```text
CreateOrderCommand
```

may return:

```text
OrderId
```

The important distinction is that the command's purpose is to change state.

---

## 8. What is the difference between CQRS and CRUD?

CRUD generally combines read/write responsibilities in a common service/model.

CQRS explicitly separates commands and queries.

---

## 9. What is CQRS vs Event Sourcing?

CQRS separates read/write responsibilities.

Event sourcing stores state changes as events.

They can be used together but are independent patterns.

---

## 10. What is eventual consistency?

When separate read/write models are synchronized asynchronously, the read model may temporarily contain stale data before eventually becoming consistent.

---

## 11. Why use AsNoTracking()?

For read-only queries, `AsNoTracking()` avoids unnecessary EF Core change tracking.

---

## 12. Why separate repositories?

Separate repositories make command and query responsibilities explicit and allow each side to evolve independently.

---

## 13. Can CQRS improve performance?

It can.

CQRS itself is not automatically a performance optimization. The architectural separation allows you to optimize read and write paths independently.

---

## 14. Can CQRS be used in a monolith?

Yes.

CQRS does not require microservices.

---

## 15. Can CQRS be used in microservices?

Yes.

It is frequently used in complex microservices and event-driven architectures.

---

# 71. Debugging Checklist

If the API does not start:

```text
Check .NET SDK
Check package restore
Check build errors
Check connection string
```

Run:

```bash
dotnet restore
dotnet build
dotnet run
```

If EF migration fails:

```bash
dotnet ef --version
dotnet ef migrations list
```

Check:

```text
DbContext registration
Connection string
SQL Server availability
Project directory
Startup project
```

If dependency injection fails:

Check that all of these are registered:

```text
Repositories
Handlers
Application Services
DbContext
```

---

# 72. Build and Run

From the project directory:

```bash
dotnet restore
dotnet build
dotnet run
```

Then open Swagger.

---

# 73. Complete Request Lifecycle

## Command Lifecycle

```text
HTTP POST/PUT/DELETE
        |
        v
Controller
        |
        v
Application Command Service
        |
        v
Command
        |
        v
Command Handler
        |
        v
Command Repository
        |
        v
EF Core
        |
        v
SQL Server
```

## Query Lifecycle

```text
HTTP GET
   |
   v
Controller
   |
   v
Application Query Service
   |
   v
Query
   |
   v
Query Handler
   |
   v
Query Repository
   |
   v
EF Core
   |
   v
SQL Server
   |
   v
DTO / Read Model
   |
   v
HTTP Response
```

---

# 74. Complete Project Responsibility Map

| Layer | Responsibility |
|---|---|
| Controller | HTTP/API concerns |
| Application Service | Coordinates application use case |
| Command | Represents write intent |
| Command Handler | Executes command |
| Query | Represents read intent |
| Query Handler | Executes query |
| Command Repository | Write-side persistence |
| Query Repository | Read-side persistence |
| Domain Entity | Business/domain data |
| DTO | API/application data contract |
| DbContext | EF Core database access |
| SQL Server | Persistent storage |

---

# 75. Recommended Enterprise Evolution

Start:

```text
Controller
    ↓
Application Service
    ↓
CQRS Handler
    ↓
Repository
    ↓
SQL Server
```

Then, when requirements justify it:

```text
                    API Gateway
                         |
                    Order Service
                         |
                  +------+------+
                  |             |
              Command        Query
                  |             |
            Write Model     Read Model
                  |             |
              Write DB      Read DB
                  |
                Events
                  |
             Message Broker
                  |
        +---------+---------+
        |                   |
 Inventory Service    Notification Service
```

This should be introduced incrementally rather than from day one.

---

# 76. Key Takeaways

```text
CQRS
 |
 +-- Commands
 |     |
 |     +-- Create
 |     +-- Update
 |     +-- Delete
 |
 +-- Queries
       |
       +-- Get
       +-- Search
       +-- List
       +-- Reports
```

The core principle is:

> **Separate the responsibility of changing data from the responsibility of reading data.**

For the enterprise-style implementation in this guide:

```text
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
```

And the most important point:

> **CQRS is an architectural pattern. MediatR is optional. Separate databases are optional. Event sourcing is optional.**

Use CQRS when the problem justifies the additional complexity.

---

# 77. Suggested Git Commit Sequence

If you are building this project step by step for GitHub or a tutorial, a clean commit history could be:

```text
1. Create .NET 10 Web API solution
2. Add EF Core SQL Server packages
3. Add Order domain entity
4. Configure ApplicationDbContext
5. Configure SQL Server connection string
6. Create InitialCreate migration
7. Create CQRS repository abstractions
8. Implement command repository
9. Implement query repository
10. Add CreateOrder command
11. Add UpdateOrder command
12. Add DeleteOrder command
13. Add GetOrderById query
14. Add GetOrders query
15. Add command handlers
16. Add query handlers
17. Add command application service
18. Add query application service
19. Register dependencies
20. Add OrdersController
21. Add Swagger
22. Test POST
23. Test GET
24. Test PUT
25. Test DELETE
26. Add validation
27. Add logging
28. Add global exception handling
29. Add unit tests
30. Add integration tests
```

---

# 78. Next-Level Improvements for Production

The sample intentionally focuses on understanding CQRS.

For a production application, consider adding:

- FluentValidation
- Global exception middleware
- Structured logging
- Serilog
- Authentication/Authorization
- API versioning
- Health checks
- OpenTelemetry
- Distributed tracing
- Retry policies
- Resilience patterns
- Pagination
- Filtering
- Sorting
- Optimized projections
- Unit tests
- Integration tests
- Docker
- CI/CD
- Azure deployment
- Secrets management
- Database indexing
- Caching
- Message broker
- Outbox pattern
- Idempotency
- SAGA
- Event-driven integration

---

# 79. Final Architecture Summary

```text
                         CLIENT
                           |
                           v
                    ASP.NET Core API
                           |
                           v
                    OrdersController
                           |
             +-------------+-------------+
             |                           |
             v                           v
       Command Service             Query Service
             |                           |
             v                           v
         Command                       Query
             |                           |
             v                           v
       Command Handler             Query Handler
             |                           |
             v                           v
     Command Repository          Query Repository
             |                           |
             +-------------+-------------+
                           |
                           v
                       EF Core
                           |
                           v
                      SQL Server
```

This is the core CQRS implementation.

As the system grows, the architecture can evolve toward:

```text
                    API Gateway
                         |
                +--------+--------+
                |                 |
             Command            Query
                |                 |
           Write Model        Read Model
                |                 |
            Write DB           Read DB
                |
              Events
                |
         Message Broker
                |
       +--------+---------+
       |                  |
 Inventory             Payment
 Service               Service
```

---

# 80. One-Line Definition for Interviews

> **CQRS is an architectural pattern that separates commands that modify application state from queries that retrieve application data, allowing each side to evolve, scale, and optimize independently when required.**

---

## License

This guide is intended for learning, interview preparation, tutorials, and personal projects.
