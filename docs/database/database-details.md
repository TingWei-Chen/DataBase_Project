# 專題進度追蹤系統 - 核心資料庫設計文檔

## 資料庫使用者與權限設計

### 權限分離原則
基於最小權限原則和角色分離概念，設計多個資料庫使用者帳號，避免使用單一root帳號進行所有操作，確保系統安全性和操作可追蹤性。

### 資料庫使用者設計

#### 1. 資料庫管理員 (pts_admin)
```sql
CREATE USER 'pts_admin'@'localhost' IDENTIFIED BY 'Admin@2024PTS!';
GRANT ALL PRIVILEGES ON project_tracking_system.* TO 'pts_admin'@'localhost';
GRANT CREATE, DROP, ALTER ON *.* TO 'pts_admin'@'localhost';
FLUSH PRIVILEGES;
```

**權限範圍**：
- 完整資料庫管理權限：可建立、刪除、修改資料表結構
- 資料管理權限：可執行所有資料操作（增刪改查）
- 使用者管理權限：可建立和管理其他資料庫使用者
- 使用時機：系統維護、資料庫結構變更、初始化設定

#### 2. 應用程式主要使用者 (pts_app)
```sql
CREATE USER 'pts_app'@'%' IDENTIFIED BY 'AppUser@2024PTS!';
GRANT SELECT, INSERT, UPDATE, DELETE ON project_tracking_system.* TO 'pts_app'@'%';
REVOKE DROP, ALTER, CREATE, INDEX ON project_tracking_system.* FROM 'pts_app'@'%';
FLUSH PRIVILEGES;
```

**權限範圍**：
- 資料操作權限：可執行基本的增刪改查操作
- 禁止結構變更：不可修改資料表結構、建立索引
- 網路存取：允許從任何主機連接（適合Web應用）
- 使用時機：Web應用程式的主要資料庫連接

#### 3. 教師角色使用者 (pts_teacher)
```sql
CREATE USER 'pts_teacher'@'localhost' IDENTIFIED BY 'Teacher@2024PTS!';
GRANT SELECT ON project_tracking_system.users TO 'pts_teacher'@'localhost';
GRANT SELECT ON project_tracking_system.students TO 'pts_teacher'@'localhost';
GRANT SELECT ON project_tracking_system.groups TO 'pts_teacher'@'localhost';
GRANT SELECT ON project_tracking_system.weekly_reports TO 'pts_teacher'@'localhost';
GRANT SELECT ON project_tracking_system.academic_years TO 'pts_teacher'@'localhost';
GRANT SELECT ON project_tracking_system.files TO 'pts_teacher'@'localhost';
GRANT SELECT, INSERT, UPDATE ON project_tracking_system.evaluations TO 'pts_teacher'@'localhost';
GRANT INSERT ON project_tracking_system.notifications TO 'pts_teacher'@'localhost';
GRANT INSERT ON project_tracking_system.system_logs TO 'pts_teacher'@'localhost';
GRANT UPDATE ON project_tracking_system.groups TO 'pts_teacher'@'localhost';
GRANT UPDATE (group_id) ON project_tracking_system.students TO 'pts_teacher'@'localhost';
FLUSH PRIVILEGES;
```

**權限範圍**：
- 查看權限：可查看學生資料、週報、組別資訊
- 評分權限：可新增、修改評分記錄
- 組別管理權限：可調整組別資訊和學生分組
- 限制範圍：不可修改使用者帳號、密碼等敏感資料

#### 4. 學生角色使用者 (pts_student)
```sql
CREATE USER 'pts_student'@'localhost' IDENTIFIED BY 'Student@2024PTS!';
GRANT SELECT ON project_tracking_system.academic_years TO 'pts_student'@'localhost';
GRANT SELECT ON project_tracking_system.groups TO 'pts_student'@'localhost';
GRANT SELECT, INSERT ON project_tracking_system.weekly_reports TO 'pts_student'@'localhost';
GRANT SELECT, INSERT ON project_tracking_system.files TO 'pts_student'@'localhost';
GRANT SELECT ON project_tracking_system.evaluations TO 'pts_student'@'localhost';
GRANT SELECT, UPDATE (is_read) ON project_tracking_system.notifications TO 'pts_student'@'localhost';
FLUSH PRIVILEGES;
```

**權限範圍**：
- 週報提交權限：可新增週報和上傳檔案
- 查看權限：可查看自己的評分和通知
- 限制查看：僅能查看與自己相關的資料

#### 5. 報表分析使用者 (pts_report)
```sql
CREATE USER 'pts_report'@'localhost' IDENTIFIED BY 'Report@2024PTS!';
GRANT SELECT ON project_tracking_system.* TO 'pts_report'@'localhost';
FLUSH PRIVILEGES;
```

