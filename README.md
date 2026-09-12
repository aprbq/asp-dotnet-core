# .NET REST API Tutorial - สรุปเนื้อหา

สรุปเนื้อหาการสร้าง REST API ด้วย ASP.NET Core และ Entity Framework Core

---

## สารบัญ

1. [Setup Environment](#setup-environment)
2. [Project Structure](#project-structure)
3. [Database Setup](#database-setup)
4. [Async/Await](#asyncawait)
5. [Key Concepts](#key-concepts)

---

## Setup Environment

### ติดตั้ง .NET
```bash
# ตรวจสอบ .NET ได้ติดตั้งแล้ว
dotnet --version

# สร้าง project ใหม่
dotnet new web -n GameStore.Api
cd GameStore.Api
```

### ติดตั้ง Packages ที่จำเป็น
```bash
# Entity Framework Core
dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.Sqlite
dotnet add package Microsoft.EntityFrameworkCore.Design
```

---

## Project Structure

```
GameStore.Api/
├── Program.cs              // Entry point, configuration
├── Models/
│   ├── Game.cs            // ข้อมูลเกม
│   └── Genre.cs           // ประเภทเกม
├── DTOs/
│   └── GameDto.cs         // Data Transfer Object
└── Data/
    └── GameStoreContext.cs // DbContext (Database)
```

---

## Database Setup

### 1. Database Migrations
```bash
# สร้าง migration แรก
dotnet ef migrations add InitialCreate

# Apply migration ไปที่ database
dotnet ef database update

# ดู migration status
dotnet ef migrations list
```

### 2. Data Seeding (เติมข้อมูลตัวอย่าง)
```csharp
// ใน Program.cs
using (var scope = app.Services.CreateScope())
{
    var db = scope.ServiceProvider.GetRequiredService<GameStoreContext>();
    db.Database.Migrate();
    
    if (!db.Genres.Any())
    {
        var genres = new[]
        {
            new Genre { Name = "Action" },
            new Genre { Name = "RPG" },
            new Genre { Name = "Strategy" }
        };
        db.Genres.AddRange(genres);
        await db.SaveChangesAsync();
    }
}
```

---

## Async/Await

### ทำไมต้อง Async?
- **Sync** → Thread block รอ DB → UI ค้าง
- **Async** → Thread ไม่ block → สามารถ handle request อื่นได้

### Pattern ที่ถูกต้อง
```csharp
// ผิด
var games = dbContext.Games.Select(...);  // IQueryable

// ถูก
var games = await dbContext.Games
    .Select(...)
    .ToListAsync();  // Task<List<T>>
```

### Terminal Operators (ต้องใช้เพื่อ await)
```
.ToListAsync()          → List<T>
.FirstAsync()           → T (รายการแรก)
.FirstOrDefaultAsync()  → T or null
.SingleAsync()          → T (ต้องมี 1 เท่านั้น)
.CountAsync()           → int
.AnyAsync()             → bool
```

---

## Key Concepts

### Dependency Injection
```csharp
// DbContext ถูก inject โดย DI Container
group.MapGet("/", async (GameStoreContext dbContext) => 
    // dbContext พร้อมใช้
);
```

### DTOs (Data Transfer Objects)
```csharp
// ส่ง DTO กลับไป (ลด expose sensitive data)
public record GameDto(
    int Id,
    string Name,
    string GenreName,
    decimal Price,
    DateTime ReleaseDate
);
```

### Input Validation
```csharp
var error = GameValidator.Validate(gameDto);
if (!error.IsEmpty)
    return Results.BadRequest(error);
```

### Logging
```csharp
// ในรูป Program.cs
builder.Logging.AddConsole();
builder.Logging.SetMinimumLevel(LogLevel.Debug);
```

### Configuration
```csharp
// appsettings.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=GameStore;Trusted_Connection=true;"
  }
}

// ใน code
var connString = builder.Configuration.GetConnectionString("DefaultConnection");
```

---

## Run Project
```bash
# Run development server
dotnet run

# Localhost
https://localhost:7043

# ทดสอบ API (ใช้ Postman หรือ Swagger)
GET https://localhost:7043/games
```

