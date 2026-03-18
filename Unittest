การเขียน **Unit Test** คือหัวใจสำคัญที่ทำให้โค้ดของเรา "พังยาก" และ "แก้ไขง่าย" ครับ ในโลกของ .NET เรามักจะใช้ **xUnit** เป็น Testing Framework และใช้ **Moq** เพื่อจำลอง (Mock) ข้อมูลจากเลเยอร์อื่น เพื่อให้เราทดสอบเฉพาะ Logic ในเลเยอร์นั้นๆ ได้จริงๆ

มาเจาะลึกการทดสอบในส่วนที่สำคัญที่สุดคือ **Service Layer** กันครับ

---

### **1. เครื่องมือที่ต้องใช้ (The Toolbox)**
ก่อนเริ่ม คุณต้องติดตั้ง NuGet Packages เหล่านี้ในโปรเจกต์ Test:
* **xUnit**: ตัวรัน Test หลัก
* **Moq**: ใช้สร้าง "ของปลอม" (Mock Objects) มาแทน Repository
* **FluentAssertions**: (เสริม) ช่วยให้เขียนคำสั่งเช็คผลลัพธ์ (Assert) ได้เหมือนภาษาพูดมากขึ้น

---

### **2. คอนเซปต์ AAA (Arrange, Act, Assert)**
ทุุกๆ Unit Test ควรมีโครงสร้าง 3 ส่วนนี้เสมอ:
1.  **Arrange (เตรียมการ):** ตั้งค่าตัวแปร, สร้าง Mock และจำลองพฤติกรรมของ Dependency
2.  **Act (ลงมือ):** เรียกใช้ Method ที่เราต้องการทดสอบจริงๆ
3.  **Assert (ตรวจสอบ):** เช็คว่าผลลัพธ์ที่ได้ ตรงกับที่เราคาดหวังไหม



---

### **3. ตัวอย่างการเขียน Unit Test สำหรับ ProductService**

โจทย์: เราจะทดสอบเมธอด `IsAvailable(id)` ว่าทำงานถูกต้องไหม โดยไม่ต้องต่อฐานข้อมูลจริง

```csharp
using Moq;
using Xunit;
using FluentAssertions;

public class ProductServiceTests
{
    private readonly Mock<IProductRepository> _mockRepo;
    private readonly ProductService _service;

    public ProductServiceTests()
    {
        // Arrange: สร้าง Repository ปลอมขึ้นมา
        _mockRepo = new Mock<IProductRepository>();
        
        // ส่ง Repo ปลอมเข้าไปใน Service จริง (Dependency Injection)
        _service = new ProductService(_mockRepo.Object);
    }

    [Fact] // บอกว่าเป็น Unit Test
    public void IsAvailable_WhenProductExistsAndHasStock_ShouldReturnTrue()
    {
        // 1. Arrange: จำลองว่าถ้าถามหา ID 1 ให้คืนค่า Product ที่มีสต็อก 10 ชิ้น
        var p = new Product { Id = 1, Name = "Laptop", StockQuantity = 10 };
        _mockRepo.Setup(repo => repo.GetById(1)).Returns(p);

        // 2. Act: เรียกใช้งาน Service จริงๆ
        var result = _service.IsAvailable(1);

        // 3. Assert: ผลลัพธ์ต้องเป็น true
        result.Should().BeTrue();
    }

    [Fact]
    public void IsAvailable_WhenStockIsZero_ShouldReturnFalse()
    {
        // 1. Arrange: จำลองสินค้าที่สต็อกหมด (0 ชิ้น)
        var p = new Product { Id = 2, Name = "Keyboard", StockQuantity = 0 };
        _mockRepo.Setup(repo => repo.GetById(2)).Returns(p);

        // 2. Act
        var result = _service.IsAvailable(2);

        // 3. Assert: ต้องเป็น false เพราะไม่มีของ
        result.Should().BeFalse();
    }
}
```

---

### **4. ทำไมเราต้อง Mock Repository?**
ลองนึกภาพว่าถ้าเราไม่ใช้ Moq แต่ใช้ Repository จริง:
* คุณต้องมีฐานข้อมูลจริงที่มีข้อมูลนั้นๆ อยู่
* ถ้าใครไปลบข้อมูลใน DB ทิ้ง Test ของคุณจะ "แดง" ทั้งที่โค้ดไม่ได้บั๊ก
* Test จะรันช้ามากเพราะต้องรอเชื่อมต่อ Network/Disk

**การ Mock** คือการตัดขาดจากปัจจัยภายนอก เพื่อทดสอบแค่ Logic ใน `ProductService` เพียวๆ ครับ

---

### **5. ข้อควรระวัง (Best Practices)**
* **1 Test = 1 Case:** อย่าเขียน Test เดียวเช็คทุกอย่าง แยกเป็น Case ที่ "สำเร็จ" และ "ล้มเหลว"
* **ชื่อ Method ต้องชัดเจน:** เช่น `MethodName_StateUnderTest_ExpectedBehavior` เพื่อให้เวลา Error แล้วรู้ทันทีว่าเคสไหนพัง
* **Isolated:** Test ตัวหนึ่งพัง ต้องไม่ส่งผลให้ Test อีกตัวพังตาม



---

> **Pro-Tip:** การเขียน Unit Test จะบังคับให้คุณเขียนโค้ดแบบเลเยอร์โดยปริยาย เพราะถ้าคุณเขียนโค้ดแบบ "ยำรวมกัน" ใน Controller คุณจะ Mock อะไรไม่ได้เลยและเขียน Test ยากมากครับ

**อยากลองให้ผมช่วยออกแบบเคสทดสอบสำหรับฟังก์ชันที่ซับซ้อนกว่านี้ เช่น "ระบบคำนวณส่วนลดตามโปรโมชั่น" ไหมครับ?**