**權限範圍**：
- 唯讀權限：僅可查詢所有資料表
- 禁止修改：不可進行任何資料異動

#### 6. 備份使用者 (pts_backup)
```sql
CREATE USER 'pts_backup'@'localhost' IDENTIFIED BY 'Backup@2024PTS!';
GRANT SELECT, LOCK TABLES, SHOW VIEW, EVENT, TRIGGER ON project_tracking_system.* TO 'pts_backup'@'localhost';
GRANT RELOAD, PROCESS ON *.* TO 'pts_backup'@'localhost';
FLUSH PRIVILEGES;
```

**權限範圍**：
- 備份權限：可執行完整資料庫備份
- 鎖定權限：備份過程中可鎖定資料表確保一致性

### 權限驗證預存程序

#### 學生週報提交驗證
```sql
DELIMITER $$
CREATE PROCEDURE secure_submit_report(
    IN p_student_id VARCHAR(20),
    IN p_this_week_work TEXT,
    IN p_next_week_plan TEXT,
    IN p_week_number INT,
    IN p_year_id INT
)
BEGIN
    DECLARE current_user_id INT;
    DECLARE student_user_id INT;
    
    SELECT user_id INTO current_user_id 
    FROM users 
    WHERE email = SUBSTRING_INDEX(USER(), '@', 1);
    
    SELECT user_id INTO student_user_id 
    FROM students 
    WHERE student_id = p_student_id;
    
    IF current_user_id = student_user_id THEN
        INSERT INTO weekly_reports (student_id, group_id, this_week_work, next_week_plan, week_number, year_id)
        SELECT p_student_id, s.group_id, p_this_week_work, p_next_week_plan, p_week_number, p_year_id
        FROM students s WHERE s.student_id = p_student_id;
        
        INSERT INTO system_logs (user_id, action_type, table_name, description)
        VALUES (current_user_id, 'create', 'weekly_reports', CONCAT('學生提交第', p_week_number, '週週報'));
    ELSE
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = '權限不足：只能提交自己的週報';
    END IF;
END$$
DELIMITER ;
```

#### 教師評分權限驗證
```sql
DELIMITER $$
CREATE PROCEDURE secure_evaluate_report(
    IN p_report_id INT,
    IN p_teacher_id VARCHAR(20),
    IN p_score INT,
    IN p_comments TEXT
)
BEGIN
    DECLARE report_group_teacher VARCHAR(20);
    
    SELECT g.teacher_id INTO report_group_teacher
    FROM weekly_reports wr
    JOIN groups g ON wr.group_id = g.group_id
    WHERE wr.report_id = p_report_id;
    
    IF report_group_teacher = p_teacher_id THEN
        INSERT INTO evaluations (report_id, teacher_id, score, comments)
        VALUES (p_report_id, p_teacher_id, p_score, p_comments);
        
        INSERT INTO notifications (user_id, title, content, type)
        SELECT s.user_id, '週報評分完成', CONCAT('您的週報已獲得評分：', p_score, '分'), 'evaluation'
        FROM weekly_reports wr
        JOIN students s ON wr.student_id = s.student_id
        WHERE wr.report_id = p_report_id;
    ELSE
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = '權限不足：只能評分自己指導組別的週報';
    END IF;
END$$
DELIMITER ;
```

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
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    -- 完整性限制
    CONSTRAINT chk_email_format CHECK (email REGEXP '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$'),
    CONSTRAINT chk_name_length CHECK (CHAR_LENGTH(name) >= 2)
);

-- 建立索引以提升查詢效能
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_type_status ON users(user_type, status);
CREATE INDEX idx_users_created_at ON users(created_at);
```

### 完整性限制

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| user_id | **主鍵 (PRIMARY KEY)**：表格主要識別碼，自動遞增<br>**非空 (NOT NULL)**：不允許空值<br>**唯一性 (UNIQUE)**：系統自動保證唯一性 |
| name | **非空 (NOT NULL)**：必須填入使用者姓名<br>**長度限制**：最長50個字元<br>**檢查約束 (CHECK)**：姓名長度至少2個字元，確保資料合理性 |
| email | **非空 (NOT NULL)**：必須填入電子郵件<br>**唯一性 (UNIQUE)**：系統內不可重複，建立唯一索引<br>**長度限制**：最長100個字元<br>**格式檢查 (CHECK)**：使用正規表達式驗證Email格式，確保包含@符號和有效域名格式 |
| user_type | **非空 (NOT NULL)**：必須指定使用者角色<br>**列舉限制 (ENUM)**：僅允許'student'、'teacher'、'admin'三種角色類型<br>資料庫層面強制執行角色限制 |
| status | **預設值 (DEFAULT)**：新註冊用戶預設為'pending'待審核狀態<br>**列舉限制 (ENUM)**：僅允許'pending'(待審核)、'active'(已啟用)、'suspended'(已停權)三種狀態 |
| created_at | **預設值 (DEFAULT CURRENT_TIMESTAMP)**：系統自動記錄帳號建立時間<br>**非空 (NOT NULL)**：確保每個記錄都有建立時間 |
| updated_at | **預設值與自動更新**：建立時設為當前時間，每次更新時自動更新為當前時間<br>**非空 (NOT NULL)**：確保記錄更新時間的完整性 |

## 2. 密碼資料表 (user_passwords)

### SQL建立語句
```sql
CREATE TABLE user_passwords (
    password_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    
    -- 外鍵約束
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE,
    
    -- 完整性限制
    CONSTRAINT chk_password_hash_length CHECK (CHAR_LENGTH(password_hash) >= 32)
);

