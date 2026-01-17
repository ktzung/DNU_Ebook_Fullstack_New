# 🟦 THỰC HÀNH TỔNG HỢP: HỆ THỐNG QUẢN LÝ THƯ VIỆN VỚI PHÂN QUYỀN (VUE JS)

> **Lưu ý**: Đây là phiên bản sử dụng **Vue.js** cho Frontend thay vì Razor Pages.
> Phần Backend API giống hoàn toàn với bài thực hành MVC.

## 🎯 MỤC TIÊU BÀI THỰC HÀNH

Xây dựng **hệ thống quản lý thư viện điện tử hoàn chỉnh** với:
- ✅ **Phân quyền rõ ràng** theo vai trò (Role-Based Access Control - RBAC)
- ✅ **Xác thực người dùng** (Authentication) với JWT
- ✅ **Phân quyền truy cập** (Authorization) theo chức năng
- ✅ **CRUD đầy đủ** với giao diện **Vue.js + Vuetify**
- ✅ **Kiến trúc SPA (Single Page Application)** hiện đại tách biệt Backend và Frontend

## 🧒 GIẢI THÍCH DỄ HIỂU (CHO NGƯỜI MỚI BẮT ĐẦU)

> **Hệ thống thư viện** giống như một thư viện thật ngoài đời:
> - **Thủ thư trưởng** (Admin): Quản lý toàn bộ, thêm/xóa sách, quản lý nhân viên
> - **Thủ thư** (Librarian): Quản lý mượn/trả sách, xem thông tin sách
> - **Độc giả** (Reader): Chỉ xem danh sách sách, mượn sách
> - **Khách** (Guest): Chỉ xem danh sách công khai
> 
> **Authentication (Xác thực)**: Kiểm tra "bạn là ai?" - giống như xuất trình thẻ thư viện
> 
> **Authorization (Phân quyền)**: Kiểm tra "bạn được làm gì?" - giống như thẻ thư viện có ghi quyền hạn

---

## 📘 PHẦN A: MÔ TẢ BÀI TOÁN & LÝ THUYẾT

## 📋 PHẦN 1: PHÂN TÍCH YÊU CẦU VÀ THIẾT KẾ HỆ THỐNG

### 1.1. Mô tả bài toán chi tiết

**Bối cảnh**: Thư viện Đại học Đà Nẵng cần một hệ thống quản lý sách điện tử. Hệ thống có nhiều loại người dùng với quyền hạn khác nhau.

**Các chức năng chính**:
1. **Quản lý sách** (Books): Thêm, sửa, xóa, xem danh sách sách
2. **Quản lý thể loại** (Categories): Thêm, sửa, xóa, xem danh sách thể loại
3. **Quản lý người dùng** (Users): Xem danh sách, phân quyền
4. **Đăng nhập/Đăng xuất**: Xác thực người dùng

### 1.2. Định nghĩa các vai trò (Roles) và quyền hạn

> ⚠️ **QUAN TRỌNG**: Đây là phần sinh viên thường bỏ qua! Phải liệt kê đầy đủ vai trò và quyền từ đầu.

#### 📊 Bảng phân quyền chi tiết (Permission Matrix)

| Chức năng | Admin | Librarian | Reader | Guest (Chưa đăng nhập) |
|-----------|-------|-----------|--------|------------------------|
| **QUẢN LÝ SÁCH** |
| Xem danh sách sách | ✅ | ✅ | ✅ | ✅ (Chỉ sách công khai) |
| Xem chi tiết sách | ✅ | ✅ | ✅ | ✅ |
| Thêm sách mới | ✅ | ✅ | ❌ | ❌ |
| Sửa thông tin sách | ✅ | ✅ | ❌ | ❌ |
| Xóa sách | ✅ | ❌ | ❌ | ❌ |
| **QUẢN LÝ THỂ LOẠI** |
| Xem danh sách thể loại | ✅ | ✅ | ✅ | ✅ |
| Thêm thể loại | ✅ | ❌ | ❌ | ❌ |
| Sửa thể loại | ✅ | ❌ | ❌ | ❌ |
| Xóa thể loại | ✅ | ❌ | ❌ | ❌ |
| **QUẢN LÝ NGƯỜI DÙNG** |
| Xem danh sách user | ✅ | ❌ | ❌ | ❌ |
| Thêm user mới | ✅ | ❌ | ❌ | ❌ |
| Phân quyền user | ✅ | ❌ | ❌ | ❌ |
| Xóa user | ✅ | ❌ | ❌ | ❌ |
| **XÁC THỰC** |
| Đăng ký tài khoản | ✅ | ✅ | ✅ | ✅ |
| Đăng nhập | ✅ | ✅ | ✅ | ❌ |
| Đăng xuất | ✅ | ✅ | ✅ | ❌ |

### 1.3. Sơ đồ kiến trúc hệ thống (Vue Implementation)

```
┌─────────────────────────────────────────────────────────┐
│                    USER (Browser)                        │
│  - Frontend: Vue 3 + Vuetify + Pinia                    │
│  - SPA (Single Page App)                                │
└────────────────┬────────────────────────────────────────┘
                 │ HTTP Requests (JSON + JWT Token)
                 ▼
┌─────────────────────────────────────────────────────────┐
│              ASP.NET Core API (Backend)                 │
│  - API Controllers (with [Authorize] attributes)        │
│  - JWT Authentication Middleware                        │
│  - Role-based Authorization                             │
│  - CORS Policy (Allow :3000 -> :5000)                   │
└────────────────┬────────────────────────────────────────┘
                 │ Entity Framework Core
                 ▼
┌─────────────────────────────────────────────────────────┐
│                   SQL Server Database                    │
│  - Books, Categories                                    │
│  - AspNetUsers, AspNetRoles (Identity tables)           │
└─────────────────────────────────────────────────────────┘
```

