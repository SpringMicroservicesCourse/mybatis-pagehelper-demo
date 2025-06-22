# MyBatis PageHelper 分頁查詢示範專案

## 專案介紹

本專案展示如何在 Spring Boot 應用程式中整合 MyBatis 與 PageHelper 進行資料庫分頁查詢。專案實作了兩種分頁方式：

1. **RowBounds 分頁**：使用 MyBatis 原生的 RowBounds 進行分頁
2. **PageHelper 分頁**：使用 PageHelper 套件進行更靈活的分頁處理

專案以咖啡店商品管理為例，展示如何對 `t_coffee` 資料表進行分頁查詢，並處理 `Money` 類型的價格欄位。

## 技術棧

本專案主要依賴以下軟體與工具：

- **Java 21**：主要開發語言
- **Spring Boot 3.4.5**：應用程式框架
- **MyBatis Spring Boot Starter 3.0.4**：ORM 框架
- **PageHelper Spring Boot Starter 2.1.1**：分頁查詢套件
- **Joda Money 2.0.2**：貨幣處理套件
- **H2 Database**：內嵌式資料庫
- **Lombok**：程式碼簡化工具
- **Maven**：專案建置工具

## 專案結構

```
mybatis-pagehelper-demo/
├── src/
│   ├── main/
│   │   ├── java/tw/spring/data/mybatisdemo/
│   │   │   ├── handler/
│   │   │   │   └── MoneyTypeHandler.java          # Money 類型轉換處理器
│   │   │   ├── mapper/
│   │   │   │   └── CoffeeMapper.java              # 咖啡資料存取介面
│   │   │   ├── model/
│   │   │   │   └── Coffee.java                    # 咖啡實體類別
│   │   │   └── MybatisDemoApplication.java        # 主應用程式類別
│   │   └── resources/
│   │       ├── application.properties             # 應用程式設定檔
│   │       ├── data.sql                           # 測試資料
│   │       └── schema.sql                         # 資料表結構
│   └── test/
│       └── java/tw/spring/data/mybatisdemo/
│           └── MybatisDemoApplicationTests.java   # 單元測試
├── pom.xml                                        # Maven 專案設定
└── README.md                                      # 專案說明文件
```

## 快速開始

### 1. 克隆此倉庫：
```bash
git clone https://github.com/SpringMicroservicesCourse/mybatis-pagehelper-demo
```

### 2. 進入專案目錄：
```bash
cd mybatis-pagehelper-demo
```

### 3. 建置專案：
```bash
mvn clean compile
```

### 4. 執行應用程式：
```bash
mvn spring-boot:run
```

### 5. 執行測試：
```bash
mvn test
```

## 核心功能說明

### MoneyTypeHandler 類型轉換器
```java
/**
 * 在 Money 與 Long 之間轉換的 TypeHandler，處理 TWD 台幣
 * 資料庫儲存以分為單位，應用程式使用 Money 物件
 */
public class MoneyTypeHandler extends BaseTypeHandler<Money> {
    // 將 Money 物件轉換為資料庫的 Long 值（以分為單位）
    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, Money parameter, JdbcType jdbcType) throws SQLException {
        ps.setLong(i, parameter.getAmountMinorLong());
    }
    
    // 將資料庫的 Long 值轉換為 Money 物件
    private Money parseMoney(Long value) {
        return Money.of(CurrencyUnit.of("TWD"), value / 100.0);
    }
}
```

### 分頁查詢實作
```java
@Mapper
public interface CoffeeMapper {
    // 使用 RowBounds 進行分頁查詢
    @Select("select * from t_coffee order by id")
    List<Coffee> findAllWithRowBounds(RowBounds rowBounds);

    // 使用 PageHelper 進行分頁查詢
    @Select("select * from t_coffee order by id")
    List<Coffee> findAllWithParam(@Param("pageNum") int pageNum,
                                  @Param("pageSize") int pageSize);
}
```

## 進階說明

### 環境變數
請設定必要的環境變數，如資料庫連線字串、API 金鑰等，確保系統能正常運作。

### 應用程式屬性設定
根據不同環境（開發、測試、正式）調整 `application.properties` 中的參數：

```properties
# MyBatis 設定
mybatis.type-handlers-package=tw.spring.data.mybatisdemo.handler
mybatis.configuration.map-underscore-to-camel-case=true

# PageHelper 分頁設定
# 使用 RowBounds.offset 作為 pageNum
pagehelper.offset-as-page-num=true
# 合理化分頁參數，頁數小於1時查詢第一頁，頁數大於總頁數時查詢最後一頁
pagehelper.reasonable=true
# 允許 pageSize=0，返回所有結果
pagehelper.page-size-zero=true
# 支援通過 Mapper 接口參數來傳遞分頁參數
pagehelper.support-methods-arguments=true
```

### 外部服務連接
若專案需連接第三方服務，請確認相關憑證與連線設定已正確配置。

## 常見問題與解決方案

### 1. 分頁查詢結果異常
**問題原因**：PageHelper 設定不當
**解決方案**：檢查 `application.properties` 中的 PageHelper 相關設定

## 參考資源

- [MyBatis 官方文件](https://mybatis.org/mybatis-3/)
- [PageHelper 官方文件](https://pagehelper.github.io/)
- [Spring Boot 官方文件](https://spring.io/projects/spring-boot)
- [Joda Money 官方文件](https://www.joda.org/joda-money/)

## 注意事項

### 程式碼品質要求
- 在重要的程式碼區塊添加清楚註解，方便團隊成員理解與維護
- 使用台灣常用的專業用語，確保溝通順暢且符合本地習慣
- 遵循 Java 命名規範，使用有意義的變數與方法名稱

### 開發注意事項
- 確保 MyBatis TypeHandler 包名設定正確
- 測試時注意包名一致性
- 分頁查詢時注意效能考量
- 貨幣處理時注意精度問題

### 部署注意事項
- 確認資料庫連線設定正確
- 檢查應用程式屬性檔案設定
- 驗證外部服務連線狀態

## 授權說明

本專案採用 MIT 授權條款，詳見 LICENSE 檔案。 