-- 建立索引
CREATE INDEX idx_passwords_user_id ON user_passwords(user_id);
```

### 完整性限制

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| password_id | **主鍵 (PRIMARY KEY)**：表格主要識別碼，自動遞增<br>**非空 (NOT NULL)**：不允許空值 |
| user_id | **非空 (NOT NULL)**：必須關聯到特定用戶<br>**外鍵約束 (FOREIGN KEY)**：與users表建立關聯，確保參照完整性<br>**級聯刪除 (ON DELETE CASCADE)**：用戶刪除時密碼記錄同步刪除 |
| password_hash | **非空 (NOT NULL)**：必須儲存加密後的密碼<br>**長度限制**：最長255個字元<br>**檢查約束 (CHECK)**：雜湊值長度至少32個字元，確保安全性 |
| created_at | **預設值 (DEFAULT CURRENT_TIMESTAMP)**：系統自動記錄密碼建立時間<br>**非空 (NOT NULL)**：確保記錄時間完整性 |

## 3. 學生資料表 (students)

### SQL建立語句
```sql
CREATE TABLE students (
    student_id VARCHAR(20) PRIMARY KEY,
    user_id INT NOT NULL UNIQUE,
    department VARCHAR(50) NOT NULL,
    admission_year YEAR NOT NULL,
    group_id INT,
    
    -- 外鍵約束
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE,
    FOREIGN KEY (group_id) REFERENCES groups(group_id) ON DELETE SET NULL,
    
    -- 完整性限制
    CONSTRAINT chk_student_id_format CHECK (student_id REGEXP '^[0-9]{8}$'),
    CONSTRAINT chk_admission_year CHECK (admission_year BETWEEN 1950 AND YEAR(CURDATE()) + 1)
);

-- 建立索引
CREATE INDEX idx_students_department ON students(department);
CREATE INDEX idx_students_admission_year ON students(admission_year);
CREATE INDEX idx_students_group_id ON students(group_id);
```

### 完整性限制

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| student_id | **主鍵 (PRIMARY KEY)**：學號作為主要識別碼<br>**格式檢查 (CHECK)**：必須為8位數字字串，使用正規表達式驗證格式如'40943212'<br>**非空 (NOT NULL)**：不允許空值 |
| user_id | **非空 (NOT NULL)**：必須關聯到特定用戶<br>**唯一性 (UNIQUE)**：一個用戶帳號只能對應一個學生身份<br>**外鍵約束 (FOREIGN KEY)**：與users表建立關聯<br>**級聯刪除 (ON DELETE CASCADE)**：用戶刪除時學生資料同步刪除 |
| department | **非空 (NOT NULL)**：必須填入就讀科系<br>**長度限制**：最長50個字元 |
| admission_year | **非空 (NOT NULL)**：必須填入入學年份<br>**年份格式 (YEAR)**：4位數年份格式<br>**範圍檢查 (CHECK)**：入學年份必須在1950年至明年之間，確保資料合理性 |
| group_id | **可空值 (NULL)**：允許未分配組別的情況<br>**外鍵約束 (FOREIGN KEY)**：與groups表建立關聯<br>**設為空值 (ON DELETE SET NULL)**：組別刪除時此欄位設為空值，保持學生記錄完整 |

## 4. 教師資料表 (teachers)

### SQL建立語句
```sql
CREATE TABLE teachers (
    teacher_id VARCHAR(20) PRIMARY KEY,
    user_id INT NOT NULL UNIQUE,
    department VARCHAR(50) NOT NULL,
    title VARCHAR(30) NOT NULL,
    
    -- 外鍵約束
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE,
    
    -- 完整性限制
    CONSTRAINT chk_teacher_id_format CHECK (teacher_id REGEXP '^T[0-9]{6}$')
);