### 1.4. Thiết kế Database

**Bảng Books**
```sql
CREATE TABLE Books (
    Id INT PRIMARY KEY IDENTITY,
    Title NVARCHAR(200) NOT NULL,
    Author NVARCHAR(100),
    ISBN NVARCHAR(20),
    PublishedYear INT,
    Price DECIMAL(18,2),
    CategoryId INT NOT NULL,
    IsPublic BIT DEFAULT 1,
    CreatedAt DATETIME2 DEFAULT GETDATE(),
    FOREIGN KEY (CategoryId) REFERENCES Categories(Id)
)
```

**Bảng Categories**
```sql
CREATE TABLE Categories (
    Id INT PRIMARY KEY IDENTITY,
    Name NVARCHAR(100) NOT NULL,
    Description NVARCHAR(500)
)
```

---

## 📘 PHẦN B: THỰC HÀNH TỪNG BƯỚC

## 🛠️ CHUẨN BỊ (PREREQUISITES)
- Cài **.NET SDK**, **SQL Server**, **SSMS**.
- Cài **Node.js** (v16 trở lên) và **npm**.
- Cài **Visual Studio Code** (khuyên dùng cho Frontend) hoặc Visual Studio.

---

## 🧭 HƯỚNG DẪN TỪNG BƯỚC (STEP-BY-STEP WALKTHROUGH)

> **LƯU Ý**: Phần Backend API này **giống hệt** bài thực hành MVC. Nếu bạn đã làm bài MVC rồi, có thể dùng lại project API cũ và nhảy sang **PHẦN 3**.

### Step 1: Chuẩn bị công cụ Migration
**Giải thích:** Migration là cách tạo/ cập nhật CSDL tự động từ code C#.
```bash
# Cài EF Tools (nếu chưa có)
dotnet tool install --global dotnet-ef

# Kiểm tra phiên bản
dotnet ef --version
```

### Step 2: Tạo CSDL bằng Migration
**Giải thích:** Tạo DB và bảng dựa vào `DbContext` + Models.
```bash
dotnet ef migrations add InitialDb
dotnet ef database update
```

### Step 3: Seed dữ liệu giả (Fake Data)
**Giải thích:** Seed giúp có dữ liệu mẫu để test UI/CRUD nhanh.

#### 3.1. Tạo file `Data/DbSeeder.cs`
```csharp
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;

public static class DbSeeder
{
    public static async Task SeedAsync(
        ApplicationDbContext context,
        UserManager<IdentityUser> userManager,
        RoleManager<IdentityRole> roleManager)
    {
        // 0) Seed Roles
        string[] roles = { "Admin", "Librarian", "Reader" };
        foreach (var role in roles)
        {
            if (!await roleManager.RoleExistsAsync(role))
            {
                await roleManager.CreateAsync(new IdentityRole(role));
            }
        }

        // 1) Seed Categories (idempotent)
        var categoriesToSeed = new List<Category>
        {
            new Category { Name = "Công nghệ thông tin", Description = "Lập trình, AI, Database" },
            new Category { Name = "Kinh tế", Description = "Quản trị, Marketing" },
            new Category { Name = "Văn học", Description = "Tiểu thuyết, thơ" },
            new Category { Name = "Khoa học", Description = "Vật lý, Hóa học" }
        };

        foreach (var cat in categoriesToSeed)
        {
            var exists = await context.Categories.AnyAsync(c => c.Name == cat.Name);
            if (!exists) context.Categories.Add(cat);
        }
        await context.SaveChangesAsync();

        // 2) Seed Books
        if (!context.Books.Any())
        {
            var categoryMap = await context.Categories
                .Where(c => new[] { "Công nghệ thông tin", "Kinh tế", "Văn học", "Khoa học" }.Contains(c.Name))
                .ToDictionaryAsync(c => c.Name, c => c.Id);

            if (!categoryMap.ContainsKey("Công nghệ thông tin")
                || !categoryMap.ContainsKey("Kinh tế")
                || !categoryMap.ContainsKey("Văn học")
                || !categoryMap.ContainsKey("Khoa học"))
            {
                return; // Không đủ category để seed sách
            }

            var itCategoryId = categoryMap["Công nghệ thông tin"];
            var ecoCategoryId = categoryMap["Kinh tế"];
            var litCategoryId = categoryMap["Văn học"];
            var sciCategoryId = categoryMap["Khoa học"];
            context.Books.AddRange(
                new Book { Title = "Lập trình C# cơ bản", Price = 120000, CategoryId = itCategoryId, IsPublic = true },
                new Book { Title = "SQL nâng cao", Price = 150000, CategoryId = itCategoryId, IsPublic = true },
                new Book { Title = "Marketing căn bản", Price = 98000, CategoryId = ecoCategoryId, IsPublic = true },
                new Book { Title = "Quản trị tài chính", Price = 135000, CategoryId = ecoCategoryId, IsPublic = false },
                new Book { Title = "Dế Mèn phiêu lưu ký", Price = 65000, CategoryId = litCategoryId, IsPublic = true },
                new Book { Title = "Vật lý đại cương", Price = 110000, CategoryId = sciCategoryId, IsPublic = true }
            );
            await context.SaveChangesAsync();
        }

        // 3) Seed Admin user
        var adminUser = await userManager.FindByNameAsync("admin");
        if (adminUser == null)
        {
            adminUser = new IdentityUser { UserName = "admin", Email = "admin@library.com" };
            var result = await userManager.CreateAsync(adminUser, "Admin123!");
            if (result.Succeeded)
            {
                await userManager.AddToRoleAsync(adminUser, "Admin");
            }
        }

        // 4) Seed Librarian user
        var librarianUser = await userManager.FindByNameAsync("librarian1");
        if (librarianUser == null)
        {
            librarianUser = new IdentityUser { UserName = "librarian1", Email = "librarian@library.com" };
            var result = await userManager.CreateAsync(librarianUser, "Lib123!");
            if (result.Succeeded)
            {
                await userManager.AddToRoleAsync(librarianUser, "Librarian");
            }
        }

        // 5) Seed Reader user
        var readerUser = await userManager.FindByNameAsync("reader1");
        if (readerUser == null)
        {
            readerUser = new IdentityUser { UserName = "reader1", Email = "reader@library.com" };
            var result = await userManager.CreateAsync(readerUser, "Reader123!");
            if (result.Succeeded)
            {
                await userManager.AddToRoleAsync(readerUser, "Reader");
            }
        }
    }
}
```

