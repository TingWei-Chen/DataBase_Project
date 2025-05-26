# 專題進度追蹤系統 - 資料庫設計文檔

## 1. 用戶資料表 (users)

### SQL建立語句
```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    user_type ENUM('student', 'teacher', 'admin') NOT NULL,
    status ENUM('pending', 'active', 'suspended') DEFAULT 'pending',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

### 完整性限制
- **實體完整性**：user_id作為主鍵，採用自動增長機制從1開始每次遞增1，不允許空值
- **參照完整性**：作為其他表的外鍵參照目標
- **值域完整性**：email須符合RFC 5322標準，user_type限制為三種角色類型
- **用戶定義完整性**：email在系統內必須唯一

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| user_id | **主鍵**：表格主要識別碼<br>**自動增長**：系統自動從1開始每次遞增1<br>**非空**：不允許空值 |
| name | **非空**：必須填入使用者姓名<br>**長度限制**：最長50個字元 |
| email | **非空**：必須填入電子郵件<br>**唯一性**：系統內不可重複<br>**長度限制**：最長100個字元<br>**格式檢查**：須符合RFC 5322標準，包含@符號且本地部分不超過64字元，域名部分不超過253字元 |
| user_type | **非空**：必須指定使用者角色<br>**值域限制**：僅允許學生、教師、管理員三種角色類型 |
| status | **預設值**：新註冊用戶預設為待審核狀態<br>**值域限制**：僅允許待審核、已啟用、已停權三種狀態<br>待審核表示等待管理員審核，已啟用表示可正常使用，已停權表示帳號被凍結 |
| created_at | **預設值**：系統自動記錄帳號建立的當前時間<br>**時間格式**：年-月-日 時:分:秒格式 |
| updated_at | **預設值**：系統自動記錄帳號建立的當前時間<br>**自動更新**：每次資料異動時自動更新為當前時間<br>**時間格式**：年-月-日 時:分:秒格式 |

### 相關VIEW設計
**用戶狀態統計檢視 (user_status_summary)**：統計各類型用戶的狀態分布，協助管理員了解用戶審核情況

## 2. 密碼資料表 (user_passwords)

### SQL建立語句
```sql
CREATE TABLE user_passwords (
    password_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE
);
```

### 完整性限制
- **實體完整性**：password_id作為主鍵，採用自動增長機制從1開始每次遞增1
- **參照完整性**：user_id與用戶資料表建立關聯，用戶刪除時密碼記錄同步刪除
- **值域完整性**：password_hash必須為非空值且長度足以容納加密結果
- **用戶定義完整性**：每個用戶可以有多個歷史密碼記錄

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| password_id | **主鍵**：表格主要識別碼<br>**自動增長**：系統自動從1開始每次遞增1<br>**非空**：不允許空值 |
| user_id | **非空**：必須關聯到特定用戶<br>**外鍵**：與用戶資料表建立關聯<br>**級聯刪除**：用戶刪除時密碼記錄同步刪除 |
| password_hash | **非空**：必須儲存加密後的密碼<br>**長度限制**：最長255個字元<br>儲存SHA-256雜湊值或bcrypt加密結果 |
| created_at | **預設值**：系統自動記錄密碼建立的當前時間<br>**時間格式**：年-月-日 時:分:秒格式 |

### 相關VIEW設計
**密碼安全檢視 (password_security_view)**：顯示密碼建立時間和安全性統計，協助系統管理員監控密碼安全

## 3. 學生資料表 (students)

### SQL建立語句
```sql
CREATE TABLE students (
    student_id VARCHAR(20) PRIMARY KEY,
    user_id INT NOT NULL UNIQUE,
    department VARCHAR(50) NOT NULL,
    admission_year YEAR NOT NULL,
    group_id INT,
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE,
    FOREIGN KEY (group_id) REFERENCES groups(group_id) ON DELETE SET NULL
);
```

### 完整性限制
- **實體完整性**：student_id作為主鍵，格式必須為8位數字字串
- **參照完整性**：user_id與用戶資料表一對一關聯，group_id與組別資料表關聯
- **值域完整性**：admission_year必須為有效年份，department必須填入科系名稱
- **用戶定義完整性**：student_id在系統內必須唯一，user_id也必須唯一

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| student_id | **主鍵**：學號作為主要識別碼<br>**格式限制**：必須為8位數字字串，例如40943212<br>**非空**：不允許空值 |
| user_id | **非空**：必須關聯到特定用戶<br>**唯一性**：一個用戶帳號只能對應一個學生身份<br>**外鍵**：與用戶資料表建立關聯<br>**級聯刪除**：用戶刪除時學生資料同步刪除 |
| department | **非空**：必須填入就讀科系<br>**長度限制**：最長50個字元，例如資訊工程系 |
| admission_year | **非空**：必須填入入學年份<br>**年份格式**：4位數年份，例如2024 |
| group_id | **可空值**：允許未分配組別的情況<br>**外鍵**：與組別資料表建立關聯<br>**設為空值**：組別刪除時此欄位設為空值 |

### 相關VIEW設計
**學生組別分配檢視 (student_group_assignment)**：顯示學生的組別分配情況，包含未分配組別的學生清單，協助進行組別管理

## 4. 教師資料表 (teachers)

### SQL建立語句
```sql
CREATE TABLE teachers (
    teacher_id VARCHAR(20) PRIMARY KEY,
    user_id INT NOT NULL UNIQUE,
    department VARCHAR(50) NOT NULL,
    title VARCHAR(30) NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE
);
```

### 完整性限制
- **實體完整性**：teacher_id作為主鍵，格式必須為T開頭加6位數字
- **參照完整性**：user_id與用戶資料表一對一關聯
- **值域完整性**：title必須填入有效職稱，department必須填入科系名稱
- **用戶定義完整性**：teacher_id在系統內必須唯一，user_id也必須唯一

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| teacher_id | **主鍵**：教師編號作為主要識別碼<br>**格式限制**：T開頭加6位數字，例如T123456<br>**非空**：不允許空值 |
| user_id | **非空**：必須關聯到特定用戶<br>**唯一性**：一個用戶帳號只能對應一個教師身份<br>**外鍵**：與用戶資料表建立關聯<br>**級聯刪除**：用戶刪除時教師資料同步刪除 |
| department | **非空**：必須填入任職科系<br>**長度限制**：最長50個字元，例如資訊工程系 |
| title | **非空**：必須填入職稱<br>**長度限制**：最長30個字元，例如教授、副教授、助理教授 |

### 相關VIEW設計
**教師指導負荷檢視 (teacher_workload_view)**：顯示每位教師指導的組別數量和學生人數，協助平衡指導負荷

## 5. 學年度資料表 (academic_years)

### SQL建立語句
```sql
CREATE TABLE academic_years (
    year_id INT PRIMARY KEY AUTO_INCREMENT,
    year_name VARCHAR(20) NOT NULL UNIQUE,
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    status ENUM('active', 'archived') DEFAULT 'active',
    CHECK (end_date > start_date)
);
```

### 完整性限制
- **實體完整性**：year_id作為主鍵，採用自動增長機制從1開始每次遞增1
- **參照完整性**：作為組別和週報資料表的外鍵參照目標
- **值域完整性**：year_name格式必須為年份-年份，日期範圍必須合理
- **用戶定義完整性**：year_name在系統內必須唯一，結束日期必須晚於開始日期

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| year_id | **主鍵**：表格主要識別碼<br>**自動增長**：系統自動從1開始每次遞增1<br>**非空**：不允許空值 |
| year_name | **非空**：必須填入學年度名稱<br>**唯一性**：學年度名稱不可重複<br>**格式限制**：須為年份-年份格式，例如2024-2025<br>**長度限制**：最長20個字元 |
| start_date | **非空**：必須填入學年度開始日期<br>**日期格式**：年-月-日格式 |
| end_date | **非空**：必須填入學年度結束日期<br>**日期格式**：年-月-日格式<br>**檢查約束**：結束日期必須晚於開始日期 |
| status | **預設值**：新建學年度預設為使用中狀態<br>**值域限制**：僅允許使用中、已歸檔兩種狀態<br>使用中表示當前學年度，已歸檔表示過去學年度 |

### 相關VIEW設計
**學年度統計檢視 (academic_year_statistics)**：顯示各學年度的組別數、學生數、專題完成率等統計資訊

## 6. 組別資料表 (groups)

### SQL建立語句
```sql
CREATE TABLE groups (
    group_id INT PRIMARY KEY AUTO_INCREMENT,
    group_name VARCHAR(50) NOT NULL,
    project_title VARCHAR(100) NOT NULL,
    teacher_id VARCHAR(20),
    year_id INT NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    status ENUM('active', 'completed', 'suspended') DEFAULT 'active',
    FOREIGN KEY (teacher_id) REFERENCES teachers(teacher_id) ON DELETE SET NULL,
    FOREIGN KEY (year_id) REFERENCES academic_years(year_id) ON DELETE RESTRICT
);
```

### 完整性限制
- **實體完整性**：group_id作為主鍵，採用自動增長機制從1開始每次遞增1
- **參照完整性**：teacher_id與教師資料表關聯，year_id與學年度資料表關聯
- **值域完整性**：group_name和project_title必須填入，status限制為三種狀態
- **用戶定義完整性**：同一學年度內組別名稱建議唯一以避免混淆

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| group_id | **主鍵**：表格主要識別碼<br>**自動增長**：系統自動從1開始每次遞增1<br>**非空**：不允許空值 |
| group_name | **非空**：必須填入組別名稱<br>**長度限制**：最長50個字元，例如第一組、智慧家居組 |
| project_title | **非空**：必須填入專題名稱<br>**長度限制**：最長100個字元 |
| teacher_id | **可空值**：允許暫時無指導教師的情況<br>**外鍵**：與教師資料表建立關聯<br>**設為空值**：教師刪除時此欄位設為空值 |
| year_id | **非空**：必須關聯到特定學年度<br>**外鍵**：與學年度資料表建立關聯<br>**限制刪除**：不允許刪除已有組別的學年度 |
| created_at | **預設值**：系統自動記錄組別建立的當前時間<br>**時間格式**：年-月-日 時:分:秒格式 |
| status | **預設值**：新建組別預設為進行中狀態<br>**值域限制**：僅允許進行中、已完成、已暫停三種狀態<br>進行中表示專題正在執行，已完成表示專題已結束，已暫停表示專題暫時停止 |

### 相關VIEW設計
**組別進度概覽檢視 (group_progress_overview)**：顯示各組別的整體進度狀況、成員數量和週報完成情況，提供組別管理參考

## 7. 週報資料表 (weekly_reports)

### SQL建立語句
```sql
CREATE TABLE weekly_reports (
    report_id INT PRIMARY KEY AUTO_INCREMENT,
    student_id VARCHAR(20) NOT NULL,
    group_id INT NOT NULL,
    week_number INT NOT NULL,
    year_id INT NOT NULL,
    this_week_work TEXT NOT NULL,
    next_week_plan TEXT NOT NULL,
    submitted_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    status ENUM('submitted', 'late', 'missing') DEFAULT 'submitted',
    FOREIGN KEY (student_id) REFERENCES students(student_id) ON DELETE CASCADE,
    FOREIGN KEY (group_id) REFERENCES groups(group_id) ON DELETE CASCADE,
    FOREIGN KEY (year_id) REFERENCES academic_years(year_id) ON DELETE RESTRICT,
    UNIQUE KEY unique_student_week_year (student_id, week_number, year_id),
    CHECK (week_number BETWEEN 1 AND 18)
);
```

### 完整性限制
- **實體完整性**：report_id作為主鍵，採用自動增長機制從1開始每次遞增1
- **參照完整性**：student_id、group_id、year_id分別與對應資料表建立關聯
- **值域完整性**：week_number限制在1到18之間，status限制為三種狀態
- **用戶定義完整性**：同一學生在同一學年度的同一週次只能提交一份週報

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| report_id | **主鍵**：表格主要識別碼<br>**自動增長**：系統自動從1開始每次遞增1<br>**非空**：不允許空值 |
| student_id | **非空**：必須關聯到特定學生<br>**外鍵**：與學生資料表建立關聯<br>**級聯刪除**：學生刪除時週報記錄同步刪除 |
| group_id | **非空**：必須關聯到特定組別<br>**外鍵**：與組別資料表建立關聯<br>**級聯刪除**：組別刪除時週報記錄同步刪除 |
| week_number | **非空**：必須填入週次<br>**範圍限制**：僅允許1到18之間的整數<br>對應學期的第1週到第18週 |
| year_id | **非空**：必須關聯到特定學年度<br>**外鍵**：與學年度資料表建立關聯<br>**限制刪除**：不允許刪除已有週報的學年度 |
| this_week_work | **非空**：必須填入本週工作內容<br>**文字類型**：支援長篇文字內容 |
| next_week_plan | **非空**：必須填入下週工作計畫<br>**文字類型**：支援長篇文字內容 |
| submitted_at | **預設值**：系統自動記錄週報提交的當前時間<br>**時間格式**：年-月-日 時:分:秒格式 |
| status | **預設值**：提交時預設為已繳交狀態<br>**值域限制**：僅允許已繳交、遲繳、未繳三種狀態<br>已繳交表示按時提交，遲繳表示逾期提交，未繳表示未提交 |

### 相關VIEW設計
**學生週報狀態檢視 (student_report_status)**：顯示學生的週報繳交狀態，包含已繳、未繳、遲繳統計資訊，協助學生了解自己的繳交情況

## 8. 評分資料表 (evaluations)

### SQL建立語句
```sql
CREATE TABLE evaluations (
    evaluation_id INT PRIMARY KEY AUTO_INCREMENT,
    report_id INT NOT NULL UNIQUE,
    teacher_id VARCHAR(20) NOT NULL,
    score INT NOT NULL,
    comments TEXT,
    evaluated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    status ENUM('pending', 'completed', 'revised') DEFAULT 'completed',
    FOREIGN KEY (report_id) REFERENCES weekly_reports(report_id) ON DELETE CASCADE,
    FOREIGN KEY (teacher_id) REFERENCES teachers(teacher_id) ON DELETE RESTRICT,
    CHECK (score BETWEEN 0 AND 100)
);
```

### 完整性限制
- **實體完整性**：evaluation_id作為主鍵，採用自動增長機制從1開始每次遞增1
- **參照完整性**：report_id與週報資料表一對一關聯，teacher_id與教師資料表關聯
- **值域完整性**：score限制在0到100之間，status限制為三種狀態
- **用戶定義完整性**：一份週報只能有一個評分記錄

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| evaluation_id | **主鍵**：表格主要識別碼<br>**自動增長**：系統自動從1開始每次遞增1<br>**非空**：不允許空值 |
| report_id | **非空**：必須關聯到特定週報<br>**唯一性**：一份週報只能有一個評分記錄<br>**外鍵**：與週報資料表建立關聯<br>**級聯刪除**：週報刪除時評分記錄同步刪除 |
| teacher_id | **非空**：必須關聯到評分教師<br>**外鍵**：與教師資料表建立關聯<br>**限制刪除**：不允許刪除已有評分記錄的教師 |
| score | **非空**：必須填入評分<br>**範圍限制**：僅允許0到100之間的整數<br>代表評分百分比 |
| comments | **可空值**：允許不填入評語<br>**文字類型**：支援長篇文字內容供教師填寫評語 |
| evaluated_at | **預設值**：系統自動記錄評分完成的當前時間<br>**時間格式**：年-月-日 時:分:秒格式 |
| status | **預設值**：評分完成時預設為已完成狀態<br>**值域限制**：僅允許待評分、已完成、已修正三種狀態<br>待評分表示尚未評分，已完成表示評分完畢，已修正表示評分已被修改 |

### 相關VIEW設計
**教師評分概覽檢視 (teacher_evaluation_overview)**：顯示教師的評分工作概覽，包含待評分、已評分的週報統計，協助教師掌握評分進度

## 9. 檔案資料表 (files)

### SQL建立語句
```sql
CREATE TABLE files (
    file_id INT PRIMARY KEY AUTO_INCREMENT,
    report_id INT NOT NULL,
    filename VARCHAR(255) NOT NULL,
    file_path VARCHAR(500) NOT NULL,
    file_size INT NOT NULL,
    file_type VARCHAR(100) NOT NULL,
    uploaded_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (report_id) REFERENCES weekly_reports(report_id) ON DELETE CASCADE,
    CHECK (file_size > 0 AND file_size <= 10485760)
);
```

### 完整性限制
- **實體完整性**：file_id作為主鍵，採用自動增長機制從1開始每次遞增1
- **參照完整性**：report_id與週報資料表建立關聯，週報刪除時檔案記錄同步刪除
- **值域完整性**：file_size限制在0到10MB之間，filename和file_path必須填入
- **用戶定義完整性**：檔案大小必須符合系統限制

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| file_id | **主鍵**：表格主要識別碼<br>**自動增長**：系統自動從1開始每次遞增1<br>**非空**：不允許空值 |
| report_id | **非空**：必須關聯到特定週報<br>**外鍵**：與週報資料表建立關聯<br>**級聯刪除**：週報刪除時檔案記錄同步刪除 |
| filename | **非空**：必須填入檔案名稱<br>**長度限制**：最長255個字元<br>記錄原始檔案名稱 |
| file_path | **非空**：必須填入檔案路徑<br>**長度限制**：最長500個字元<br>記錄檔案在伺服器上的儲存位置 |
| file_size | **非空**：必須記錄檔案大小<br>**範圍限制**：檔案大小必須大於0且不超過10485760位元組（10MB）<br>確保系統儲存空間合理使用 |
| file_type | **非空**：必須記錄檔案類型<br>**長度限制**：最長100個字元<br>記錄MIME類型例如application/pdf或image/jpeg |
| uploaded_at | **預設值**：系統自動記錄檔案上傳的當前時間<br>**時間格式**：年-月-日 時:分:秒格式 |

### 相關VIEW設計
**檔案管理檢視 (file_management_view)**：顯示各週報的檔案上傳情況和儲存空間使用統計，協助系統管理

## 10. 通知資料表 (notifications)

### SQL建立語句
```sql
CREATE TABLE notifications (
    notification_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    title VARCHAR(100) NOT NULL,
    content TEXT NOT NULL,
    type ENUM('system', 'evaluation', 'reminder', 'announcement') NOT NULL,
    is_read BOOLEAN DEFAULT FALSE,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE
);
```

### 完整性限制
- **實體完整性**：notification_id作為主鍵，採用自動增長機制從1開始每次遞增1
- **參照完整性**：user_id與用戶資料表建立關聯，用戶刪除時通知記錄同步刪除
- **值域完整性**：type限制為四種通知類型，is_read為布林值
- **用戶定義完整性**：通知內容必須完整填入

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| notification_id | **主鍵**：表格主要識別碼<br>**自動增長**：系統自動從1開始每次遞增1<br>**非空**：不允許空值 |
| user_id | **非空**：必須關聯到特定用戶<br>**外鍵**：與用戶資料表建立關聯<br>**級聯刪除**：用戶刪除時通知記錄同步刪除 |
| title | **非空**：必須填入通知標題<br>**長度限制**：最長100個字元 |
| content | **非空**：必須填入通知內容<br>**文字類型**：支援長篇文字內容 |
| type | **非空**：必須指定通知類型<br>**值域限制**：僅允許系統通知、評分通知、提醒、公告四種類型<br>系統通知表示系統自動產生，評分通知表示評分完成通知，提醒表示期限提醒，公告表示重要訊息 |
| is_read | **預設值**：新通知預設為未讀狀態<br>**布林值**：僅允許真或假兩種值<br>假表示未讀，真表示已讀 |
| created_at | **預設值**：系統自動記錄通知建立的當前時間<br>**時間格式**：年-月-日 時:分:秒格式 |

### 相關VIEW設計
**用戶通知檢視 (user_notification_view)**：顯示用戶的通知狀態和未讀通知數量，協助用戶管理通知訊息

## 11. 審核資料表 (user_approvals)

### SQL建立語句
```sql
CREATE TABLE user_approvals (
    approval_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    admin_id INT,
    request_type ENUM('registration', 'role_change', 'account_recovery') NOT NULL,
    status ENUM('pending', 'approved', 'rejected') DEFAULT 'pending',
    request_data TEXT,
    admin_notes TEXT,
    requested_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    processed_at DATETIME,
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE,
    FOREIGN KEY (admin_id) REFERENCES users(user_id) ON DELETE SET NULL
);
```

### 完整性限制
- **實體完整性**：approval_id作為主鍵，採用自動增長機制從1開始每次遞增1
- **參照完整性**：user_id和admin_id分別與用戶資料表建立關聯
- **值域完整性**：request_type限制為三種申請類型，status限制為三種狀態
- **用戶定義完整性**：申請記錄必須關聯到有效用戶

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| approval_id | **主鍵**：表格主要識別碼<br>**自動增長**：系統自動從1開始每次遞增1<br>**非空**：不允許空值 |
| user_id | **非空**：必須關聯到申請審核的用戶<br>**外鍵**：與用戶資料表建立關聯<br>**級聯刪除**：用戶刪除時審核記錄同步刪除 |
| admin_id | **可空值**：允許尚未指派審核管理員的情況<br>**外鍵**：與用戶資料表建立關聯<br>**設為空值**：管理員刪除時此欄位設為空值 |
| request_type | **非空**：必須指定申請類型<br>**值域限制**：僅允許註冊申請、角色變更、帳號恢復三種類型<br>註冊申請表示新用戶申請，角色變更表示變更用戶權限，帳號恢復表示恢復被停權帳號 |
| status | **預設值**：新申請預設為待審核狀態<br>**值域限制**：僅允許待審核、已核准、已拒絕三種狀態<br>待審核表示等待處理，已核准表示申請通過，已拒絕表示申請被駁回 |
| request_data | **可空值**：允許不附加額外資料<br>**文字類型**：支援長篇文字內容<br>以JSON格式儲存申請相關的詳細資料 |
| admin_notes | **可空值**：允許管理員不填寫備註<br>**文字類型**：支援長篇文字內容供管理員記錄審核意見 |
| requested_at | **預設值**：系統自動記錄申請提交的當前時間<br>**時間格式**：年-月-日 時:分:秒格式 |
| processed_at | **可空值**：允許尚未處理的申請<br>**時間格式**：年-月-日 時:分:秒格式<br>記錄審核處理完成的時間 |

### 相關VIEW設計
**審核管理檢視 (approval_management_view)**：顯示待審核申請列表和審核處理統計，協助管理員進行審核工作

## 12. 系統日誌資料表 (system_logs)

### SQL建立語句
```sql
CREATE TABLE system_logs (
    log_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT,
    action_type ENUM('login', 'logout', 'create', 'update', 'delete', 'view') NOT NULL,
    table_name VARCHAR(50),
    record_id INT,
    description TEXT,
    ip_address VARCHAR(45),
    user_agent VARCHAR(500),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE SET NULL
);
```

### 完整性限制
- **實體完整性**：log_id作為主鍵，採用自動增長機制從1開始每次遞增1
- **參照完整性**：user_id與用戶資料表建立關聯，用戶刪除時此欄位設為空值
- **值域完整性**：action_type限制為六種操作類型，ip_address支援IPv4和IPv6格式
- **用戶定義完整性**：系統操作必須記錄詳細資訊以供審計

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| log_id | **主鍵**：表格主要識別碼<br>**自動增長**：系統自動從1開始每次遞增1<br>**非空**：不允許空值 |
| user_id | **可空值**：允許系統自動操作無關聯用戶的情況<br>**外鍵**：與用戶資料表建立關聯<br>**設為空值**：用戶刪除時此欄位設為空值 |
| action_type | **非空**：必須記錄操作類型<br>**值域限制**：僅允許登入、登出、新增、更新、刪除、檢視六種操作類型 |
| table_name | **可空值**：允許非資料庫操作的情況<br>**長度限制**：最長50個字元<br>記錄操作的資料表名稱 |
| record_id | **可空值**：允許非特定記錄操作的情況<br>記錄操作的特定記錄識別碼 |
| description | **可空值**：允許不填寫操作描述<br>**文字類型**：支援長篇文字內容記錄操作詳細說明 |
| ip_address | **可空值**：允許內部系統操作無IP位址的情況<br>**長度限制**：最長45個字元<br>支援IPv4和IPv6位址格式 |
| user_agent | **可空值**：允許非瀏覽器操作的情況<br>**長度限制**：最長500個字元<br>記錄用戶端瀏覽器或應用程式資訊 |
| created_at | **預設值**：系統自動記錄日誌建立的當前時間<br>**時間格式**：年-月-日 時:分:秒格式 |

### 相關VIEW設計
**系統使用統計檢視 (system_usage_stats)**：顯示系統使用統計，包含用戶活躍度、操作頻率等管理資訊，協助系統監控和優化

## 審核資料表關聯檢討

經檢視審核資料表 (user_approvals) 的關聯設計：

### 現行設計問題分析
1. **管理員權限驗證不足**：admin_id欄位雖然關聯到用戶資料表，但缺乏進一步的權限驗證機制確保該用戶具有管理員身份
2. **審核流程追蹤不完整**：缺少審核過程的詳細記錄，無法追蹤審核決策的完整歷程
3. **業務邏輯約束不足**：未在資料庫層面限制只有管理員角色能執行審核操作

### 建議改善方案
1. **強化權限檢查**：新增約束條件確保admin_id對應的用戶必須具有管理員權限
2. **完善審核記錄**：考慮新增審核歷史表詳細記錄每次審核的過程和理由  
3. **增強完整性控制**：實作觸發器機制確保審核操作符合業務邏輯要求

### 關聯合理性驗證
審核資料表的現行關聯設計基本合理，主要關聯包括：
- **申請用戶關聯**：每個審核記錄都明確關聯到提出申請的用戶
- **審核管理員關聯**：記錄處理審核的管理員身份
- **級聯操作適當**：用戶刪除時審核記錄同步刪除符合業務邏輯
- **空值處理合理**：管理員可為空值適應系統自動審核的情況

### 整體關聯架構評估
整個資料庫的關聯設計呈現良好的層次結構：
- **核心用戶管理**：用戶表作為中心，透過一對一關係擴展為學生和教師
- **學術組織結構**：學年度、組別、學生形成清晰的組織關係
- **業務流程支援**：週報、評分、檔案形成完整的專題管理流程
- **系統管理功能**：通知、審核、日誌提供完善的系統管理支援

所有外鍵關係都設計了適當的級聯操作，確保資料一致性和完整性。
