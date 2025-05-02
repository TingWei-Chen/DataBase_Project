# 專題題目追蹤系統 - 詳細資料庫設計

本文檔提供專題題目追蹤系統資料庫的完整定義，包含資料表結構、完整性限制和 SQL 建立語句。系統採用 MariaDB 作為資料庫管理系統，使用 UTF-8 編碼以支援中文字元。

## 資料庫創建

```sql
CREATE DATABASE pts_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE pts_db;
```

## 1. 用戶表 (User)

用戶表存儲系統所有用戶的基本資訊，包括學生、教師和管理員。

### 表格結構與完整性限制

| 欄位名稱 | 資料類型 | 說明 | 完整性限制 |
|----------|----------|------|------------|
| user_id | INT | 用戶唯一識別碼 | 主鍵、自動增長 |
| name | VARCHAR(50) | 用戶姓名 | 非空 |
| email | VARCHAR(100) | 電子郵件地址 | 非空、唯一、符合電子郵件格式 |
| password | VARCHAR(100) | 密碼（加密儲存） | 非空、長度至少8個字元 |
| user_type | ENUM | 用戶類型 | 非空、僅可為'student', 'teacher', 'admin'其中一種 |
| create_date | DATETIME | 帳號建立日期 | 非空、預設為當前時間 |
| status | ENUM | 帳號狀態 | 非空、僅可為'pending', 'active', 'inactive'其中一種，預設為'pending' |

### SQL 定義

```sql
CREATE TABLE `user` (
  `user_id` INT AUTO_INCREMENT,
  `name` VARCHAR(50) NOT NULL,
  `email` VARCHAR(100) NOT NULL,
  `password` VARCHAR(100) NOT NULL,
  `user_type` ENUM('student', 'teacher', 'admin') NOT NULL,
  `create_date` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `status` ENUM('pending', 'active', 'inactive') NOT NULL DEFAULT 'pending',
  PRIMARY KEY (`user_id`),
  UNIQUE KEY `UK_email` (`email`),
  CONSTRAINT `CHK_user_password` CHECK (LENGTH(`password`) >= 8)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 範例資料

```sql
-- 管理員
INSERT INTO `user` (`name`, `email`, `password`, `user_type`, `status`) VALUES
('系統管理員', 'admin@nfu.edu.tw', '$2a$10$XEg4G4mGLPp/uVFAzK0KPueH5ZZHCy.JZ1EK4NF9lopvIZWFxOKXO', 'admin', 'active');

-- 教師（密碼為加密後的形式）
INSERT INTO `user` (`name`, `email`, `password`, `user_type`, `status`) VALUES
('張教授', 'chang@nfu.edu.tw', '$2a$10$MIk5qP3QHqI9hPUB4KKdCu7lS/A2xLaUvAcJ.Vx1tBjNtwzN7AcgO', 'teacher', 'active'),
('王教授', 'wang@nfu.edu.tw', '$2a$10$MIk5qP3QHqI9hPUB4KKdCu7lS/A2xLaUvAcJ.Vx1tBjNtwzN7AcgO', 'teacher', 'active');