#### 3.2. Gọi Seeder trong `Program.cs`
```csharp
using (var scope = app.Services.CreateScope())
{
    var services = scope.ServiceProvider;
    var context = services.GetRequiredService<ApplicationDbContext>();
    var userManager = services.GetRequiredService<UserManager<IdentityUser>>();
    var roleManager = services.GetRequiredService<RoleManager<IdentityRole>>();

    await DbSeeder.SeedAsync(context, userManager, roleManager);
}
```

### Step 4: Rollback migration và reset DB (khi cần làm lại)

**4.1. Xoá migration cuối (chỉ khi chưa chạy update)**
```bash
dotnet ef migrations remove
```

**4.2. Rollback DB về migration trước**
```bash
# Liệt kê migrations
dotnet ef migrations list

# Rollback về migration trước (ví dụ: InitialDb)
dotnet ef database update InitialDb
```

**4.3. Reset toàn bộ DB (xoá sạch và tạo lại)**
```bash
# Xoá DB hiện tại
dotnet ef database drop -f

# Tạo lại migration và DB
dotnet ef migrations add InitialDb
dotnet ef database update
```

> Nếu seed không chạy: kiểm tra đã gọi `DbSeeder.SeedAsync(...)` trong `Program.cs` hay chưa.

---

## ✅ Seed dữ liệu Users + Books theo role (hướng dẫn chi tiết)

### 1) Tạo các tài khoản mẫu theo role
Sau khi chạy seeding, bạn sẽ có 3 tài khoản mẫu:

| Role | Username | Password | Mục đích test |
|------|----------|----------|---------------|
| Admin | `admin` | `Admin123!` | Full quyền CRUD |
| Librarian | `librarian1` | `Lib123!` | Thêm/Sửa Books |
| Reader | `reader1` | `Reader123!` | Chỉ xem |

> Nếu muốn đổi mật khẩu, sửa trực tiếp trong `DbSeeder.cs`.

### 2) Seed Books để kiểm thử đúng quyền
Trong seeding, đã có:
- **Books công khai** (`IsPublic = true`) → Guest và Reader đều xem được.
- **Books riêng tư** (`IsPublic = false`) → Guest không xem được.

Ví dụ seed đã có (trích):
```csharp
new Book { Title = "Quản trị tài chính", Price = 135000, CategoryId = ecoCategory.Id, IsPublic = false }
```

### 3) Cách test theo role
1. **Login Admin** → xem thêm/sửa/xóa tất cả.
2. **Login Librarian** → thêm/sửa Books, **không xóa**.
3. **Login Reader** → chỉ xem (nút CRUD bị ẩn).
4. **Không login (Guest)** → chỉ thấy sách công khai.

### 4) Mở rộng: thêm Books đa dạng theo từng nhóm
Bạn có thể thêm nhiều sách mẫu hơn theo từng lĩnh vực:
```csharp
context.Books.AddRange(
    new Book { Title = "Python căn bản", Price = 90000, CategoryId = itCategory.Id, IsPublic = true },
    new Book { Title = "Kinh tế học vi mô", Price = 125000, CategoryId = ecoCategory.Id, IsPublic = true },
    new Book { Title = "Truyện ngắn chọn lọc", Price = 70000, CategoryId = litCategory.Id, IsPublic = true }
);
```

> Lưu ý: nếu đã có dữ liệu, hãy xóa DB hoặc reset migration để seed lại.

---

## 📋 PHẦN 2: XÂY DỰNG BACKEND API VỚI PHÂN QUYỀN

### 2.1. Khởi tạo dự án API

**Bước 1**: Tạo dự án API mới
```bash
mkdir LibraryManagement
cd LibraryManagement
dotnet new sln -n LibraryManagement
dotnet new webapi -n LibraryManagement.API
dotnet sln add LibraryManagement.API/LibraryManagement.API.csproj
```

**Bước 2**: Cài đặt package
```bash
cd LibraryManagement.API
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
dotnet add package AutoMapper.Extensions.Microsoft.DependencyInjection
```

### 2.2. Models & Context
*(Xem chi tiết code trong bài thực hành MVC gốc hoặc copy từ source code mẫu)*
- `Models/Book.cs`
- `Models/Category.cs`
- `Data/ApplicationDbContext.cs` (Kế thừa `IdentityDbContext`)

