

# 📂 .NET Layered Architecture Guide (Full Version)

เอกสารฉบับนี้อธิบายโครงสร้างการเขียนโปรแกรม .NET แบบแบ่งเลเยอร์ (Clean Architecture / Repository Pattern) เพื่อความยืดหยุ่นและการบำรุงรักษาโค้ดในระยะยาว

---

## 🏗️ 1. ตารางสรุปหน้าที่แต่ละเลเยอร์ (Layer Responsibilities)

| โฟลเดอร์ (Folder) | หน้าที่หลัก | หน้าที่เปรียบเทียบ |
| :--- | :--- | :--- |
| **Models/Entities** | นิยามโครงสร้างข้อมูล (Database Schema) | **วัตถุดิบ / พัสดุ** |
| **Repositories** | จัดการ Database (CRUD) โดยตรง | **พนักงานคลังสินค้า** |
| **Services** | เก็บ Business Logic และกฎการคำนวณ | **พ่อครัว / ผู้จัดการ** |
| **Controllers** | รับ HTTP Request และควบคุมทิศทาง | **พนักงานรับออเดอร์** |
| **Views** | หน้าจอแสดงผล (UI) สำหรับ User | **เมนู / หน้าร้าน** |

---

## 🔄 2. แผนผังลำดับการทำงาน (Workflow Flowchart)

ลำดับการไหลของข้อมูล (Data Flow) จะเป็นไปตามแผนผังด้านล่างนี้:



### **ขั้นตอนการทำงาน (The Sequence):**
1.  **User Interaction:** ผู้ใช้คลิกปุ่มบน **View** (เช่น ปุ่ม "ตรวจสอบสต็อกสินค้า")
2.  **Request:** ข้อมูลถูกส่งไปยัง **Controller** ผ่าน URL/API Route
3.  **Service Call:** Controller เรียกใช้งาน **Service** (เพื่อคำนวณหรือตรวจสอบเงื่อนไข)
4.  **Database Access:** Service เรียกใช้งาน **Repository** เพื่อขอข้อมูลจริงจากฐานข้อมูล
5.  **Query:** Repository ใช้ **Model** เป็นโครงสร้างในการดึงข้อมูลจาก **Database**
6.  **Response Return:** ข้อมูลวิ่งย้อนกลับเลเยอร์เดิม (Database → Repo → Service → Controller)
7.  **Final Result:** Controller ส่งข้อมูลที่ประมวลผลแล้วไปแสดงผลที่ **View**

---

## 💻 3. ตัวอย่างโค้ดแบบสมบูรณ์ (Complete Code Snippets)

### **เลเยอร์ 1: Model (Entities/Product.cs)**
```csharp
namespace MyApp.Models
{
    public class Product 
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public decimal Price { get; set; }
        public int StockQuantity { get; set; }
    }
}
```

### **เลเยอร์ 2: Repository (Repositories/ProductRepository.cs)**
```csharp
public interface IProductRepository {
    Product GetById(int id);
}

public class ProductRepository : IProductRepository 
{
    private readonly MyDbContext _db;
    public ProductRepository(MyDbContext db) => _db = db;

    public Product GetById(int id) => _db.Products.Find(id);
}
```

### **เลเยอร์ 3: Service (Services/ProductService.cs)**
```csharp
public interface IProductService {
    bool IsAvailable(int id);
}

public class ProductService : IProductService 
{
    private readonly IProductRepository _repo;
    public ProductService(IProductRepository repo) => _repo = repo;

    public bool IsAvailable(int id) 
    {
        var product = _repo.GetById(id);
        // Business Logic: ต้องมีสินค้ามากกว่า 0 ชิ้นถึงจะถือว่า Available
        return product != null && product.StockQuantity > 0;
    }
}
```

### **เลเยอร์ 4: Controller (Controllers/ProductController.cs)**
```csharp
public class ProductController : Controller 
{
    private readonly IProductService _service;
    public ProductController(IProductService service) => _service = service;

    public IActionResult CheckStock(int id) 
    {
        var result = _service.IsAvailable(id);
        return View(result); // ส่งค่า True/False ไปที่ View
    }
}
```

---

## ⚙️ 4. การตั้งค่าระบบ (Dependency Injection)

เพื่อให้เลเยอร์ต่างๆ คุยกันได้ คุณต้องลงทะเบียนในไฟล์ **`Program.cs`** ดังนี้:

```csharp
// Program.cs
builder.Services.AddScoped<IProductRepository, ProductRepository>();
builder.Services.AddScoped<IProductService, ProductService>();
```

---

> **Note:** การแยกเลเยอร์แบบนี้ช่วยให้คุณสามารถทำ **Unit Test** เฉพาะในส่วนของ Service ได้โดยไม่ต้องต่อฐานข้อมูลจริง ซึ่งเป็นมาตรฐานที่องค์กรใหญ่ๆ ใช้กันครับ

---