-- 建立索引
CREATE INDEX idx_teachers_department ON teachers(department);
CREATE INDEX idx_teachers_title ON teachers(title);
```

### 完整性限制

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| teacher_id | **主鍵 (PRIMARY KEY)**：教師編號作為主要識別碼<br>**格式檢查 (CHECK)**：必須為T開頭加6位數字格式，如'T123456'<br>**非空 (NOT NULL)**：不允許空值 |
| user_id | **非空 (NOT NULL)**：必須關聯到特定用戶<br>**唯一性 (UNIQUE)**：一個用戶帳號只能對應一個教師身份<br>**外鍵約束 (FOREIGN KEY)**：與users表建立關聯<br>**級聯刪除 (ON DELETE CASCADE)**：用戶刪除時教師資料同步刪除 |
| department | **非空 (NOT NULL)**：必須填入任職科系<br>**長度限制**：最長50個字元 |
| title | **非空 (NOT NULL)**：必須填入職稱<br>**長度限制**：最長30個字元，如教授、副教授、助理教授、講師 |

## 5. 學年度資料表 (academic_years)

### SQL建立語句
```sql
CREATE TABLE academic_years (
    year_id INT PRIMARY KEY AUTO_INCREMENT,
    year_name VARCHAR(20) NOT NULL UNIQUE,
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    status ENUM('active', 'archived') DEFAULT 'active',
    
    -- 完整性限制
    CONSTRAINT chk_year_name_format CHECK (year_name REGEXP '^[0-9]{4}-[0-9]{4}$'),
    CONSTRAINT chk_end_after_start CHECK (end_date > start_date),
    CONSTRAINT chk_year_span CHECK (
        CAST(SUBSTRING(year_name, 6, 4) AS UNSIGNED) = 
        CAST(SUBSTRING(year_name, 1, 4) AS UNSIGNED) + 1
    )
);

-- 建立索引
CREATE INDEX idx_academic_years_status ON academic_years(status);
CREATE INDEX idx_academic_years_dates ON academic_years(start_date, end_date);
```

### 完整性限制

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| year_id | **主鍵 (PRIMARY KEY)**：表格主要識別碼，自動遞增<br>**非空 (NOT NULL)**：不允許空值 |
| year_name | **非空 (NOT NULL)**：必須填入學年度名稱<br>**唯一性 (UNIQUE)**：學年度名稱不可重複<br>**格式檢查 (CHECK)**：必須為'YYYY-YYYY'格式，如'2024-2025'<br>**年份邏輯檢查 (CHECK)**：後一年份必須等於前一年份加1<br>**長度限制**：最長20個字元 |
| start_date | **非空 (NOT NULL)**：必須填入學年度開始日期<br>**日期格式 (DATE)**：年-月-日格式 |
| end_date | **非空 (NOT NULL)**：必須填入學年度結束日期<br>**日期格式 (DATE)**：年-月-日格式<br>**邏輯檢查 (CHECK)**：結束日期必須晚於開始日期 |
| status | **列舉限制 (ENUM)**：僅允許'active'(進行中)、'archived'(已歸檔)兩種狀態<br>**預設值 (DEFAULT)**：新建學年度預設為進行中狀態 |

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
    
    -- 外鍵約束
    FOREIGN KEY (teacher_id) REFERENCES teachers(teacher_id) ON DELETE SET NULL,
    FOREIGN KEY (year_id) REFERENCES academic_years(year_id) ON DELETE RESTRICT,
    
    -- 完整性限制
    CONSTRAINT chk_group_name_length CHECK (CHAR_LENGTH(group_name) >= 2),
    CONSTRAINT chk_project_title_length CHECK (CHAR_LENGTH(project_title) >= 5)
);

-- 建立索引
CREATE INDEX idx_groups_teacher_year ON groups(teacher_id, year_id);
CREATE INDEX idx_groups_year_status ON groups(year_id, status);
CREATE INDEX idx_groups_created_at ON groups(created_at);

-- 建立同一學年度組別名稱唯一性約束
CREATE UNIQUE INDEX idx_groups_unique_name_year ON groups(group_name, year_id);
```

