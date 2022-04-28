---
title: EF Core Fluent API Entity Configuration
author: Jandari
date: 2022-04-28 22:20:00 -0500
categories: [.NET, EF Core]
tags: [.NET, C#, EF Core]
math: true
mermaid: true
---

## 소개

이번 포스트는 EF Core에서 Fluent API 패턴을 기반으로 한 Entity Configuration에 대해 정리하도록 하겠습니다.

## Entity Configuration

Entity Configuration은 테이블 및 관계 매핑에 대해 구성합니다.

PrimaryKey(기본 키), AlternateKey(대체 키), Index, Table Name, 1대1 관계, 1대N 관계, N대N 관계에 대해 정의 합니다.

|Methods| 사용법|
|-|-|
| HasAlternateKey()	| 엔티티에 대한 EF 모델의 대체 키 구성. |
| HasIndex()	    | 지정 된 속성의 인덱스 구성 |
| HasKey()	        | 속성 또는 속성 목록을 기본 키로 구성 |
| HasMany()	        | 일대다 또는 다대다 관계에 대해 다른 유형의 참조 수집 속성을 포함하는 관계에서 여러 부분을 구성한다. |
| HasOne()	        | 일대일 관계 또는 일대다 관계에 대한 다른 유형의 참조 속성을 포함하는 관계 중 하나 부분 구성. |
| Ignore()	        | 클래스 또는 속성을 테이블 또는 열에 매핑하지 않도록 구성. |
| OwnsOne()	        | 이 엔티티가 대상 엔티티를 소유하는 관계 구성대상 엔티티 키 값은 해당 엔티티가 속한 엔티티에서 전파된다. |
| ToTable()	        | 엔티티가 매핑할 데이터베이스 테이블 구성 |

## [HasAlternateKey()](https://www.tektutorialshub.com/entity-framework-core/ef-core-fluent-api-hasalternatekey-method/)

EF Core의 핵심 API 중 하나인 `HasAlternateKey` 메서드는 기본 키를 제외하고 제약 조건을 구성하여 `대체 키`를 생성할 수 있도록 합니다.

쉽게 말해 기본 키를 제외하고 나머지 컬럼을 조합하여 키를 생성하는 방법입니다. 이는 일반적으로 데이터의 고유성을 보장하기 위해 수행합니다.

아래와 같이 Employee Entity의 경우 EmployeeId가 기본 키로 매핑이 됩니다.

```cs
public class Employee
{
    public int EmployeeID { get; set; }
    public int BranchCode { get; set; }
    public int EmployeeCode { get; set; }
    public string Name { get; set; }
    public DateTime DOB { get; set; }
    public int Age { get; set; }
}
```

여기서 EmployeeCode를 사용하여 대체키를 구성하기 위해서는 아래와 같이 사용해야 합니다.

```cs
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
  modelBuilder.Entity<Employee>()
             .HasAlternateKey(e=> e.EmployeeCode);
}
```

대체 키의 이름은 `AK_<엔티티이름>_<속성이름>`으로 생성 됩니다.

여러 속성을 조합하여 대체 키를 생성하고 싶다면 아래와 같이 `익명 객체(new {})`를 사용해서 생성하면 됩니다.

```cs
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
  modelBuilder.Entity<Employee>()
        .HasAlternateKey(e => new { e.BranchCode, e.EmployeeCode });
}
```

복합 키의 경우 `AK_<엔티티이름>_<속성이름1>_<속성이름2>`로 생성됩니다.

## [HasIndex()](https://www.learnentityframeworkcore.com/configuration/fluent-api/hasindex-method)

`HasIndex` 메서드는 지정된 엔티티 속성에 매핑된 열에 데이터베이스 인덱스를 만드는데 사용합니다.

기본적으로 인덱스는 외래 키와 대체 키로 생성되는데 데이터 검색 속도를 높이기 위해 특정 속성을 사용 할 수 있습니다.

```cs
public class SampleContext : DbContext
{
    public DbSet<Book> Books { get; set; }
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Book>()
            .HasIndex(b => b.Isbn);
    }
}
public class Book
{
    public int BookId { get; set; }
    public string Title { get; set; }
    public string Isbn { get; set; }
}
```

위와 같이 생성했을 때 `Isbn`는 중복되는 경우가 발생 할 수 있습니다.

이럴 때 `.IsUnique` 메서드를 사용하여 고유한 값을 가질 수 있도록 설정합니다.

```cs
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Book>()
        .HasIndex(b => b.Isbn)
        .IsUnique();
}
```

또한 `HasName` 메서드로 인덱스에 이름을 지정 할 수 있습니다.

```cs
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Book>()
        .HasIndex(b => b.Isbn)
        .IsUnique();
        .HasName("MyIndexName");
}
```

### 복합 인덱스

둘 이상의 속성으로 인덱스를 사용 하려면 `익명 객체(new{})`를 사용하여 지정 합니다.

```cs
public class SampleContext : DbContext
{
    public DbSet<Patient> Patients { get; set; }
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Patient>()
            .HasIndex(p => new { p.Ssn, p.DateOfBirth});
    }
}
public class Patient
{
    public int PatientId { get; set; }
    public string Ssn { get; set; }
    public DateTime DateOfBirth { get; set; }
}
```

## [HasKey()](https://www.learnentityframeworkcore.com/configuration/fluent-api/haskey-method)

`HasKey` 메서드는 엔티티의 고유 식별키(EntityKey)하고 데이터베이스의 기본키 필드와 매핑하는데 사용됩니다.

```cs
public class SampleContext : DbContext
{
    public DbSet<Order> Orders { get; set; }
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Order>()
            .HasKey(o => o.OrderNumber);
    }
}
public class Order
{
    public int OrderNumber { get; set; }
    public DateTime DateCreated { get; set; }
    public Customer Customer { get; set; }
    ...
)
```

위에서 `Order` 엔티티를 보면 `OrderId`가 아닌 `OrderNumber`이다. 이 경우 EF Core의 네이밍 룰에 맞지 않아 정상적으로 키를 인식하지 못합니다.

이럴 경우 `HasKey`를 사용하여 지정합니다.

### 복합키

두 개 이상의 필드를 사용하여 복합 키를 만들 경우 `HasKey`를 사용합니다. 복합키의 경우 Annotation으로는 지정을 하지 못하기 때문에 FluentAPI에서 HasKey를 사용하는 방법 밖에 없습니다.

```cs
public class SampleContext : DbContext
{
    public DbSet<Order> Orders { get; set; }
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Order>()
            .HasKey(o => new { o.CustomerAbbreviation, o.OrderNumber });
    }
}
public class Order
{
    public string CustomerAbbreviation { get; set; }
    public int OrderNumber { get; set; }
    public DateTime DateCreated { get; set; }
    public Customer Customer { get; set; }
    ...
)
```


## [HasMany()](https://www.learnentityframeworkcore.com/configuration/fluent-api/hasmany-method)

`HasMany` 메서드는 1대N 관계에서 N의 측면에서 구성할 때 사용됩니다.

여기서 관계를 구성하기 위해서는 `Has/With` 패턴을 사용하여 유효한 관계를 구성해야 합니다.

```cs
public class Company
{
    public int Id { get; set; }
    public string Name { get; set; }
    public ICollection<Employee> Employees { get; set; }
}
public class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }
    public Company Company { get; set; }
}
```

`Employee` 엔티티에서 Company 속성의 경우 역 탐색을 위해 정의한 속성입니다.

회사에는 여러 직원이 있고 직원 개개인은 한 회사에 다닌다는 관계는 아래와 같이 표현 할 수 있습니다.

```cs
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Company>()
        .HasMany(c => c.Employees)
        .WithOne(e => e.Company);
}
```


## [HasOne()](https://www.learnentityframeworkcore.com/configuration/fluent-api/hasone-method)

`HasOne` 메서드는 1대N 관계에서 1 또는 1대1 관계에서 한쪽 부분을 나타내는데 사용합니다.

1:1관계인지 1:N 관계인지에 따라 `Has/With` 패턴에 따라 `HasOne`과 `WithOne`, `WithMany`로 구성해야 합니다.

### 1:1 관계

아래는 `HasOne`과 `WithOne`을 사용하여 1:1 관계를 표현한 예제입니다.

```cs
public class SampleContext : DbContext
{
    public DbSet<Author> Authors { get; set; }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Author>()
            .HasOne(a => a.Biography)
            .WithOne(b => b.Author);
    }
}
public class Author
{
    public int AuthorId { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public AuthorBiography Biography { get; set; }
}
public class AuthorBiography
{
    public int AuthorBiographyId { get; set; }
    public string Biography { get; set; }
    public int AuthorId { get; set; }
    public Author Author { get; set; }
}
```

### 1:N 관계

아래는 `HasOne`과 `WithMany`를 사용하여 1:N 관계를 표현 한 것입니다.

```cs
public class SampleContext : DbContext
{
    public DbSet<Employee> Employees { get; set; }
    public DbSet<Company> Companies { get; set; }
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Employee>()
        .HasOne(e => e.Company)
        .WithMany(c => c.Employees);
    }
public class Company
{
    public int Id { get; set; }
    public string Name { get; set; }
    public ICollection<Employee> Employees { get; set; }
}
public class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }
    public Company Company { get; set; }
}
```

## [Ignore ()](https://www.learnentityframeworkcore.com/configuration/fluent-api/ignore-method)

`Ignore` 메서드는 데이터베이스에 매핑되는 속성 또는 엔티티(테이블)을 무시하는데 사용되며, EF Core에서는 두가지 방법으로 사용 가능합니다. `ModelBuilder`를 사용하면 테이블 자체를 매핑되지 않도록 제외 할 수 있고, `EntityTypeBuilder`를 사용하면 개별 속성을 제외 할 수 있습니다.

### 엔티티 제외

아래 예제는 `AuditLog` 엔티티는 테이블 매핑에 제외 됩니다.

```cs
public class SampleContext : DbContext
{
    public DbSet<Contact> Contacts { get; set; }
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Ignore<AuditLog>();
    }
}
public class Contact
{
    public int ContactId { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Email { get; set; } 
    public AuditLog AuditLog { get; set; }
}
public class AuditLog
{
    public int EntityId { get; set; }
    public int UserId { get; set; }
    public DateTime Modified { get; set; }
}
```

### 속성 제외

아래의 예제는 `Contact` 엔티티에서 `FullName`이 매핑에서 제외 됩니다.

```cs
public class SampleContext : DbContext
{
    public DbSet<Contact> Contacts { get; set; }
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Contact>().Ignore(c => c.FullName);
    }
}
public class Contact
{
    public int ContactId { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string FullName => $"{FirstName} {LastName}";
    public string Email { get; set; } 
}
```

## [ToTable()](https://www.learnentityframeworkcore.com/configuration/fluent-api/totable-method)

`ToTable` 메서드는 매핑되어야 하는 데이터베이스 테이블의 이름을 지정하기 위해 엔티티에 적용됩니다.

아래의 예제는 `Book` 엔티티가 `tbl_Book` 테이블에 매핑되어야 함을 지정합니다.

```cs
public class SampleContext : DbContext
{
    public DbSet<Book> Books { get; set; }
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Book>()
            .ToTable("tbl_Book`");
    }
}
public class Book
{
    public int BookId { get; set; }
    public string Title { get; set; }
    public Author Author { get; set; }
}
```

아래와 같이 스키마까지 지정이 가능합니다.

```cs
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Book>().ToTable("tbl_Book", "library");
}
```