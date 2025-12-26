# 📘 Entity Framework Core 

---

## 1️⃣ Introduction

**ما هو Entity Framework Core؟**
Entity Framework Core هو ORM (Object Relational Mapper) بيسمح لنا نتعامل مع قاعدة البيانات باستخدام كائنات C# بدل ما نكتب SQL Queries يدوي.

**ليه نستخدم EF Core؟**

* يقلل كتابة SQL
* يسهل التعامل مع البيانات
* مناسب مع ASP.NET Core
* يدعم Migrations

**ORM يعني إيه؟**
هو أداة بتربط بين:

* Classes في الكود
* Tables في قاعدة البيانات

بحيث كل Class يمثل Table وكل Property تمثل Column.

---

## 2️⃣ Add DbContext and Connection String

**DbContext** هو الكلاس الأساسي اللي بيتم من خلاله التواصل مع قاعدة البيانات.

### الخطوات:

1. إنشاء كلاس يرث من `DbContext`
2. تعريف `DbSet` لكل Table
3. إضافة Connection String في `appsettings.json`
4. تسجيل DbContext داخل `Program.cs`

### مثال DbContext:

```csharp
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) {}

    public DbSet<Student> Students { get; set; }
}
```

### مثال Connection String:

```json
"ConnectionStrings": {
  "DefaultConnection": "Server=.;Database=EfCoreDb;Trusted_Connection=True;"
}
```

---

## 3️⃣ Add First Migration

**Migration** هي طريقة EF Core لتتبع أي تغيير يحصل في الـ Models.

### ليه نستخدم Migration؟

* إنشاء قاعدة البيانات من الكود
* تحديث الجداول بسهولة
* التحكم في تاريخ التعديلات

### أوامر مهمة:

```powershell
Add-Migration InitialCreate
Update-Database
```

* `Add-Migration` → بيعمل Snapshot للتغييرات
* `Update-Database` → بيطبق التغييرات على قاعدة البيانات

---

## 4️⃣ Save New Record (.Add())

في EF Core، إضافة بيانات جديدة بتتم عن طريق DbContext.

### الخطوات:

1. إنشاء Object جديد
2. استخدام `Add()`
3. حفظ التغييرات باستخدام `SaveChanges()`

### مثال:

```csharp
var student = new Student
{
    Name = "Ahmed",
    Age = 22
};

context.Students.Add(student);
context.SaveChanges();
```

⚠️ بدون `SaveChanges()` البيانات مش هتتحفظ في قاعدة البيانات.

---

## 5️⃣ Migration Rollback

Rollback يعني الرجوع لنسخة أقدم من قاعدة البيانات.

### نستخدمه إمتى؟

* لو Migration فيها خطأ
* لو غيرت رأيك في Structure
* أثناء التطوير فقط

### أوامر:

```powershell
Update-Database MigrationName
Remove-Migration
```

* `Update-Database` → بيرجع لنسخة معينة
* `Remove-Migration` → يحذف آخر Migration (لو لم تُطبق)

⚠️ يُفضّل عدم استخدام Rollback في Production.

---

## 📝 ملاحظات عامة

* EF Core بيعتمد على Code First
* أي تغيير في Model لازم Migration جديدة
* DbContext لازم يكون مسجل في Dependency Injection



---

## 6️⃣ Add Your Own Migration

 إزاي نضيف Migration بإسم مخصص يعبر عن التغيير اللي حصل في المشروع.

### الفكرة:

* أي تعديل في الـ Model (إضافة Property / تعديل نوع بيانات)
* لازم يقابله Migration جديدة

### مثال:

```powershell
Add-Migration AddStudentEmailColumn
Update-Database
```

🎯 الهدف: إن اسم الـ Migration يكون واضح وسهل الرجوع له بعدين.

---

## 7️⃣ DataAnnotations vs Fluent API

عندي طريقتين لتعريف القيود (Constraints) على الجداول.

### 🔹 Data Annotations

* بتتكتب فوق الـ Properties
* سهلة وسريعة

```csharp
[Required]
[StringLength(50)]
public string Name { get; set; }
```

### 🔹 Fluent API

