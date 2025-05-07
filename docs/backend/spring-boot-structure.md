# 專題題目追蹤系統 - Spring Boot 實作指南

<div align="center">

![Spring Boot](https://img.shields.io/badge/Spring_Boot-6DB33F?style=for-the-badge&logo=spring-boot&logoColor=white)
![MariaDB](https://img.shields.io/badge/MariaDB-003545?style=for-the-badge&logo=mariadb&logoColor=white)
![Java](https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)
![REST API](https://img.shields.io/badge/REST_API-FF6C37?style=for-the-badge&logo=postman&logoColor=white)

</div>

## 📋 目錄

- [專案概述](#專案概述)
- [技術棧](#技術棧)
- [專案結構](#專案結構)
- [開發指南](#開發指南)
- [API文檔](#api文檔)
- [安全性實作](#安全性實作)
- [部署指南](#部署指南)
- [開發者](#開發者)

## 📝 專案概述

專題題目追蹤系統是一個基於Spring Boot的Web應用程式，旨在幫助教師有效地追蹤和記錄學生的專題進展。本系統實作了完整的用戶管理、專題組別管理、週報提交、評分回饋和記錄查看等功能。

## 🛠️ 技術棧

- **後端框架**：Spring Boot 3.0.x
- **資料庫**：MariaDB 10.6
- **ORM**：Spring Data JPA / Hibernate
- **安全性**：Spring Security、JWT認證
- **API設計**：RESTful API
- **專案構建**：Maven
- **其他**：Lombok, ModelMapper, Swagger(OpenAPI)

## 📂 專案結構

專案採用分層架構，遵循領域驅動設計(DDD)原則：

```
project-tracking-system/
│
├── src/
│   ├── main/
│   │   ├── java/com/example/pts/
│   │   │   ├── config/                 # 配置類
│   │   │   ├── controller/             # API控制器
│   │   │   ├── dto/                    # 數據傳輸對象
│   │   │   ├── entity/                 # JPA實體類
│   │   │   ├── exception/              # 異常處理
│   │   │   ├── repository/             # 數據訪問層
│   │   │   ├── security/               # 安全配置
│   │   │   ├── service/                # 業務邏輯層
│   │   │   ├── util/                   # 公共工具類
│   │   │   └── ProjectTrackingSystemApplication.java
│   │   │
│   │   └── resources/
│   │       ├── application.properties  # 應用配置
│   │       ├── application-dev.properties  # 開發環境配置
│   │       ├── application-prod.properties # 生產環境配置
│   │       └── db/
│   │           └── migration/          # Flyway 數據庫遷移腳本
│   │
│   └── test/                           # 單元測試和整合測試
│
├── pom.xml                             # Maven 配置
└── README.md                           # 項目說明文件
```

### 關鍵文件和功能說明

#### 1. 配置類 (config/)

- **WebConfig.java**: 處理CORS、靜態資源等Web配置
  - `corsConfigurer()`: 配置跨域資源共享
  - `addFormatters()`: 註冊自定義的格式轉換器

- **SwaggerConfig.java**: API文檔配置
  - `api()`: 配置Swagger/OpenAPI文檔

#### 2. 控制器 (controller/)

分為六大主要控制器，對應系統主要功能：

- **AuthController.java**: 認證相關端點
  - `login()`: 用戶登入
  - `register()`: 用戶註冊

- **UserController.java**: 用戶管理
  - `getUserProfile()`: 獲取用戶信息
  - `updateUser()`: 更新用戶信息
  - `changePassword()`: 變更密碼

- **StudentController.java**: 學生相關操作
  - `getAllStudents()`: 獲取所有學生
  - `getStudentById()`: 獲取特定學生詳情
  - `assignGroup()`: 分配學生到小組

- **TeacherController.java**: 教師相關操作
  - `getAllTeachers()`: 獲取所有教師
  - `getTeacherById()`: 獲取特定教師詳情
  - `getTeacherGroups()`: 獲取教師指導的小組

- **GroupController.java**: 小組管理
  - `createGroup()`: 創建新小組
  - `getGroupDetails()`: 獲取小組詳情
  - `updateGroup()`: 更新小組信息
  - `getGroupMembers()`: 獲取小組成員

- **WeeklyReportController.java**: 週報管理
  - `submitReport()`: 提交週報
  - `getReportById()`: 獲取特定週報
  - `getReportsByStudent()`: 獲取學生的週報
  - `getReportsByGroup()`: 獲取小組的所有週報

- **EvaluationController.java**: 評分管理
  - `evaluateReport()`: 評分週報
  - `getEvaluationById()`: 獲取特定評分
  - `getEvaluationsByTeacher()`: 獲取教師的所有評分

- **FileController.java**: 檔案管理
  - `uploadFile()`: 上傳檔案
  - `downloadFile()`: 下載檔案
  - `getFilesByReport()`: 獲取週報相關檔案

- **AdminController.java**: 管理員操作
  - `approveUser()`: 審核用戶註冊
  - `getSystemLogs()`: 查看系統日誌
  - `getStatistics()`: 獲取系統統計數據

#### 3. 數據傳輸對象 (dto/)

- **request/**: 請求DTO
  - `LoginRequest.java`: 登入請求
  - `RegisterRequest.java`: 註冊請求
  - `ReportSubmissionRequest.java`: 週報提交請求
  - `EvaluationRequest.java`: 評分請求

- **response/**: 響應DTO
  - `UserResponse.java`: 用戶資訊響應
  - `ReportResponse.java`: 週報響應
  - `GroupResponse.java`: 小組信息響應
  - `ApiResponse.java`: 通用API響應

#### 4. 實體類 (entity/)

實體類對應資料庫表結構：

- `User.java`: 用戶表
- `Student.java`: 學生表
- `Teacher.java`: 教師表
- `Group.java`: 小組表
- `AcademicYear.java`: 學年表
- `WeeklyReport.java`: 週報表
- `Evaluation.java`: 評分表
- `EvaluationCriteria.java`: 評分標準表
- `EvaluationItem.java`: 評分項目表
- `File.java`: 檔案表
- `Notification.java`: 通知表
- `SystemLog.java`: 系統日誌表

每個實體類包含：
- JPA註解(@Entity, @Table等)
- 關係映射(@OneToMany, @ManyToOne等)
- 欄位定義和約束(@Column, @NotNull等)
- Lombok註解簡化代碼(@Data, @Builder等)

#### 5. 異常處理 (exception/)

- `GlobalExceptionHandler.java`: 全局異常處理
- `ResourceNotFoundException.java`: 資源未找到異常
- `BadRequestException.java`: 錯誤請求異常
- `UnauthorizedException.java`: 未授權異常
- `AccessDeniedException.java`: 訪問拒絕異常

#### 6. 數據訪問層 (repository/)

JPA 儲存庫接口：

- `UserRepository.java`: 用戶數據訪問
- `StudentRepository.java`: 學生數據訪問
- `TeacherRepository.java`: 教師數據訪問
- `GroupRepository.java`: 小組數據訪問
- `WeeklyReportRepository.java`: 週報數據訪問
- `EvaluationRepository.java`: 評分數據訪問
- `FileRepository.java`: 檔案數據訪問
- `NotificationRepository.java`: 通知數據訪問
- `SystemLogRepository.java`: 系統日誌數據訪問

每個儲存庫都包含自定義查詢方法，如：
- `findByEmailAndStatus()`
- `findAllByGroupId()`
- `countBySubmitDateBetween()`

#### 7. 安全配置 (security/)

- `SecurityConfig.java`: Spring Security配置
  - `securityFilterChain()`: 配置安全過濾器鏈
  - `authenticationProvider()`: 配置認證提供者

- `JwtService.java`: JWT令牌服務
  - `generateToken()`: 生成JWT令牌
  - `validateToken()`: 驗證JWT令牌
  - `extractUsername()`: 提取用戶名

- `UserDetailsServiceImpl.java`: 用戶詳情服務實現
  - `loadUserByUsername()`: 載入用戶詳情

#### 8. 業務邏輯層 (service/)

- **接口**:
  - `UserService.java`
  - `StudentService.java`
  - `TeacherService.java`
  - `GroupService.java`
  - `WeeklyReportService.java`
  - `EvaluationService.java`
  - `FileService.java`
  - `NotificationService.java`
  - `SystemLogService.java`

- **實現**:
  - `UserServiceImpl.java`: 用戶服務實現
    - `registerUser()`: 註冊新用戶
    - `authenticateUser()`: 認證用戶
    
  - `StudentServiceImpl.java`: 學生服務實現
    - `createStudent()`: 創建學生
    - `assignStudentToGroup()`: 分配學生到小組
    
  - `TeacherServiceImpl.java`: 教師服務實現
    - `createTeacher()`: 創建教師
    - `getTeacherGroups()`: 獲取教師指導的小組
    
  - `GroupServiceImpl.java`: 小組服務實現
    - `createGroup()`: 創建小組
    - `updateGroup()`: 更新小組信息
    
  - `WeeklyReportServiceImpl.java`: 週報服務實現
    - `submitReport()`: 提交週報
    - `getReportById()`: 獲取週報詳情
    
  - `EvaluationServiceImpl.java`: 評分服務實現
    - `evaluateReport()`: 評分週報
    - `calculateFinalScore()`: 計算最終得分
    
  - `FileServiceImpl.java`: 檔案服務實現
    - `storeFile()`: 儲存檔案
    - `getFile()`: 獲取檔案
    
  - `NotificationServiceImpl.java`: 通知服務實現
    - `createNotification()`: 創建通知
    - `markAsRead()`: 標記為已讀
    
  - `SystemLogServiceImpl.java`: 系統日誌服務實現
    - `logAction()`: 記錄操作日誌
    - `searchLogs()`: 搜索日誌

#### 9. 工具類 (util/)

- `DateUtils.java`: 日期處理工具
- `ValidationUtils.java`: 數據驗證工具
- `FileUtils.java`: 檔案處理工具
- `SecurityUtils.java`: 安全相關工具

## 🚀 開發指南

### 前置條件

- JDK 17 或更高版本
- Maven 3.8 或更高版本
- MariaDB 10.6 或更高版本
- IntelliJ IDEA (推薦)

### 初始設置

1. **複製專案**

```bash
git clone https://github.com/yourusername/project-tracking-system.git
cd project-tracking-system
```

2. **配置資料庫**

編輯 `src/main/resources/application-dev.properties`:

```properties
spring.datasource.url=jdbc:mariadb://localhost:3306/pts_db
spring.datasource.username=your_username
spring.datasource.password=your_password
spring.datasource.driver-class-name=org.mariadb.jdbc.Driver

spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

3. **構建並運行**

```bash
mvn clean install
mvn spring-boot:run -Dspring-boot.run.profiles=dev
```

應用程式將在 http://localhost:8080 啟動

### 資料庫遷移

專案使用 Flyway 進行資料庫遷移管理。遷移腳本位於 `src/main/resources/db/migration/` 目錄。

主要遷移腳本：

- `V1__Create_Initial_Schema.sql`: 創建基本表結構
- `V2__Add_Foreign_Keys.sql`: 添加外鍵約束
- `V3__Insert_Initial_Data.sql`: 插入初始數據

## 📚 API文檔

專案整合了 Swagger (OpenAPI) 用於 API 文檔。運行應用程式後，可以訪問：

- Swagger UI: http://localhost:8080/swagger-ui.html
- OpenAPI JSON: http://localhost:8080/v3/api-docs

## 🔒 安全性實作

系統安全性基於以下方面：

1. **認證與授權**:
   - JWT (JSON Web Token) 用於無狀態認證
   - 基於角色的訪問控制 (RBAC)
   - 密碼加密存儲 (使用 BCrypt)

2. **保護敏感數據**:
   - 資料庫密碼加密
   - 敏感信息傳輸加密
   - 防止 SQL 注入攻擊

3. **事務管理**:
   - 使用 `@Transactional` 註解確保資料一致性
   - 實現 ACID 特性

## 📦 部署指南

### JAR 檔部署

1. 構建生產 JAR 檔:

```bash
mvn clean package -Pprod
```

2. 運行 JAR 檔:

```bash
java -jar target/project-tracking-system-0.0.1-SNAPSHOT.jar --spring.profiles.active=prod
```

### Docker 部署

1. 構建 Docker 映像:

```bash
docker build -t project-tracking-system .
```

2. 運行容器:

```bash
docker run -p 8080:8080 -e SPRING_PROFILES_ACTIVE=prod project-tracking-system
```

## 👨‍💻 開發者

<div align="center">
  
![Profile Banner](https://img.shields.io/badge/Database%20Developer-MariaDB-003545?style=for-the-badge&logo=mariadb&logoColor=white)

</div>

### 陳廷威 40943212

我是「專題題目追蹤系統」的開發者，專精於資料庫設計與實作、系統架構規劃及前後端整合。在資料庫正規化設計與ACID特性實踐方面有深入研究經驗。

#### 🧰 技術棧

**資料庫技術**
![MariaDB](https://img.shields.io/badge/MariaDB-003545?style=for-the-badge&logo=mariadb&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)

**後端/前端技術**
![Java](https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring_Boot-6DB33F?style=for-the-badge&logo=spring-boot&logoColor=white)
![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)

**開發工具**
![IntelliJ IDEA](https://img.shields.io/badge/IntelliJ_IDEA-000000?style=for-the-badge&logo=intellij-idea&logoColor=white)
![Git](https://img.shields.io/badge/Git-F05032?style=for-the-badge&logo=git&logoColor=white)

#### 🔗 聯絡方式

[![Email](https://img.shields.io/badge/Email-40943212%40gm.nfu.edu.tw-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:40943212@gm.nfu.edu.tw)
[![GitHub](https://img.shields.io/badge/GitHub-TingWei--Chen-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/TingWei-Chen)