### 完整性限制

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| group_id | **主鍵 (PRIMARY KEY)**：表格主要識別碼，自動遞增<br>**非空 (NOT NULL)**：不允許空值 |
| group_name | **非空 (NOT NULL)**：必須填入組別名稱<br>**長度限制**：最長50個字元<br>**長度檢查 (CHECK)**：組別名稱至少2個字元<br>**學年度內唯一 (UNIQUE INDEX)**：同一學年度內組別名稱不可重複 |
| project_title | **非空 (NOT NULL)**：必須填入專題名稱<br>**長度限制**：最長100個字元<br>**長度檢查 (CHECK)**：專題名稱至少5個字元，確保描述性 |
| teacher_id | **可空值 (NULL)**：允許暫時無指導教師的情況<br>**外鍵約束 (FOREIGN KEY)**：與teachers表建立關聯<br>**設為空值 (ON DELETE SET NULL)**：教師刪除時此欄位設為空值，保持組別記錄 |
| year_id | **非空 (NOT NULL)**：必須關聯到特定學年度<br>**外鍵約束 (FOREIGN KEY)**：與academic_years表建立關聯<br>**限制刪除 (ON DELETE RESTRICT)**：不允許刪除已有組別的學年度 |
| created_at | **預設值 (DEFAULT CURRENT_TIMESTAMP)**：系統自動記錄組別建立時間<br>**非空 (NOT NULL)**：確保記錄時間完整性 |
| status | **列舉限制 (ENUM)**：僅允許'active'(進行中)、'completed'(已完成)、'suspended'(已暫停)三種狀態<br>**預設值 (DEFAULT)**：新建組別預設為進行中狀態 |

---

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
    
    -- 外鍵約束
    FOREIGN KEY (student_id) REFERENCES students(student_id) ON DELETE CASCADE,
    FOREIGN KEY (group_id) REFERENCES groups(group_id) ON DELETE CASCADE,
    FOREIGN KEY (year_id) REFERENCES academic_years(year_id) ON DELETE RESTRICT,
    
    -- 完整性限制
    CONSTRAINT chk_week_number_range CHECK (week_number BETWEEN 1 AND 18),
    CONSTRAINT chk_work_content_length CHECK (CHAR_LENGTH(this_week_work) >= 10),
    CONSTRAINT chk_plan_content_length CHECK (CHAR_LENGTH(next_week_plan) >= 10),
    
    -- 學生每週每學年度只能有一份週報
    UNIQUE KEY unique_student_week_year (student_id, week_number, year_id)
);

-- 建立索引
CREATE INDEX idx_weekly_reports_student_year ON weekly_reports(student_id, year_id);
CREATE INDEX idx_weekly_reports_group_week ON weekly_reports(group_id, week_number);
CREATE INDEX idx_weekly_reports_status ON weekly_reports(status);
CREATE INDEX idx_weekly_reports_submitted_at ON weekly_reports(submitted_at);
```

### 完整性限制

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| report_id | **主鍵 (PRIMARY KEY)**：表格主要識別碼，自動遞增<br>**非空 (NOT NULL)**：不允許空值 |
| student_id | **非空 (NOT NULL)**：必須關聯到特定學生<br>**外鍵約束 (FOREIGN KEY)**：與students表建立關聯<br>**級聯刪除 (ON DELETE CASCADE)**：學生刪除時週報記錄同步刪除 |
| group_id | **非空 (NOT NULL)**：必須關聯到特定組別<br>**外鍵約束 (FOREIGN KEY)**：與groups表建立關聯<br>**級聯刪除 (ON DELETE CASCADE)**：組別刪除時週報記錄同步刪除 |
| week_number | **非空 (NOT NULL)**：必須填入週次<br>**範圍檢查 (CHECK)**：週次必須在1到18之間，涵蓋完整學期<br>**唯一性約束**：與student_id和year_id組成複合唯一鍵 |
| year_id | **非空 (NOT NULL)**：必須關聯到特定學年度<br>**外鍵約束 (FOREIGN KEY)**：與academic_years表建立關聯<br>**限制刪除 (ON DELETE RESTRICT)**：不允許刪除已有週報的學年度 |
| this_week_work | **非空 (NOT NULL)**：必須填入本週工作內容<br>**文字類型 (TEXT)**：支援長篇文字內容<br>**長度檢查 (CHECK)**：內容至少10個字元，確保有意義的描述 |
| next_week_plan | **非空 (NOT NULL)**：必須填入下週工作計畫<br>**文字類型 (TEXT)**：支援長篇文字內容<br>**長度檢查 (CHECK)**：內容至少10個字元，確保有意義的規劃 |
| submitted_at | **預設值 (DEFAULT CURRENT_TIMESTAMP)**：系統自動記錄週報提交時間<br>**非空 (NOT NULL)**：確保記錄提交時間 |
| status | **列舉限制 (ENUM)**：僅允許'submitted'(已提交)、'late'(遲交)、'missing'(未交)三種狀態<br>**預設值 (DEFAULT)**：預設為已提交狀態 |

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
    status ENUM('pending', 'completed') DEFAULT 'completed',
    
    -- 外鍵約束
    FOREIGN KEY (report_id) REFERENCES weekly_reports(report_id) ON DELETE CASCADE,
    FOREIGN KEY (teacher_id) REFERENCES teachers(teacher_id) ON DELETE RESTRICT,
    
    -- 完整性限制
    CONSTRAINT chk_score_range CHECK (score BETWEEN 0 AND 100),
    CONSTRAINT chk_comments_meaningful CHECK (comments IS NULL OR CHAR_LENGTH(comments) >= 5)
);

-- 建立索引
CREATE INDEX idx_evaluations_teacher_id ON evaluations(teacher_id);
CREATE INDEX idx_evaluations_score ON evaluations(score);
CREATE INDEX idx_evaluations_status ON evaluations(status);
CREATE INDEX idx_evaluations_evaluated_at ON evaluations(evaluated_at);
```

