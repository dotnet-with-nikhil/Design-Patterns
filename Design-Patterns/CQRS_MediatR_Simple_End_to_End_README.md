# CQRS + MediatR in .NET 10 — Simple End-to-End Example

A beginner-friendly implementation of **CQRS + MediatR** in ASP.NET Core / .NET 10.

This example intentionally keeps the application simple and implements only:

- Create Order
- Get All Orders
- Get Order By Id

It uses:

- .NET 10
- ASP.NET Core Web API
- C#
- Entity Framework Core
- SQL Server
- EF Core Migrations
- Swagger
- CQRS
- MediatR
- Repository Pattern
- Application Service
- Dependency Injection

The goal is to understand **what MediatR does, why we use it, what problem it solves, and how a request travels through the application**.

---

# 1. What is MediatR?

**MediatR** is a .NET library that implements the **Mediator Pattern**.

It provides a central mechanism through which a request can be sent to the appropriate handler.

Instead of directly calling a specific handler:

```text
Controller
    ↓
CreateOrderCommandHandler
```

we can do:

```text
Controller
    ↓
MediatR
    ↓
CreateOrderCommandHandler
```

The controller does not need to know which concrete handler will execute the command.

---

# 2. What is CQRS?

CQRS stands for:

**Command Query Responsibility Segregation**

It separates:

```text
COMMAND
    ↓
Changes data

QUERY
    ↓
Reads data
```

In this example:

### Commands

```text
CreateOrderCommand
```

### Queries

```text
GetOrdersQuery
GetOrderByIdQuery
```

---

# 3. CQRS and MediatR Are Not the Same

This is one of the most important concepts.

```text
CQRS
    ↓
Architectural Pattern

MediatR
    ↓
Library implementing the Mediator Pattern
```

CQRS does not require MediatR.

You can implement CQRS manually.

You can also use MediatR to simplify command/query dispatching.

---

# 4. What Problem Does MediatR Solve?

Without MediatR, the controller or application service may need to directly depend on handlers.

For example:

```csharp
private readonly CreateOrderCommandHandler _createHandler;
private readonly GetOrdersQueryHandler _getOrdersHandler;
private readonly GetOrderByIdQueryHandler _getByIdHandler;
```

As the application grows, this can become difficult to manage.

Imagine:

```text
50 Commands
40 Queries
```

The application layer could end up knowing about dozens of handlers.

MediatR provides a common entry point:

```csharp
await _mediator.Send(command);
```

or:

```csharp
await _mediator.Send(query);
```

The caller does not need to directly invoke the handler.

---

# 5. Without MediatR

The flow might look like:

```text
Controller
    ↓
Application Service
    ↓
CreateOrderCommandHandler
    ↓
Repository
    ↓
EF Core
    ↓
SQL Server
```

The application service must know about the concrete handler.

---

# 6. With MediatR

The flow becomes:

```text
Controller
    ↓
Application Service
    ↓
IMediator
    ↓
CreateOrderCommandHandler
    ↓
Repository
    ↓
EF Core
    ↓
SQL Server
```

The service only needs:

```csharp
IMediator
```

instead of knowing every concrete handler.

---

# 7. Why Use MediatR with CQRS?

MediatR is useful because it gives us:

### 1. Loose Coupling

The caller does not directly depend on a specific handler.

### 2. Central Request Dispatching

Commands and queries are sent through:

```csharp
_mediator.Send(...)
```

### 3. Cleaner Application Services

Instead of injecting many handlers:

```text
CreateOrderHandler
UpdateOrderHandler
DeleteOrderHandler
GetOrdersHandler
GetOrderByIdHandler
...
```

the service can depend on:

```text
IMediator
```

### 4. Pipeline Behaviors

MediatR supports pipeline behaviors that are useful for cross-cutting concerns such as:

- Logging
- Validation
- Performance measurement
- Authorization
- Transaction handling

Conceptually:

```text
Request
   ↓
Validation
   ↓
Logging
   ↓
Authorization
   ↓
Handler
```

### 5. Better Separation

The command/query represents the request.

The handler represents the execution.

MediatR connects them.

---

# 8. What MediatR Does NOT Do

MediatR does not:

- Create your database
- Replace EF Core
- Replace SQL Server
- Implement CQRS automatically
- Implement repositories automatically
- Implement business rules automatically
- Make your application distributed
- Create microservices automatically

MediatR primarily provides **request/handler dispatching**.

---

# 9. Example Application

We will build a simple:

**Order Management API**

Only three operations are required:

```text
POST /api/orders
GET  /api/orders
GET  /api/orders/{id}
```

Architecture:

```text
                 Controller
                     |
                     v
              Order Application
                  Service
                     |
                     v
                  IMediator
                     |
          +----------+----------+
          |                     |
          v                     v
   Command Handler       Query Handlers
          |                     |
          v                     v
 Command Repository      Query Repository
          |                     |
          +----------+----------+
                     |
                     v
                   EF Core
                     |
                     v
                 SQL Server
```

---

# 10. Project Structure

Recommended structure:

```text
CQRS.MediatR.Demo
│
├── Controllers
│   └── OrdersController.cs
│
├── Application
│   │
│   ├── Commands
│   │   └── CreateOrder
│   │       ├── CreateOrderCommand.cs
│   │       └── CreateOrderCommandHandler.cs
│   │
│   ├── Queries
│   │   ├── GetOrders
│   │   │   ├── GetOrdersQuery.cs
│   │   │   └── GetOrdersQueryHandler.cs
│   │   │
│   │   └── GetOrderById
│   │       ├── GetOrderByIdQuery.cs
│   │       └── GetOrderByIdQueryHandler.cs
│   │
│   ├── Services
│   │   ├── IOrderService.cs
│   │   └── OrderService.cs
│   │
│   └── DTOs
│       ├── CreateOrderRequest.cs
│       └── OrderResponse.cs
│
├── Domain
│   └── Entities
│       └── Order.cs
│
├── Infrastructure
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
└── CQRS.MediatR.Demo.csproj
```

---

# 11. Create the .NET 10 Project

Create a solution:

```bash
mkdir CQRS-MediatR-Demo
cd CQRS-MediatR-Demo
```

Create solution:

```bash
dotnet new sln -n CQRS.MediatR.Demo
```

Create Web API:

```bash
dotnet new webapi -n CQRS.MediatR.Demo --framework net10.0
```

Add it to the solution:

```bash
dotnet sln add CQRS.MediatR.Demo/CQRS.MediatR.Demo.csproj
```

Go into the project:

```bash
cd CQRS.MediatR.Demo
```

---

# 12. Install NuGet Packages

Install EF Core SQL Server:

```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
```

Install EF Core Design:

```bash
dotnet add package Microsoft.EntityFrameworkCore.Design
```

Install MediatR:

```bash
dotnet add package MediatR
```

Depending on the .NET 10 project template, Swagger/OpenAPI packages may already be present. If required, install the Swagger package used by your selected project template.

Install EF CLI if necessary:

```bash
dotnet tool install --global dotnet-ef
```

Verify:

```bash
dotnet ef --version
```

---

# 13. Create the Domain Entity

Create:

```text
Domain/Entities/Order.cs
```

```csharp
namespace CQRS.MediatR.Demo.Domain.Entities;

public class Order
{
    public int Id { get; set; }

    public string CustomerName { get; set; } = string.Empty;

    public string ProductName { get; set; } = string.Empty;

    public int Quantity { get; set; }

    public decimal UnitPrice { get; set; }

    public decimal TotalAmount { get; set; }

    public DateTime CreatedAt { get; set; }
}
```

---

# 14. Create DTOs

## CreateOrderRequest.cs

Create:

```text
Application/DTOs/CreateOrderRequest.cs
```

```csharp
namespace CQRS.MediatR.Demo.Application.DTOs;

public class CreateOrderRequest
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
namespace CQRS.MediatR.Demo.Application.DTOs;

public class OrderResponse
{
    public int Id { get; set; }

    public string CustomerName { get; set; } = string.Empty;

    public string ProductName { get; set; } = string.Empty;

    public int Quantity { get; set; }

    public decimal UnitPrice { get; set; }

    public decimal TotalAmount { get; set; }

    public DateTime CreatedAt { get; set; }
}
```

---

# 15. Create DbContext

Create:

```text
Infrastructure/Data/ApplicationDbContext.cs
```

```csharp
using CQRS.MediatR.Demo.Domain.Entities;
using Microsoft.EntityFrameworkCore;

namespace CQRS.MediatR.Demo.Infrastructure.Data;

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

# 16. Add SQL Server Connection String

Open:

```text
appsettings.json
```

Add:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=CQRSMediatRDb;Trusted_Connection=True;TrustServerCertificate=True"
  }
}
```

