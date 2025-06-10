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

# 專題進度追蹤系統 - 核心資料庫設計文檔 (第一部分)

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
```

### 完整性限制

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| user_id | **主鍵約束 (PRIMARY KEY)**：確保每筆記錄的唯一識別<br>**自動遞增 (AUTO_INCREMENT)**：從1開始，每次新增記錄自動遞增1<br>**非空約束 (NOT NULL)**：主鍵自動具有非空屬性<br>**資料型別**：INT，範圍1-2147483647 |
| name | **非空約束 (NOT NULL)**：姓名為必填欄位，不允許空值<br>**長度檢查約束 (CHECK)**：姓名至少2個字元<br>**資料型別**：VARCHAR(50)，最多50個UTF-8字元<br>**字元集**：支援中英文、數字、空格，UTF-8編碼 |
| email | **非空約束 (NOT NULL)**：電子郵件為必填欄位<br>**唯一性約束 (UNIQUE)**：系統內電子郵件不可重複<br>**格式檢查約束 (CHECK)**：必須符合正規表達式 `^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$`<br>　• 本地部分：允許英數字、點號(.)、底線(_)、百分號(%)、加號(+)、減號(-)<br>　• 必須包含一個@符號<br>　• 網域部分：允許英數字、點號(.)、減號(-)<br>　• 頂級網域：至少2個英文字母<br>**資料型別**：VARCHAR(100)，最多100個字元 |
| user_type | **非空約束 (NOT NULL)**：使用者類型為必填欄位<br>**列舉約束 (ENUM)**：只能輸入'student'、'teacher'、'admin'三種值<br>**值域完整性**：資料庫層面強制執行角色限制<br>**大小寫敏感**：嚴格匹配列舉值 |
| status | **列舉約束 (ENUM)**：只能輸入'pending'、'active'、'suspended'三種值<br>**預設值約束 (DEFAULT)**：新註冊用戶預設為'pending'狀態<br>**狀態定義**：pending(待審核)、active(已啟用)、suspended(已停權) |
| created_at | **預設值約束 (DEFAULT CURRENT_TIMESTAMP)**：系統自動記錄建立時間<br>**資料型別**：DATETIME，格式YYYY-MM-DD HH:mm:ss<br>**時間範圍**：1000-01-01 00:00:00 到 9999-12-31 23:59:59<br>**時區**：使用資料庫系統時區 |
| updated_at | **預設值約束 (DEFAULT CURRENT_TIMESTAMP)**：建立時設為當前時間<br>**自動更新約束 (ON UPDATE CURRENT_TIMESTAMP)**：每次UPDATE操作時自動更新為當前時間<br>**資料型別**：DATETIME，格式YYYY-MM-DD HH:mm:ss<br>**自動維護**：由資料庫引擎自動管理，無需手動更新 |

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
```