* بتتكتب داخل `OnModelCreating`
* أقوى ومناسبة للعلاقات المعقدة

```csharp
modelBuilder.Entity<Student>()
    .Property(s => s.Name)
    .IsRequired()
    .HasMaxLength(50);
```

### الفرق بينهم:

* DataAnnotations → بسيطة
* Fluent API → مرنة وأقوى

---

## 8️⃣ Mark Column as Required

بيركّز على جعل العمود **إجباري (NOT NULL)**.

### باستخدام Data Annotation:

```csharp
[Required]
public string Email { get; set; }
```

### باستخدام Fluent API:

```csharp
modelBuilder.Entity<Student>()
    .Property(s => s.Email)
    .IsRequired();
```

⚠️ بعد التعديل لازم نعمل Migration جديدة.

---

## 9️⃣ Add Entity To Model

إزاي نضيف Entity جديدة للمشروع.

### الخطوات:

1. إنشاء Class جديد (Entity)
2. إضافته كـ `DbSet` داخل DbContext
3. عمل Migration
4. Update Database

### مثال:

```csharp
public DbSet<Course> Courses { get; set; }
```

🎯 الهدف: إضافة جدول جديد لقاعدة البيانات من خلال الكود.

---

## 🔟 Exclude Entity From Model Or From Migration

هنا بيشرح إزاي نستبعد Entity من EF Core.

### استبعاد Entity من Model:

```csharp
[NotMapped]
public class TempData
{
    public string Value { get; set; }
}
```

### استبعاد Entity باستخدام Fluent API:

```csharp
modelBuilder.Ignore<TempData>();
```

### نستخدمه إمتى؟

* Classes مساعدة
* DTOs
* بيانات مش محتاجينها في Database



---

## 1️⃣1️⃣ Change Table Name

هنا بنتعلم إزاي نغير اسم الـ Table في قاعدة البيانات بدون ما نغيّر اسم الـ Class.

### باستخدام Data Annotation:

```csharp
[Table("StudentsTable")]
public class Student
{
    public int Id { get; set; }
}
```

### باستخدام Fluent API:

```csharp
modelBuilder.Entity<Student>()
    .ToTable("StudentsTable");
```

🎯 الهدف: التحكم في أسماء الجداول بما يناسب تصميم قاعدة البيانات.

---

## 1️⃣2️⃣ Change Schema & Map Model To View

### 🔹 Change Schema

Schema هو تقسيم منطقي داخل قاعدة البيانات (مثل: dbo).

```csharp
modelBuilder.Entity<Student>()
    .ToTable("Students", "school");
```

### 🔹 Map Model To View

نستخدمه لما نحب نربط Model بـ View بدل Table.

```csharp
modelBuilder.Entity<Student>()
    .HasNoKey()
    .ToView("StudentsView");
```

⚠️ الـ View عادة للقراءة فقط.

---

## 1️⃣3️⃣ Exclude Properties
هنا بيشرح إزاي نستبعد Property معينة من إنها تتحفظ في قاعدة البيانات.

### باستخدام Data Annotation:

```csharp
[NotMapped]
public string FullName { get; set; }
```

### باستخدام Fluent API:

```csharp
modelBuilder.Entity<Student>()
    .Ignore(s => s.FullName);
```

🎯 نستخدمه للـ Properties المحسوبة أو المؤقتة.

---

## 1️⃣4️⃣ Change Column Name

بنستخدمه لما اسم الـ Property في الكود يختلف عن اسم العمود في قاعدة البيانات.

### Data Annotation:

```csharp
[Column("student_name")]
public string Name { get; set; }
```

### Fluent API:

```csharp
modelBuilder.Entity<Student>()
    .Property(s => s.Name)
    .HasColumnName("student_name");
```

---

## 1️⃣5️⃣ Column Data Types

هنا بيركّز على تحديد نوع البيانات للعمود في قاعدة البيانات.

### Data Annotation:

```csharp
[Column(TypeName = "varchar(100)")]
public string Email { get; set; }
```

### Fluent API:

```csharp
modelBuilder.Entity<Student>()
    .Property(s => s.Email)
    .HasColumnType("varchar(100)");
```

🎯 الهدف: التحكم في حجم ونوع البيانات وتحسين الأداء.