For SQL authentication:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=CQRSMediatRDb;User Id=sa;Password=YourPassword;TrustServerCertificate=True"
  }
}
```

Do not commit real production credentials to GitHub.

---

# 17. Create Database Using EF Core Migration

Run:

```bash
dotnet ef migrations add InitialCreate
```

Then:

```bash
dotnet ef database update
```

The database will be created:

```text
CQRSMediatRDb
```

with:

```text
Orders
```

table.

---

# 18. Create Command Repository

Create:

```text
Infrastructure/Repositories/IOrderCommandRepository.cs
```

```csharp
using CQRS.MediatR.Demo.Domain.Entities;

namespace CQRS.MediatR.Demo.Infrastructure.Repositories;

public interface IOrderCommandRepository
{
    Task AddAsync(Order order);

    Task SaveChangesAsync();
}
```

---

# 19. Implement Command Repository

Create:

```text
Infrastructure/Repositories/OrderCommandRepository.cs
```

```csharp
using CQRS.MediatR.Demo.Domain.Entities;
using CQRS.MediatR.Demo.Infrastructure.Data;

namespace CQRS.MediatR.Demo.Infrastructure.Repositories;

public class OrderCommandRepository : IOrderCommandRepository
{
    private readonly ApplicationDbContext _context;

    public OrderCommandRepository(
        ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task AddAsync(Order order)
    {
        await _context.Orders.AddAsync(order);
    }

    public async Task SaveChangesAsync()
    {
        await _context.SaveChangesAsync();
    }
}
```

---

# 20. Create Query Repository

Create:

```text
Infrastructure/Repositories/IOrderQueryRepository.cs
```

```csharp
using CQRS.MediatR.Demo.Domain.Entities;

namespace CQRS.MediatR.Demo.Infrastructure.Repositories;

public interface IOrderQueryRepository
{
    Task<List<Order>> GetAllAsync();

    Task<Order?> GetByIdAsync(int id);
}
```

---

# 21. Implement Query Repository

Create:

```text
Infrastructure/Repositories/OrderQueryRepository.cs
```

```csharp
using CQRS.MediatR.Demo.Domain.Entities;
using CQRS.MediatR.Demo.Infrastructure.Data;
using Microsoft.EntityFrameworkCore;

namespace CQRS.MediatR.Demo.Infrastructure.Repositories;

public class OrderQueryRepository : IOrderQueryRepository
{
    private readonly ApplicationDbContext _context;

    public OrderQueryRepository(
        ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task<List<Order>> GetAllAsync()
    {
        return await _context.Orders
            .AsNoTracking()
            .OrderByDescending(x => x.Id)
            .ToListAsync();
    }

    public async Task<Order?> GetByIdAsync(int id)
    {
        return await _context.Orders
            .AsNoTracking()
            .FirstOrDefaultAsync(x => x.Id == id);
    }
}
```

---

# 22. Create the CreateOrder Command

Create:

```text
Application/Commands/CreateOrder/CreateOrderCommand.cs
```

```csharp
using MediatR;

namespace CQRS.MediatR.Demo.Application.Commands.CreateOrder;

public record CreateOrderCommand(
    string CustomerName,
    string ProductName,
    int Quantity,
    decimal UnitPrice
) : IRequest<int>;
```

The important part is:

```csharp
IRequest<int>
```

This tells MediatR:

> This request expects an `int` response.

The `int` will be the newly created Order Id.

---

# 23. Create the CreateOrder Handler

Create:

```text
Application/Commands/CreateOrder/CreateOrderCommandHandler.cs
```

```csharp
using CQRS.MediatR.Demo.Domain.Entities;
using CQRS.MediatR.Demo.Infrastructure.Repositories;
using MediatR;

namespace CQRS.MediatR.Demo.Application.Commands.CreateOrder;

public class CreateOrderCommandHandler
    : IRequestHandler<CreateOrderCommand, int>
{
    private readonly IOrderCommandRepository _repository;

    public CreateOrderCommandHandler(
        IOrderCommandRepository repository)
    {
        _repository = repository;
    }

    public async Task<int> Handle(
        CreateOrderCommand request,
        CancellationToken cancellationToken)
    {
        var order = new Order
        {
            CustomerName = request.CustomerName,
            ProductName = request.ProductName,
            Quantity = request.Quantity,
            UnitPrice = request.UnitPrice,
            TotalAmount = request.Quantity * request.UnitPrice,
            CreatedAt = DateTime.UtcNow
        };

        await _repository.AddAsync(order);

        await _repository.SaveChangesAsync();

        return order.Id;
    }
}
```

---

# 24. Understand IRequestHandler

This:

```csharp
IRequestHandler<CreateOrderCommand, int>
```

means:

```text
Request:
    CreateOrderCommand

Response:
    int
```

MediatR knows:

```text
CreateOrderCommand
        ↓
CreateOrderCommandHandler
```

because the handler implements:

```csharp
IRequestHandler<CreateOrderCommand, int>
```

---

# 25. Create GetOrders Query

Create:

```text
Application/Queries/GetOrders/GetOrdersQuery.cs
```

```csharp
using CQRS.MediatR.Demo.Application.DTOs;
using MediatR;

namespace CQRS.MediatR.Demo.Application.Queries.GetOrders;

public record GetOrdersQuery
    : IRequest<List<OrderResponse>>;
```

The query expects:

```text
List<OrderResponse>
```

as the response.

---

# 26. Create GetOrders Query Handler

Create:

```text
Application/Queries/GetOrders/GetOrdersQueryHandler.cs
```

```csharp
using CQRS.MediatR.Demo.Application.DTOs;
using CQRS.MediatR.Demo.Infrastructure.Repositories;
using MediatR;

namespace CQRS.MediatR.Demo.Application.Queries.GetOrders;

public class GetOrdersQueryHandler
    : IRequestHandler<GetOrdersQuery, List<OrderResponse>>
{
    private readonly IOrderQueryRepository _repository;

    public GetOrdersQueryHandler(
        IOrderQueryRepository repository)
    {
        _repository = repository;
    }

    public async Task<List<OrderResponse>> Handle(
        GetOrdersQuery request,
        CancellationToken cancellationToken)
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
            CreatedAt = order.CreatedAt
        }).ToList();
    }
}
```

---

# 27. Create GetOrderById Query

Create:

```text
Application/Queries/GetOrderById/GetOrderByIdQuery.cs
```

```csharp
using CQRS.MediatR.Demo.Application.DTOs;
using MediatR;