### 完整性限制

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| password_id | **主鍵約束 (PRIMARY KEY)**：密碼記錄的唯一識別碼<br>**自動遞增 (AUTO_INCREMENT)**：從1開始自動遞增<br>**非空約束 (NOT NULL)**：主鍵自動具有非空屬性<br>**資料型別**：INT，範圍1-2147483647 |
| user_id | **非空約束 (NOT NULL)**：必須關聯到特定用戶<br>**外鍵約束 (FOREIGN KEY)**：與users表的user_id建立參照關係<br>**級聯刪除 (ON DELETE CASCADE)**：用戶刪除時密碼記錄同步刪除<br>**參照完整性**：確保每個密碼記錄都對應有效的用戶帳號<br>**資料型別**：INT，必須為正整數 |
| password_hash | **非空約束 (NOT NULL)**：密碼雜湊值為必填欄位<br>**長度檢查約束 (CHECK)**：雜湊值至少32個字元<br>**資料型別**：VARCHAR(255)，最多255個字元<br>**安全性要求**：僅允許雜湊值，禁止明文密碼<br>**字元集**：支援十六進位字元(0-9, a-f, A-F)和Base64字元 |
| created_at | **預設值約束 (DEFAULT CURRENT_TIMESTAMP)**：系統自動記錄密碼建立時間<br>**資料型別**：DATETIME，格式YYYY-MM-DD HH:mm:ss<br>**審計追蹤**：記錄密碼設定或變更的確切時間<br>**不可更新**：密碼記錄建立後此欄位不應被修改 |

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
```

### 完整性限制

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| student_id | **主鍵約束 (PRIMARY KEY)**：學號作為學生記錄的主要識別碼<br>**格式檢查約束 (CHECK)**：必須符合正規表達式 `^[0-9]{8}$`<br>**格式要求**：恰好8個連續數字，如40943212<br>**字元限制**：僅允許數字0-9，不允許英文字母、特殊符號或空格<br>**非空約束 (NOT NULL)**：主鍵自動具有非空屬性<br>**資料型別**：VARCHAR(20)，實際使用8個字元 |
| user_id | **非空約束 (NOT NULL)**：必須關聯到特定用戶帳號<br>**唯一性約束 (UNIQUE)**：一個用戶帳號只能對應一個學生身份<br>**外鍵約束 (FOREIGN KEY)**：與users表的user_id建立參照關係<br>**級聯刪除 (ON DELETE CASCADE)**：用戶帳號刪除時學生資料同步刪除<br>**一對一關係**：確保用戶帳號與學生身份的一對一映射<br>**資料型別**：INT，必須為正整數 |
| department | **非空約束 (NOT NULL)**：就讀科系為必填欄位<br>**資料型別**：VARCHAR(50)，最多50個UTF-8字元<br>**字元集**：支援中英文科系名稱<br>**內容範例**：資訊工程系、電機工程系、資訊管理系等 |
| admission_year | **非空約束 (NOT NULL)**：入學年份為必填欄位<br>**範圍檢查約束 (CHECK)**：入學年份必須在1950年至明年之間<br>**最小值**：1950年，涵蓋歷史資料需求<br>**最大值**：當前年度+1年，允許預先建立下學年資料<br>**資料型別**：YEAR，4位數年份格式(1901-2155)<br>**動態範圍**：使用YEAR(CURDATE())確保範圍隨時間更新 |
| group_id | **可空值 (NULL)**：允許學生暫時未分配組別<br>**外鍵約束 (FOREIGN KEY)**：與groups表的group_id建立參照關係<br>**設為空值 (ON DELETE SET NULL)**：組別刪除時將此欄位設為NULL，保持學生記錄完整<br>**分組管理**：支援動態分組調整，一個學生同時只能屬於一個組別<br>**資料型別**：INT，若不為NULL則必須為正整數 |

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
```

### 完整性限制

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| teacher_id | **主鍵約束 (PRIMARY KEY)**：教師編號作為教師記錄的唯一識別碼<br>**格式檢查約束 (CHECK)**：必須符合正規表達式 `^T[0-9]{6}$`<br>**格式要求**：T開頭 + 6位數字，如T123456<br>**首字元**：必須為大寫英文字母T<br>**後續字元**：必須為6個連續數字(0-9)<br>**總長度**：恰好7個字元<br>**非空約束 (NOT NULL)**：主鍵自動具有非空屬性<br>**資料型別**：VARCHAR(20)，實際使用7個字元 |
| user_id | **非空約束 (NOT NULL)**：必須關聯到特定用戶帳號<br>**唯一性約束 (UNIQUE)**：一個用戶帳號只能對應一個教師身份<br>**外鍵約束 (FOREIGN KEY)**：與users表的user_id建立參照關係<br>**級聯刪除 (ON DELETE CASCADE)**：用戶帳號刪除時教師資料同步刪除<br>**一對一關係**：確保用戶帳號與教師身份的一對一映射<br>**資料型別**：INT，必須為正整數 |
| department | **非空約束 (NOT NULL)**：任職科系為必填欄位<br>**資料型別**：VARCHAR(50)，最多50個UTF-8字元<br>**字元集**：支援中英文科系名稱<br>**內容範例**：資訊工程系、電機工程系、資訊管理系等 |
| title | **非空約束 (NOT NULL)**：職稱為必填欄位<br>**資料型別**：VARCHAR(30)，最多30個UTF-8字元<br>**字元集**：支援中英文職稱<br>**職稱範例**：教授、副教授、助理教授、講師、系主任等 |

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
```

### 完整性限制

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| year_id | **主鍵約束 (PRIMARY KEY)**：學年度記錄的唯一識別碼<br>**自動遞增 (AUTO_INCREMENT)**：從1開始自動遞增<br>**非空約束 (NOT NULL)**：主鍵自動具有非空屬性<br>**資料型別**：INT，範圍1-2147483647 |
| year_name | **非空約束 (NOT NULL)**：學年度名稱為必填欄位<br>**唯一性約束 (UNIQUE)**：學年度名稱不可重複<br>**格式檢查約束 (CHECK)**：必須符合正規表達式 `^[0-9]{4}-[0-9]{4}$`<br>**格式要求**：YYYY-YYYY格式，如2024-2025<br>**前4位**：起始年份，必須為4位數字<br>**第5位**：分隔符號，必須為減號(-)<br>**後4位**：結束年份，必須為4位數字<br>**年份邏輯檢查 (CHECK)**：後一年份必須等於前一年份加1<br>**資料型別**：VARCHAR(20)，實際使用9個字元 |
| start_date | **非空約束 (NOT NULL)**：學年度開始日期為必填欄位<br>**邏輯檢查約束 (CHECK)**：開始日期必須早於結束日期<br>**資料型別**：DATE，格式YYYY-MM-DD<br>**日期範圍**：1000-01-01 到 9999-12-31<br>**建議值**：通常為每年9月1日 |
| end_date | **非空約束 (NOT NULL)**：學年度結束日期為必填欄位<br>**邏輯檢查約束 (CHECK)**：結束日期必須晚於開始日期<br>**資料型別**：DATE，格式YYYY-MM-DD<br>**日期範圍**：1000-01-01 到 9999-12-31<br>**建議值**：通常為隔年6月30日 |
| status | **列舉約束 (ENUM)**：只能輸入'active'、'archived'兩種值<br>**預設值約束 (DEFAULT)**：新建學年度預設為'active'狀態<br>**狀態定義**：active(進行中或未來學年度)、archived(已結束的歷史學年度) |

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

-- 建立同一學年度組別名稱唯一性約束
CREATE UNIQUE INDEX idx_groups_unique_name_year ON groups(group_name, year_id);
```