### 完整性限制

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| evaluation_id | **主鍵 (PRIMARY KEY)**：表格主要識別碼，自動遞增<br>**非空 (NOT NULL)**：不允許空值 |
| report_id | **非空 (NOT NULL)**：必須關聯到特定週報<br>**唯一性 (UNIQUE)**：一份週報只能有一個評分記錄<br>**外鍵約束 (FOREIGN KEY)**：與weekly_reports表建立關聯<br>**級聯刪除 (ON DELETE CASCADE)**：週報刪除時評分記錄同步刪除 |
| teacher_id | **非空 (NOT NULL)**：必須關聯到評分教師<br>**外鍵約束 (FOREIGN KEY)**：與teachers表建立關聯<br>**限制刪除 (ON DELETE RESTRICT)**：不允許刪除已有評分記錄的教師 |
| score | **非空 (NOT NULL)**：必須填入評分<br>**範圍檢查 (CHECK)**：評分必須在0到100之間 |
| comments | **可空值 (NULL)**：評語<br>**文字類型 (TEXT)**：支援長篇評語<br>**長度檢查 (CHECK)**：若填入評語則至少5個字元 |
| evaluated_at | **預設值 (DEFAULT CURRENT_TIMESTAMP)**：系統自動記錄評分時間<br>**非空 (NOT NULL)**：確保記錄評分時間 |
| status | **列舉限制 (ENUM)**：僅允許'pending'(待評分)、'completed'(已完成)兩種狀態<br>**預設值 (DEFAULT)**：預設為已完成狀態 |

## 9. 檔案資料表 (files)

### SQL建立語句
```sql
CREATE TABLE files (
    file_id INT PRIMARY KEY AUTO_INCREMENT,
    report_id INT NOT NULL,
    filename VARCHAR(255) NOT NULL,
    file_path VARCHAR(500) NOT NULL,
    file_size BIGINT NOT NULL,
    file_type VARCHAR(100) NOT NULL,
    uploaded_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    
    -- 外鍵約束
    FOREIGN KEY (report_id) REFERENCES weekly_reports(report_id) ON DELETE CASCADE,
    
    -- 完整性限制
    CONSTRAINT chk_file_size_positive CHECK (file_size > 0),
    CONSTRAINT chk_file_size_limit CHECK (file_size <= 104857600), -- 100MB限制
    CONSTRAINT chk_filename_not_empty CHECK (CHAR_LENGTH(filename) > 0)
);

-- 建立索引
CREATE INDEX idx_files_report_id ON files(report_id);
CREATE INDEX idx_files_file_type ON files(file_type);
CREATE INDEX idx_files_uploaded_at ON files(uploaded_at);
```

### 完整性限制

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| file_id | **主鍵 (PRIMARY KEY)**：表格主要識別碼，自動遞增<br>**非空 (NOT NULL)**：不允許空值 |
| report_id | **非空 (NOT NULL)**：必須關聯到特定週報<br>**外鍵約束 (FOREIGN KEY)**：與weekly_reports表建立關聯<br>**級聯刪除 (ON DELETE CASCADE)**：週報刪除時檔案記錄同步刪除 |
| filename | **非空 (NOT NULL)**：必須記錄檔案名稱<br>**長度限制**：最長255個字元<br>**非空檢查 (CHECK)**：檔案名稱不能為空字串 |
| file_path | **非空 (NOT NULL)**：檔案儲存路徑<br>**長度限制**：最長500個字元<br>記錄檔案在檔案系統中的完整路徑 |
| file_size | **非空 (NOT NULL)**：檔案大小（位元組）<br>**大整數類型 (BIGINT)**：支援大檔案<br>**正數檢查 (CHECK)**：檔案大小必須大於0<br>**大小限制 (CHECK)**：檔案大小不超過100MB |
| file_type | **非空 (NOT NULL)**：檔案類型描述<br>**長度限制**：最長100個字元<br>如'文件'、'程式碼'、'圖片'等 |
| uploaded_at | **預設值 (DEFAULT CURRENT_TIMESTAMP)**：系統自動記錄檔案上傳時間<br>**非空 (NOT NULL)**：確保記錄上傳時間 |

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
    
    -- 外鍵約束
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE,
    
    -- 完整性限制
    CONSTRAINT chk_title_not_empty CHECK (CHAR_LENGTH(title) > 0),
    CONSTRAINT chk_content_meaningful CHECK (CHAR_LENGTH(content) >= 5)
);