namespace CQRS.MediatR.Demo.Application.Queries.GetOrderById;

public record GetOrderByIdQuery(
    int Id
) : IRequest<OrderResponse?>;
```

The query returns:

```text
OrderResponse?
```

The `?` means the order may not exist.

---

# 28. Create GetOrderById Handler

Create:

```text
Application/Queries/GetOrderById/GetOrderByIdQueryHandler.cs
```

```csharp
using CQRS.MediatR.Demo.Application.DTOs;
using CQRS.MediatR.Demo.Infrastructure.Repositories;
using MediatR;

namespace CQRS.MediatR.Demo.Application.Queries.GetOrderById;

public class GetOrderByIdQueryHandler
    : IRequestHandler<GetOrderByIdQuery, OrderResponse?>
{
    private readonly IOrderQueryRepository _repository;

    public GetOrderByIdQueryHandler(
        IOrderQueryRepository repository)
    {
        _repository = repository;
    }

    public async Task<OrderResponse?> Handle(
        GetOrderByIdQuery request,
        CancellationToken cancellationToken)
    {
        var order = await _repository.GetByIdAsync(request.Id);

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
            CreatedAt = order.CreatedAt
        };
    }
}
```

---

# 29. Create Application Service

We will still use an application service because this keeps the Controller focused on HTTP concerns.

Create:

```text
Application/Services/IOrderService.cs
```

```csharp
using CQRS.MediatR.Demo.Application.DTOs;

namespace CQRS.MediatR.Demo.Application.Services;

public interface IOrderService
{
    Task<int> CreateAsync(CreateOrderRequest request);

    Task<List<OrderResponse>> GetAllAsync();

    Task<OrderResponse?> GetByIdAsync(int id);
}
```

---

# 30. Implement Application Service

Create:

```text
Application/Services/OrderService.cs
```

```csharp
using CQRS.MediatR.Demo.Application.Commands.CreateOrder;
using CQRS.MediatR.Demo.Application.DTOs;
using CQRS.MediatR.Demo.Application.Queries.GetOrderById;
using CQRS.MediatR.Demo.Application.Queries.GetOrders;
using MediatR;

namespace CQRS.MediatR.Demo.Application.Services;

public class OrderService : IOrderService
{
    private readonly IMediator _mediator;

    public OrderService(IMediator mediator)
    {
        _mediator = mediator;
    }

    public async Task<int> CreateAsync(
        CreateOrderRequest request)
    {
        var command = new CreateOrderCommand(
            request.CustomerName,
            request.ProductName,
            request.Quantity,
            request.UnitPrice);

        return await _mediator.Send(command);
    }