### 完整性限制

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| group_id | **主鍵約束 (PRIMARY KEY)**：組別記錄的唯一識別碼<br>**自動遞增 (AUTO_INCREMENT)**：從1開始自動遞增<br>**非空約束 (NOT NULL)**：主鍵自動具有非空屬性<br>**資料型別**：INT，範圍1-2147483647 |
| group_name | **非空約束 (NOT NULL)**：組別名稱為必填欄位<br>**長度檢查約束 (CHECK)**：組別名稱至少2個字元<br>**學年度內唯一約束 (UNIQUE INDEX)**：同一學年度內組別名稱不可重複<br>**複合唯一性**：(group_name, year_id)組成複合唯一索引<br>**資料型別**：VARCHAR(50)，最多50個UTF-8字元<br>**字元集**：支援中英文組別名稱 |
| project_title | **非空約束 (NOT NULL)**：專題名稱為必填欄位<br>**長度檢查約束 (CHECK)**：專題名稱至少5個字元<br>**資料型別**：VARCHAR(100)，最多100個UTF-8字元<br>**字元集**：支援中英文混合的專題名稱和描述<br>**內容要求**：應包含專題的主要技術或應用領域描述 |
| teacher_id | **可空值 (NULL)**：允許組別暫時無指導教師<br>**外鍵約束 (FOREIGN KEY)**：與teachers表的teacher_id建立參照關係<br>**設為空值 (ON DELETE SET NULL)**：指導教師刪除時將此欄位設為NULL<br>**格式要求**：若不為NULL，必須符合教師編號格式(T+6位數字)<br>**資料型別**：VARCHAR(20)，實際使用7個字元或NULL |
| year_id | **非空約束 (NOT NULL)**：必須關聯到特定學年度<br>**外鍵約束 (FOREIGN KEY)**：與academic_years表的year_id建立參照關係<br>**限制刪除 (ON DELETE RESTRICT)**：不允許刪除已有組別的學年度<br>**參照完整性**：確保組別必須歸屬於有效的學年度<br>**資料型別**：INT，必須為正整數 |
| created_at | **預設值約束 (DEFAULT CURRENT_TIMESTAMP)**：系統自動記錄組別建立時間<br>**資料型別**：DATETIME，格式YYYY-MM-DD HH:mm:ss<br>**時間範圍**：1000-01-01 00:00:00 到 9999-12-31 23:59:59<br>**不可更新**：組別建立後此欄位不應被修改 |
| status | **列舉約束 (ENUM)**：只能輸入'active'、'completed'、'suspended'三種值<br>**預設值約束 (DEFAULT)**：新建組別預設為'active'狀態<br>**狀態定義**：active(進行中)、completed(已完成)、suspended(已暫停) |