---


## 1️⃣6️⃣ Maximum Length

هنا بيشرح إزاي نحدد أقصى طول مسموح بيه للـ Column.

### Data Annotation:

```csharp
[MaxLength(100)]
public string Name { get; set; }
```

### Fluent API:

```csharp
modelBuilder.Entity<Student>()
    .Property(s => s.Name)
    .HasMaxLength(100);
```

🎯 الهدف: منع إدخال بيانات أطول من المطلوب وتحسين تصميم الجدول.

---

## 1️⃣7️⃣ Column Comments

نستخدم Comments لشرح العمود داخل قاعدة البيانات.

### Fluent API:

```csharp
modelBuilder.Entity<Student>()
    .Property(s => s.Email)
    .HasComment("Student email address");
```

📌 مفيد للتوثيق داخل DB خصوصًا في المشاريع الكبيرة.

---

## 1️⃣8️⃣ Set Primary Key

هنا بيشرح تحديد الـ Primary Key.

### Data Annotation:

```csharp
[Key]
public int Id { get; set; }
```

### Fluent API:

```csharp
modelBuilder.Entity<Student>()
    .HasKey(s => s.Id);
```

🎯 كل Entity لازم يكون لها Primary Key.

---

## 1️⃣9️⃣ Change Primary Key Name

نقدر نغيّر اسم الـ Constraint الخاص بالـ Primary Key.

```csharp
modelBuilder.Entity<Student>()
    .HasKey(s => s.Id)
    .HasName("PK_Students_Id");
```

📌 مفيد للالتزام بقواعد Naming في المشروع.

---

## 2️⃣0️⃣ Set Composite Key

Composite Key يعني أكثر من عمود مع بعض يكونوا Primary Key.

```csharp
modelBuilder.Entity<Enrollment>()
    .HasKey(e => new { e.StudentId, e.CourseId });
```

🎯 يُستخدم في جداول الربط (Join Tables).

---

## 2️⃣1️⃣ Set Default Value

هنا بيشرح إزاي نحدد قيمة افتراضية للعمود.

```csharp
modelBuilder.Entity<Student>()
    .Property(s => s.IsActive)
    .HasDefaultValue(true);
```

---

## 2️⃣2️⃣ Computed Columns

Computed Column هو عمود قيمته بتتحسب تلقائيًا.

```csharp
modelBuilder.Entity<Student>()
    .Property(s => s.FullName)
    .HasComputedColumnSql("FirstName + ' ' + LastName");
```

⚠️ لا يتم إدخال قيمة له يدويًا.

---

## 2️⃣3️⃣ Primary Key Default Value

نستخدمه غالبًا مع GUID.

```csharp
modelBuilder.Entity<Student>()
    .Property(s => s.Id)
    .HasDefaultValueSql("NEWID()");
```

🎯 إنشاء الـ Primary Key تلقائيًا.

---

## 2️⃣4️⃣ One To One Relationship

هنا بيشرح علاقة One-To-One بين جدولين.

### مثال:

```csharp
modelBuilder.Entity<Student>()
    .HasOne(s => s.Profile)
    .WithOne(p => p.Student)
    .HasForeignKey<StudentProfile>(p => p.StudentId);
```

📌 كل Record في جدول يقابله Record واحد فقط في الجدول الآخر.

---

## 2️⃣5️⃣ One To Many Relationship – Part 1

العلاقة One-To-Many معناها إن Record واحد من الجدول الأساسي يرتبط بعدة Records من جدول آخر.

### مثال:

* Department → يحتوي على عدة Employees

### التعريف:

```csharp
modelBuilder.Entity<Employee>()
    .HasOne(e => e.Department)
    .WithMany(d => d.Employees)
    .HasForeignKey(e => e.DepartmentId);
```

🎯 الهدف: ربط كيان رئيسي بعدة كيانات فرعية باستخدام Foreign Key.

---

## 2️⃣6️⃣ One To Many Relationship – Part 2

في الجزء ده بنكمّل:

* Navigation Properties
* Required vs Optional Relationship

### مثال:

```csharp
public class Department
{
    public int Id { get; set; }
    public ICollection<Employee> Employees { get; set; }
}
```