    public async Task<List<OrderResponse>> GetAllAsync()
    {
        return await _mediator.Send(
            new GetOrdersQuery());
    }

    public async Task<OrderResponse?> GetByIdAsync(int id)
    {
        return await _mediator.Send(
            new GetOrderByIdQuery(id));
    }
}
```

---

# 31. This Is Where MediatR Becomes Useful

Notice the application service only knows:

```csharp
IMediator
```

It does not need:

```csharp
CreateOrderCommandHandler
GetOrdersQueryHandler
GetOrderByIdQueryHandler
```

Instead:

```csharp
_mediator.Send(command);
```

MediatR finds the correct handler.

---

# 32. Create Orders Controller

Create:

```text
Controllers/OrdersController.cs
```

```csharp
using CQRS.MediatR.Demo.Application.DTOs;
using CQRS.MediatR.Demo.Application.Services;
using Microsoft.AspNetCore.Mvc;

namespace CQRS.MediatR.Demo.Controllers;

[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    private readonly IOrderService _orderService;

    public OrdersController(
        IOrderService orderService)
    {
        _orderService = orderService;
    }

    [HttpPost]
    public async Task<IActionResult> Create(
        CreateOrderRequest request)
    {
        var orderId =
            await _orderService.CreateAsync(request);

        return CreatedAtAction(
            nameof(GetById),
            new { id = orderId },
            new { id = orderId });
    }

    [HttpGet]
    public async Task<IActionResult> GetAll()
    {
        var orders =
            await _orderService.GetAllAsync();

        return Ok(orders);
    }

    [HttpGet("{id:int}")]
    public async Task<IActionResult> GetById(
        int id)
    {
        var order =
            await _orderService.GetByIdAsync(id);

        if (order == null)
        {
            return NotFound();
        }

        return Ok(order);
    }
}
```

Notice how clean the controller is.

It does not know about:

```text
Commands
Queries
Handlers
Repositories
EF Core
MediatR
```

The Controller only knows:

```text
IOrderService
```

---

# 33. Configure Program.cs

Update `Program.cs`:

```csharp
using CQRS.MediatR.Demo.Application.Services;
using CQRS.MediatR.Demo.Infrastructure.Data;
using CQRS.MediatR.Demo.Infrastructure.Repositories;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

builder.Services.AddDbContext<ApplicationDbContext>(options =>
{
    options.UseSqlServer(
        builder.Configuration
            .GetConnectionString("DefaultConnection"));
});

// Repositories

builder.Services.AddScoped<
    IOrderCommandRepository,
    OrderCommandRepository>();

builder.Services.AddScoped<
    IOrderQueryRepository,
    OrderQueryRepository>();

// Application Service

builder.Services.AddScoped<
    IOrderService,
    OrderService>();

// MediatR