# 專題進度追蹤系統 - 核心資料庫設計文檔 (第二部分)

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
```

### 完整性限制

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| report_id | **主鍵約束 (PRIMARY KEY)**：週報記錄的唯一識別碼<br>**自動遞增 (AUTO_INCREMENT)**：從1開始自動遞增<br>**非空約束 (NOT NULL)**：主鍵自動具有非空屬性<br>**資料型別**：INT，範圍1-2147483647 |
| student_id | **非空約束 (NOT NULL)**：必須關聯到特定學生<br>**外鍵約束 (FOREIGN KEY)**：與students表的student_id建立參照關係<br>**級聯刪除 (ON DELETE CASCADE)**：學生刪除時週報記錄同步刪除<br>**複合唯一約束 (UNIQUE KEY)**：與week_number、year_id組成複合唯一鍵<br>**格式要求**：必須為8位數字格式<br>**資料型別**：VARCHAR(20)，實際使用8個字元 |
| group_id | **非空約束 (NOT NULL)**：必須關聯到特定組別<br>**外鍵約束 (FOREIGN KEY)**：與groups表的group_id建立參照關係<br>**級聯刪除 (ON DELETE CASCADE)**：組別刪除時週報記錄同步刪除<br>**資料型別**：INT，必須為正整數 |
| week_number | **非空約束 (NOT NULL)**：必須填入週次<br>**範圍檢查約束 (CHECK)**：週次必須在1到18之間<br>**複合唯一約束 (UNIQUE KEY)**：與student_id、year_id組成複合唯一鍵<br>**學期制度**：對應18週制學期安排<br>**資料型別**：INT，範圍1-18 |
| year_id | **非空約束 (NOT NULL)**：必須關聯到特定學年度<br>**外鍵約束 (FOREIGN KEY)**：與academic_years表的year_id建立參照關係<br>**限制刪除 (ON DELETE RESTRICT)**：不允許刪除已有週報的學年度<br>**複合唯一約束 (UNIQUE KEY)**：與student_id、week_number組成複合唯一鍵<br>**資料型別**：INT，必須為正整數 |
| this_week_work | **非空約束 (NOT NULL)**：必須填入本週工作內容<br>**長度檢查約束 (CHECK)**：內容至少10個字元<br>**資料型別**：TEXT，最大65,535個字元<br>**字元集**：支援UTF-8編碼，容納中英文混合內容<br>**內容要求**：記錄學生本週實際完成的專題工作項目 |
| next_week_plan | **非空約束 (NOT NULL)**：必須填入下週工作計畫<br>**長度檢查約束 (CHECK)**：內容至少10個字元<br>**資料型別**：TEXT，最大65,535個字元<br>**字元集**：支援UTF-8編碼，容納中英文混合內容<br>**內容要求**：記錄學生下週預計進行的專題工作安排 |
| submitted_at | **預設值約束 (DEFAULT CURRENT_TIMESTAMP)**：系統自動記錄週報提交時間<br>**資料型別**：DATETIME，格式YYYY-MM-DD HH:mm:ss<br>**時間範圍**：1000-01-01 00:00:00 到 9999-12-31 23:59:59<br>**時效管理**：用於判定提交狀態和遲交情況 |
| status | **列舉約束 (ENUM)**：只能輸入'submitted'、'late'、'missing'三種值<br>**預設值約束 (DEFAULT)**：預設為'submitted'狀態<br>**狀態定義**：submitted(已提交)、late(遲交)、missing(未交) |

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
```