### 2.3. Cấu hình Program.cs (Rất quan trọng cho Vue)

> ⚠️ **LƯU Ý CORS**: Frontend Vue chạy ở port khác (ví dụ 3000), nên Backend (ví dụ 5000) phải mở CORS.

**File: `Program.cs`**
```csharp
// ... (Các phần config DB, Identity giống bài trước)

// Gợi ý: nới lỏng password rules để seed tài khoản demo
builder.Services.AddIdentity<IdentityUser, IdentityRole>(options =>
{
    options.Password.RequireDigit = false;
    options.Password.RequireLowercase = false;
    options.Password.RequireUppercase = false;
    options.Password.RequireNonAlphanumeric = false;
    options.Password.RequiredLength = 6;
})
.AddEntityFrameworkStores<ApplicationDbContext>()
.AddDefaultTokenProviders();

// QUAN TRỌNG: Cấu hình CORS
// Giải thích: Trình duyệt mặc định chặn request từ domain khác (vd: localhost:3000 gọi localhost:5000).
// Ta cần mở policy để cho phép Vue App gọi API.
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowVueApp", policy =>
    {
        policy.WithOrigins("http://localhost:5173", "http://localhost:3000") // Port của Vue
              .AllowAnyHeader()
              .AllowAnyMethod()
              .AllowCredentials(); // Cho phép gửi cookie/auth headers
    });
});

var app = builder.Build();

// ...

app.UseCors("AllowVueApp"); // Phải đặt trước Authentication để browser check pre-flight request

app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
app.Run();
```

### 2.4. Controllers API (Chi tiết)

**AuthController (Đăng ký/Đăng nhập)**
```csharp
[Route("api/[controller]")]
[ApiController]
public class AuthController : ControllerBase
{
    private readonly UserManager<IdentityUser> _userManager;
    private readonly IConfiguration _configuration;

    // POST: api/Auth/login
    // Giải thích: 
    // 1. Kiểm tra username/password từ database.
    // 2. Lấy danh sách Roles của user.
    // 3. Tạo JWT Token chứa Claims (thông tin user, roles).
    // 4. Client sẽ giữ token này để gửi kèm các request sau.
    [HttpPost("login")]
    public async Task<IActionResult> Login([FromBody] LoginDto model)
    {
        var user = await _userManager.FindByNameAsync(model.Username);
        if (user != null && await _userManager.CheckPasswordAsync(user, model.Password))
        {
            var userRoles = await _userManager.GetRolesAsync(user);

            var authClaims = new List<Claim>
            {
                new Claim(ClaimTypes.Name, user.UserName!),
                new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()),
            };

            foreach (var role in userRoles)
            {
                authClaims.Add(new Claim(ClaimTypes.Role, role));
            }

            var token = CreateToken(authClaims);

            return Ok(new
            {
                token = new JwtSecurityTokenHandler().WriteToken(token),
                expiration = token.ValidTo,
                username = user.UserName,
                roles = userRoles
            });
        }
        return Unauthorized();
    }
}
```

**CategoriesController (Phân quyền Admin)**
```csharp
[Authorize(Roles = "Admin")]
[Route("api/[controller]")]
[ApiController]
public class CategoriesController : ControllerBase
{
    // GET: api/Categories (Ai cũng xem được)
    [AllowAnonymous]
    [HttpGet]
    public async Task<ActionResult<IEnumerable<CategoryDto>>> GetCategories() { ... }

    // POST: api/Categories (Chỉ Admin)
    [HttpPost]
    public async Task<ActionResult<CategoryDto>> PostCategory(CategoryCreateDto dto) { ... }

    // DELETE: api/Categories/5 (Chỉ Admin)
    [HttpDelete("{id}")]
    public async Task<IActionResult> DeleteCategory(int id) { ... }
}
```

**BooksController (Phân quyền phức tạp)**
> 💡 **Ý tưởng**:
> - Dùng `[Authorize]` chung cho POST/PUT để chặn Guest.
> - Trong `GetBooks`, kiểm tra thủ công `User.dentity.IsAuthenticated` để quyết định trả về list full hay list public.
> - Cách này linh hoạt hơn là dùng Attribute cứng nhắc.

```csharp
[Route("api/[controller]")]
[ApiController]
public class BooksController : ControllerBase
{
    // GET: api/Books (Guest chỉ xem sách Public)
    [AllowAnonymous]
    [HttpGet]
    public async Task<ActionResult<IEnumerable<BookDto>>> GetBooks()
    {
        // Logic kiểm tra User.Identity.IsAuthenticated và lọc sách
    }

    // POST: api/Books (Admin hoặc Librarian)
    [Authorize(Roles = "Admin, Librarian")]
    [HttpPost]
    public async Task<ActionResult<BookDto>> PostBook(BookCreateDto dto) { ... }

    // DELETE: api/Books/5 (Chỉ Admin)
    [Authorize(Roles = "Admin")]
    [HttpDelete("{id}")]
    public async Task<IActionResult> DeleteBook(int id) { ... }
}
```

---

## 🧪 PHẦN 2.5: KIỂM THỬ API BẰNG POSTMAN (CHI TIẾT)

### 1) Tạo Collection và Environment
1. Mở Postman → **New** → **Collection** → đặt tên `Library API`.
2. Tạo **Environment** tên `Local`.
3. Thêm biến:
   - `baseUrl` = `https://localhost:5001`
   - `token` = (để trống)

### 2) Login để lấy token
**POST** `{{baseUrl}}/api/Auth/login`

**Body (raw JSON):**
```json
{
  "username": "admin",
  "password": "Admin123!"
}
```

