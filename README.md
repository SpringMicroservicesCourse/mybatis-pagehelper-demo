# mybatis-pagehelper-demo

> MyBatis PageHelper pagination with RowBounds and parameter methods

[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.4.5-brightgreen.svg)](https://spring.io/projects/spring-boot)
[![Java](https://img.shields.io/badge/Java-21-orange.svg)](https://openjdk.org/)
[![MyBatis](https://img.shields.io/badge/MyBatis-3.0.5-red.svg)](https://mybatis.org/mybatis-3/)
[![PageHelper](https://img.shields.io/badge/PageHelper-2.1.1-blue.svg)](https://pagehelper.github.io/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A comprehensive demonstration of MyBatis PageHelper featuring two pagination methods (RowBounds and parameters), PageInfo wrapper, and advanced configuration options.

## Features

- MyBatis PageHelper integration with Spring Boot Starter
- RowBounds pagination method (MyBatis native style)
- Parameter pagination method (RESTful API friendly)
- PageInfo wrapper for complete pagination information
- Advanced PageHelper configurations
- Custom TypeHandler for Joda Money
- H2 in-memory database with automatic data initialization
- Demonstration of pageSize=0 behavior

## Tech Stack

- Spring Boot 3.4.5
- MyBatis Spring Boot Starter 3.0.5
- PageHelper Spring Boot Starter 2.1.1
- Java 21
- H2 Database 2.3.232
- Joda Money 2.0.2
- Lombok
- Maven 3.8+

## Getting Started

### Prerequisites

- JDK 21 or higher
- Maven 3.8+ (or use included Maven Wrapper)

### Quick Start

**Step 1: Clone the repository**

```bash
git clone https://github.com/SpringMicroservicesCourse/mybatis-pagehelper-demo
cd mybatis-pagehelper-demo
```

**Step 2: Run the application**

```bash
./mvnw spring-boot:run
```

## Pagination Methods

### Method 1: RowBounds Pagination

**Mapper Definition:**

```java
@Mapper
public interface CoffeeMapper {
    @Select("select * from t_coffee order by id")
    List<Coffee> findAllWithRowBounds(RowBounds rowBounds);
}
```

**Usage:**

```java
// Page 1, 3 records per page
RowBounds rowBounds = new RowBounds(1, 3);
List<Coffee> page1 = coffeeMapper.findAllWithRowBounds(rowBounds);

// Page 2, 3 records per page
rowBounds = new RowBounds(2, 3);
List<Coffee> page2 = coffeeMapper.findAllWithRowBounds(rowBounds);
```

**Advantages:**
- ✅ MyBatis native API style
- ✅ Perfect integration with MyBatis Generator
- ✅ No @Param annotation needed
- ✅ Clean method signature

**Sample Output:**

```
Page(1) Coffee Coffee(id=1, name=espresso, price=TWD 100.00, ...)
Page(1) Coffee Coffee(id=2, name=latte, price=TWD 125.00, ...)
Page(1) Coffee Coffee(id=3, name=capuccino, price=TWD 125.00, ...)

Page(2) Coffee Coffee(id=4, name=mocha, price=TWD 150.00, ...)
Page(2) Coffee Coffee(id=5, name=macchiato, price=TWD 150.00, ...)
```

### Method 2: Parameter Pagination

**Mapper Definition:**

```java
@Mapper
public interface CoffeeMapper {
    @Select("select * from t_coffee order by id")
    List<Coffee> findAllWithParam(@Param("pageNum") int pageNum,
                                  @Param("pageSize") int pageSize);
}
```

**Usage:**

```java
// Page 1, 3 records per page
List<Coffee> page1 = coffeeMapper.findAllWithParam(1, 3);

// Page 2, 3 records per page - Get PageInfo
List<Coffee> page2 = coffeeMapper.findAllWithParam(2, 3);
PageInfo<Coffee> pageInfo = new PageInfo<>(page2);
```

**Advantages:**
- ✅ Explicit method signature
- ✅ RESTful API friendly
- ✅ No RowBounds object creation needed
- ✅ Better for Controller layer

**Sample Output:**

```
Page(1) Coffee Coffee(id=1, name=espresso, price=TWD 100.00, ...)
Page(1) Coffee Coffee(id=2, name=latte, price=TWD 125.00, ...)
Page(1) Coffee Coffee(id=3, name=capuccino, price=TWD 125.00, ...)
```

### PageInfo Details

**PageInfo Wrapper:**

```java
List<Coffee> list = coffeeMapper.findAllWithParam(2, 3);
PageInfo<Coffee> pageInfo = new PageInfo<>(list);

System.out.println("Current Page: " + pageInfo.getPageNum());      // 2
System.out.println("Page Size: " + pageInfo.getPageSize());        // 3
System.out.println("Total Records: " + pageInfo.getTotal());       // 5
System.out.println("Total Pages: " + pageInfo.getPages());         // 2
System.out.println("Has Next: " + pageInfo.isHasNextPage());       // false
```

**Complete PageInfo Output:**

```
PageInfo{
  pageNum=2,                    # Current page number
  pageSize=3,                   # Records per page
  size=2,                       # Actual records on current page
  startRow=4,                   # Start row number
  endRow=5,                     # End row number
  total=5,                      # Total records
  pages=2,                      # Total pages
  list=[...],                   # Data list
  prePage=1,                    # Previous page number
  nextPage=0,                   # Next page number (0 = none)
  isFirstPage=false,            # Is first page
  isLastPage=true,              # Is last page
  hasPreviousPage=true,         # Has previous page
  hasNextPage=false,            # Has next page
  navigatePages=8,              # Navigation page count
  navigateFirstPage=1,          # First navigation page
  navigateLastPage=2,           # Last navigation page
  navigatepageNums=[1, 2]       # Navigation page numbers
}
```

## Configuration

### Application Properties

```properties
# MyBatis configuration
mybatis.type-handlers-package=tw.fengqing.spring.data.mybatisdemo.handler
mybatis.configuration.map-underscore-to-camel-case=true

# PageHelper configuration
# Use RowBounds.offset as pageNum (not offset)
pagehelper.offset-as-page-num=true

# Rationalize pagination parameters
# pageNum < 1 → query first page
# pageNum > total pages → query last page
pagehelper.reasonable=true

# Allow pageSize=0 to return all results
pagehelper.page-size-zero=true

# Support pagination via method parameters
pagehelper.support-methods-arguments=true
```

**Configuration Comparison:**

| Configuration | Description | Recommended | Effect |
|---------------|-------------|-------------|--------|
| `offset-as-page-num` | RowBounds.offset semantics | `true` | offset as page number (not offset) |
| `reasonable` | Rationalize page parameters | `true` | Auto-adjust out-of-range pages |
| `page-size-zero` | pageSize=0 behavior | `true` | Return all records |
| `support-methods-arguments` | Method parameter pagination | `true` | Auto-detect pageNum/pageSize |

### Database Schema

**schema.sql:**

```sql
create table t_coffee (
    id bigint not null auto_increment,
    name varchar(255),
    price bigint not null,
    create_time timestamp,
    update_time timestamp,
    primary key (id)
);
```

**data.sql:**

```sql
insert into t_coffee (name, price, create_time, update_time) 
  values ('espresso', 10000, now(), now());
insert into t_coffee (name, price, create_time, update_time) 
  values ('latte', 12500, now(), now());
insert into t_coffee (name, price, create_time, update_time) 
  values ('capuccino', 12500, now(), now());
insert into t_coffee (name, price, create_time, update_time) 
  values ('mocha', 15000, now(), now());
insert into t_coffee (name, price, create_time, update_time) 
  values ('macchiato', 15000, now(), now());
```

**Note:** Price stored in cents (10000 = TWD 100.00)

## Usage

### Application Flow

```
1. Spring Boot starts
   ↓
2. Auto-executes schema.sql & data.sql
   ↓
3. ApplicationRunner executes 4 demonstrations:
   - Demo 1: RowBounds pagination (page 1 & 2)
   - Demo 2: pageSize=0 behavior (returns all)
   - Demo 3: Parameter pagination (page 1)
   - Demo 4: PageInfo usage (page 2)
```

### Code Example

```java
@SpringBootApplication
@MapperScan("tw.fengqing.spring.data.mybatisdemo.mapper")
public class MybatisDemoApplication implements ApplicationRunner {
    
    @Autowired
    private CoffeeMapper coffeeMapper;
    
    @Override
    public void run(ApplicationArguments args) throws Exception {
        // Demo 1: RowBounds pagination
        coffeeMapper.findAllWithRowBounds(new RowBounds(1, 3))
                .forEach(c -> log.info("Page(1) Coffee {}", c));
        coffeeMapper.findAllWithRowBounds(new RowBounds(2, 3))
                .forEach(c -> log.info("Page(2) Coffee {}", c));
        
        // Demo 2: pageSize=0 returns all
        coffeeMapper.findAllWithRowBounds(new RowBounds(1, 0))
                .forEach(c -> log.info("Page(1) Coffee {}", c));
        
        // Demo 3: Parameter pagination
        coffeeMapper.findAllWithParam(1, 3)
                .forEach(c -> log.info("Page(1) Coffee {}", c));
        
        // Demo 4: PageInfo usage
        List<Coffee> list = coffeeMapper.findAllWithParam(2, 3);
        PageInfo<Coffee> page = new PageInfo<>(list);
        log.info("PageInfo: {}", page);
    }
}
```

## Best Practices

### 1. Method Selection Guide

**Use RowBounds When:**
- Working with MyBatis Generator
- Internal service calls
- Prefer MyBatis native API style
- Don't need explicit pagination parameters in signature

**Use Parameters When:**
- Building RESTful APIs
- Controller layer needs explicit pagination
- Want clear method signatures
- Working with Spring MVC

### 2. Web API Integration

```java
@RestController
@RequestMapping("/api/coffees")
public class CoffeeController {
    
    @Autowired
    private CoffeeMapper coffeeMapper;
    
    @GetMapping
    public ResponseEntity<PageInfo<Coffee>> getCoffees(
            @RequestParam(defaultValue = "1") int pageNum,
            @RequestParam(defaultValue = "10") int pageSize) {
        
        // Parameter validation
        if (pageNum < 1) pageNum = 1;
        if (pageSize < 1 || pageSize > 100) pageSize = 10;
        
        // Execute pagination query
        List<Coffee> list = coffeeMapper.findAllWithParam(pageNum, pageSize);
        PageInfo<Coffee> pageInfo = new PageInfo<>(list);
        
        return ResponseEntity.ok(pageInfo);
    }
}
```

**API Usage:**

```bash
# Query page 1, 10 records per page
curl http://localhost:8080/api/coffees?pageNum=1&pageSize=10

# Query page 2, 20 records per page
curl http://localhost:8080/api/coffees?pageNum=2&pageSize=20
```

### 3. Configuration Recommendations

**Development Environment:**

```properties
pagehelper.offset-as-page-num=true
pagehelper.reasonable=true
pagehelper.page-size-zero=true
pagehelper.support-methods-arguments=true

# Enable SQL logging for debugging
logging.level.tw.fengqing.spring.data.mybatisdemo.mapper=DEBUG
```

**Production Environment:**

```properties
pagehelper.offset-as-page-num=true
pagehelper.reasonable=true
pagehelper.page-size-zero=false    # Security: don't allow query all
pagehelper.support-methods-arguments=true

# Disable detailed SQL logging
logging.level.tw.fengqing.spring.data.mybatisdemo.mapper=WARN
```

### 4. Performance Optimization

**Avoid COUNT Query:**

```java
// Don't execute COUNT query if not needed
PageHelper.startPage(1, 10, false);  // false = skip COUNT
List<Coffee> coffees = coffeeMapper.findAll();
```

**Add Indexes:**

```sql
-- Index for common sort columns
CREATE INDEX idx_coffee_id ON t_coffee(id);
CREATE INDEX idx_coffee_name ON t_coffee(name);
CREATE INDEX idx_coffee_price ON t_coffee(price);
```

**Limit Max pageSize:**

```java
// Controller layer limit
if (pageSize > 100) {
    pageSize = 100;  // Maximum 100 records per page
}
```

## Testing

```bash
# Run tests
./mvnw test

# Run application
./mvnw spring-boot:run
```

## Key Components

### MoneyTypeHandler

```java
/**
 * TypeHandler for Money ↔ Long conversion (TWD currency)
 * Database stores as cents, application uses Money object
 */
public class MoneyTypeHandler extends BaseTypeHandler<Money> {
    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, 
                                     Money parameter, JdbcType jdbcType) 
                                     throws SQLException {
        ps.setLong(i, parameter.getAmountMinorLong());
    }
    
    private Money parseMoney(Long value) {
        return Money.of(CurrencyUnit.of("TWD"), value / 100.0);
    }
}
```

### CoffeeMapper

```java
@Mapper
public interface CoffeeMapper {
    // RowBounds method
    @Select("select * from t_coffee order by id")
    List<Coffee> findAllWithRowBounds(RowBounds rowBounds);

    // Parameter method
    @Select("select * from t_coffee order by id")
    List<Coffee> findAllWithParam(@Param("pageNum") int pageNum,
                                  @Param("pageSize") int pageSize);
}
```

**Important:** 
- SQL doesn't include LIMIT/OFFSET
- PageHelper automatically adds pagination syntax
- Supports multiple databases (MySQL, PostgreSQL, Oracle, H2, etc.)

## Common Issues

### Issue 1: Pagination Not Working

**Problem:** Returns all records instead of paginated results

**Causes:**
- `support-methods-arguments=false`
- Parameter names not `pageNum` and `pageSize`

**Solution:**

```properties
# Ensure parameter support is enabled
pagehelper.support-methods-arguments=true
```

Or use explicit pagination:

```java
PageHelper.startPage(pageNum, pageSize);
List<Coffee> coffees = coffeeMapper.findAll();
```

### Issue 2: Multiple Query Pagination

**Problem:** Subsequent queries also paginated after `PageHelper.startPage()`

**Cause:** PageHelper uses ThreadLocal to store pagination parameters

**Solution:**

```java
try {
    PageHelper.startPage(1, 10);
    List<Coffee> coffees = coffeeMapper.findAll();
    // Process results
} finally {
    PageHelper.clearPage();  // Clear pagination parameters
}
```

Or use parameter method (auto-clears):

```java
List<Coffee> coffees = coffeeMapper.findAllWithParam(1, 10);
```

## Method Comparison

| Method | SQL Execution | COUNT Query | Performance | Use Case |
|--------|--------------|-------------|-------------|----------|
| **RowBounds** | Auto LIMIT | Auto COUNT | Normal | MyBatis Generator |
| **Parameters** | Auto LIMIT | Auto COUNT | Normal | Custom Mapper, Web API |

**Note:** PageHelper executes 2 SQL queries:
1. COUNT query for total records
2. SELECT query with LIMIT for data

## References

- [MyBatis PageHelper Documentation](https://pagehelper.github.io/)
- [PageHelper GitHub](https://github.com/pagehelper/Mybatis-PageHelper)
- [PageHelper Spring Boot Starter](https://github.com/pagehelper/pagehelper-spring-boot)
- [MyBatis Documentation](https://mybatis.org/mybatis-3/)
- [Joda Money Documentation](https://www.joda.org/joda-money/)

## License

MIT License - see [LICENSE](LICENSE) file for details.

## About Us

我們主要專注在敏捷專案管理、物聯網（IoT）應用開發和領域驅動設計（DDD）。喜歡把先進技術和實務經驗結合，打造好用又靈活的軟體解決方案。近來也積極結合 AI 技術，推動自動化工作流，讓開發與運維更有效率、更智慧。持續學習與分享，希望能一起推動軟體開發的創新和進步。

## Contact

**風清雲談** - 專注於敏捷專案管理、物聯網（IoT）應用開發和領域驅動設計（DDD）。

- 🌐 官方網站：[風清雲談部落格](https://blog.fengqing.tw/)
- 📘 Facebook：[風清雲談粉絲頁](https://www.facebook.com/profile.php?id=61576838896062)
- 💼 LinkedIn：[Chu Kuo-Lung](https://www.linkedin.com/in/chu-kuo-lung)
- 📺 YouTube：[雲談風清頻道](https://www.youtube.com/channel/UCXDqLTdCMiCJ1j8xGRfwEig)
- 📧 Email：[fengqing.tw@gmail.com](mailto:fengqing.tw@gmail.com)

---

**⭐ 如果這個專案對您有幫助，歡迎給個 Star！**

*最後更新：2025年1月27日*