### 完整性限制

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| evaluation_id | **主鍵約束 (PRIMARY KEY)**：評分記錄的唯一識別碼<br>**自動遞增 (AUTO_INCREMENT)**：從1開始自動遞增<br>**非空約束 (NOT NULL)**：主鍵自動具有非空屬性<br>**資料型別**：INT，範圍1-2147483647 |
| report_id | **非空約束 (NOT NULL)**：必須關聯到特定週報<br>**唯一性約束 (UNIQUE)**：一份週報只能有一個評分記錄<br>**外鍵約束 (FOREIGN KEY)**：與weekly_reports表的report_id建立參照關係<br>**級聯刪除 (ON DELETE CASCADE)**：週報刪除時評分記錄同步刪除<br>**一對一關係**：確保週報與評分的一對一映射<br>**資料型別**：INT，必須為正整數 |
| teacher_id | **非空約束 (NOT NULL)**：必須關聯到評分教師<br>**外鍵約束 (FOREIGN KEY)**：與teachers表的teacher_id建立參照關係<br>**限制刪除 (ON DELETE RESTRICT)**：不允許刪除已有評分記錄的教師<br>**格式要求**：必須符合教師編號格式(T+6位數字)<br>**資料型別**：VARCHAR(20)，實際使用7個字元 |
| score | **非空約束 (NOT NULL)**：必須填入評分<br>**範圍檢查約束 (CHECK)**：評分必須在0到100之間<br>**評分制度**：採用百分制評分系統<br>**資料型別**：INT，範圍0-100 |
| comments | **可空值 (NULL)**：評語為選填欄位<br>**長度檢查約束 (CHECK)**：若填入評語則至少5個字元<br>**條件約束**：comments IS NULL OR CHAR_LENGTH(comments) >= 5<br>**資料型別**：TEXT，最大65,535個字元<br>**字元集**：支援UTF-8編碼，容納中英文評語 |
| evaluated_at | **預設值約束 (DEFAULT CURRENT_TIMESTAMP)**：系統自動記錄評分時間<br>**資料型別**：DATETIME，格式YYYY-MM-DD HH:mm:ss<br>**時間範圍**：1000-01-01 00:00:00 到 9999-12-31 23:59:59<br>**審計追蹤**：記錄評分完成的確切時間 |
| status | **列舉約束 (ENUM)**：只能輸入'pending'、'completed'兩種值<br>**預設值約束 (DEFAULT)**：預設為'completed'狀態<br>**狀態定義**：pending(待評分)、completed(已完成) |

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
```

### 完整性限制

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| file_id | **主鍵約束 (PRIMARY KEY)**：檔案記錄的唯一識別碼<br>**自動遞增 (AUTO_INCREMENT)**：從1開始自動遞增<br>**非空約束 (NOT NULL)**：主鍵自動具有非空屬性<br>**資料型別**：INT，範圍1-2147483647 |
| report_id | **非空約束 (NOT NULL)**：必須關聯到特定週報<br>**外鍵約束 (FOREIGN KEY)**：與weekly_reports表的report_id建立參照關係<br>**級聯刪除 (ON DELETE CASCADE)**：週報刪除時檔案記錄同步刪除<br>**一對多關係**：一份週報可包含多個檔案<br>**資料型別**：INT，必須為正整數 |
| filename | **非空約束 (NOT NULL)**：必須記錄檔案名稱<br>**非空檢查約束 (CHECK)**：檔案名稱不能為空字串<br>**資料型別**：VARCHAR(255)，最多255個UTF-8字元<br>**字元集**：支援中英文檔案名稱和副檔名<br>**內容要求**：應包含完整檔案名稱和副檔名 |
| file_path | **非空約束 (NOT NULL)**：檔案儲存路徑為必填<br>**資料型別**：VARCHAR(500)，最多500個字元<br>**路徑格式**：記錄檔案在檔案系統中的完整路徑<br>**字元集**：支援各種作業系統的路徑格式 |
| file_size | **非空約束 (NOT NULL)**：檔案大小為必填<br>**正數檢查約束 (CHECK)**：檔案大小必須大於0位元組<br>**大小限制約束 (CHECK)**：檔案大小不超過104,857,600位元組(100MB)<br>**資料型別**：BIGINT，支援大檔案，範圍1-104857600 |
| file_type | **非空約束 (NOT NULL)**：檔案類型描述為必填<br>**資料型別**：VARCHAR(100)，最多100個UTF-8字元<br>**分類用途**：用於檔案管理和權限控制<br>**內容範例**：文件、程式碼、圖片、壓縮檔等 |
| uploaded_at | **預設值約束 (DEFAULT CURRENT_TIMESTAMP)**：系統自動記錄檔案上傳時間<br>**資料型別**：DATETIME，格式YYYY-MM-DD HH:mm:ss<br>**時間範圍**：1000-01-01 00:00:00 到 9999-12-31 23:59:59<br>**檔案追蹤**：記錄檔案上傳的確切時間 |

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
```