📌 وجود Navigation Properties بيسهّل قراءة البيانات والتنقل بينها.

---

## 2️⃣7️⃣ Many To Many Relationship

العلاقة Many-To-Many معناها:

* كل Record في الجدول الأول يرتبط بعدة Records من الجدول الثاني والعكس.

### مثال مباشر (EF Core ):

```csharp
modelBuilder.Entity<Student>()
    .HasMany(s => s.Courses)
    .WithMany(c => c.Students);
```

🎯 EF Core بيعمل جدول ربط تلقائي.

---

## 2️⃣8️⃣ Indirect Many To Many Relationship

نستخدمها لما نحتاج نضيف بيانات إضافية في جدول الربط.

### مثال:

```csharp
public class Enrollment
{
    public int StudentId { get; set; }
    public int CourseId { get; set; }
    public DateTime EnrollDate { get; set; }
}
```

📌 هنا جدول الربط بقى Entity مستقل.

---

## 2️⃣9️⃣ Indexes

الـ Index بيساعد على تسريع عمليات البحث.

### Fluent API:

```csharp
modelBuilder.Entity<Student>()
    .HasIndex(s => s.Email);
```

🎯 تحسين الأداء خصوصًا مع الجداول الكبيرة.

---

## 3️⃣0️⃣ Composite Index

Composite Index يعني Index على أكثر من Column.

```csharp
modelBuilder.Entity<Student>()
    .HasIndex(s => new { s.Email, s.Name });
```

📌 مفيد في Queries اللي بتستخدم أكتر من شرط.

---

## 3️⃣1️⃣ Index Uniqueness

هنا بيشرح إزاي نخلي الـ Index **Unique** بحيث يمنع تكرار القيم.

```csharp
modelBuilder.Entity<Student>()
    .HasIndex(s => s.Email)
    .IsUnique();
```

🎯 الهدف: منع إدخال بيانات مكررة (مثل الإيميل أو رقم الهوية).

---

## 3️⃣2️⃣ Index Name

نقدر نحدد اسم مخصص للـ Index بدل الاسم الافتراضي.

```csharp
modelBuilder.Entity<Student>()
    .HasIndex(s => s.Email)
    .HasDatabaseName("IX_Student_Email");
```

📌 مفيد للالتزام بمعايير التسمية داخل قاعدة البيانات.

---

## 3️⃣3️⃣ Index Filter

Index Filter بيخلي الـ Index يتطبق على جزء من البيانات فقط.

```csharp
modelBuilder.Entity<Student>()
    .HasIndex(s => s.Email)
    .HasFilter("Email IS NOT NULL");
```

🎯 تحسين الأداء وتقليل حجم الـ Index.

---

## 3️⃣4️⃣ Sequences

Sequence هو كائن في قاعدة البيانات بيولّد أرقام متسلسلة.

```csharp
modelBuilder.HasSequence<int>("StudentSeq")
    .StartsAt(1)
    .IncrementsBy(1);
```

📌 يُستخدم كبديل أو مع الـ Identity.

---

## 3️⃣5️⃣ Data Seeding

Data Seeding يعني إدخال بيانات افتراضية مع إنشاء قاعدة البيانات.

```csharp
modelBuilder.Entity<Department>().HasData(
    new Department { Id = 1, Name = "IT" },
    new Department { Id = 2, Name = "HR" }
);
```

🎯 مفيد للبيانات الأساسية.

---

## 3️⃣6️⃣ Manage Migration and Generate SQL Scripts

هنا بيشرح:

* إدارة الـ Migrations
* توليد SQL Script من Migration

```powershell
Script-Migration
```

📌 مفيد للنشر على Production.

---

## 3️⃣7️⃣ Working With an Existing Database (Database Scaffolding)

Scaffolding يعني توليد Models و DbContext من قاعدة بيانات موجودة.

```powershell
Scaffold-DbContext "ConnectionString" Microsoft.EntityFrameworkCore.SqlServer
```

🎯 مناسب للمشاريع القديمة.

---

## 3️⃣8️⃣ Select All Data, Select One Item Using .Find

```csharp
var students = context.Students.ToList();
var student = context.Students.Find(1);
```