**Test script (tab Tests):**
```javascript
const json = pm.response.json();
pm.environment.set("token", json.token);
```

> Chạy request này xong thì `token` sẽ được lưu cho các request sau.

### 3) Test Categories
**GET** `{{baseUrl}}/api/Categories` (public)

**POST** `{{baseUrl}}/api/Categories` (Admin)
Headers:
- `Authorization: Bearer {{token}}`
- `Content-Type: application/json`

Body:
```json
{
  "name": "Khoa học dữ liệu",
  "description": "Data Science"
}
```

### 4) Test Books
**GET** `{{baseUrl}}/api/Books` (Guest chỉ thấy `IsPublic = true`)

**POST** `{{baseUrl}}/api/Books` (Admin/Librarian)
Headers:
- `Authorization: Bearer {{token}}`
- `Content-Type: application/json`

Body:
```json
{
  "title": "Nhập môn AI",
  "author": "Nguyễn Văn A",
  "isbn": "978-1111111111",
  "publishedYear": 2024,
  "price": 200000,
  "categoryId": 1,
  "isPublic": true
}
```

**PUT** `{{baseUrl}}/api/Books/1` (Admin/Librarian)
Headers:
- `Authorization: Bearer {{token}}`
- `Content-Type: application/json`

Body:
```json
{
  "title": "Nhập môn AI (Updated)",
  "author": "Nguyễn Văn A",
  "isbn": "978-1111111111",
  "publishedYear": 2024,
  "price": 220000,
  "categoryId": 1,
  "isPublic": true
}
```

**DELETE** `{{baseUrl}}/api/Books/1` (chỉ Admin)

### 5) Checklist nhanh
- [ ] Login OK, token lưu vào environment
- [ ] GET Categories OK (200)
- [ ] POST Categories: Admin OK, Librarian 403
- [ ] POST Books: Admin/Librarian OK
- [ ] PUT Books: Admin/Librarian OK
- [ ] DELETE Books: chỉ Admin OK

### 6) Import trực tiếp Postman Collection
Bạn có thể import file sau vào Postman:
- `postman/LibraryManagement_API.postman_collection.json`

Cách import:
1. Mở Postman → **Import**.
2. Chọn file JSON ở trên.
3. Tạo Environment `Local` với biến:
   - `baseUrl` = `https://localhost:5001`
   - `token` = (để trống, sẽ tự set sau khi login)

---

## 📋 PHẦN 3: XÂY DỰNG FRONTEND VUE.JS (CHI TIẾT)

### 3.1. Khởi tạo dự án Vue + Vite

**Bước 1**: Tạo dự án Vue mới bằng Vite
```bash
# Quay ra thư mục gốc
cd ..

# Tạo dự án Vue (chọn Vue, JavaScript/TypeScript)
npm create vite@latest LibraryManagement.Client -- --template vue

cd LibraryManagement.Client
npm install
```

**Bước 2**: Cài đặt các thư viện cần thiết
```bash
# 1. Vuetify (UI Component Library) + Font icon
npm add vuetify @mdi/font

# 2. Router (Điều hướng trang)
npm add vue-router@4

# 3. Pinia (Quản lý State - thay thế Vuex)
npm add pinia

# 4. Axios (Gọi API)
npm add axios

# 5. Jwt Decode (Giải mã token để lấy role)
npm add jwt-decode
```

### 3.2. Cấu trúc thư mục chuẩn (Best Practice)
Hãy tạo cấu trúc thư mục như sau để dễ quản lý:

```
src/
├── assets/          # CSS, Images
├── components/      # Các component nhỏ (Button, Modal...)
├── layouts/         # Bố cục trang (AppLayout, GuestLayout)
├── plugins/         # Cấu hình Vuetify, Axios...
├── router/          # Cấu hình route và Guard
├── services/        # Các hàm gọi API (AuthService, BookService...)
├── stores/          # Pinia stores (AuthStore...)
├── views/           # Các trang chính (Pages)
│   ├── Auth/        # Login.vue, Register.vue
│   ├── Admin/       # Dashboard.vue, Users.vue
│   ├── Public/      # Home.vue, BookList.vue
├── App.vue
└── main.js
```

### 3.3. Cấu hình Vuetify
**File: `src/plugins/vuetify.js`**
```javascript
import 'vuetify/styles'
import { createVuetify } from 'vuetify'
import * as components from 'vuetify/components'
import * as directives from 'vuetify/directives'
import '@mdi/font/css/materialdesignicons.css'

export default createVuetify({
  components,
  directives,
  theme: {
    defaultTheme: 'light'
  }
})
```

**File: `src/main.js`**
```javascript
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'
import router from './router'
import vuetify from './plugins/vuetify'

const app = createApp(App)

app.use(createPinia()) // State Management
app.use(router)        // Routing
app.use(vuetify)       // UI

app.mount('#app')
```

### 3.4. Cấu hình Axios & JWT (Cực quan trọng)

> 💡 **Giải thích ý tưởng**:
> - Thay vì mỗi lần gọi API phải thủ công thêm `headers: { Authorization: ... }`, ta dùng **Interceptor**.
> - Interceptor giống như một "trạm kiểm soát": mọi request đi ra đều phải qua trạm này để được đóng dấu (gắn token).
> - Tương tự, mọi response về nếu bị lỗi 401 (hết hạn token) sẽ bị chặn lại để xử lý (logout).

Chúng ta cần tạo một instance Axios để tự động gắn token vào mỗi request.