builder.Services.AddMediatR(cfg =>
{
    cfg.RegisterServicesFromAssembly(
        typeof(Program).Assembly);
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

# 34. Important MediatR Registration

This part is important:

```csharp
builder.Services.AddMediatR(cfg =>
{
    cfg.RegisterServicesFromAssembly(
        typeof(Program).Assembly);
});
```

MediatR scans the assembly and finds handlers such as:

```text
CreateOrderCommandHandler
GetOrdersQueryHandler
GetOrderByIdQueryHandler
```

because they implement:

```text
IRequestHandler<TRequest, TResponse>
```

---

# 35. Complete Dependency Injection Flow

At application startup:

```text
Program.cs
    |
    +-- DbContext
    |
    +-- Command Repository
    |
    +-- Query Repository
    |
    +-- Order Service
    |
    +-- MediatR
            |
            +-- CreateOrderCommandHandler
            +-- GetOrdersQueryHandler
            +-- GetOrderByIdQueryHandler
```

---

# 36. Complete Create Flow

Request:

```http
POST /api/orders
```

Body:

```json
{
  "customerName": "Nikhil",
  "productName": "Laptop",
  "quantity": 2,
  "unitPrice": 50000
}
```

Flow:

```text
HTTP Request
     |
     v
OrdersController
     |
     v
IOrderService.CreateAsync()
     |
     v
CreateOrderCommand
     |
     v
IMediator.Send(command)
     |
     v
MediatR
     |
     v
CreateOrderCommandHandler
     |
     v
IOrderCommandRepository
     |
     v
EF Core
     |
     v
SQL Server
     |
     v
Order Id
     |
     v
HTTP 201 Created
```

---

# 37. What Exactly Does MediatR Do?

When we write:

```csharp
await _mediator.Send(command);
```

conceptually:

```text
CreateOrderCommand
        |
        v
      MediatR
        |
        | Find handler for CreateOrderCommand
        v
CreateOrderCommandHandler
        |
        v
Handle(...)
```

MediatR performs the dispatching.

You do not manually write:

```csharp
new CreateOrderCommandHandler(...)
```

or:

```csharp
handler.Handle(...)
```

---

# 38. Complete Get All Flow

Request:

```http
GET /api/orders
```

Flow:

```text
HTTP Request
     |
     v
OrdersController
     |
     v
OrderService.GetAllAsync()
     |
     v
GetOrdersQuery
     |
     v
IMediator.Send(query)
     |
     v
MediatR
     |
     v
GetOrdersQueryHandler
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
List<OrderResponse>
     |
     v
HTTP 200
```

---

# 39. Complete Get By Id Flow

Request:

```http
GET /api/orders/1
```

Flow:

```text
HTTP Request
     |
     v
OrdersController
     |
     v
OrderService.GetByIdAsync(1)
     |
     v
GetOrderByIdQuery(1)
     |
     v
IMediator.Send(query)
     |
     v
MediatR
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
OrderResponse
     |
     v
HTTP 200
```

---

# 40. Why Not Inject All Handlers?

Without MediatR, you might have:

```csharp
public class OrderService
{
    private readonly CreateOrderCommandHandler _createHandler;

    private readonly GetOrdersQueryHandler _getOrdersHandler;

    private readonly GetOrderByIdQueryHandler _getByIdHandler;
}
```

With MediatR:

```csharp
public class OrderService
{
    private readonly IMediator _mediator;

    public OrderService(IMediator mediator)
    {
        _mediator = mediator;
    }
}
```

Now adding another operation is simpler.

For example:

```text
CancelOrderCommand
```

Add:

```text
CancelOrderCommand
CancelOrderCommandHandler
```

Then:

```csharp
await _mediator.Send(command);
```

The service does not need to directly depend on:

```text
CancelOrderCommandHandler
```

---

# 41. MediatR's Main Value

The main value is **decoupling request senders from request handlers**.

Instead of:

```text
Sender
  |
  +----> Handler A
  |
  +----> Handler B
  |
  +----> Handler C
```

we get:

```text
Sender
   |
   v
Mediator
   |
   +----> Handler A
   +----> Handler B
   +----> Handler C
```

The mediator determines which handler should receive the request.

---

# 42. MediatR Pipeline Behaviors

One of the most useful advanced features is pipeline behavior.

Conceptually:

```text
Request
   |
   v
Logging Behavior
   |
   v
Validation Behavior
   |
   v
Performance Behavior
   |
   v
Authorization Behavior
   |
   v
Handler
```

For example:

```text
CreateOrderCommand
       |
       v
Validation
       |
       v
Logging
       |
       v
Handler
```

This avoids duplicating the same logic inside every handler.

---

# 43. Example: Why Pipeline Behaviors Are Useful

Imagine 100 handlers.

You want to log:

```text
Request Started
Request Completed
Execution Time
```

Without a common pipeline, you might duplicate code in 100 handlers.

With a pipeline behavior:

```text
MediatR
   |
   v
LoggingBehavior
   |
   v
Handler
```

the logging can be centralized.

---

# 44. CQRS + MediatR Architecture

The complete architecture is:

```text
                           CLIENT
                              |
                              v
                         Controller
                              |
                              v
                       Application Service
                              |
                              v
                           IMediator
                              |
             +----------------+----------------+
             |                                 |
             v                                 v
        COMMAND SIDE                      QUERY SIDE
             |                                 |
             v                                 v
      Command Handler                    Query Handler
             |                                 |
             v                                 v
      Command Repository                Query Repository
             |                                 |
             +----------------+----------------+
                              |
                              v
                           EF Core
                              |
                              v
                         SQL Server
```

---

# 45. Test the API

Run:

```bash
dotnet restore
dotnet build
dotnet run
```

Open Swagger:

```text
https://localhost:<port>/swagger
```

---

# 46. Test Create

Endpoint:

```text
POST /api/Orders
```

Body:

```json
{
  "customerName": "Nikhil",
  "productName": "Laptop",
  "quantity": 2,
  "unitPrice": 50000
}
```

Expected:

```json
{
  "id": 1
}
```

---

# 47. Test Get All

Endpoint:

```text
GET /api/Orders
```

Expected:

```json
[
  {
    "id": 1,
    "customerName": "Nikhil",
    "productName": "Laptop",
    "quantity": 2,
    "unitPrice": 50000,
    "totalAmount": 100000,
    "createdAt": "2026-08-19T..."
  }
]
```

---

# 48. Test Get By Id

Endpoint:

```text
GET /api/Orders/1
```

Expected:

```json
{
  "id": 1,
  "customerName": "Nikhil",
  "productName": "Laptop",
  "quantity": 2,
  "unitPrice": 50000,
  "totalAmount": 100000,
  "createdAt": "2026-08-19T..."
}
```

---

# 49. What Happens If ID Does Not Exist?

Request:

```text
GET /api/Orders/999
```

Handler returns:

```csharp
return null;
```

The service returns the result to the controller.

Controller:

```csharp
if (order == null)
{
    return NotFound();
}
```

Response:

```text
404 Not Found
```

---

# 50. CQRS Without MediatR vs CQRS With MediatR

## Without MediatR

```text
Controller
    ↓
Application Service
    ↓
Handler
```

The service knows the handler.

Example:

```csharp
_createHandler.HandleAsync(command);
```

## With MediatR

```text
Controller
    ↓
Application Service
    ↓
IMediator
    ↓
Handler
```

Example:

```csharp
_mediator.Send(command);
```

---

# 51. Comparison

| Area | Without MediatR | With MediatR |
|---|---|---|
| CQRS possible | Yes | Yes |
| Direct handler dependency | Yes | No |
| Central dispatching | Manual | MediatR |
| Number of handlers | Can become cumbersome | Easier to manage |
| Pipeline behaviors | Must build yourself | Supported by MediatR |
| Learning CQRS | Very explicit | Slight abstraction |
| Additional dependency | No | Yes |
| Request/handler decoupling | Lower | Higher |

---

# 52. When Should You Use MediatR?

MediatR can be useful when:

- You have many commands and queries
- You want consistent request/handler dispatching
- You want pipeline behaviors
- You want handlers to be independent
- You have a large application layer
- You want centralized cross-cutting behavior
- You are implementing CQRS

---

# 53. When Might You Avoid MediatR?

You may not need it when:

- The application is very small
- There are only a few operations
- Direct service calls are clearer
- The team wants fewer abstractions/dependencies
- The mediator adds more indirection than value

Do not add MediatR simply because CQRS examples commonly use it.

---

# 54. Important Interview Question

### Is MediatR CQRS?

**No.**

Correct answer:

> MediatR is a mediator library that can be used to implement CQRS by dispatching commands and queries to their respective handlers. CQRS itself is an architectural pattern for separating commands from queries.

---

# 55. Important Interview Question

### Does CQRS require MediatR?

**No.**

CQRS can be implemented manually:

```text
Service
   ↓
Handler
```

or with MediatR:

```text
Service
   ↓
MediatR
   ↓
Handler
```

---

# 56. Important Interview Question

### What problem does MediatR solve?

A good answer:

> MediatR decouples the sender of a request from the concrete handler that processes it. Instead of directly depending on and invoking individual handlers, the application sends a request through `IMediator`, which dispatches it to the appropriate handler. It also provides pipeline behaviors for cross-cutting concerns such as validation, logging and performance monitoring.

---

# 57. Important Interview Question

### How does MediatR know which handler to call?

The handler implements:

```csharp
IRequestHandler<TRequest, TResponse>
```

For example:

```csharp
IRequestHandler<CreateOrderCommand, int>
```

MediatR uses the request type to locate the corresponding handler.

Conceptually:

```text
CreateOrderCommand
       |
       v
IRequestHandler<CreateOrderCommand, int>
       |
       v
CreateOrderCommandHandler
```

---

# 58. Important Interview Question

### What is IRequest?

`IRequest<TResponse>` represents a request that expects a response.

Example:

```csharp
public record GetOrdersQuery
    : IRequest<List<OrderResponse>>;
```

This means:

```text
Request:
    GetOrdersQuery

Response:
    List<OrderResponse>
```

---

# 59. Important Interview Question

### What is IRequestHandler?

It defines the class responsible for processing a request.

Example:

```csharp
public class GetOrdersQueryHandler
    : IRequestHandler<GetOrdersQuery, List<OrderResponse>>
```

The handler implements:

```csharp
Handle(...)
```

---

# 60. Important Interview Question

### What is IMediator?

`IMediator` is the abstraction used by the application to send requests.

Example:

```csharp
await _mediator.Send(command);
```

or:

```csharp
await _mediator.Send(query);
```

---

# 61. Simple Mental Model

Think of MediatR like a receptionist.

Without MediatR:

```text
You
 |
 +----> Handler A
```

With MediatR:

```text
You
 |
 v
Receptionist
 |
 +----> Handler A
```

You tell the receptionist:

```text
"I want to create an order."
```

The receptionist determines:

```text
This request belongs to CreateOrderCommandHandler.
```

Then it sends the request there.

---

# 62. The Most Important Code

If you remember only one piece of MediatR code, remember:

```csharp
await _mediator.Send(request);
```

The request could be:

```csharp
var command = new CreateOrderCommand(...);

await _mediator.Send(command);
```

or:

```csharp
var query = new GetOrdersQuery();

await _mediator.Send(query);
```

---

# 63. Complete Request Flow — Final Revision

## Create

```text
POST /api/orders
       |
       v
Controller
       |
       v
OrderService
       |
       v
CreateOrderCommand
       |
       v
IMediator.Send()
       |
       v
MediatR
       |
       v
CreateOrderCommandHandler
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

## Get All

```text
GET /api/orders
       |
       v
Controller
       |
       v
OrderService
       |
       v
GetOrdersQuery
       |
       v
IMediator.Send()
       |
       v
MediatR
       |
       v
GetOrdersQueryHandler
       |
       v
Query Repository
       |
       v
EF Core
       |
       v
SQL Server
```

## Get By Id

```text
GET /api/orders/1
       |
       v
Controller
       |
       v
OrderService
       |
       v
GetOrderByIdQuery
       |
       v
IMediator.Send()
       |
       v
MediatR
       |
       v
GetOrderByIdQueryHandler
       |
       v
Query Repository
       |
       v
EF Core
       |
       v
SQL Server
```

---

# 64. What Problem Is Solved at Each Layer?

| Layer | Main Problem Solved |
|---|---|
| Controller | HTTP/API handling |
| Application Service | Keeps controller independent of application execution details |
| CQRS Command | Represents a write intention |
| CQRS Query | Represents a read intention |
| MediatR | Decouples sender from handler |
| Handler | Executes one specific use case |
| Repository | Abstracts persistence |
| EF Core | ORM/database communication |
| SQL Server | Persistent data storage |

---

# 65. Final Architecture

```text
                        HTTP CLIENT
                             |
                             v
                     OrdersController
                             |
                             v
                      IOrderService
                             |
                             v
                         IMediator
                             |
                 +-----------+-----------+
                 |                       |
                 v                       v
             COMMAND                   QUERY
                 |                       |
                 v                       v
       CreateOrderCommand       GetOrdersQuery
                 |               GetOrderByIdQuery
                 v                       |
       CreateOrderCommandHandler        |
                 |                       v
                 |             Query Handlers
                 |                       |
                 v                       v
       Command Repository       Query Repository
                 |                       |
                 +-----------+-----------+
                             |
                             v
                          EF Core
                             |
                             v
                         SQL Server
```

---

# 66. Key Takeaways

### CQRS

Separates:

```text
Commands
    ↓
Write

Queries
    ↓
Read
```

### MediatR

Provides:

```text
Request
   ↓
Mediator
   ↓
Handler
```

### Together

```text
Controller
    ↓
Application Service
    ↓
MediatR
    ↓
Command/Query Handler
    ↓
Repository
    ↓
EF Core
    ↓
SQL Server
```

The most important point is:

> **CQRS decides how we separate reads and writes. MediatR helps us decouple the caller from the handler that processes each request.**

---

# 67. Recommended Learning Order

If you are learning this for interviews or enterprise .NET development, follow this order:

```text
1. CRUD
   ↓
2. Repository Pattern
   ↓
3. Service Layer
   ↓
4. CQRS Without MediatR
   ↓
5. Mediator Pattern
   ↓
6. CQRS + MediatR
   ↓
7. MediatR Pipeline Behaviors
   ↓
8. CQRS + Events
   ↓
9. CQRS + SAGA
   ↓
10. CQRS + Eventual Consistency
   ↓
11. CQRS + Separate Read/Write Databases
```

This progression helps you understand **why each abstraction exists** instead of simply memorizing the code.

---

# 68. One-Line Interview Answer

> **MediatR is a .NET mediator library that decouples a request sender from its handler by allowing commands and queries to be dispatched through `IMediator`, and it also provides pipeline behaviors for cross-cutting concerns such as validation, logging and performance monitoring.**