📌 `Find` بيبحث باستخدام Primary Key.

---

## 3️⃣9️⃣ Select One Item Using .Single

```csharp
var student = context.Students.Single(s => s.Id == 1);
```

⚠️ بيرمي Exception لو مفيش أو في أكتر من Record.

---

## 4️⃣0️⃣ Select One Item Using .First

```csharp
var student = context.Students.First(s => s.Age > 20);
```

📌 بيرجع أول عنصر مطابق.

---

## 4️⃣1️⃣ Select One Item Using .Last

```csharp
var student = context.Students.OrderBy(s => s.Id).Last();
```

⚠️ لازم ترتيب قبل الاستخدام.

---

## 4️⃣2️⃣ Filtering Data Using .Where

```csharp
var students = context.Students.Where(s => s.Age > 20).ToList();
```

🎯 تصفية البيانات حسب شرط.

---

## 4️⃣3️⃣ .Any vs .All

```csharp
context.Students.Any(s => s.Age > 30);
context.Students.All(s => s.Age > 18);
```

* `Any` → هل يوجد عنصر واحد على الأقل؟
* `All` → هل كل العناصر تحقق الشرط؟

---

## 4️⃣4️⃣ .Append vs .Prepend

```csharp
list.Append(newItem);
list.Prepend(newItem);
```


* `Append` → إضافة في النهاية
* `Prepend` → إضافة في البداية

---

## 4️⃣5️⃣ .Average vs .Count vs .Sum

```csharp
context.Students.Average(s => s.Age);
context.Students.Count();
context.Students.Sum(s => s.Age);
```

🎯 عمليات تجميع البيانات.

---

## 46️⃣ .Max vs .Min
- `.Max()` يستخدم للحصول على أكبر قيمة من مجموعة بيانات.
- `.Min()` يستخدم للحصول على أصغر قيمة من مجموعة بيانات.

```csharp
var maxAge = context.Students.Max(s => s.Age);
var minAge = context.Students.Min(s => s.Age);
```
---

## 47️⃣ Data Sorting Using .OrderBy
- `.OrderBy()` لترتيب البيانات تصاعديًا.
- `.OrderByDescending()` لترتيب البيانات تنازليًا.

```csharp
var studentsAsc = context.Students.OrderBy(s => s.Name).ToList();
var studentsDesc = context.Students.OrderByDescending(s => s.Age).ToList();
```
---

## 48️⃣ Projection Using .Select
- `.Select()` لاختيار أعمدة معينة أو عمل Projection.

```csharp
var studentNames = context.Students.Select(s => s.Name).ToList();
```

---

## 49️⃣ Select Unique Values Using .Distinct
- `.Distinct()` للحصول على القيم الفريدة فقط.

```csharp
var uniqueAges = context.Students.Select(s => s.Age).Distinct().ToList();
```

---

## 50️⃣ .Take vs .Skip
- `.Take(n)` للحصول على أول n عناصر.
- `.Skip(n)` لتجاوز أول n عناصر.

```csharp
var first5 = context.Students.Take(5).ToList();
var skip2 = context.Students.Skip(2).ToList();
```

---

## 51️⃣ .GroupBy
- `.GroupBy()` لتجميع البيانات حسب عمود معين.

```csharp
var grouped = context.Students
    .GroupBy(s => s.Age)
    .Select(g => new { Age = g.Key, Count = g.Count() })
    .ToList();
```

---

## 52️⃣ Inner Join Using .Join (Extension Method)
- عمل Inner Join بين جدولين.

```csharp
var result = context.Students
    .Join(context.Courses,
          s => s.Id,
          c => c.StudentId,
          (s, c) => new { s.Name, c.CourseName })
    .ToList();
```

---

## 53️⃣ Left Join Using .GroupJoin (Extension Method)
- عمل Left Join باستخدام GroupJoin.

```csharp
var result = context.Students
    .GroupJoin(context.Courses,
               s => s.Id,
               c => c.StudentId,
               (s, courses) => new { s.Name, Courses = courses.DefaultIfEmpty() })
    .ToList();
```

---