-- 學生
INSERT INTO `user` (`name`, `email`, `password`, `user_type`, `status`) VALUES
('陳同學', 'chen@mail.nfu.edu.tw', '$2a$10$MIk5qP3QHqI9hPUB4KKdCu7lS/A2xLaUvAcJ.Vx1tBjNtwzN7AcgO', 'student', 'active'),
('林同學', 'lin@mail.nfu.edu.tw', '$2a$10$MIk5qP3QHqI9hPUB4KKdCu7lS/A2xLaUvAcJ.Vx1tBjNtwzN7AcgO', 'student', 'active');
```

## 2. 學年度表 (Academic_Year)

學年度表管理系統中的不同學年度資訊，用於將專題與特定學年關聯。

### 表格結構與完整性限制

| 欄位名稱 | 資料類型 | 說明 | 完整性限制 |
|----------|----------|------|------------|
| year_id | INT | 學年度唯一識別碼 | 主鍵、自動增長 |
| year_name | VARCHAR(20) | 學年度名稱 | 非空、唯一 (例如: "2023-2024") |
| start_date | DATE | 學年度開始日期 | 非空 |
| end_date | DATE | 學年度結束日期 | 非空、必須晚於開始日期 |
| status | ENUM | 學年度狀態 | 非空、僅可為'active', 'inactive'其中一種，預設為'inactive' |

### SQL 定義

```sql
CREATE TABLE `academic_year` (
  `year_id` INT AUTO_INCREMENT,
  `year_name` VARCHAR(20) NOT NULL,
  `start_date` DATE NOT NULL,
  `end_date` DATE NOT NULL,
  `status` ENUM('active', 'inactive') NOT NULL DEFAULT 'inactive',
  PRIMARY KEY (`year_id`),
  UNIQUE KEY `UK_year_name` (`year_name`),
  CONSTRAINT `CHK_year_date` CHECK (`end_date` > `start_date`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 範例資料

```sql
INSERT INTO `academic_year` (`year_name`, `start_date`, `end_date`, `status`) VALUES
('2023-2024', '2023-09-01', '2024-06-30', 'active'),
('2022-2023', '2022-09-01', '2023-06-30', 'inactive');
```

## 3. 教師表 (Teacher)

教師表存儲教師特有的資訊，與用戶表建立關聯。

### 表格結構與完整性限制

| 欄位名稱 | 資料類型 | 說明 | 完整性限制 |
|----------|----------|------|------------|
| teacher_id | VARCHAR(20) | 教師編號 | 主鍵 |
| user_id | INT | 關聯用戶ID | 非空、外鍵參照用戶表、唯一 |
| department | VARCHAR(50) | 所屬科系 | 非空 |
| title | ENUM | 職稱 | 非空、限定為預設職稱之一 |

### SQL 定義

```sql
CREATE TABLE `teacher` (
  `teacher_id` VARCHAR(20) NOT NULL,
  `user_id` INT NOT NULL,
  `department` VARCHAR(50) NOT NULL,
  `title` ENUM('professor', 'associate_professor', 'assistant_professor', 'lecturer') NOT NULL,
  PRIMARY KEY (`teacher_id`),
  UNIQUE KEY `UK_user_id` (`user_id`),
  CONSTRAINT `FK_teacher_user` FOREIGN KEY (`user_id`) REFERENCES `user` (`user_id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 範例資料

```sql
INSERT INTO `teacher` (`teacher_id`, `user_id`, `department`, `title`) VALUES
('T001', 2, '資訊工程系', 'professor'),
('T002', 3, '資訊工程系', 'associate_professor');
```

## 4. 組別表 (Group)

組別表管理專題組別資訊，記錄各組的基本資訊和指導教師。

### 表格結構與完整性限制

| 欄位名稱 | 資料類型 | 說明 | 完整性限制 |
|----------|----------|------|------------|
| group_id | INT | 組別唯一識別碼 | 主鍵、自動增長 |
| group_name | VARCHAR(100) | 組別名稱 | 非空 |
| project_name | VARCHAR(200) | 專題名稱 | 非空 |
| teacher_id | VARCHAR(20) | 指導教師編號 | 非空、外鍵參照教師表 |
| create_date | DATETIME | 建立日期 | 非空、預設為當前時間 |
| academic_year_id | INT | 關聯學年度ID | 非空、外鍵參照學年度表 |

### SQL 定義

```sql
CREATE TABLE `group` (
  `group_id` INT AUTO_INCREMENT,
  `group_name` VARCHAR(100) NOT NULL,
  `project_name` VARCHAR(200) NOT NULL,
  `teacher_id` VARCHAR(20) NOT NULL,
  `create_date` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `academic_year_id` INT NOT NULL,
  PRIMARY KEY (`group_id`),
  UNIQUE KEY `UK_group_teacher` (`group_name`, `teacher_id`),
  CONSTRAINT `FK_group_teacher` FOREIGN KEY (`teacher_id`) REFERENCES `teacher` (`teacher_id`),
  CONSTRAINT `FK_group_academic_year` FOREIGN KEY (`academic_year_id`) REFERENCES `academic_year` (`year_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 範例資料

```sql
INSERT INTO `group` (`group_name`, `project_name`, `teacher_id`, `academic_year_id`) VALUES
('A組', '智慧家居系統開發', 'T001', 1),
('B組', '區塊鏈應用設計', 'T002', 1);
```

## 5. 學生表 (Student)

學生表存儲學生特有的資訊，與用戶表和組別表建立關聯。

### 表格結構與完整性限制

| 欄位名稱 | 資料類型 | 說明 | 完整性限制 |
|----------|----------|------|------------|
| student_id | VARCHAR(20) | 學號 | 主鍵 |
| user_id | INT | 關聯用戶ID | 非空、外鍵參照用戶表、唯一 |
| group_id | INT | 所屬組別ID | 可為空、外鍵參照組別表 |
| department | VARCHAR(50) | 所屬科系 | 非空 |
| year | INT | 入學年份 | 非空、必須為合理年份 |

### SQL 定義

```sql
CREATE TABLE `student` (
  `student_id` VARCHAR(20) NOT NULL,
  `user_id` INT NOT NULL,
  `group_id` INT DEFAULT NULL,
  `department` VARCHAR(50) NOT NULL,
  `year` INT NOT NULL,
  PRIMARY KEY (`student_id`),
  UNIQUE KEY `UK_user_id` (`user_id`),
  KEY `IX_group_id` (`group_id`),
  CONSTRAINT `FK_student_user` FOREIGN KEY (`user_id`) REFERENCES `user` (`user_id`) ON DELETE CASCADE,
  CONSTRAINT `FK_student_group` FOREIGN KEY (`group_id`) REFERENCES `group` (`group_id`) ON DELETE SET NULL,
  CONSTRAINT `CHK_student_year` CHECK (`year` > 1900 AND `year` < 2100)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 範例資料

```sql
INSERT INTO `student` (`student_id`, `user_id`, `group_id`, `department`, `year`) VALUES
('S110001', 4, 1, '資訊工程系', 2022),
('S110002', 5, 1, '資訊工程系', 2022);
```

## 6. 週報表 (WeeklyReport)

週報表記錄學生每週提交的專題進度報告。

### 表格結構與完整性限制

| 欄位名稱 | 資料類型 | 說明 | 完整性限制 |
|----------|----------|------|------------|
| report_id | INT | 週報唯一識別碼 | 主鍵、自動增長 |
| student_id | VARCHAR(20) | 提交學生學號 | 非空、外鍵參照學生表 |
| current_work | TEXT | 本週工作內容 | 非空 |
| next_plan | TEXT | 下週工作計畫 | 非空 |
| submit_date | DATETIME | 提交日期 | 非空、預設為當前時間 |
| week_number | INT | 週次 | 非空、必須在1-52之間 |
| year | INT | 年份 | 非空、必須為合理年份 |

### SQL 定義

```sql
CREATE TABLE `weekly_report` (
  `report_id` INT AUTO_INCREMENT,
  `student_id` VARCHAR(20) NOT NULL,
  `current_work` TEXT NOT NULL,
  `next_plan` TEXT NOT NULL,
  `submit_date` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `week_number` INT NOT NULL,
  `year` INT NOT NULL,
  PRIMARY KEY (`report_id`),
  UNIQUE KEY `UK_student_week_year` (`student_id`, `week_number`, `year`),
  CONSTRAINT `FK_report_student` FOREIGN KEY (`student_id`) REFERENCES `student` (`student_id`) ON DELETE CASCADE,
  CONSTRAINT `CHK_report_week` CHECK (`week_number` >= 1 AND `week_number` <= 52),
  CONSTRAINT `CHK_report_year` CHECK (`year` > 1900 AND `year` < 2100)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 範例資料

```sql
INSERT INTO `weekly_report` (`student_id`, `current_work`, `next_plan`, `week_number`, `year`) VALUES
('S110001', '完成了系統需求分析和初步的資料庫設計', '開始進行使用者介面設計和後端API規劃', 1, 2024),
('S110002', '研究了各種智慧家居協議和通訊標準', '開始設計系統架構和選擇適合的技術棧', 1, 2024);
```

## 7. 評分標準表 (Evaluation_Criteria)

評分標準表定義每個學年度的評分項目和權重。

### 表格結構與完整性限制

| 欄位名稱 | 資料類型 | 說明 | 完整性限制 |
|----------|----------|------|------------|
| criteria_id | INT | 評分標準唯一識別碼 | 主鍵、自動增長 |
| name | VARCHAR(100) | 評分項目名稱 | 非空 |
| description | TEXT | 評分項目描述 | 可為空 |
| weight | INT | 權重百分比 | 非空、必須在1-100之間 |
| year_id | INT | 關聯學年度ID | 非空、外鍵參照學年度表 |

### SQL 定義

```sql
CREATE TABLE `evaluation_criteria` (
  `criteria_id` INT AUTO_INCREMENT,
  `name` VARCHAR(100) NOT NULL,
  `description` TEXT,
  `weight` INT NOT NULL,
  `year_id` INT NOT NULL,
  PRIMARY KEY (`criteria_id`),
  KEY `IX_year_id` (`year_id`),
  CONSTRAINT `FK_criteria_year` FOREIGN KEY (`year_id`) REFERENCES `academic_year` (`year_id`) ON DELETE CASCADE,
  CONSTRAINT `CHK_criteria_weight` CHECK (`weight` >= 1 AND `weight` <= 100)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 範例資料

```sql
INSERT INTO `evaluation_criteria` (`name`, `description`, `weight`, `year_id`) VALUES
('內容完整性', '週報內容是否完整地描述了本週工作和下週計畫', 40, 1),
('技術深度', '對技術問題的理解和處理深度', 30, 1),
('進度達成率', '本週計畫的完成程度', 30, 1);
```

## 8. 評分表 (Evaluation)

評分表記錄教師對學生週報的評分和評語。

### 表格結構與完整性限制

| 欄位名稱 | 資料類型 | 說明 | 完整性限制 |
|----------|----------|------|------------|
| evaluation_id | INT | 評分唯一識別碼 | 主鍵、自動增長 |
| report_id | INT | 關聯週報ID | 非空、外鍵參照週報表、唯一 |
| teacher_id | VARCHAR(20) | 評分教師編號 | 非空、外鍵參照教師表 |
| score | INT | 評分分數 | 非空、必須在0-100之間 |
| comment | TEXT | 評語 | 可為空 |
| evaluate_date | DATETIME | 評分日期 | 非空、預設為當前時間 |

### SQL 定義

```sql
CREATE TABLE `evaluation` (
  `evaluation_id` INT AUTO_INCREMENT,
  `report_id` INT NOT NULL,
  `teacher_id` VARCHAR(20) NOT NULL,
  `score` INT NOT NULL,
  `comment` TEXT,
  `evaluate_date` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`evaluation_id`),
  UNIQUE KEY `UK_report_id` (`report_id`),
  KEY `IX_teacher_id` (`teacher_id`),
  CONSTRAINT `FK_evaluation_report` FOREIGN KEY (`report_id`) REFERENCES `weekly_report` (`report_id`) ON DELETE CASCADE,
  CONSTRAINT `FK_evaluation_teacher` FOREIGN KEY (`teacher_id`) REFERENCES `teacher` (`teacher_id`) ON DELETE CASCADE,
  CONSTRAINT `CHK_evaluation_score` CHECK (`score` >= 0 AND `score` <= 100)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 範例資料

```sql
INSERT INTO `evaluation` (`report_id`, `teacher_id`, `score`, `comment`) VALUES
(1, 'T001', 85, '內容完整，但技術細節可以更豐富'),
(2, 'T001', 90, '研究充分，下週計畫明確可行');
```

## 9. 評分項目表 (Evaluation_Item)

評分項目表記錄具體評分標準的得分和評語。

### 表格結構與完整性限制

| 欄位名稱 | 資料類型 | 說明 | 完整性限制 |
|----------|----------|------|------------|
| item_id | INT | 評分項目唯一識別碼 | 主鍵、自動增長 |
| evaluation_id | INT | 關聯評分ID | 非空、外鍵參照評分表 |
| criteria_id | INT | 關聯評分標準ID | 非空、外鍵參照評分標準表 |
| score | INT | 該項目的得分 | 非空、必須在0-100之間 |
| comment | TEXT | 該項目的具體評語 | 可為空 |

### SQL 定義

```sql
CREATE TABLE `evaluation_item` (
  `item_id` INT AUTO_INCREMENT,
  `evaluation_id` INT NOT NULL,
  `criteria_id` INT NOT NULL,
  `score` INT NOT NULL,
  `comment` TEXT,
  PRIMARY KEY (`item_id`),
  UNIQUE KEY `UK_evaluation_criteria` (`evaluation_id`, `criteria_id`),
  KEY `IX_criteria_id` (`criteria_id`),
  CONSTRAINT `FK_item_evaluation` FOREIGN KEY (`evaluation_id`) REFERENCES `evaluation` (`evaluation_id`) ON DELETE CASCADE,
  CONSTRAINT `FK_item_criteria` FOREIGN KEY (`criteria_id`) REFERENCES `evaluation_criteria` (`criteria_id`),
  CONSTRAINT `CHK_item_score` CHECK (`score` >= 0 AND `score` <= 100)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 範例資料

```sql
INSERT INTO `evaluation_item` (`evaluation_id`, `criteria_id`, `score`, `comment`) VALUES
(1, 1, 90, '週報內容詳盡'),
(1, 2, 80, '技術分析可再深入'),
(1, 3, 85, '進度符合預期'),
(2, 1, 95, '內容非常完整'),
(2, 2, 85, '技術研究良好'),
(2, 3, 90, '進度超前');
```

## 10. 檔案表 (File)

檔案表管理系統中的所有上傳檔案資訊。

### 表格結構與完整性限制

| 欄位名稱 | 資料類型 | 說明 | 完整性限制 |
|----------|----------|------|------------|
| file_id | INT | 檔案唯一識別碼 | 主鍵、自動增長 |
| report_id | INT | 關聯週報ID | 可為空、外鍵參照週報表 |
| file_name | VARCHAR(200) | 檔案名稱 | 非空 |
| file_path | VARCHAR(500) | 檔案路徑 | 非空 |
| file_size | INT | 檔案大小 | 非空、必須大於0且小於系統限制 |
| file_type | VARCHAR(50) | 檔案類型 | 非空 |
| upload_date | DATETIME | 上傳日期 | 非空、預設為當前時間 |
| reference_type | VARCHAR(20) | 檔案關聯類型 | 非空 |
| reference_id | INT | 關聯對象的ID | 非空 |

### SQL 定義

```sql
CREATE TABLE `file` (
  `file_id` INT AUTO_INCREMENT,
  `report_id` INT DEFAULT NULL,
  `file_name` VARCHAR(200) NOT NULL,
  `file_path` VARCHAR(500) NOT NULL,
  `file_size` INT NOT NULL,
  `file_type` VARCHAR(50) NOT NULL,
  `upload_date` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `reference_type` VARCHAR(20) NOT NULL,
  `reference_id` INT NOT NULL,
  PRIMARY KEY (`file_id`),
  KEY `IX_report_id` (`report_id`),
  KEY `IX_reference` (`reference_type`, `reference_id`),
  CONSTRAINT `FK_file_report` FOREIGN KEY (`report_id`) REFERENCES `weekly_report` (`report_id`) ON DELETE CASCADE,
  CONSTRAINT `CHK_file_size` CHECK (`file_size` > 0 AND `file_size` <= 10485760) -- 10MB
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 範例資料

```sql
INSERT INTO `file` (`report_id`, `file_name`, `file_path`, `file_size`, `file_type`, `reference_type`, `reference_id`) VALUES
(1, '系統需求分析文檔.pdf', '/uploads/S110001/doc1.pdf', 1024000, 'application/pdf', 'report', 1),
(2, '智慧家居協議比較表.xlsx', '/uploads/S110002/doc1.xlsx', 512000, 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet', 'report', 2);
```

## 11. 通知表 (Notification)

通知表管理系統中的各種通知資訊。

### 表格結構與完整性限制

| 欄位名稱 | 資料類型 | 說明 | 完整性限制 |
|----------|----------|------|------------|
| notification_id | INT | 通知唯一識別碼 | 主鍵、自動增長 |
| user_id | INT | 接收通知的用戶ID | 非空、外鍵參照用戶表 |
| content | TEXT | 通知內容 | 非空 |
| type | ENUM | 通知類型 | 非空、限定為系統定義的類型 |
| is_read | BOOLEAN | 已讀狀態 | 非空、預設為FALSE |
| create_date | DATETIME | 通知建立時間 | 非空、預設為當前時間 |

### SQL 定義

```sql
CREATE TABLE `notification` (
  `notification_id` INT AUTO_INCREMENT,
  `user_id` INT NOT NULL,
  `content` TEXT NOT NULL,
  `type` ENUM('report_submission', 'evaluation_completed', 'group_change', 'system') NOT NULL,
  `is_read` BOOLEAN NOT NULL DEFAULT FALSE,
  `create_date` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`notification_id`),
  KEY `IX_user_id` (`user_id`),
  KEY `IX_is_read` (`is_read`),
  CONSTRAINT `FK_notification_user` FOREIGN KEY (`user_id`) REFERENCES `user` (`user_id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 範例資料

```sql
INSERT INTO `notification` (`user_id`, `content`, `type`) VALUES
(2, '學生 陳同學 已提交本週週報', 'report_submission'),
(4, '教師 張教授 已評閱您的週報', 'evaluation_completed');
```

## 12. 系統日誌表 (System_Log)

系統日誌表記錄系統中的各種操作記錄，用於審計和故障排除。

### 表格結構與完整性限制

| 欄位名稱 | 資料類型 | 說明 | 完整性限制 |
|----------|----------|------|------------|
| log_id | INT | 日誌唯一識別碼 | 主鍵、自動增長 |
| user_id | INT | 執行操作的用戶ID | 可為空、外鍵參照用戶表 |
| action | VARCHAR(50) | 執行的操作類型 | 非空 |
| entity_type | VARCHAR(50) | 操作對象類型 | 非空 |
| entity_id | INT | 操作對象ID | 可為空 |
| description | TEXT | 詳細描述 | 可為空 |
| ip_address | VARCHAR(50) | 操作來源IP | 可為空 |
| log_time | DATETIME | 操作時間 | 非空、預設為當前時間 |

### SQL 定義

```sql
CREATE TABLE `system_log` (
  `log_id` INT AUTO_INCREMENT,
  `user_id` INT DEFAULT NULL,
  `action` VARCHAR(50) NOT NULL,
  `entity_type` VARCHAR(50) NOT NULL,
  `entity_id` INT DEFAULT NULL,
  `description` TEXT,
  `ip_address` VARCHAR(50),
  `log_time` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`log_id`),
  KEY `IX_user_id` (`user_id`),
  KEY `IX_entity` (`entity_type`, `entity_id`),
  KEY `IX_log_time` (`log_time`),
  CONSTRAINT `FK_log_user` FOREIGN KEY (`user_id`) REFERENCES `user` (`user_id`) ON DELETE SET NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 範例資料

```sql
INSERT INTO `system_log` (`user_id`, `action`, `entity_type`, `entity_id`, `description`, `ip_address`) VALUES
(1, 'create', 'group', 1, '建立新組別：A組', '192.168.1.1'),
(2, 'update', 'report', 1, '評分週報', '192.168.1.2');
```

## 總結

以上是專題題目追蹤系統的完整資料庫定義。系統採用關聯式資料庫模型，通過各表之間的外鍵關係確保資料的完整性和一致性。完整性約束包括主鍵唯一性、外鍵參照完整性和檢查約束等機制，共同保障系統資料的正確性和安全性。

資料庫設計特點：

1. **合理的資料關聯**：通過外鍵關係建立各實體之間的關聯，如用戶-學生/教師、組別-學生、週報-評分等，確保資料間的一致性。

2. **完善的完整性約束**：每個表都實現了實體完整性（主鍵唯一性）、參照完整性（外鍵關係）和值域完整性（檢查約束）。

3. **規範化設計**：遵循資料庫設計的規範化原則，避免資料冗餘，提高資料維護效率。

4. **易於擴展**：系統允許未來新增表格或欄位，以適應系統功能擴展。

5. **安全性考慮**：用戶密碼以加密形式存儲，系統操作皆記錄在日誌表中，便於審計和安全監控。

## 資料庫關係總覽

以下是各表之間的主要關聯關係：

1. **用戶表 (User)**
   - 與學生表 (Student) 是一對一關係
   - 與教師表 (Teacher) 是一對一關係
   - 與通知表 (Notification) 是一對多關係
   - 與系統日誌表 (System_Log) 是一對多關係

2. **學生表 (Student)**
   - 與用戶表 (User) 是一對一關係
   - 與組別表 (Group) 是多對一關係
   - 與週報表 (WeeklyReport) 是一對多關係

3. **教師表 (Teacher)**
   - 與用戶表 (User) 是一對一關係
   - 與組別表 (Group) 是一對多關係
   - 與評分表 (Evaluation) 是一對多關係

4. **組別表 (Group)**
   - 與教師表 (Teacher) 是多對一關係
   - 與學生表 (Student) 是一對多關係
   - 與學年度表 (Academic_Year) 是多對一關係

5. **週報表 (WeeklyReport)**
   - 與學生表 (Student) 是多對一關係
   - 與評分表 (Evaluation) 是一對一關係
   - 與檔案表 (File) 是一對多關係

6. **評分表 (Evaluation)**
   - 與週報表 (WeeklyReport) 是一對一關係
   - 與教師表 (Teacher) 是多對一關係
   - 與評分項目表 (Evaluation_Item) 是一對多關係

7. **評分標準表 (Evaluation_Criteria)**
   - 與學年度表 (Academic_Year) 是多對一關係
   - 與評分項目表 (Evaluation_Item) 是一對多關係

8. **評分項目表 (Evaluation_Item)**
   - 與評分表 (Evaluation) 是多對一關係
   - 與評分標準表 (Evaluation_Criteria) 是多對一關係

## 索引策略

系統在多個表上設置了索引以提升查詢效能：

1. **主鍵索引**：每個表都有主鍵索引，確保主鍵查詢的高效性。

2. **外鍵索引**：所有外鍵欄位都建立了索引，優化關聯查詢性能。

3. **唯一索引**：如用戶表的電子郵件欄位、學年度表的學年名稱等設置了唯一索引。

4. **複合索引**：針對特定的查詢模式，如組別表的組別名稱和教師ID組合、週報表的學生ID、週次和年份組合等設置了複合索引。

5. **功能性索引**：針對系統常用查詢功能，如通知表的is_read欄位、系統日誌表的log_time欄位等設置了功能性索引。

## 資料庫維護建議

為確保系統長期穩定運行，建議進行以下資料庫維護工作：

1. **定期備份**：建立自動備份機制，防止資料丟失。

```sql
-- 備份策略示例（需在資料庫伺服器執行）
mysqldump -u username -p pts_db > pts_db_backup_$(date +%Y%m%d).sql
```

2. **定期優化**：對表進行定期優化，提升系統性能。

```sql
-- 表優化示例
OPTIMIZE TABLE user, student, teacher, `group`, weekly_report, evaluation;
```

3. **監控索引使用情況**：定期檢查索引使用情況，優化查詢性能。

```sql
-- 查看索引使用情況
SHOW INDEX FROM weekly_report;
```

4. **資料清理**：定期清理過期資料，特別是系統日誌和通知。

```sql
-- 日誌清理示例（刪除一年前的日誌）
DELETE FROM system_log WHERE log_time < DATE_SUB(NOW(), INTERVAL 1 YEAR);
```

5. **資料庫參數調整**：根據系統實際使用情況，調整資料庫參數以優化性能。

```sql
-- 檢查當前參數設置
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';
```

## 典型查詢示例

以下是系統常用的一些典型查詢示例：

### 1. 查詢特定組別的所有學生及其提交的週報數量

```sql
SELECT s.student_id, u.name, COUNT(wr.report_id) AS report_count
FROM student s
JOIN user u ON s.user_id = u.user_id
LEFT JOIN weekly_report wr ON s.student_id = wr.student_id
WHERE s.group_id = 1
GROUP BY s.student_id, u.name;
```

### 2. 查詢本週未提交週報的學生

```sql
SELECT s.student_id, u.name, g.group_name
FROM student s
JOIN user u ON s.user_id = u.user_id
JOIN `group` g ON s.group_id = g.group_id
WHERE s.student_id NOT IN (
    SELECT student_id FROM weekly_report 
    WHERE week_number = WEEK(CURDATE()) AND year = YEAR(CURDATE())
);
```

### 3. 查詢教師的評分統計

```sql
SELECT t.teacher_id, u.name, COUNT(e.evaluation_id) AS evaluation_count, AVG(e.score) AS avg_score
FROM teacher t
JOIN user u ON t.user_id = u.user_id
LEFT JOIN evaluation e ON t.teacher_id = e.teacher_id
GROUP BY t.teacher_id, u.name;
```

### 4. 查詢學生的週報評分趨勢

```sql
SELECT wr.week_number, wr.year, e.score
FROM weekly_report wr
JOIN evaluation e ON wr.report_id = e.report_id
WHERE wr.student_id = 'S110001'
ORDER BY wr.year, wr.week_number;
```

### 5. 查詢最近活躍的組別

```sql
SELECT g.group_id, g.group_name, g.project_name, COUNT(wr.report_id) AS activity_count
FROM `group` g
JOIN student s ON g.group_id = s.group_id
JOIN weekly_report wr ON s.student_id = wr.student_id
WHERE wr.submit_date > DATE_SUB(CURDATE(), INTERVAL 30 DAY)
GROUP BY g.group_id, g.group_name, g.project_name
ORDER BY activity_count DESC;
```

## 交易管理

系統在關鍵操作中使用交易來確保資料一致性，如學生週報提交、教師評分等。以下是一些關鍵交易示例：

### 1. 週報提交交易

```sql
START TRANSACTION;

-- 插入週報記錄
INSERT INTO weekly_report (student_id, current_work, next_plan, week_number, year)
VALUES ('S110001', '本週工作內容', '下週計畫內容', 15, 2024);

-- 獲取插入的週報ID
SET @report_id = LAST_INSERT_ID();

-- 上傳相關檔案
INSERT INTO file (report_id, file_name, file_path, file_size, file_type, reference_type, reference_id)
VALUES (@report_id, '週報附件.pdf', '/uploads/S110001/doc15.pdf', 512000, 'application/pdf', 'report', @report_id);

-- 新增通知給指導教師
INSERT INTO notification (user_id, content, type)
SELECT t.user_id, CONCAT('學生 S110001 已提交第15週週報'), 'report_submission'
FROM student s
JOIN `group` g ON s.group_id = g.group_id
JOIN teacher t ON g.teacher_id = t.teacher_id
WHERE s.student_id = 'S110001';

-- 記錄系統日誌
INSERT INTO system_log (user_id, action, entity_type, entity_id, description)
SELECT user_id, 'create', 'report', @report_id, CONCAT('提交第15週週報')
FROM student
WHERE student_id = 'S110001';

COMMIT;
```

### 2. 週報評分交易

```sql
START TRANSACTION;

-- 插入評分記錄
INSERT INTO evaluation (report_id, teacher_id, score, comment)
VALUES (1, 'T001', 85, '內容完整，但技術細節可以更豐富');

-- 獲取插入的評分ID
SET @evaluation_id = LAST_INSERT_ID();

-- 插入各評分項目
INSERT INTO evaluation_item (evaluation_id, criteria_id, score, comment)
VALUES
(@evaluation_id, 1, 90, '週報內容詳盡'),
(@evaluation_id, 2, 80, '技術分析可再深入'),
(@evaluation_id, 3, 85, '進度符合預期');

-- 新增通知給學生
INSERT INTO notification (user_id, content, type)
SELECT s.user_id, CONCAT('您的第', wr.week_number, '週週報已被評分，分數：85'), 'evaluation_completed'
FROM weekly_report wr
JOIN student s ON wr.student_id = s.student_id
WHERE wr.report_id = 1;

-- 記錄系統日誌
INSERT INTO system_log (user_id, action, entity_type, entity_id, description)
VALUES 
((SELECT user_id FROM teacher WHERE teacher_id = 'T001'), 'create', 'evaluation', @evaluation_id, '評分週報');

COMMIT;
```

## 資料庫安全考量

1. **密碼加密**：用戶密碼以加密形式存儲，使用如BCrypt等強加密算法。

2. **權限分離**：根據不同角色（學生、教師、管理員）設置不同的資料庫訪問權限。

3. **SQL注入防護**：所有查詢使用參數化查詢或預處理語句。

4. **敏感資料保護**：限制對敏感個人資訊的訪問。

5. **審計追蹤**：系統日誌表記錄所有關鍵操作，便於追溯和審計。

6. **備份策略**：定期備份資料庫，確保資料安全。

7. **連接加密**：使用SSL/TLS加密資料庫連接。

## 附錄：資料庫設計思考過程

在設計專題題目追蹤系統的資料庫時，我們考慮了以下幾點：

1. **用戶角色區分**：系統需要支持學生、教師和管理員三種角色，每種角色有不同的權限和功能需求。我們選擇在用戶表中設置用戶類型欄位，並為每種角色創建獨立的關聯表，以存儲各角色特有的屬性。

2. **專題組別管理**：專題組別是系統的核心概念，我們設計了組別表來管理專題組別資訊，並通過外鍵關係與教師和學生建立關聯。

3. **週報與評分**：週報是學生定期提交的專題進度報告，教師需要對週報進行評分和評語。我們設計了週報表和評分表，並通過外鍵關係建立一對一的關聯。

4. **評分標準**：為了支持多元化的評分標準，我們設計了評分標準表和評分項目表，允許管理員定義不同的評分項目和權重。

5. **檔案管理**：專題過程中會產生各種檔案，我們設計了檔案表來管理這些檔案，並通過reference_type和reference_id欄位實現多態關聯。

6. **通知系統**：為了及時通知用戶系統中的重要事件，我們設計了通知表，支持不同類型的通知訊息。

7. **系統日誌**：為了記錄系統中的各種操作，便於審計和故障排除，我們設計了系統日誌表。

8. **學年度管理**：為了支持跨學年的專題管理，我們設計了學年度表，並通過外鍵關係與組別表和評分標準表建立關聯。
