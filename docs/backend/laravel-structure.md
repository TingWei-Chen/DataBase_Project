# 專題進度追蹤系統 - Laravel 實作指南

<div align="center">

![Laravel](https://img.shields.io/badge/Laravel-FF2D20?style=for-the-badge&logo=laravel&logoColor=white)
![MariaDB](https://img.shields.io/badge/MariaDB-003545?style=for-the-badge&logo=mariadb&logoColor=white)
![PHP](https://img.shields.io/badge/PHP-777BB4?style=for-the-badge&logo=php&logoColor=white)
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

## 📝 專案概述

專題進度追蹤系統是一個基於Laravel的Web應用程式，目的在於協助教師有效追蹤和記錄學生的專題進展。本系統實現了用戶管理、專題組別管理、週報提交、評分回饋和記錄查看等核心功能。

## 🛠️ 技術棧

- **後端框架**：Laravel 9.x
- **資料庫**：MariaDB 10.6
- **ORM**：Eloquent ORM
- **認證**：Laravel Sanctum (API Token) / JWT
- **API設計**：RESTful API
- **前端整合**：Blade 模板 / API 模式
- **自動化測試**：PHPUnit
- **依賴管理**：Composer
- **其他**：Laravel Breeze、Laravel Mix、Intervention Image

## 📂 專案結構

Laravel專案遵循MVC架構，並採用Laravel標準的目錄結構：

```
project-tracking-system/
│
├── app/                         # 應用程式核心代碼
│   ├── Console/                 # Artisan命令
│   ├── Exceptions/              # 異常處理類
│   ├── Http/                    # HTTP層
│   │   ├── Controllers/         # 控制器
│   │   │   ├── API/             # API控制器
│   │   │   └── Web/             # Web控制器
│   │   ├── Middleware/          # 中間件
│   │   └── Requests/            # 表單請求驗證
│   ├── Models/                  # 資料庫模型
│   ├── Policies/                # 授權策略
│   ├── Providers/               # 服務提供者
│   └── Services/                # 業務邏輯服務
│
├── bootstrap/                   # 啟動框架
├── config/                      # 設定檔案
├── database/                    # 資料庫
│   ├── factories/               # 數據工廠
│   ├── migrations/              # 遷移檔案
│   └── seeders/                 # 資料填充
│
├── public/                      # 公共文件
│   ├── css/                     # 編譯後的CSS
│   ├── js/                      # 編譯後的JS
│   └── uploads/                 # 用戶上傳文件
│
├── resources/                   # 前端資源
│   ├── css/                     # CSS源文件
│   ├── js/                      # JavaScript源文件
│   └── views/                   # Blade視圖
│
├── routes/                      # 路由定義
│   ├── api.php                  # API路由
│   ├── web.php                  # Web路由
│   └── channels.php             # 廣播通道
│
├── storage/                     # 應用程式生成的檔案
│   ├── app/                     # 應用程式儲存
│   ├── framework/               # 框架儲存
│   └── logs/                    # 日誌檔案
│
├── tests/                       # 測試代碼
│
├── .env                         # 環境變數
└── composer.json                # Composer依賴
```

### 關鍵文件和功能說明

#### 1. 模型 (app/Models/)

- **User.php**: 用戶模型，繼承自Laravel的認證模型
  - 關聯方法: `student()`, `teacher()`
  - 屬性: `fillable`, `hidden`

- **Student.php**: 學生模型
  - 關聯方法: `user()`, `group()`, `weeklyReports()`
  - 屬性: `fillable`, `appends`

- **Teacher.php**: 教師模型
  - 關聯方法: `user()`, `groups()`, `evaluations()`
  - 屬性: `fillable`

- **Group.php**: 專題組別模型
  - 關聯方法: `students()`, `teacher()`, `academicYear()`
  - 屬性: `fillable`

- **WeeklyReport.php**: 週報模型
  - 關聯方法: `student()`, `evaluation()`, `files()`
  - 屬性: `fillable`, `dates`

- **Evaluation.php**: 評分模型
  - 關聯方法: `report()`, `teacher()`, `items()`
  - 屬性: `fillable`

- **EvaluationCriteria.php**: 評分標準模型
  - 關聯方法: `year()`, `items()`
  - 屬性: `fillable`

- **EvaluationItem.php**: 評分項目模型
  - 關聯方法: `evaluation()`, `criteria()`
  - 屬性: `fillable`

- **File.php**: 檔案模型
  - 關聯方法: `report()`
  - 屬性: `fillable`
  - 方法: `getUrlAttribute()`

- **AcademicYear.php**: 學年模型
  - 關聯方法: `groups()`, `criteria()`
  - 屬性: `fillable`

- **Notification.php**: 通知模型
  - 關聯方法: `user()`
  - 屬性: `fillable`

- **SystemLog.php**: 系統日誌模型
  - 關聯方法: `user()`
  - 屬性: `fillable`, `casts`

#### 2. 控制器 (app/Http/Controllers/)

##### API控制器 (API/)

- **AuthController.php**: 認證控制器
  - `register()`: 註冊新用戶
  - `login()`: 用戶登入
  - `logout()`: 用戶登出
  - `refreshToken()`: 更新令牌

- **UserController.php**: 用戶控制器
  - `show()`: 獲取用戶資料
  - `update()`: 更新用戶資料
  - `changePassword()`: 修改密碼

- **StudentController.php**: 學生控制器
  - `index()`: 列出所有學生
  - `show()`: 獲取特定學生
  - `update()`: 更新學生資料
  - `assignGroup()`: 分配學生到組別

- **TeacherController.php**: 教師控制器
  - `index()`: 列出所有教師
  - `show()`: 獲取特定教師
  - `update()`: 更新教師資料
  - `getGroups()`: 獲取教師指導的組別

- **GroupController.php**: 組別控制器
  - `index()`: 列出所有組別
  - `show()`: 獲取特定組別
  - `store()`: 創建組別
  - `update()`: 更新組別資料
  - `destroy()`: 刪除組別
  - `getMembers()`: 獲取組別成員

- **WeeklyReportController.php**: 週報控制器
  - `index()`: 列出所有週報
  - `show()`: 獲取特定週報
  - `store()`: 提交週報
  - `update()`: 更新週報
  - `getByStudent()`: 獲取學生的週報
  - `getByGroup()`: 獲取組別的週報

- **EvaluationController.php**: 評分控制器
  - `store()`: 創建評分
  - `show()`: 獲取特定評分
  - `getByTeacher()`: 獲取教師的評分記錄
  - `getByReport()`: 獲取週報的評分

- **FileController.php**: 檔案控制器
  - `store()`: 上傳檔案
  - `show()`: 獲取檔案資訊
  - `download()`: 下載檔案
  - `getByReport()`: 獲取週報的檔案

- **NotificationController.php**: 通知控制器
  - `index()`: 獲取用戶通知
  - `markAsRead()`: 標記通知為已讀
  - `destroy()`: 刪除通知

- **AdminController.php**: 管理員控制器
  - `approveUser()`: 審核用戶
  - `getSystemLogs()`: 獲取系統日誌
  - `getStatistics()`: 獲取統計數據

##### Web控制器 (Web/)

根據需求類似實現對應的網頁版控制器。

#### 3. 表單請求驗證 (app/Http/Requests/)

- **LoginRequest.php**: 登入請求驗證
- **RegisterRequest.php**: 註冊請求驗證
- **WeeklyReportRequest.php**: 週報提交驗證
- **EvaluationRequest.php**: 評分提交驗證
- **GroupRequest.php**: 組別創建/更新驗證

#### 4. 服務類 (app/Services/)

- **AuthService.php**: 認證相關業務邏輯
  - `register()`: 註冊邏輯
  - `login()`: 登入邏輯
  - `createToken()`: 創建令牌

- **WeeklyReportService.php**: 週報相關業務邏輯
  - `create()`: 創建週報
  - `findByStudent()`: 查找學生週報
  - `findByDateRange()`: 依日期範圍查找

- **EvaluationService.php**: 評分相關業務邏輯
  - `evaluate()`: 評分週報
  - `calculateFinalScore()`: 計算最終得分

- **FileService.php**: 檔案操作相關邏輯
  - `store()`: 儲存檔案
  - `retrievePath()`: 獲取檔案路徑

- **NotificationService.php**: 通知相關業務邏輯
  - `notify()`: 發送通知
  - `getUnread()`: 獲取未讀通知

- **SystemLogService.php**: 系統日誌相關邏輯
  - `log()`: 記錄系統操作
  - `search()`: 搜索日誌

#### 5. 中間件 (app/Http/Middleware/)

- **RoleMiddleware.php**: 角色權限控制
- **ActivityLogger.php**: 活動日誌記錄
- **LogRequestResponse.php**: API請求響應日誌

#### 6. 路由定義 (routes/)

- **api.php**: API路由定義
  ```php
  // Auth Routes
  Route::post('/login', [AuthController::class, 'login']);
  Route::post('/register', [AuthController::class, 'register']);
  
  // Protected Routes
  Route::middleware('auth:sanctum')->group(function () {
      // User Routes
      Route::get('/user', [UserController::class, 'show']);
      Route::put('/user', [UserController::class, 'update']);
      
      // Student Routes
      Route::middleware('role:student')->group(function () {
          Route::post('/reports', [WeeklyReportController::class, 'store']);
          Route::get('/reports/my', [WeeklyReportController::class, 'getByStudent']);
      });
      
      // Teacher Routes
      Route::middleware('role:teacher')->group(function () {
          Route::get('/groups/my', [TeacherController::class, 'getGroups']);
          Route::post('/evaluations', [EvaluationController::class, 'store']);
      });
      
      // Admin Routes
      Route::middleware('role:admin')->group(function () {
          Route::put('/users/{id}/approve', [AdminController::class, 'approveUser']);
          Route::get('/logs', [AdminController::class, 'getSystemLogs']);
      });
  });
  ```

- **web.php**: Web路由定義（如果同時支援網頁版）

#### 7. 資料庫遷移 (database/migrations/)

- **2014_10_12_000000_create_users_table.php**
- **2023_01_01_000001_create_students_table.php**
- **2023_01_01_000002_create_teachers_table.php**
- **2023_01_01_000003_create_academic_years_table.php**
- **2023_01_01_000004_create_groups_table.php**
- **2023_01_01_000005_create_weekly_reports_table.php**
- **2023_01_01_000006_create_evaluation_criteria_table.php**
- **2023_01_01_000007_create_evaluations_table.php**
- **2023_01_01_000008_create_evaluation_items_table.php**
- **2023_01_01_000009_create_files_table.php**
- **2023_01_01_000010_create_notifications_table.php**
- **2023_01_01_000011_create_system_logs_table.php**

#### 8. 資料填充 (database/seeders/)

- **DatabaseSeeder.php**: 主要填充器
- **UserSeeder.php**: 用戶資料
- **AcademicYearSeeder.php**: 學年資料
- **EvaluationCriteriaSeeder.php**: 評分標準資料

## 🚀 開發指南

### 前置條件

- PHP 8.1 或更高版本
- Composer 2.0 或更高版本
- MariaDB 10.6 或更高版本
- Node.js 和 npm (用於前端資源)

### 初始設置

1. **複製專案**

```bash
git clone https://github.com/yourusername/project-tracking-system.git
cd project-tracking-system
```

2. **安裝依賴**

```bash
composer install
npm install
npm run dev
```

3. **配置環境**

複製 `.env.example` 為 `.env` 並設定資料庫連接：

```bash
cp .env.example .env
php artisan key:generate
```

編輯 `.env` 檔案：

```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=pts_db
DB_USERNAME=your_username
DB_PASSWORD=your_password
```

4. **遷移和填充資料庫**

```bash
php artisan migrate
php artisan db:seed
```

5. **啟動開發伺服器**

```bash
php artisan serve
```

應用程式將在 http://localhost:8000 啟動

### 常用開發命令

- 創建新控制器: `php artisan make:controller API/NewController --api`
- 創建新模型: `php artisan make:model NewModel -m`
- 運行測試: `php artisan test`
- 清除快取: `php artisan optimize:clear`

## 📚 API文檔

API路由和文檔可以通過整合 Laravel Scribe 或 Swagger 自動生成。

設置 Scribe:

1. 安裝套件:

```bash
composer require knuckleswtf/scribe:3.*
```

2. 發布設定:

```bash
php artisan vendor:publish --provider="Knuckleswtf\Scribe\ScribeServiceProvider" --tag=config
```

3. 生成文檔:

```bash
php artisan scribe:generate
```

生成的API文檔將可以在 `/docs` 路徑訪問。

## 🔒 安全性實作

### 1. 認證與授權

Laravel專案使用以下安全機制：

- **Laravel Sanctum**: 提供API令牌認證
- **中間件控制**: 使用角色中間件控制路由訪問
- **授權策略**: 使用Laravel Policy定義細粒度的授權控制

```php
// app/Policies/WeeklyReportPolicy.php
public function view(User $user, WeeklyReport $report)
{
    // 學生只能查看自己的週報
    if ($user->isStudent()) {
        return $user->student->id === $report->student_id;
    }
    
    // 教師只能查看其指導組別的週報
    if ($user->isTeacher()) {
        return $user->teacher->groups()
            ->whereHas('students', function ($query) use ($report) {
                $query->where('id', $report->student_id);
            })->exists();
    }
    
    // 管理員可以查看所有週報
    return $user->isAdmin();
}
```

### 2. 資料驗證與防護

- **表單請求驗證**: 確保所有輸入數據符合規則
- **跨站請求偽造防護**: 使用Laravel CSRF保護
- **參數綁定**: 防止SQL注入攻擊
- **XSS防護**: 自動轉義HTML標籤

### 3. 檔案安全

- **檔案名驗證**: 確保安全的檔案名
- **MIME類型檢測**: 確保檔案類型安全
- **隨機檔案名**: 使用UUID生成檔案名避免衝突
- **路徑處理**: 防止目錄遍歷攻擊

## 📦 部署指南

### 共享主機部署

1. 上傳專案到主機的公共目錄 (通常是 `public_html`)
2. 修改 `index.php` 指向正確的 `bootstrap/app.php` 路徑
3. 設定 `.htaccess` 重定向規則
4. 設定環境變數

### VPS部署

1. 在伺服器安裝必要套件:

```bash
apt update
apt install -y php8.1-cli php8.1-fpm php8.1-mysql php8.1-mbstring php8.1-xml php8.1-curl nginx
```

2. 設定Nginx:

```nginx
server {
    listen 80;
    server_name your-domain.com;
    root /var/www/project-tracking-system/public;

    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

3. 部署代碼並設定權限:

```bash
cd /var/www
git clone https://github.com/yourusername/project-tracking-system.git
cd project-tracking-system
composer install --optimize-autoloader --no-dev
npm install
npm run build

chown -R www-data:www-data /var/www/project-tracking-system
chmod -R 755 /var/www/project-tracking-system/storage
```

4. 設定環境並執行遷移:

```bash
cp .env.example .env
php artisan key:generate
php artisan migrate --force
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

### Docker部署

1. 創建Dockerfile:

```dockerfile
FROM php:8.1-fpm

# 安裝依賴
RUN apt-get update && apt-get install -y \
    git \
    curl \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    zip \
    unzip

# 安裝PHP擴展
RUN docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd

# 安裝Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# 設定工作目錄
WORKDIR /var/www

# 複製專案檔案
COPY . /var/www

# 安裝依賴
RUN composer install --optimize-autoloader --no-dev

# 設定權限
RUN chown -R www-data:www-data /var/www/storage
RUN chmod -R 755 /var/www/storage

# 暴露埠
EXPOSE 9000

CMD ["php-fpm"]
```

2. 創建docker-compose.yml:

```yaml
version: '3'
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      - .:/var/www
    networks:
      - pts-network

  webserver:
    image: nginx:alpine
    volumes:
      - .:/var/www
      - ./nginx/conf.d/:/etc/nginx/conf.d/
    ports:
      - "80:80"
    networks:
      - pts-network

  db:
    image: mariadb:10.6
    environment:
      MYSQL_DATABASE: pts_db
      MYSQL_ROOT_PASSWORD: your_password
      MYSQL_USER: your_username
      MYSQL_PASSWORD: your_password
    volumes:
      - dbdata:/var/lib/mysql
    networks:
      - pts-network

networks:
  pts-network:

volumes:
  dbdata:
```

3. 啟動容器:

```bash
docker-compose up -d
```

4. 在容器內執行遷移:

```bash
docker-compose exec app php artisan migrate --force
```