## 54️⃣ Tracking vs. NoTracking
- Tracking: تتبع التغييرات تلقائيًا.
- NoTracking: لا يتتبع، أسرع للقراءة فقط.

```csharp
var tracked = context.Students.ToList();
var notTracked = context.Students.AsNoTracking().ToList();
```
---

## 55️⃣ Eager Loading
- تحميل العلاقات مباشرة عند الاستعلام.

```csharp
var students = context.Students.Include(s => s.Courses).ToList();
```

---

## 56️⃣ Explicit Loading
- تحميل العلاقات عند الحاجة صراحة.

```csharp
var student = context.Students.First();
context.Entry(student).Collection(s => s.Courses).Load();
```
---

## 57️⃣ Lazy Loading
- تحميل العلاقات عند الوصول إليها فقط.

```csharp
var courses = student.Courses;
```
---

## 58️⃣ Split Queries
- تقسيم الاستعلامات لتقليل مشاكل الأداء مع Include كبير.

```csharp
var students = context.Students.Include(s => s.Courses).AsSplitQuery().ToList();
```

---

## 59️⃣ Join Using LINQ
- استخدام LINQ Query Syntax لعمل Join.

```csharp
var result = from s in context.Students
             join c in context.Courses on s.Id equals c.StudentId
             select new { s.Name, c.CourseName };
```
---

## 60️⃣ Select Data Using SQL Statement or Stored Procedure - Part 1
- تنفيذ SQL Statement مباشر.

```csharp
var students = context.Students.FromSqlRaw("SELECT * FROM Students").ToList();
```
---

## 61️⃣ Select Data Using SQL Statement or Stored Procedure - Part 2
- تنفيذ Stored Procedure.

```csharp
var students = context.Students.FromSqlRaw("EXEC GetAllStudents").ToList();
```

---

## 62️⃣ Global Query Filters
- فلترة بيانات عامة لكل الاستعلامات على Entity.

```csharp
modelBuilder.Entity<Student>().HasQueryFilter(s => s.IsActive);
```

---

## 63️⃣ Add New Record(s) and Save Related Data
- إضافة Record مرتبط بكيانات أخرى.

```csharp
var student = new Student { Name = "Ali", Courses = new List<Course> { new Course { Name = "Math" } } };
context.Students.Add(student);
context.SaveChanges();
```
---

## 64️⃣ Update Record(s)
- تعديل Record موجود.

```csharp
var student = context.Students.Find(1);
student.Name = "Ahmed";
context.SaveChanges();
```
---

## 65️⃣ Remove Record(s)
- حذف Record محدد.

```csharp
var student = context.Students.Find(1);
context.Students.Remove(student);
context.SaveChanges();
```
---

## 66️⃣ Delete Related Data
- حذف بيانات مرتبطة بعلاقات.

```csharp
var student = context.Students.Include(s => s.Courses).First();
context.Students.Remove(student);
context.SaveChanges();
```
---

## 67️⃣ Transactions
- استخدام Transaction لحماية البيانات.

```csharp
using var transaction = context.Database.BeginTransaction();
try
{
    context.Students.Add(new Student { Name = "Test" });
    context.SaveChanges();
    transaction.Commit();
}
catch
{
    transaction.Rollback();
}
```
---

## 68️⃣ Save Data with SQL Statement and Stored Procedures (ExecuteSqlRaw)
- تنفيذ SQL أو Stored Procedure لتحديث البيانات.

```csharp
context.Database.ExecuteSqlRaw("UPDATE Students SET Name = 'Test' WHERE Id = 1");
```
---

## 69️⃣ MaxBy vs MinBy (EF Core 7+)
- الحصول على العنصر بأكبر أو أصغر قيمة لخاصية معينة.

```csharp
var oldestStudent = context.Students.MaxBy(s => s.Age);
var youngestStudent = context.Students.MinBy(s => s.Age);
```
---

## 70️⃣ EF7 Updates - Bulk Delete/Bulk Update
- حذف أو تحديث عدة Records دفعة واحدة.

```csharp
context.Students.Where(s => s.Age < 18).ExecuteDelete();
context.Students.Where(s => s.IsActive).ExecuteUpdate(s => s.SetProperty(st => st.IsActive, false));
```
---