-- 建立索引
CREATE INDEX idx_notifications_user_id ON notifications(user_id);
CREATE INDEX idx_notifications_type ON notifications(type);
CREATE INDEX idx_notifications_read_status ON notifications(user_id, is_read);
CREATE INDEX idx_notifications_created_at ON notifications(created_at);
```

### 完整性限制

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| notification_id | **主鍵 (PRIMARY KEY)**：表格主要識別碼，自動遞增<br>**非空 (NOT NULL)**：不允許空值 |
| user_id | **非空 (NOT NULL)**：必須關聯到特定用戶<br>**外鍵約束 (FOREIGN KEY)**：與users表建立關聯<br>**級聯刪除 (ON DELETE CASCADE)**：用戶刪除時通知記錄同步刪除 |
| title | **非空 (NOT NULL)**：必須填入通知標題<br>**長度限制**：最長100個字元<br>**非空檢查 (CHECK)**：標題不能為空字串 |
| content | **非空 (NOT NULL)**：必須填入通知內容<br>**文字類型 (TEXT)**：支援長篇通知內容<br>**長度檢查 (CHECK)**：內容至少5個字元，確保有意義的通知 |
| type | **非空 (NOT NULL)**：必須指定通知類型<br>**列舉限制 (ENUM)**：僅允許'system'(系統)、'evaluation'(評分)、'reminder'(提醒)、'announcement'(公告)四種類型 |
| is_read | **布林值 (BOOLEAN)**：已讀狀態<br>**預設值 (DEFAULT FALSE)**：新通知預設為未讀 |
| created_at | **預設值 (DEFAULT CURRENT_TIMESTAMP)**：系統自動記錄通知建立時間<br>**非空 (NOT NULL)**：確保記錄建立時間 |

## 11. 審核資料表 (user_approvals)

### SQL建立語句
```sql
CREATE TABLE user_approvals (
    approval_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    admin_id INT,
    request_type ENUM('registration', 'role_change', 'account_recovery') NOT NULL,
    status ENUM('pending', 'approved', 'rejected') DEFAULT 'pending',
    request_data JSON,
    admin_notes TEXT,
    requested_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    processed_at DATETIME,
    
    -- 外鍵約束
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE,
    FOREIGN KEY (admin_id) REFERENCES users(user_id) ON DELETE SET NULL,
    
    -- 完整性限制
    CONSTRAINT chk_processed_after_requested CHECK (processed_at IS NULL OR processed_at >= requested_at)
);

