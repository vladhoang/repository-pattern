# Repository Pattern in C#

## Introduction

The Repository Pattern is a design pattern that provides an abstraction layer between the data access logic and business logic of an application. By implementing this pattern, applications become more maintainable, testable, and reusable.

## Why Use the Repository Pattern?

- **Decouples** business logic from data access logic.
- **Improves testability** by allowing mock repositories.
- **Reduces duplication** of data access code.
- **Supports multiple data sources** (databases, file systems, etc.).

## Core Components

### Generic Repository Interface
```csharp
public interface IRepository<T>
{
    T Add(T entity);
    T Update(T entity);
    T Get(Guid id);
    IEnumerable<T> All();
    IEnumerable<T> Find(Expression<Func<T, bool>> predicate);
    void SaveChanges();
}
```

### Generic Repository Implementation
```csharp
public class GenericRepository<T> : IRepository<T> where T : class
{
    protected readonly DbContext context;

    public GenericRepository(DbContext context)
    {
        this.context = context;
    }

    public virtual T Add(T entity)
    {
        context.Set<T>().Add(entity);
        return entity;
    }

    public virtual IEnumerable<T> All()
    {
        return context.Set<T>().ToList();
    }

    public virtual IEnumerable<T> Find(Expression<Func<T, bool>> predicate)
    {
        return context.Set<T>().Where(predicate).ToList();
    }

    public virtual T Get(Guid id)
    {
        return context.Set<T>().Find(id);
    }

    public virtual T Update(T entity)
    {
        context.Set<T>().Update(entity);
        return entity;
    }

    public void SaveChanges()
    {
        context.SaveChanges();
    }
}
```

## Extending the Generic Repository

Concrete implementations can override methods for specific logic:

```csharp
public class OrderRepository : GenericRepository<Order>
{
    public OrderRepository(DbContext context) : base(context) { }

    public override IEnumerable<Order> Find(Expression<Func<Order, bool>> predicate)
    {
        return context.Set<Order>()
                      .Include(o => o.LineItems)
                      .ThenInclude(li => li.Product)
                      .Where(predicate)
                      .ToList();
    }
}
```

## Using Repository in Controllers

Controllers become independent of specific data access technologies:

```csharp
public class OrderController : Controller
{
    private readonly IRepository<Order> orderRepository;

    public OrderController(IRepository<Order> orderRepository)
    {
        this.orderRepository = orderRepository;
    }

    public IActionResult Index()
    {
        var orders = orderRepository.Find(o => o.Date >= DateTime.Today.AddDays(-1));
        return View(orders);
    }
}
```

## Dependency Injection Setup

Register repositories for dependency injection in `Startup.cs`:

```csharp
services.AddScoped<IRepository<Order>, OrderRepository>();
services.AddScoped<IRepository<Product>, ProductRepository>();
services.AddScoped<IRepository<Customer>, CustomerRepository>();
```

## Step-by-Step Guide to Setup Sample Project

### Step 1: Create Project

1. Open Visual Studio.
2. Create a new ASP.NET Core Web Application.
3. Choose the template MVC or Web API.

### Step 2: Add Entity Framework Core

Install Entity Framework Core and SQLite (for simplicity):
```bash
dotnet add package Microsoft.EntityFrameworkCore
```
```bash
dotnet add package Microsoft.EntityFrameworkCore.Sqlite
```

### Step 3: Configure Database Context

Create your `DbContext`:
```csharp
public class ShoppingContext : DbContext
{
    public ShoppingContext(DbContextOptions<ShoppingContext> options) : base(options) { }

    public DbSet<Order> Orders { get; set; }
    public DbSet<Product> Products { get; set; }
}
```

Configure context in `Startup.cs`:
```csharp
services.AddDbContext<ShoppingContext>(options =>
    options.UseSqlite(Configuration.GetConnectionString("DefaultConnection")));
```

### Step 4: Implement Repository Pattern

- Add the generic repository interface and implementation.
- Create concrete repositories like `OrderRepository`.

### Step 5: Configure Dependency Injection

As shown previously, configure your repositories in `Startup.cs`.

### Step 6: Use Repositories in Controllers

Inject your repositories in controllers as shown above.

### Step 7: Run and Test the Application

- Apply migrations and update the database:
```bash
dotnet ef migrations add InitialCreate
dotnet ef database update
```
- Run the application and verify it works correctly.

## Testing with Fake Repositories

Using a mocking framework (e.g., Moq), tests become straightforward without side effects:

```csharp
[TestMethod]
public void CreateOrder_ShouldInvokeRepositoryAddMethod()
{
    var mockOrderRepo = new Mock<IRepository<Order>>();
    var controller = new OrderController(mockOrderRepo.Object);

    var model = new CreateOrderModel { /* properties */ };

    controller.Create(model);

    mockOrderRepo.Verify(repo => repo.Add(It.IsAny<Order>()), Times.Once);
}
```

## Advantages and Considerations

### Advantages:
- Enhanced maintainability and reusability.
- Easier unit testing without direct database dependency.

### Considerations:
- Introducing the Repository Pattern adds complexity.
- Evaluate if the additional abstraction truly benefits your project's maintainability and scalability.

## Conclusion

The Repository Pattern is a valuable tool for building robust, maintainable, and testable applications. By abstracting data access logic, it simplifies interactions with different data sources and enhances code modularity.

---

*References: Filip Ekberg, Pluralsight course on Repository Pattern in C#.*