**File: `src/services/axiosClient.js`**
```javascript
import axios from 'axios'
import { useAuthStore } from '../stores/auth'

const axiosClient = axios.create({
  baseURL: 'https://localhost:5001', // URL API Backend
  headers: {
    'Content-Type': 'application/json'
  }
})

// Interceptor: Gắn Token vào Header trước khi gửi Request
axiosClient.interceptors.request.use((config) => {
  const authStore = useAuthStore()
  if (authStore.token) {
    config.headers.Authorization = `Bearer ${authStore.token}`
  }
  return config
})

// Interceptor: Xử lý lỗi tr
axiosClient.interceptors.response.use(
  (response) => response,
  (error) => {
    // Nếu lỗi 401 (Unauthorized) -> Token hết hạn hoặc không hợp lệ
    if (error.response && error.response.status === 401) {
      const authStore = useAuthStore()
      authStore.logout() // Xóa token và redirect về login
    }
    return Promise.reject(error)
  }
)

export default axiosClient
```

### 3.5. Pinia Store: Quản lý Auth State

> 💡 **Giải thích ý tưởng**:
> - Frontend cần biết "User đã đăng nhập chưa?" ở mọi nơi (Header, trang Admin, trang Books).
> - **Pinia** giúp lưu trạng thái này vào một chỗ chung (Store).
> - **LocalStorage**: Giúp giữ trạng thái đăng nhập khi người dùng nhấn F5 (Refresh trang).

Lưu trữ Token và thông tin User.

**File: `src/stores/auth.js`**
```javascript
import { defineStore } from 'pinia'
import axiosClient from '../services/axiosClient'
import jwt_decode from 'jwt-decode'

export const useAuthStore = defineStore('auth', {
  state: () => ({
    token: localStorage.getItem('token') || '',
    user: JSON.parse(localStorage.getItem('user')) || null,
    roles: JSON.parse(localStorage.getItem('roles')) || []
  }),

  getters: {
    isAuthenticated: (state) => !!state.token,
    isAdmin: (state) => state.roles.includes('Admin'),
    isLibrarian: (state) => state.roles.includes('Librarian'),
  },

  actions: {
    async login(username, password) {
      try {
        const response = await axiosClient.post('/api/Auth/login', { username, password })
        const { token, username: user, roles } = response.data

        // Lưu vào State
        this.token = token
        this.user = user
        this.roles = roles

        // Lưu vào LocalStorage (để F5 không mất)
        localStorage.setItem('token', token)
        localStorage.setItem('user', JSON.stringify(user))
        localStorage.setItem('roles', JSON.stringify(roles))

        return true
      } catch (error) {
        console.error('Login failed', error)
        return false
      }
    },

    logout() {
      this.token = ''
      this.user = null
      this.roles = []
      localStorage.clear()
      window.location.href = '/login'
    }
  }
})
```

### 3.6. Router & Navigation Guard (Bảo vệ trang)

> 💡 **Giải thích ý tưởng**:
> - Router Guard (`beforeEach`) chạy trước mỗi lần chuyển trang.
> - Nó đóng vai trò "Bảo vệ": Khách không có vé (token) thì không cho vào khu Admin.
> - **Lưu ý**: Đây chỉ là bảo vệ mặt giao diện (UX). Bảo vệ thật sự nằm ở Backend API (`[Authorize]`).

Chặn không cho user chưa đăng nhập vào trang Admin.

**File: `src/router/index.js`**
```javascript
import { createRouter, createWebHistory } from 'vue-router'
import { useAuthStore } from '../stores/auth'
import DefaultLayout from '../layouts/DefaultLayout.vue'

const routes = [
  {
    path: '/login',
    component: () => import('../views/Auth/LoginPage.vue'),
    meta: { layout: 'guest' }
  },
  {
    path: '/',
    component: DefaultLayout,
    meta: { requiresAuth: true }, // Yêu cầu đăng nhập
    children: [
      { path: '', redirect: '/books' },
      { 
        path: 'books', 
        component: () => import('../views/Public/BookList.vue') 
      },
      { 
        path: 'categories', 
        component: () => import('../views/Admin/Categories.vue'),
        meta: { roles: ['Admin'] } // Chỉ Admin mới vào được
      }
    ]
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

// Guard kiểm tra quyền
router.beforeEach((to, from, next) => {
  const authStore = useAuthStore()
  const requiresAuth = to.matched.some(record => record.meta.requiresAuth)
  const allowedRoles = to.meta.roles

  // 1. Nếu trang yêu cầu Auth mà chưa login -> Về Login
  if (requiresAuth && !authStore.isAuthenticated) {
    return next('/login')
  }

  // 2. Nếu đã login nhưng không đúng Role -> Chặn
  if (allowedRoles) {
    const hasPermission = allowedRoles.some(role => authStore.roles.includes(role))
    if (!hasPermission) {
      alert('Bạn không có quyền truy cập trang này!')
      return next('/')
    }
  }

  next()
})

export default router
```

---

## 📋 PHẦN 4: GIAO DIỆN ĐĂNG NHẬP (LOGIN PAGE)