### 完整性限制

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| notification_id | **主鍵約束 (PRIMARY KEY)**：通知記錄的唯一識別碼<br>**自動遞增 (AUTO_INCREMENT)**：從1開始自動遞增<br>**非空約束 (NOT NULL)**：主鍵自動具有非空屬性<br>**資料型別**：INT，範圍1-2147483647 |
| user_id | **非空約束 (NOT NULL)**：必須關聯到特定用戶<br>**外鍵約束 (FOREIGN KEY)**：與users表的user_id建立參照關係<br>**級聯刪除 (ON DELETE CASCADE)**：用戶刪除時通知記錄同步刪除<br>**一對多關係**：一個用戶可有多個通知<br>**資料型別**：INT，必須為正整數 |
| title | **非空約束 (NOT NULL)**：必須填入通知標題<br>**非空檢查約束 (CHECK)**：標題不能為空字串<br>**資料型別**：VARCHAR(100)，最多100個UTF-8字元<br>**字元集**：支援中英文通知標題<br>**內容要求**：應簡潔明確地描述通知主題 |
| content | **非空約束 (NOT NULL)**：必須填入通知內容<br>**長度檢查約束 (CHECK)**：內容至少5個字元<br>**資料型別**：TEXT，最大65,535個字元<br>**字元集**：支援UTF-8編碼，容納詳細的通知內容<br>**內容要求**：提供完整的通知詳細資訊 |
| type | **非空約束 (NOT NULL)**：必須指定通知類型<br>**列舉約束 (ENUM)**：只能輸入'system'、'evaluation'、'reminder'、'announcement'四種值<br>**類型定義**：system(系統通知)、evaluation(評分通知)、reminder(提醒通知)、announcement(公告通知)<br>**分類用途**：用於通知的分類顯示和處理 |
| is_read | **布林值約束 (BOOLEAN)**：已讀狀態標記<br>**預設值約束 (DEFAULT FALSE)**：新通知預設為未讀狀態<br>**資料型別**：BOOLEAN，TRUE(已讀)或FALSE(未讀)<br>**狀態管理**：用於追蹤用戶的通知閱讀狀態 |
| created_at | **預設值約束 (DEFAULT CURRENT_TIMESTAMP)**：系統自動記錄通知建立時間<br>**資料型別**：DATETIME，格式YYYY-MM-DD HH:mm:ss<br>**時間範圍**：1000-01-01 00:00:00 到 9999-12-31 23:59:59<br>**時序排列**：用於通知的時間排序顯示 |

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
```

### 完整性限制

| 欄位名稱 | 完整性限制 |
|---------|-----------|
| approval_id | **主鍵約束 (PRIMARY KEY)**：審核記錄的唯一識別碼<br>**自動遞增 (AUTO_INCREMENT)**：從1開始自動遞增<br>**非空約束 (NOT NULL)**：主鍵自動具有非空屬性<br>**資料型別**：INT，範圍1-2147483647 |
| user_id | **非空約束 (NOT NULL)**：必須關聯到申請審核的用戶<br>**外鍵約束 (FOREIGN KEY)**：與users表的user_id建立參照關係<br>**級聯刪除 (ON DELETE CASCADE)**：用戶刪除時審核記錄同步刪除<br>**一對多關係**：一個用戶可有多個審核申請<br>**資料型別**：INT，必須為正整數 |
| admin_id | **可空值 (NULL)**：允許尚未指派審核管理員的情況<br>**外鍵約束 (FOREIGN KEY)**：與users表的user_id建立參照關係<br>**設為空值 (ON DELETE SET NULL)**：管理員刪除時此欄位設為NULL<br>**角色限制**：應為admin類型用戶<br>**資料型別**：INT，若不為NULL則必須為正整數 |
| request_type | **非空約束 (NOT NULL)**：必須指定申請類型<br>**列舉約束 (ENUM)**：只能輸入'registration'、'role_change'、'account_recovery'三種值<br>**類型定義**：registration(註冊申請)、role_change(角色變更)、account_recovery(帳號恢復)<br>**業務邏輯**：用於區分不同類型的審核流程 |
| status | **列舉約束 (ENUM)**：只能輸入'pending'、'approved'、'rejected'三種值<br>**預設值約束 (DEFAULT)**：預設為'pending'待審核狀態<br>**狀態定義**：pending(待審核)、approved(已核准)、rejected(已拒絕)<br>**工作流程**：追蹤審核申請的處理狀態 |
| request_data | **可空值 (NULL)**：申請相關資料為選填<br>**資料型別**：JSON，儲存結構化的申請資料<br>**JSON格式**：支援複雜的申請資料結構<br>**內容範例**：用戶資料、變更內容、申請原因等 |
| admin_notes | **可空值 (NULL)**：管理員備註為選填<br>**資料型別**：TEXT，最大65,535個字元<br>**字元集**：支援UTF-8編碼，容納中英文備註<br>**用途**：記錄審核過程和決策依據 |
| requested_at | **預設值約束 (DEFAULT CURRENT_TIMESTAMP)**：系統自動記錄申請提交時間<br>**資料型別**：DATETIME，格式YYYY-MM-DD HH:mm:ss<br>**時間範圍**：1000-01-01 00:00:00 到 9999-12-31 23:59:59<br>**申請追蹤**：記錄申請的確切提交時間 |
| processed_at | **可空值 (NULL)**：審核處理完成時間為選填<br>**邏輯檢查約束 (CHECK)**：處理時間不能早於申請時間<br>**條件約束**：processed_at IS NULL OR processed_at >= requested_at<br>**資料型別**：DATETIME，格式YYYY-MM-DD HH:mm:ss<br>**處理追蹤**：記錄審核決定的確切時間 |

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
        ip_address REGEXP '^([0-9]{1,3}\.){3}[0-9]{1,3}$|^([0-9a-fA-F]{0,4}:){1,7}[0-9a-fA-F]{0,4}(FOREIGN KEY)**：與students表的student_id建立參照關係<br>**級聯刪除 (ON DELETE CASCADE)**：學生刪除時週報記錄同步刪除<br>**複合唯一約束 (UNIQUE KEY)**：與week_number、year_id組成複合唯一鍵<br>**格式要求**：必須為8位數字格式<br>**資料型別**：VARCHAR(20)，實際使用8個字元 |
| group_id | **非空約束 (NOT NULL)**：必須關聯到特定組別<br>**外鍵約束 (FOREIGN KEY)**：與groups表的group_id建立參照關係<br>**級聯刪除 (ON DELETE CASCADE)**：組別刪除時週報記錄同步刪除<br>**資料型別**：INT，必須為正整數 |
| week_number | **非空約束 (NOT NULL)**：必須填入週次<br>**範圍檢查約束 (CHECK)**：週次必須在1到18之間<br>**複合唯一約束 (UNIQUE KEY)**：與student_id、year_id組成複合唯一鍵<br>**學期制度**：對應18週制學期安排<br>**資料型別**：INT，範圍1-18 |
| year_id | **非空約束 (NOT NULL)**：必須關聯到特定學年度<br>**外鍵約束 (FOREIGN KEY)**：與academic_years表的year_id建立參照關係<br>**限制刪除 (ON DELETE RESTRICT)**：不允許刪除已有週報的學年度<br>**複合唯一約束 (UNIQUE KEY)**：與student_id、week_number組成複合唯一鍵<br>**資料型別**：INT，必須為正整數 |
| this_week_work | **非空約束 (NOT NULL)**：必須填入本週工作內容<br>**長度檢查約束 (CHECK)**：內容至少10個字元<br>**資料型別**：TEXT，最大65,535個字元<br>**字元集**：支援UTF-8編碼，容納中英文混合內容<br>**內容要求**：記錄學生本週實際完成的專題工作項目 |
| next_week_plan | **非空約束 (NOT NULL)**：必須填入下週工作計畫<br>**長度檢查約束 (CHECK)**：內容至少10個字元<br>**資料型別**：TEXT，最大65,535個字元<br>**字元集**：支援UTF-8編碼，容納中英文混合內容<br>**內容要求**：記錄學生下週預計進行的專題工作安排 |
| submitted_at | **預設值約束 (DEFAULT CURRENT_TIMESTAMP)**：系統自動記錄週報提交時間<br>**資料型別**：DATETIME，格式YYYY-MM-DD HH:mm:ss<br>**時間範圍**：1000-01-01 00:00:00 到 9999-12-31 23:59:59<br>**時效管理**：用於判定提交狀態和遲交情況 |
| status | **列舉約束 (ENUM)**：只能輸入'submitted'、'late'、'missing'三種值<br>**預設值約束 (DEFAULT)**：預設為'submitted'狀態<br>**狀態定義**：submitted(已提交)、late(遲交)、missing(未交) |

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
```