-- 建立索引
CREATE INDEX idx_approvals_user_id ON user_approvals(user_id);
CREATE INDEX idx_approvals_admin_id ON user_approvals(admin_id);
CREATE INDEX idx_approvals_status ON user_approvals(status);
CREATE INDEX idx_approvals_type ON user_approvals(request_type);
CREATE INDEX idx_approvals_requested_at ON user_approvals(requested_at);
```

### 完整性限制

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| approval_id | **主鍵 (PRIMARY KEY)**：表格主要識別碼，自動遞增<br>**非空 (NOT NULL)**：不允許空值 |
| user_id | **非空 (NOT NULL)**：必須關聯到申請審核的用戶<br>**外鍵約束 (FOREIGN KEY)**：與users表建立關聯<br>**級聯刪除 (ON DELETE CASCADE)**：用戶刪除時審核記錄同步刪除 |
| admin_id | **可空值 (NULL)**：審核管理員，允許尚未指派的情況<br>**外鍵約束 (FOREIGN KEY)**：與users表建立關聯<br>**設為空值 (ON DELETE SET NULL)**：管理員刪除時此欄位設為空值 |
| request_type | **非空 (NOT NULL)**：必須指定申請類型<br>**列舉限制 (ENUM)**：僅允許'registration'(註冊)、'role_change'(角色變更)、'account_recovery'(帳號恢復)三種申請類型 |
| status | **非空 (NOT NULL)**：審核狀態<br>**列舉限制 (ENUM)**：僅允許'pending'(待審核)、'approved'(已核准)、'rejected'(已拒絕)三種狀態<br>**預設值 (DEFAULT)**：預設為待審核狀態 |
| request_data | **可空值 (NULL)**：申請相關資料<br>**JSON格式**：以JSON格式儲存結構化申請資料 |
| admin_notes | **可空值 (NULL)**：管理員備註<br>**文字類型 (TEXT)**：供管理員記錄審核過程和決策依據 |
| requested_at | **預設值 (DEFAULT CURRENT_TIMESTAMP)**：系統自動記錄申請提交時間<br>**非空 (NOT NULL)**：確保記錄申請時間 |
| processed_at | **可空值 (NULL)**：審核處理完成時間<br>**日期時間格式 (DATETIME)**：記錄審核決定的時間<br>**邏輯檢查 (CHECK)**：處理時間不能早於申請時間 |

## 12. 系統日誌資料表 (system_logs)

### SQL建立語句
```sql
CREATE TABLE system_logs (
    log_id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id INT,
    action_type ENUM('login', 'logout', 'create', 'update', 'delete', 'view') NOT NULL,
    table_name VARCHAR(50),
    record_id INT,
    description TEXT,
    ip_address VARCHAR(45),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    
    -- 外鍵約束
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE SET NULL,
    
    -- 完整性限制
    CONSTRAINT chk_ip_address_format CHECK (
        ip_address IS NULL OR 
        ip_address REGEXP '^([0-9]{1,3}\.){3}[0-9]{1,3}$|^([0-9a-fA-F]{0,4}:){1,7}[0-9a-fA-F]{0,4}$'
    )
);

-- 建立索引
CREATE INDEX idx_system_logs_user_id ON system_logs(user_id);
CREATE INDEX idx_system_logs_action_type ON system_logs(action_type);
CREATE INDEX idx_system_logs_table_record ON system_logs(table_name, record_id);
CREATE INDEX idx_system_logs_created_at ON system_logs(created_at);
CREATE INDEX idx_system_logs_ip_address ON system_logs(ip_address);

-- 建立按月分區提升效能（可選）
ALTER TABLE system_logs PARTITION BY RANGE (YEAR(created_at) * 100 + MONTH(created_at)) (
    PARTITION p202401 VALUES LESS THAN (202402),
    PARTITION p202402 VALUES LESS THAN (202403),
    PARTITION p202403 VALUES LESS THAN (202404),
    PARTITION p202404 VALUES LESS THAN (202405),
    PARTITION p202405 VALUES LESS THAN (202406),
    PARTITION p202406 VALUES LESS THAN (202407),
    PARTITION p202407 VALUES LESS THAN (202408),
    PARTITION p202408 VALUES LESS THAN (202409),
    PARTITION p202409 VALUES LESS THAN (202410),
    PARTITION p202410 VALUES LESS THAN (202411),
    PARTITION p202411 VALUES LESS THAN (202412),
    PARTITION p202412 VALUES LESS THAN (202501),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);
```

### 完整性限制

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| log_id | **主鍵 (PRIMARY KEY)**：表格主要識別碼，使用BIGINT支援大量日誌記錄<br>**非空 (NOT NULL)**：不允許空值 |
| user_id | **可空值 (NULL)**：允許系統自動操作無關聯用戶的情況<br>**外鍵約束 (FOREIGN KEY)**：與users表建立關聯<br>**設為空值 (ON DELETE SET NULL)**：用戶刪除時此欄位設為空值，保持日誌完整性 |
| action_type | **非空 (NOT NULL)**：必須記錄操作類型<br>**列舉限制 (ENUM)**：僅允許'login'(登入)、'logout'(登出)、'create'(新增)、'update'(更新)、'delete'(刪除)、'view'(檢視)六種操作類型 |
| table_name | **可空值 (NULL)**：操作的資料表名稱<br>**長度限制**：最長50個字元<br>記錄資料庫操作涉及的表格 |
| record_id | **可空值 (NULL)**：操作的記錄識別碼<br>**整數類型 (INT)**：記錄具體操作的資料記錄ID |
| description | **可空值 (NULL)**：操作描述<br>**文字類型 (TEXT)**：記錄操作的詳細說明 |
| ip_address | **可空值 (NULL)**：客戶端IP位址<br>**長度限制**：最長45個字元<br>**格式檢查 (CHECK)**：使用正規表達式驗證IPv4和IPv6格式 |
| created_at | **預設值 (DEFAULT CURRENT_TIMESTAMP)**：系統自動記錄日誌建立時間<br>**非空 (NOT NULL)**：確保記錄時間完整性 |

---