**File: `src/views/Auth/LoginPage.vue`**
```html
<template>
  <v-container class="fill-height justify-center">
    <v-card width="400" elevation="10" class="rounded-lg">
      <v-card-title class="text-h5 text-center bg-primary text-white py-4">
        📚 Đăng Nhập Thư Viện
      </v-card-title>
      
      <v-card-text class="mt-4">
        <v-form @submit.prevent="handleLogin">
          <v-text-field
            v-model="username"
            label="Tên đăng nhập"
            prepend-inner-icon="mdi-account"
            variant="outlined"
          ></v-text-field>

          <v-text-field
            v-model="password"
            label="Mật khẩu"
            type="password"
            prepend-inner-icon="mdi-lock"
            variant="outlined"
          ></v-text-field>

          <v-btn
            type="submit"
            color="primary"
            block
            size="large"
            :loading="loading"
          >
            Đăng nhập
          </v-btn>
        </v-form>
      </v-card-text>
    </v-card>
  </v-container>
</template>

<script setup>
import { ref } from 'vue'
import { useAuthStore } from '../../stores/auth'
import { useRouter } from 'vue-router'

const username = ref('')
const password = ref('')
const loading = ref(false)
const authStore = useAuthStore()
const router = useRouter()

const handleLogin = async () => {
  loading.value = true
  const success = await authStore.login(username.value, password.value)
  loading.value = false

  if (success) {
    router.push('/')
  } else {
    alert('Đăng nhập thất bại! Kiểm tra lại thông tin.')
  }
}
</script>
```

---

## 📋 PHẦN 5: QUẢN LÝ CATEGORIES (ROLE ADMIN)

> 💡 **Giải thích ý tưởng**:
> - Trang này dùng **Composition API** (`<script setup>`) giúp code gọn hơn.
> - **Reactivity**: Các biến `categories`, `dialog` dùng `ref()` để khi dữ liệu đổi, giao diện tự cập nhật.
> - **Tái sử dụng Dialog**: Một Dialog dùng chung cho cả Thêm và Sửa (phân biệt bằng `editedId`).

Trang này chỉ dành cho Admin. Chúng ta sẽ dùng `v-data-table` để hiển thị danh sách và `v-dialog` để tạo form thêm/sửa.

**File: `src/views/Admin/Categories.vue`**
```html
<template>
  <v-container>
    <v-row class="mb-4" align="center">
      <v-col>
        <h2 class="text-h4">📂 Quản lý Thể loại</h2>
      </v-col>
      <v-col class="text-right">
        <!-- Chặn nút thêm ở giao diện nếu không phải Admin (Double check) -->
        <v-btn v-if="authStore.isAdmin" color="primary" @click="openDialog()">
          <v-icon start>mdi-plus</v-icon> Thêm mới
        </v-btn>
      </v-col>
    </v-row>

    <v-data-table
      :headers="headers"
      :items="categories"
      :loading="loading"
      class="elevation-1"
    >
      <template v-slot:item.actions="{ item }">
        <v-btn
          v-if="authStore.isAdmin"
          icon="mdi-pencil"
          size="small"
          color="warning"
          class="mr-2"
          @click="openDialog(item)"
        ></v-btn>
        <v-btn
          v-if="authStore.isAdmin"
          icon="mdi-delete"
          size="small"
          color="error"
          @click="deleteCategory(item.id)"
        ></v-btn>
      </template>
    </v-data-table>

    <!-- Dialog Thêm/Sửa -->
    <v-dialog v-model="dialog" max-width="500px">
      <v-card>
        <v-card-title>
          {{ editedId ? 'Sửa thể loại' : 'Thêm thể loại mới' }}
        </v-card-title>
        <v-card-text>
          <v-text-field v-model="form.name" label="Tên thể loại"></v-text-field>
          <v-textarea v-model="form.description" label="Mô tả"></v-textarea>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue-darken-1" variant="text" @click="dialog = false">Hủy</v-btn>
          <v-btn color="blue-darken-1" variant="text" @click="save">Lưu</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useAuthStore } from '../../stores/auth'
import axiosClient from '../../services/axiosClient'

const authStore = useAuthStore()
const categories = ref([])
const loading = ref(false)
const dialog = ref(false)
const editedId = ref(null)

const form = ref({ name: '', description: '' })

const headers = [
  { title: 'ID', key: 'id' },
  { title: 'Tên thể loại', key: 'name' },
  { title: 'Mô tả', key: 'description' },
  { title: 'Thao tác', key: 'actions', sortable: false },
]

// Hàm load dữ liệu
const fetchCategories = async () => {
  loading.value = true
  try {
    const res = await axiosClient.get('/api/Categories')
    categories.value = res.data
  } catch (err) {
    console.error(err)
  } finally {
    loading.value = false
  }
}

// Hàm mở dialog
const openDialog = (item = null) => {
  if (item) {
    editedId.value = item.id
    form.value = { ...item }
  } else {
    editedId.value = null
    form.value = { name: '', description: '' }
  }
  dialog.value = true
}

// Hàm lưu (Quyết định Create hay Update)
const save = async () => {
  try {
    if (editedId.value) {
      // Update
      await axiosClient.put(`/api/Categories/${editedId.value}`, form.value)
    } else {
      // Create
      await axiosClient.post('/api/Categories', form.value)
    }
    dialog.value = false
    fetchCategories() // Reload lại bảng
  } catch (err) {
    alert('Lỗi: ' + (err.response?.data?.message || err.message))
  }
}

const deleteCategory = async (id) => {
  if (!confirm('Bạn có chắc muốn xóa?')) return
  try {
    await axiosClient.delete(`/api/Categories/${id}`)
    fetchCategories()
  } catch (err) {
    alert('Không thể xóa! Có thể thể loại này đang có sách.')
  }
}

onMounted(fetchCategories)
</script>
```

---

## 📋 PHẦN 6: QUẢN LÝ BOOKS (ADMIN + LIBRARIAN)

> 💡 **Giải thích ý tưởng**:
> - **Phân quyền UI**: Thay vì viết `v-if="authStore.isAdmin || authStore.isLibrarian"` nhiều lần, ta dùng `computed` để tạo biến `isAdminOrLib`.
> - **Logic hiển thị**:
>   - Nút **Thêm/Sửa**: Hiện với Admin & Librarian.
>   - Nút **Xóa**: CHỈ hiện với Admin.
>   - Guest/Reader: Không thấy nút chức năng nào.

Trang này thể hiện rõ nhất quyền hạn:
- **Reader/Guest**: Chỉ xem.
- **Librarian**: Xem + Thêm + Sửa.
- **Admin**: Xem + Thêm + Sửa + Xóa.

**File: `src/views/Public/BookList.vue`**
```html
<template>
  <v-container>
    <v-row class="mb-4">
      <v-col><h2>📚 Danh sách Sách</h2></v-col>
      <v-col class="text-right">
        <!-- Chỉ Admin hoặc Librarian mới thấy nút Thêm -->
        <v-btn 
          v-if="authStore.isAdmin || authStore.isLibrarian" 
          color="success"
          @click="openDialog()"
        >
          <v-icon start>mdi-plus</v-icon> Thêm Sách
        </v-btn>
      </v-col>
    </v-row>

    <v-row>
      <v-col v-for="book in books" :key="book.id" cols="12" md="4">
        <v-card class="h-100" elevation="2">
          <v-card-title>{{ book.title }}</v-card-title>
          <v-card-subtitle>{{ book.author }}</v-card-subtitle>
          <v-card-text>
            <p><strong>Giá:</strong> {{ book.price }} VND</p>
            <p><strong>Thể loại:</strong> {{ book.categoryName }}</p>
          </v-card-text>
          
          <v-divider></v-divider>
          
          <v-card-actions>
            <!-- Nút Chi tiết (Ai cũng thấy) -->
            <v-btn variant="text" color="primary">Chi tiết</v-btn>
            <v-spacer></v-spacer>
            
            <!-- Nút Sửa (Admin + Librarian) -->
            <v-btn 
              v-if="isAdminOrLib" 
              icon="mdi-pencil" 
              color="warning" 
              variant="text"
              @click="openDialog(book)"
            ></v-btn>

            <!-- Nút Xóa (CHỈ ADMIN) -->
            <v-btn 
              v-if="authStore.isAdmin" 
              icon="mdi-delete" 
              color="error" 
              variant="text"
              @click="deleteBook(book.id)"
            ></v-btn>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>

    <!-- Dialog Thêm/Sửa (Lược giản cho gọn, logic giống Categories) -->
    <!-- ... -->
  </v-container>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useAuthStore } from '../../stores/auth'
import axiosClient from '../../services/axiosClient'

const authStore = useAuthStore()
const books = ref([])

// Computed property để check quyền gọn hơn
const isAdminOrLib = computed(() => authStore.isAdmin || authStore.isLibrarian)

const fetchBooks = async () => {
  const res = await axiosClient.get('/api/Books')
  books.value = res.data
}

const deleteBook = async (id) => {
  if (!confirm('Xóa sách này?')) return
  await axiosClient.delete(`/api/Books/${id}`)
  fetchBooks()
}

onMounted(fetchBooks)
</script>
```

---

## 📋 PHẦN 7: TEST TOÀN BỘ HỆ THỐNG

### 7.1. Chạy Backend API
```bash
cd LibraryManagement.API
dotnet run
# API sẽ chạy tại https://localhost:5001
```

### 7.2. Chạy Frontend Vue
```bash
cd LibraryManagement.Client
npm run dev
# Vue sẽ chạy tại http://localhost:5173
```

### 7.3. Kịch bản Test (Test Cases)

**Test 1: Admin Login**
1. Đăng nhập với `admin` / `Admin123`.
2. Kiểm tra LocalStorage: Phải có `token` và roles `["Admin"]`.
3. Vào trang Categories: Thấy nút Thêm/Sửa/Xóa.
4. Thử Xóa 1 category -> Thành công.

**Test 2: Librarian Login**
1. Đăng nhập với `librarian` / `Lib123`.
2. Vào trang Categories: KHÔNG thấy nút Thêm/Sửa/Xóa (chỉ xem).
3. Vào trang Books: Thấy nút Thêm/Sửa, NHƯNG KHÔNG thấy nút Xóa.

**Test 3: Guest Access**
1. Đăng xuất (Xóa token).
2. Thử truy cập `/admin/categories` -> Bị đẩy về `/login`.
3. Vào trang chủ -> Chỉ thấy danh sách sách công khai (`IsPublic=true`).

---

## ✅ CHECKLIST HOÀN THÀNH

- [ ] API trả về Token JWT chuẩn (có Roles).
- [ ] Frontend Vue Login được và lưu Token.
- [ ] Axios tự động gắn Token vào Header.
- [ ] Router chặn được người không có quyền.
- [ ] Admin có full quyền CRUD.
- [ ] Librarian bị giới hạn quyền Xóa.

---

## 🎯 KẾT LUẬN

Phiên bản Vue.js này hiện đại hơn MVC Razor ở chỗ:
1. **Trải nghiệm người dùng tốt hơn (SPA)**: Không load lại trang.
2. **Tách biệt Frontend/Backend**: Team Frontend và Backend có thể làm việc độc lập.
3. **Bảo mật**: Sử dụng JWT chuẩn công nghiệp.
4. **Kiến trúc sạch**: Pinia Store quản lý state, Router quản lý điều hướng.

Chúc các bạn thực hành tốt! 🚀




