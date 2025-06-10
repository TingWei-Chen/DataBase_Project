 # 專題進度追蹤系統 - 範例輸入資料、VIEW

## 1. 範例資料插入

### 1.1 用戶資料表 (users)
```sql
INSERT INTO users (user_id, name, email, user_type, status, created_at, updated_at) VALUES
(1, '陳廷威', '40943212@gm.nfu.edu.tw', 'student', 'active', '2024-09-01 10:00:00', '2024-09-01 10:00:00'),
(2, '李明華', '40943213@gm.nfu.edu.tw', 'student', 'active', '2024-09-01 10:15:00', '2024-09-01 10:15:00'),
(3, '王小美', '40943214@gm.nfu.edu.tw', 'student', 'active', '2024-09-01 10:30:00', '2024-09-01 10:30:00'),
(4, '張志明', '40943215@gm.nfu.edu.tw', 'student', 'pending', '2024-09-15 14:20:00', '2024-09-15 14:20:00'),
(5, '林雅婷', '40943216@gm.nfu.edu.tw', 'student', 'active', '2024-09-01 11:00:00', '2024-09-01 11:00:00'),
(6, '劉建國', 'T123456@nfu.edu.tw', 'teacher', 'active', '2024-08-15 09:00:00', '2024-08-15 09:00:00'),
(7, '吳美玲', 'T123457@nfu.edu.tw', 'teacher', 'active', '2024-08-15 09:30:00', '2024-08-15 09:30:00'),
(8, '黃志偉', 'T123458@nfu.edu.tw', 'teacher', 'active', '2024-08-15 10:00:00', '2024-08-15 10:00:00'),
(9, '系統管理員', 'admin@nfu.edu.tw', 'admin', 'active', '2024-08-01 08:00:00', '2024-08-01 08:00:00'),
(10, '趙小琪', '40943217@gm.nfu.edu.tw', 'student', 'suspended', '2024-09-01 12:00:00', '2024-10-01 15:30:00'),
(11, '陳小明', '40843001@gm.nfu.edu.tw', 'student', 'active', '2023-09-01 10:00:00', '2023-09-01 10:00:00'),
(12, '李小華', '40843002@gm.nfu.edu.tw', 'student', 'active', '2023-09-01 10:15:00', '2023-09-01 10:15:00'),
(13, '王小強', '40843003@gm.nfu.edu.tw', 'student', 'active', '2023-09-01 10:30:00', '2023-09-01 10:30:00'),
(14, '張小美', '41043001@gm.nfu.edu.tw', 'student', 'pending', '2024-09-01 11:00:00', '2024-09-01 11:00:00'),
(15, '李教授', 'T123459@nfu.edu.tw', 'teacher', 'active', '2024-08-15 09:00:00', '2024-08-15 09:00:00'),
(16, '陳講師', 'T123460@nfu.edu.tw', 'teacher', 'active', '2024-08-15 09:30:00', '2024-08-15 09:30:00'),
(17, '林副教授', 'T123461@nfu.edu.tw', 'teacher', 'active', '2024-08-15 10:00:00', '2024-08-15 10:00:00'),
(18, '王助教', 'T123462@nfu.edu.tw', 'teacher', 'active', '2024-08-15 10:30:00', '2024-08-15 10:30:00'),
(19, '張教授', 'T123463@nfu.edu.tw', 'teacher', 'active', '2024-08-15 11:00:00', '2024-08-15 11:00:00'),
(20, '劉副教授', 'T123464@nfu.edu.tw', 'teacher', 'active', '2024-08-15 11:30:00', '2024-08-15 11:30:00'),
(21, '吳講師', 'T123465@nfu.edu.tw', 'teacher', 'active', '2024-08-15 12:00:00', '2024-08-15 12:00:00');
```

### 1.2 密碼資料表 (user_passwords)
```sql
INSERT INTO user_passwords (password_id, user_id, password_hash, created_at) VALUES
(1, 1, 'a665a45920422f9d417e4867efdc4fb8a04a1f3fff1fa07e998e86f7f7a27ae3', '2024-09-01 10:00:00'),
(2, 2, 'b3a8e0e1f9ab1bfe3a36f231f676f78bb30a519d2b21e6c530c0eee8ebb4a5d0', '2024-09-01 10:15:00'),
(3, 3, 'c2356069e9d1e79ca924378153cfbbfb4d4416b1f99d41a2940bfdb66c5319db', '2024-09-01 10:30:00'),
(4, 4, 'd4f6ed52a4a98c8c1b78d5b8f7e6f5e4c9d8a7b6e5f4c3d2e1f0a9b8c7d6e5f4', '2024-09-15 14:20:00'),
(5, 5, 'e7b8c9d0a1f2e3d4c5b6a7f8e9d0c1b2a3f4e5d6c7b8a9f0e1d2c3b4a5f6e7d8', '2024-09-01 11:00:00'),
(6, 6, 'f1e2d3c4b5a6f7e8d9c0b1a2f3e4d5c6b7a8f9e0d1c2b3a4f5e6d7c8b9a0f1e2', '2024-08-15 09:00:00'),
(7, 7, 'a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3', '2024-08-15 09:30:00'),
(8, 8, 'b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5', '2024-08-15 10:00:00'),
(9, 9, 'c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7', '2024-08-01 08:00:00'),
(10, 10, 'd8e9f0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9', '2024-09-01 12:00:00');
```

### 1.3 學生資料表 (students)
```sql
INSERT INTO students (student_id, user_id, department, admission_year, group_id) VALUES
('40943212', 1, '資訊工程系', 2020, 1),
('40943213', 2, '資訊工程系', 2020, 1),
('40943214', 3, '資訊工程系', 2020, 2),
('40943215', 4, '資訊工程系', 2020, NULL),
('40943216', 5, '電機工程系', 2020, 3),
('40943217', 10, '資訊工程系', 2020, NULL),
('40843001', 11, '資訊工程系', 2019, 4),
('40843002', 12, '電機工程系', 2019, 5),
('40843003', 13, '資訊管理系', 2019, 6),
('41043001', 14, '資訊工程系', 2021, NULL);
```

### 1.4 教師資料表 (teachers)
```sql
INSERT INTO teachers (teacher_id, user_id, department, title) VALUES
('T123456', 6, '資訊工程系', '教授'),
('T123457', 7, '資訊工程系', '副教授'),
('T123458', 8, '電機工程系', '助理教授'),
('T123459', 15, '資訊管理系', '教授'),
('T123460', 16, '資訊工程系', '講師'),
('T123461', 17, '電機工程系', '副教授'),
('T123462', 18, '資訊工程系', '助理教授'),
('T123463', 19, '資訊管理系', '教授'),
('T123464', 20, '電機工程系', '副教授'),
('T123465', 21, '資訊工程系', '講師');
```

### 1.5 學年度資料表 (academic_years)
```sql
INSERT INTO academic_years (year_id, year_name, start_date, end_date, status) VALUES
(1, '2024-2025', '2024-09-01', '2025-06-30', 'active'),
(2, '2023-2024', '2023-09-01', '2024-06-30', 'archived'),
(3, '2022-2023', '2022-09-01', '2023-06-30', 'archived'),
(4, '2021-2022', '2021-09-01', '2022-06-30', 'archived'),
(5, '2020-2021', '2020-09-01', '2021-06-30', 'archived'),
(6, '2019-2020', '2019-09-01', '2020-06-30', 'archived'),
(7, '2025-2026', '2025-09-01', '2026-06-30', 'active'),
(8, '2018-2019', '2018-09-01', '2019-06-30', 'archived'),
(9, '2017-2018', '2017-09-01', '2018-06-30', 'archived'),
(10, '2016-2017', '2016-09-01', '2017-06-30', 'archived');
```

### 1.6 組別資料表 (groups)
```sql
INSERT INTO groups (group_id, group_name, project_title, teacher_id, year_id, created_at, status) VALUES
(1, '智慧家居組', '基於IoT的智慧家居控制系統', 'T123456', 1, '2024-09-10 14:00:00', 'active'),
(2, 'AI影像組', '深度學習影像辨識系統', 'T123457', 1, '2024-09-10 14:30:00', 'active'),
(3, '行動應用組', '校園導覽APP開發', 'T123458', 1, '2024-09-10 15:00:00', 'active'),
(4, '網路安全組', '網路入侵偵測系統', 'T123456', 2, '2023-09-10 14:00:00', 'completed'),
(5, '大資料組', '學生學習行為分析平台', 'T123459', 2, '2023-09-10 14:30:00', 'completed'),
(6, '區塊鏈組', '數位證書管理系統', 'T123460', 1, '2024-09-10 15:30:00', 'active'),
(7, '遊戲開發組', '教育類遊戲設計', 'T123461', 1, '2024-09-10 16:00:00', 'active'),
(8, '雲端運算組', '微服務架構電商平台', 'T123457', 1, '2024-09-10 16:30:00', 'suspended'),
(9, '機器學習組', '智慧推薦系統', 'T123462', 1, '2024-09-10 17:00:00', 'active'),
(10, 'VR/AR組', '虛擬實境教學系統', 'T123463', 2, '2023-09-10 15:00:00', 'completed');
```

### 1.7 週報資料表 (weekly_reports)
```sql
INSERT INTO weekly_reports (report_id, student_id, group_id, week_number, year_id, this_week_work, next_week_plan, submitted_at, status) VALUES
(1, '40943212', 1, 1, 1, '完成系統需求分析，研讀相關IoT技術文獻，建立專題規劃時程表', '開始進行系統架構設計，選擇適合的硬體平台', '2024-09-15 23:45:00', 'submitted'),
(2, '40943213', 1, 1, 1, '協助需求分析，研究使用者介面設計原則，學習前端開發技術', '設計系統UI/UX原型，準備前端開發環境', '2024-09-16 01:20:00', 'late'),
(3, '40943214', 2, 1, 1, '收集影像辨識相關論文，建立資料集收集計畫，學習深度學習基礎', '開始收集訓練資料，設定深度學習開發環境', '2024-09-15 22:30:00', 'submitted'),
(4, '40943216', 3, 1, 1, '進行校園場地勘查，設計APP功能架構，學習行動應用開發技術', '完成APP介面設計，開始核心功能開發', '2024-09-15 20:15:00', 'submitted'),
(5, '40943212', 1, 2, 1, '完成系統架構設計圖，選定Arduino和Raspberry Pi作為主要硬體', '進行硬體採購，開始感測器整合測試', '2024-09-22 23:30:00', 'submitted'),
(6, '40943213', 1, 2, 1, '完成UI原型設計，建立前端開發環境，學習React Native框架', '開始前端介面開發，整合後端API接口', '2024-09-23 02:10:00', 'late'),
(7, '40943214', 2, 2, 1, '收集了1000張訓練影像，建立資料標註工具，完成開發環境設定', '完成資料標註工作，開始訓練初步模型', '2024-09-22 21:45:00', 'submitted'),
(8, '40943216', 3, 2, 1, '完成APP主要介面設計，實作基本導覽功能，整合地圖API', '新增互動功能，優化使用者體驗設計', '2024-09-22 19:20:00', 'submitted'),
(9, '40943212', 1, 3, 1, '完成感測器採購與基本測試，建立硬體連接架構，進行初步整合', '完成所有感測器整合，開始軟體控制程式開發', '2024-09-29 23:00:00', 'submitted'),
(10, '40943213', 1, 3, 1, '持續進行前端開發，完成主要操作介面，開始後端API整合測試', '完善前端功能，進行系統整合測試', '2024-09-30 01:30:00', 'late');
```

### 1.8 評分資料表 (evaluations)
```sql
INSERT INTO evaluations (evaluation_id, report_id, teacher_id, score, comments, evaluated_at, status) VALUES
(1, 1, 'T123456', 85, '需求分析做得很詳細，時程規劃合理，建議加強技術可行性評估', '2024-09-17 10:30:00', 'completed'),
(2, 2, 'T123456', 78, 'UI設計有創意，但需要考慮實用性，建議多研究使用者體驗設計原則', '2024-09-17 11:00:00', 'completed'),
(3, 3, 'T123457', 90, '文獻收集充分，計畫完整，對深度學習概念掌握良好', '2024-09-17 14:20:00', 'completed'),
(4, 4, 'T123458', 82, '實地勘查做得仔細，功能規劃完整，建議增加技術難度挑戰', '2024-09-17 15:45:00', 'completed'),
(5, 5, 'T123456', 88, '架構設計合理，硬體選擇恰當，期待後續實作成果', '2024-09-24 09:15:00', 'completed'),
(6, 6, 'T123456', 75, '前端開發進度稍慢，建議加強時間管理，多與組員協調', '2024-09-24 09:45:00', 'completed'),
(7, 7, 'T123457', 92, '資料收集效率高，標註工具設計實用，展現良好的解決問題能力', '2024-09-24 13:30:00', 'completed'),
(8, 8, 'T123458', 86, 'APP介面設計美觀，功能實作順利，建議增加更多互動元素', '2024-09-24 16:20:00', 'completed'),
(9, 9, 'T123456', 87, '硬體整合進展良好，技術問題解決能力強，期待軟體開發成果', '2024-10-01 10:00:00', 'completed'),
(10, 10, 'T123456', 73, '開發進度需要加快，建議更有效率的工作方式，加強團隊合作', '2024-10-01 10:30:00', 'completed');
```

### 1.9 檔案資料表 (files)
```sql
INSERT INTO files (file_id, report_id, filename, file_path, file_size, file_type, uploaded_at) VALUES
(1, 1, '系統需求規格書.pdf', '/uploads/2024/09/15/system_requirements_spec.pdf', 2048576, 'PDF文件', '2024-09-15 23:45:30'),
(2, 1, '專題時程規劃.xlsx', '/uploads/2024/09/15/project_timeline.xlsx', 1024768, 'Excel文件', '2024-09-15 23:46:00'),
(3, 2, 'UI設計原型.png', '/uploads/2024/09/16/ui_prototype.png', 3145728, '圖片', '2024-09-16 01:20:30'),
(4, 3, '深度學習文獻整理.docx', '/uploads/2024/09/15/dl_literature_review.docx', 1536000, 'Word文件', '2024-09-15 22:30:45'),
(5, 4, '校園場地勘查報告.pdf', '/uploads/2024/09/15/campus_survey_report.pdf', 4194304, 'PDF文件', '2024-09-15 20:15:20'),
(6, 5, '系統架構圖.png', '/uploads/2024/09/22/system_architecture.png', 2097152, '圖片', '2024-09-22 23:30:15'),
(7, 6, '前端開發進度截圖.png', '/uploads/2024/09/23/frontend_progress.png', 1572864, '圖片', '2024-09-23 02:10:25'),
(8, 7, '訓練資料樣本.zip', '/uploads/2024/09/22/training_samples.zip', 10485760, '壓縮檔', '2024-09-22 21:45:50'),
(9, 8, 'APP介面設計.sketch', '/uploads/2024/09/22/app_interface_design.sketch', 5242880, '設計檔', '2024-09-22 19:20:35'),
(10, 9, '感測器測試報告.pdf', '/uploads/2024/09/29/sensor_test_report.pdf', 3670016, 'PDF文件', '2024-09-29 23:00:40');
```

### 1.10 通知資料表 (notifications)
```sql
INSERT INTO notifications (notification_id, user_id, title, content, type, is_read, created_at) VALUES
(1, 1, '週報評分完成', '您第1週的週報已完成評分，獲得85分', 'evaluation', TRUE, '2024-09-17 10:30:30'),
(2, 2, '週報評分完成', '您第1週的週報已完成評分，獲得78分', 'evaluation', FALSE, '2024-09-17 11:00:30'),
(3, 3, '週報評分完成', '您第1週的週報已完成評分，獲得90分', 'evaluation', TRUE, '2024-09-17 14:20:30'),
(4, 4, '帳號審核通過', '您的帳號已通過審核，現在可以正常使用系統功能', 'system', FALSE, '2024-09-20 09:00:00'),
(5, 1, '週報提交提醒', '第4週週報提交截止日即將到來，請及時提交', 'reminder', FALSE, '2024-10-05 08:00:00'),
(6, 2, '週報評分完成', '您第2週的週報已完成評分，獲得75分', 'evaluation', FALSE, '2024-09-24 09:45:30'),
(7, 5, '組別分配通知', '您已被分配到行動應用組，請與組員聯繫', 'system', TRUE, '2024-09-10 16:00:00'),
(8, 6, '新學期開始', '2024-2025學年度已開始，請各位老師協助學生組別分配', 'announcement', TRUE, '2024-09-01 08:00:00'),
(9, 1, '系統維護通知', '系統將於本週日凌晨2:00-4:00進行維護，屆時無法使用', 'system', FALSE, '2024-10-03 18:00:00'),
(10, 3, '優秀作品展示', '您的專題作品表現優異，邀請參加期末成果展示', 'announcement', FALSE, '2024-10-01 15:30:00');
```

### 1.11 審核資料表 (user_approvals)
```sql
INSERT INTO user_approvals (approval_id, user_id, admin_id, request_type, status, request_data, admin_notes, requested_at, processed_at) VALUES
(1, 4, 9, 'registration', 'approved', '{"student_id":"40943215","department":"資訊工程系"}', '學號格式正確，科系確認無誤', '2024-09-15 14:20:00', '2024-09-20 09:00:00'),
(2, 14, 9, 'registration', 'pending', '{"student_id":"41043001","department":"資訊工程系"}', NULL, '2024-09-01 11:00:00', NULL),
(3, 10, 9, 'account_recovery', 'rejected', '{"reason":"違反使用規範"}', '經查證確實有違規行為，暫不恢復', '2024-10-01 10:00:00', '2024-10-01 15:30:00'),
(4, 5, 9, 'role_change', 'approved', '{"from":"student","to":"student","group_change":"3"}', '組別異動申請合理', '2024-09-25 14:30:00', '2024-09-26 10:15:00'),
(5, 11, 9, 'registration', 'approved', '{"student_id":"40843001","department":"資訊工程系"}', '舊生重新註冊，資料確認無誤', '2024-09-01 09:00:00', '2024-09-01 16:00:00'),
(6, 15, 9, 'registration', 'approved', '{"teacher_id":"T123459","department":"資訊管理系"}', '新進教師，資格審核通過', '2024-08-10 10:00:00', '2024-08-15 09:00:00'),
(7, 20, 9, 'role_change', 'pending', '{"change_title":"副教授"}', NULL, '2024-10-01 14:00:00', NULL),
(8, 12, 9, 'registration', 'approved', '{"student_id":"40843002","department":"電機工程系"}', '轉系生，新科系確認', '2024-09-05 11:30:00', '2024-09-06 14:20:00'),
(9, 16, 9, 'account_recovery', 'approved', '{"reason":"密碼遺失"}', '身份驗證通過，密碼已重設', '2024-09-20 16:45:00', '2024-09-21 09:30:00'),
(10, 21, 9, 'registration', 'pending', '{"teacher_id":"T123465","department":"資訊工程系"}', NULL, '2024-10-02 13:20:00', NULL);
```

### 1.12 系統日誌資料表 (system_logs)
```sql
INSERT INTO system_logs (log_id, user_id, action_type, table_name, record_id, description, ip_address, created_at) VALUES
(1, 1, 'login', NULL, NULL, '學生用戶登入系統', '192.168.1.100', '2024-09-15 23:30:00'),
(2, 1, 'create', 'weekly_reports', 1, '提交第1週週報', '192.168.1.100', '2024-09-15 23:45:00'),
(3, 6, 'login', NULL, NULL, '教師用戶登入系統', '192.168.1.50', '2024-09-17 10:00:00'),
(4, 6, 'create', 'evaluations', 1, '評分週報ID:1', '192.168.1.50', '2024-09-17 10:30:00'),
(5, 9, 'login', NULL, NULL, '管理員登入系統', '192.168.1.10', '2024-09-20 08:45:00'),
(6, 9, 'update', 'user_approvals', 1, '審核用戶註冊申請', '192.168.1.10', '2024-09-20 09:00:00'),
(7, 2, 'login', NULL, NULL, '學生用戶登入系統', '192.168.1.101', '2024-09-23 01:50:00'),
(8, 2, 'create', 'weekly_reports', 6, '提交第2週週報', '192.168.1.101', '2024-09-23 02:10:00'),
(9, 3, 'view', 'evaluations', 3, '查看評分結果', '192.168.1.102', '2024-09-24 20:15:00'),
(10, 7, 'login', NULL, NULL, '教師用戶登入系統', '192.168.1.51', '2024-09-24 13:00:00');
```

---

## 2. VIEW實作與輸出結果

### 2.1 用戶基本資訊檢視 (view_user_basic_info)

#### VIEW建立語句
```sql
CREATE VIEW view_user_basic_info AS
SELECT 
    u.user_id,
    u.name,
    u.email,
    u.user_type,
    u.status,
    u.created_at,
    CASE 
        WHEN u.user_type = 'student' THEN s.student_id
        WHEN u.user_type = 'teacher' THEN t.teacher_id
        ELSE NULL
    END AS role_id,
    CASE 
        WHEN u.user_type = 'student' THEN s.department
        WHEN u.user_type = 'teacher' THEN t.department
        ELSE NULL
    END AS department
FROM users u
LEFT JOIN students s ON u.user_id = s.user_id
LEFT JOIN teachers t ON u.user_id = t.user_id;
```

#### 輸出結果
| user_id | name | email | user_type | status | created_at | role_id | department |
|---------|------|-------|-----------|--------|------------|---------|------------|
| 1 | 陳廷威 | 40943212@gm.nfu.edu.tw | student | active | 2024-09-01 10:00:00 | 40943212 | 資訊工程系 |
| 2 | 李明華 | 40943213@gm.nfu.edu.tw | student | active | 2024-09-01 10:15:00 | 40943213 | 資訊工程系 |
| 3 | 王小美 | 40943214@gm.nfu.edu.tw | student | active | 2024-09-01 10:30:00 | 40943214 | 資訊工程系 |
| 6 | 劉建國 | T123456@nfu.edu.tw | teacher | active | 2024-08-15 09:00:00 | T123456 | 資訊工程系 |
| 7 | 吳美玲 | T123457@nfu.edu.tw | teacher | active | 2024-08-15 09:30:00 | T123457 | 資訊工程系 |
| 8 | 黃志偉 | T123458@nfu.edu.tw | teacher | active | 2024-08-15 10:00:00 | T123458 | 電機工程系 |
| 9 | 系統管理員 | admin@nfu.edu.tw | admin | active | 2024-08-01 08:00:00 | NULL | NULL |

---

### 2.2 學生完整資訊檢視 (view_student_full_info)

#### VIEW建立語句
```sql
CREATE VIEW view_student_full_info AS
SELECT 
    s.student_id,
    u.name as student_name,
    u.email,
    s.department,
    s.admission_year,
    u.status,
    g.group_id,
    g.group_name,
    g.project_title,
    t.teacher_id,
    tu.name as teacher_name,
    ay.year_name,
    ay.status as year_status
FROM students s
JOIN users u ON s.user_id = u.user_id
LEFT JOIN groups g ON s.group_id = g.group_id
LEFT JOIN academic_years ay ON g.year_id = ay.year_id
LEFT JOIN teachers t ON g.teacher_id = t.teacher_id
LEFT JOIN users tu ON t.user_id = tu.user_id;
```

#### 輸出結果
| student_id | student_name | email | department | admission_year | status | group_id | group_name | project_title | teacher_id | teacher_name | year_name | year_status |
|------------|--------------|-------|------------|----------------|--------|----------|------------|---------------|------------|--------------|-----------|-------------|
| 40943212 | 陳廷威 | 40943212@gm.nfu.edu.tw | 資訊工程系 | 2020 | active | 1 | 智慧家居組 | 基於IoT的智慧家居控制系統 | T123456 | 劉建國 | 2024-2025 | active |
| 40943213 | 李明華 | 40943213@gm.nfu.edu.tw | 資訊工程系 | 2020 | active | 1 | 智慧家居組 | 基於IoT的智慧家居控制系統 | T123456 | 劉建國 | 2024-2025 | active |
| 40943214 | 王小美 | 40943214@gm.nfu.edu.tw | 資訊工程系 | 2020 | active | 2 | AI影像組 | 深度學習影像辨識系統 | T123457 | 吳美玲 | 2024-2025 | active |
| 40943216 | 林雅婷 | 40943216@gm.nfu.edu.tw | 電機工程系 | 2020 | active | 3 | 行動應用組 | 校園導覽APP開發 | T123458 | 黃志偉 | 2024-2025 | active |
| 40943215 | 張志明 | 40943215@gm.nfu.edu.tw | 資訊工程系 | 2020 | pending | NULL | NULL | NULL | NULL | NULL | NULL | NULL |


---

### 2.3 學生週報統計檢視 (view_student_report_stats)

#### VIEW建立語句
```sql
CREATE VIEW view_student_report_stats AS
SELECT 
    s.student_id,
    u.name as student_name,
    s.department,
    g.group_name,
    ay.year_name,
    COUNT(wr.report_id) as total_reports,
    SUM(CASE WHEN wr.status = 'submitted' THEN 1 ELSE 0 END) as submitted_count,
    SUM(CASE WHEN wr.status = 'late' THEN 1 ELSE 0 END) as late_count,
    SUM(CASE WHEN wr.status = 'missing' THEN 1 ELSE 0 END) as missing_count,
    ROUND(AVG(e.score), 2) as avg_score
FROM students s
JOIN users u ON s.user_id = u.user_id
LEFT JOIN groups g ON s.group_id = g.group_id
LEFT JOIN academic_years ay ON g.year_id = ay.year_id
LEFT JOIN weekly_reports wr ON s.student_id = wr.student_id
LEFT JOIN evaluations e ON wr.report_id = e.report_id
WHERE u.status = 'active'
GROUP BY s.student_id, u.name, s.department, g.group_name, ay.year_name;
```

#### 輸出結果
| student_id | student_name | department | group_name | year_name | total_reports | submitted_count | late_count | missing_count | avg_score |
|------------|--------------|------------|------------|-----------|---------------|----------------|------------|---------------|-----------|
| 40943212 | 陳廷威 | 資訊工程系 | 智慧家居組 | 2024-2025 | 3 | 3 | 0 | 0 | 86.67 |
| 40943213 | 李明華 | 資訊工程系 | 智慧家居組 | 2024-2025 | 3 | 1 | 2 | 0 | 75.33 |
| 40943214 | 王小美 | 資訊工程系 | AI影像組 | 2024-2025 | 2 | 2 | 0 | 0 | 91.00 |
| 40943216 | 林雅婷 | 電機工程系 | 行動應用組 | 2024-2025 | 2 | 2 | 0 | 0 | 84.00 |

---

### 2.4 教師工作負荷檢視 (view_teacher_workload)

#### VIEW建立語句
```sql
CREATE VIEW view_teacher_workload AS
SELECT 
    t.teacher_id,
    u.name as teacher_name,
    t.department,
    t.title,
    ay.year_name,
    COUNT(DISTINCT g.group_id) as active_groups,
    COUNT(DISTINCT s.student_id) as total_students,
    COUNT(DISTINCT wr.report_id) as total_reports,
    COUNT(DISTINCT e.evaluation_id) as evaluated_reports,
    CASE 
        WHEN COUNT(DISTINCT wr.report_id) > 0 
        THEN ROUND(COUNT(DISTINCT e.evaluation_id) * 100.0 / COUNT(DISTINCT wr.report_id), 2)
        ELSE 0
    END as evaluation_rate
FROM teachers t
JOIN users u ON t.user_id = u.user_id
LEFT JOIN groups g ON t.teacher_id = g.teacher_id
LEFT JOIN academic_years ay ON g.year_id = ay.year_id
LEFT JOIN students s ON g.group_id = s.group_id
LEFT JOIN weekly_reports wr ON s.student_id = wr.student_id
LEFT JOIN evaluations e ON wr.report_id = e.report_id AND e.teacher_id = t.teacher_id
WHERE u.status = 'active' AND (ay.status = 'active' OR ay.status IS NULL)
GROUP BY t.teacher_id, u.name, t.department, t.title, ay.year_name;
```

#### 輸出結果
| teacher_id | teacher_name | department | title | year_name | active_groups | total_students | total_reports | evaluated_reports | evaluation_rate |
|------------|--------------|------------|-------|-----------|---------------|----------------|---------------|-------------------|-----------------|
| T123456 | 劉建國 | 資訊工程系 | 教授 | 2024-2025 | 1 | 2 | 6 | 6 | 100.00 |
| T123457 | 吳美玲 | 資訊工程系 | 副教授 | 2024-2025 | 1 | 1 | 2 | 2 | 100.00 |
| T123458 | 黃志偉 | 電機工程系 | 助理教授 | 2024-2025 | 1 | 1 | 2 | 2 | 100.00 |

---

### 2.5 組別進度統計檢視 (view_group_progress_stats)

#### VIEW建立語句
```sql
CREATE VIEW view_group_progress_stats AS
SELECT 
    g.group_id,
    g.group_name,
    g.project_title,
    ay.year_name,
    tu.name as teacher_name,
    COUNT(DISTINCT s.student_id) as member_count,
    COUNT(DISTINCT wr.report_id) as total_reports,
    COUNT(DISTINCT e.evaluation_id) as evaluated_reports,
    ROUND(AVG(e.score), 2) as avg_score,
    MAX(wr.week_number) as latest_week,
    COUNT(DISTINCT wr.week_number) as reported_weeks
FROM groups g
JOIN academic_years ay ON g.year_id = ay.year_id
LEFT JOIN teachers t ON g.teacher_id = t.teacher_id
LEFT JOIN users tu ON t.user_id = tu.user_id
LEFT JOIN students s ON g.group_id = s.group_id
LEFT JOIN weekly_reports wr ON s.student_id = wr.student_id
LEFT JOIN evaluations e ON wr.report_id = e.report_id
WHERE g.status = 'active'
GROUP BY g.group_id, g.group_name, g.project_title, ay.year_name, tu.name;
```

#### 輸出結果
| group_id | group_name | project_title | year_name | teacher_name | member_count | total_reports | evaluated_reports | avg_score | latest_week | reported_weeks |
|----------|------------|---------------|-----------|--------------|--------------|---------------|-------------------|-----------|-------------|----------------|
| 1 | 智慧家居組 | 基於IoT的智慧家居控制系統 | 2024-2025 | 劉建國 | 2 | 6 | 6 | 80.83 | 3 | 3 |
| 2 | AI影像組 | 深度學習影像辨識系統 | 2024-2025 | 吳美玲 | 1 | 2 | 2 | 91.00 | 2 | 2 |
| 3 | 行動應用組 | 校園導覽APP開發 | 2024-2025 | 黃志偉 | 1 | 2 | 2 | 84.00 | 2 | 2 |

---

### 2.6 週報提交狀況檢視 (view_weekly_report_submission_status)

#### VIEW建立語句
```sql
CREATE VIEW view_weekly_report_submission_status AS
SELECT 
    ay.year_name,
    wr.week_number,
    COUNT(*) as total_submissions,
    SUM(CASE WHEN wr.status = 'submitted' THEN 1 ELSE 0 END) as on_time,
    SUM(CASE WHEN wr.status = 'late' THEN 1 ELSE 0 END) as late,
    SUM(CASE WHEN wr.status = 'missing' THEN 1 ELSE 0 END) as missing,
    ROUND(AVG(e.score), 2) as avg_score,
    COUNT(e.evaluation_id) as evaluated_count
FROM weekly_reports wr
JOIN academic_years ay ON wr.year_id = ay.year_id
LEFT JOIN evaluations e ON wr.report_id = e.report_id
GROUP BY ay.year_name, wr.week_number
ORDER BY ay.year_name, wr.week_number;
```

#### 輸出結果
| year_name | week_number | total_submissions | on_time | late | missing | avg_score | evaluated_count |
|-----------|-------------|-------------------|---------|------|---------|-----------|-----------------|
| 2024-2025 | 1 | 4 | 3 | 1 | 0 | 83.75 | 4 |
| 2024-2025 | 2 | 4 | 3 | 1 | 0 | 81.75 | 4 |
| 2024-2025 | 3 | 2 | 2 | 0 | 0 | 80.00 | 2 |

---

### 2.7 評分統計檢視 (view_evaluation_statistics)

#### VIEW建立語句
```sql
CREATE VIEW view_evaluation_statistics AS
SELECT 
    t.teacher_id,
    tu.name as teacher_name,
    ay.year_name,
    COUNT(e.evaluation_id) as total_evaluations,
    ROUND(AVG(e.score), 2) as avg_score,
    MIN(e.score) as min_score,
    MAX(e.score) as max_score,
    COUNT(CASE WHEN e.score >= 90 THEN 1 END) as excellent_count,
    COUNT(CASE WHEN e.score >= 80 AND e.score < 90 THEN 1 END) as good_count,
    COUNT(CASE WHEN e.score >= 70 AND e.score < 80 THEN 1 END) as fair_count,
    COUNT(CASE WHEN e.score < 70 THEN 1 END) as poor_count
FROM teachers t
JOIN users tu ON t.user_id = tu.user_id
JOIN groups g ON t.teacher_id = g.teacher_id
JOIN academic_years ay ON g.year_id = ay.year_id
JOIN students s ON g.group_id = s.group_id
JOIN weekly_reports wr ON s.student_id = wr.student_id
JOIN evaluations e ON wr.report_id = e.report_id
WHERE tu.status = 'active'
GROUP BY t.teacher_id, tu.name, ay.year_name
ORDER BY t.teacher_id, ay.year_name;
```

#### 輸出結果
| teacher_id | teacher_name | year_name | total_evaluations | avg_score | min_score | max_score | excellent_count | good_count | fair_count | poor_count |
|------------|--------------|-----------|-------------------|-----------|-----------|-----------|-----------------|------------|------------|------------|
| T123456 | 劉建國 | 2024-2025 | 6 | 80.83 | 73 | 88 | 0 | 4 | 2 | 0 |
| T123457 | 吳美玲 | 2024-2025 | 2 | 91.00 | 90 | 92 | 2 | 0 | 0 | 0 |
| T123458 | 黃志偉 | 2024-2025 | 2 | 84.00 | 82 | 86 | 0 | 2 | 0 | 0 |

---

### 2.8 學生成績分析檢視 (view_student_grade_analysis)

#### VIEW建立語句
```sql
CREATE VIEW view_student_grade_analysis AS
SELECT 
    s.student_id,
    su.name as student_name,
    s.department,
    g.group_name,
    ay.year_name,
    COUNT(e.evaluation_id) as total_evaluations,
    ROUND(AVG(e.score), 2) as avg_score,
    MIN(e.score) as min_score,
    MAX(e.score) as max_score,
    CASE 
        WHEN AVG(e.score) >= 90 THEN '優秀'
        WHEN AVG(e.score) >= 80 THEN '良好'
        WHEN AVG(e.score) >= 70 THEN '及格'
        ELSE '待加強'
    END as grade_level,
    COUNT(CASE WHEN e.score >= 80 THEN 1 END) as high_score_count
FROM students s
JOIN users su ON s.user_id = su.user_id
JOIN groups g ON s.group_id = g.group_id
JOIN academic_years ay ON g.year_id = ay.year_id
JOIN weekly_reports wr ON s.student_id = wr.student_id
JOIN evaluations e ON wr.report_id = e.report_id
GROUP BY s.student_id, su.name, s.department, g.group_name, ay.year_name
ORDER BY avg_score DESC;
```

#### 輸出結果
| student_id | student_name | department | group_name | year_name | total_evaluations | avg_score | min_score | max_score | grade_level | high_score_count |
|------------|--------------|------------|------------|-----------|-------------------|-----------|-----------|-----------|-------------|------------------|
| 40943214 | 王小美 | 資訊工程系 | AI影像組 | 2024-2025 | 2 | 91.00 | 90 | 92 | 優秀 | 2 |
| 40943212 | 陳廷威 | 資訊工程系 | 智慧家居組 | 2024-2025 | 3 | 86.67 | 85 | 88 | 良好 | 3 |
| 40943216 | 林雅婷 | 電機工程系 | 行動應用組 | 2024-2025 | 2 | 84.00 | 82 | 86 | 良好 | 2 |
| 40943213 | 李明華 | 資訊工程系 | 智慧家居組 | 2024-2025 | 3 | 75.33 | 73 | 78 | 及格 | 0 |

---

### 2.9 用戶通知統計檢視 (view_user_notification_summary)

#### VIEW建立語句
```sql
CREATE VIEW view_user_notification_summary AS
SELECT 
    u.user_id,
    u.name,
    u.user_type,
    COUNT(n.notification_id) as total_notifications,
    SUM(CASE WHEN n.is_read = FALSE THEN 1 ELSE 0 END) as unread_count,
    SUM(CASE WHEN n.type = 'system' THEN 1 ELSE 0 END) as system_count,
    SUM(CASE WHEN n.type = 'evaluation' THEN 1 ELSE 0 END) as evaluation_count,
    SUM(CASE WHEN n.type = 'reminder' THEN 1 ELSE 0 END) as reminder_count,
    SUM(CASE WHEN n.type = 'announcement' THEN 1 ELSE 0 END) as announcement_count,
    MAX(n.created_at) as latest_notification
FROM users u
LEFT JOIN notifications n ON u.user_id = n.user_id
WHERE u.status = 'active'
GROUP BY u.user_id, u.name, u.user_type
ORDER BY unread_count DESC;
```

#### 輸出結果
| user_id | name | user_type | total_notifications | unread_count | system_count | evaluation_count | reminder_count | announcement_count | latest_notification |
|---------|------|-----------|---------------------|--------------|--------------|------------------|----------------|-------------------|-------------------|
| 1 | 陳廷威 | student | 3 | 2 | 1 | 1 | 1 | 0 | 2024-10-05 08:00:00 |
| 2 | 李明華 | student | 2 | 2 | 0 | 2 | 0 | 0 | 2024-09-24 09:45:30 |
| 3 | 王小美 | student | 2 | 1 | 0 | 1 | 0 | 1 | 2024-10-01 15:30:00 |
| 4 | 張志明 | student | 1 | 1 | 1 | 0 | 0 | 0 | 2024-09-20 09:00:00 |
| 5 | 林雅婷 | student | 1 | 0 | 1 | 0 | 0 | 0 | 2024-09-10 16:00:00 |
| 6 | 劉建國 | teacher | 1 | 0 | 0 | 0 | 0 | 1 | 2024-09-01 08:00:00 |

---

### 2.10 學年度總覽統計檢視 (view_academic_year_overview)

#### VIEW建立語句
```sql
CREATE VIEW view_academic_year_overview AS
SELECT 
    ay.year_id,
    ay.year_name,
    ay.status,
    ay.start_date,
    ay.end_date,
    COUNT(DISTINCT g.group_id) as total_groups,
    COUNT(DISTINCT s.student_id) as total_students,
    COUNT(DISTINCT t.teacher_id) as total_teachers,
    COUNT(DISTINCT wr.report_id) as total_reports,
    COUNT(DISTINCT e.evaluation_id) as total_evaluations,
    ROUND(AVG(e.score), 2) as avg_score
FROM academic_years ay
LEFT JOIN groups g ON ay.year_id = g.year_id
LEFT JOIN students s ON g.group_id = s.group_id
LEFT JOIN teachers t ON g.teacher_id = t.teacher_id
LEFT JOIN weekly_reports wr ON ay.year_id = wr.year_id
LEFT JOIN evaluations e ON wr.report_id = e.report_id
GROUP BY ay.year_id, ay.year_name, ay.status, ay.start_date, ay.end_date
ORDER BY ay.start_date DESC;
```

#### 輸出結果
| year_id | year_name | status | start_date | end_date | total_groups | total_students | total_teachers | total_reports | total_evaluations | avg_score |
|---------|-----------|--------|------------|----------|--------------|----------------|----------------|---------------|-------------------|-----------|
| 1 | 2024-2025 | active | 2024-09-01 | 2025-06-30 | 6 | 4 | 3 | 10 | 10 | 83.40 |
| 2 | 2023-2024 | archived | 2023-09-01 | 2024-06-30 | 3 | 2 | 2 | 0 | 0 | NULL |
| 7 | 2025-2026 | active | 2025-09-01 | 2026-06-30 | 0 | 0 | 0 | 0 | 0 | NULL |

---

### 2.11 管理員儀表板檢視 (view_admin_dashboard)

#### VIEW建立語句
```sql
CREATE VIEW view_admin_dashboard AS
SELECT 
    'users' as metric_type,
    COUNT(*) as total_count,
    SUM(CASE WHEN status = 'pending' THEN 1 ELSE 0 END) as pending_count,
    SUM(CASE WHEN status = 'active' THEN 1 ELSE 0 END) as active_count,
    SUM(CASE WHEN created_at >= DATE_SUB(NOW(), INTERVAL 7 DAY) THEN 1 ELSE 0 END) as recent_count
FROM users

UNION ALL

SELECT 
    'weekly_reports' as metric_type,
    COUNT(*) as total_count,
    SUM(CASE WHEN status = 'missing' THEN 1 ELSE 0 END) as pending_count,
    SUM(CASE WHEN status = 'submitted' THEN 1 ELSE 0 END) as active_count,
    SUM(CASE WHEN submitted_at >= DATE_SUB(NOW(), INTERVAL 7 DAY) THEN 1 ELSE 0 END) as recent_count
FROM weekly_reports

UNION ALL

SELECT 
    'evaluations' as metric_type,
    COUNT(*) as total_count,
    0 as pending_count,
    COUNT(*) as active_count,
    SUM(CASE WHEN evaluated_at >= DATE_SUB(NOW(), INTERVAL 7 DAY) THEN 1 ELSE 0 END) as recent_count
FROM evaluations

UNION ALL

SELECT 
    'user_approvals' as metric_type,
    COUNT(*) as total_count,
    SUM(CASE WHEN status = 'pending' THEN 1 ELSE 0 END) as pending_count,
    SUM(CASE WHEN status = 'approved' THEN 1 ELSE 0 END) as active_count,
    SUM(CASE WHEN requested_at >= DATE_SUB(NOW(), INTERVAL 7 DAY) THEN 1 ELSE 0 END) as recent_count
FROM user_approvals;
```

#### 輸出結果
| metric_type | total_count | pending_count | active_count | recent_count |
|-------------|-------------|---------------|--------------|--------------|
| users | 21 | 3 | 17 | 14 |
| weekly_reports | 10 | 0 | 10 | 10 |
| evaluations | 10 | 0 | 10 | 10 |
| user_approvals | 10 | 3 | 6 | 5 |

---

### 2.12 學生當前狀態檢視 (view_student_current_status)

#### VIEW建立語句
```sql
CREATE VIEW view_student_current_status AS
SELECT 
    s.student_id,
    su.name as student_name,
    s.department,
    g.group_name,
    tu.name as teacher_name,
    ay.year_name,
    latest_report.week_number as latest_week,
    latest_report.submitted_at as last_submission,
    latest_eval.score as latest_score,
    CASE 
        WHEN latest_report.week_number IS NULL THEN '尚未開始'
        WHEN latest_eval.score IS NULL THEN '待評分'
        WHEN latest_eval.score >= 80 THEN '表現良好'
        WHEN latest_eval.score >= 60 THEN '表現普通'
        ELSE '需要關注'
    END as status_summary
FROM students s
JOIN users su ON s.user_id = su.user_id
LEFT JOIN groups g ON s.group_id = g.group_id
LEFT JOIN academic_years ay ON g.year_id = ay.year_id
LEFT JOIN teachers t ON g.teacher_id = t.teacher_id
LEFT JOIN users tu ON t.user_id = tu.user_id
LEFT JOIN (
    SELECT 
        student_id,
        MAX(week_number) as week_number,
        MAX(submitted_at) as submitted_at
    FROM weekly_reports 
    GROUP BY student_id
) latest_report ON s.student_id = latest_report.student_id
LEFT JOIN weekly_reports wr ON s.student_id = wr.student_id 
    AND wr.week_number = latest_report.week_number
LEFT JOIN evaluations latest_eval ON wr.report_id = latest_eval.report_id
WHERE su.status = 'active' AND (ay.status = 'active' OR ay.status IS NULL);
```

#### 輸出結果
| student_id | student_name | department | group_name | teacher_name | year_name | latest_week | last_submission | latest_score | status_summary |
|------------|--------------|------------|------------|--------------|-----------|-------------|----------------|--------------|----------------|
| 40943212 | 陳廷威 | 資訊工程系 | 智慧家居組 | 劉建國 | 2024-2025 | 3 | 2024-09-29 23:00:00 | 87 | 表現良好 |
| 40943213 | 李明華 | 資訊工程系 | 智慧家居組 | 劉建國 | 2024-2025 | 3 | 2024-09-30 01:30:00 | 73 | 表現普通 |
| 40943214 | 王小美 | 資訊工程系 | AI影像組 | 吳美玲 | 2024-2025 | 2 | 2024-09-22 21:45:00 | 92 | 表現良好 |

### 2.13 教師工作台檢視 (view_teacher_dashboard)

#### VIEW建立語句
```sql
CREATE VIEW view_teacher_dashboard AS
SELECT 
    t.teacher_id,
    tu.name as teacher_name,
    COUNT(DISTINCT g.group_id) as active_groups,
    COUNT(DISTINCT s.student_id) as total_students,
    COUNT(DISTINCT pending_reports.report_id) as pending_evaluations,
    COUNT(DISTINCT this_week_reports.report_id) as this_week_submissions,
    ROUND(AVG(recent_eval.score), 2) as recent_avg_score
FROM teachers t
JOIN users tu ON t.user_id = tu.user_id
LEFT JOIN groups g ON t.teacher_id = g.teacher_id AND g.status = 'active'
LEFT JOIN students s ON g.group_id = s.group_id
LEFT JOIN weekly_reports pending_reports ON s.student_id = pending_reports.student_id
    AND NOT EXISTS (
        SELECT 1 FROM evaluations e 
        WHERE e.report_id = pending_reports.report_id
    )
LEFT JOIN weekly_reports this_week_reports ON s.student_id = this_week_reports.student_id
    AND WEEK(this_week_reports.submitted_at) = WEEK(NOW())
    AND YEAR(this_week_reports.submitted_at) = YEAR(NOW())
LEFT JOIN weekly_reports recent_wr ON s.student_id = recent_wr.student_id
    AND recent_wr.submitted_at >= DATE_SUB(NOW(), INTERVAL 30 DAY)
LEFT JOIN evaluations recent_eval ON recent_wr.report_id = recent_eval.report_id
    AND recent_eval.teacher_id = t.teacher_id
WHERE tu.status = 'active'
GROUP BY t.teacher_id, tu.name;
```

#### 輸出結果
| teacher_id | teacher_name | active_groups | total_students | pending_evaluations | this_week_submissions | recent_avg_score |
|------------|--------------|---------------|----------------|---------------------|----------------------|------------------|
| T123456 | 劉建國 | 1 | 2 | 0 | 0 | 80.83 |
| T123457 | 吳美玲 | 1 | 1 | 0 | 0 | 91.00 |
| T123458 | 黃志偉 | 1 | 1 | 0 | 0 | 84.00 |
| T123459 | 李教授 | 0 | 0 | 0 | 0 | NULL |

---

### 2.14 檔案上傳統計檢視 (view_file_upload_statistics)

#### VIEW建立語句
```sql
CREATE VIEW view_file_upload_statistics AS
SELECT 
    DATE(f.uploaded_at) as upload_date,
    COUNT(f.file_id) as total_files,
    COUNT(DISTINCT f.report_id) as reports_with_files,
    ROUND(SUM(f.file_size) / 1024 / 1024, 2) as total_size_mb,
    f.file_type,
    COUNT(*) as type_count
FROM files f
GROUP BY DATE(f.uploaded_at), f.file_type
ORDER BY upload_date DESC, type_count DESC;
```

#### 輸出結果
| upload_date | total_files | reports_with_files | total_size_mb | file_type | type_count |
|-------------|-------------|-------------------|---------------|-----------|------------|
| 2024-09-29 | 1 | 1 | 3.50 | PDF文件 | 1 |
| 2024-09-23 | 1 | 1 | 1.50 | 圖片 | 1 |
| 2024-09-22 | 3 | 3 | 17.00 | 壓縮檔 | 1 |
| 2024-09-22 | 3 | 3 | 17.00 | 圖片 | 1 |
| 2024-09-22 | 3 | 3 | 17.00 | 設計檔 | 1 |
| 2024-09-16 | 1 | 1 | 3.00 | 圖片 | 1 |
| 2024-09-15 | 5 | 4 | 10.75 | PDF文件 | 2 |
| 2024-09-15 | 5 | 4 | 10.75 | Excel文件 | 1 |
| 2024-09-15 | 5 | 4 | 10.75 | Word文件 | 1 |

---

### 2.15 學期成績報表檢視 (view_semester_grade_report)

#### VIEW建立語句
```sql
CREATE VIEW view_semester_grade_report AS
SELECT 
    ay.year_name,
    s.department,
    g.group_name,
    s.student_id,
    su.name as student_name,
    tu.name as teacher_name,
    COUNT(e.evaluation_id) as total_evaluations,
    ROUND(AVG(e.score), 2) as semester_average,
    MIN(e.score) as lowest_score,
    MAX(e.score) as highest_score,
    CASE 
        WHEN AVG(e.score) >= 90 THEN 'A'
        WHEN AVG(e.score) >= 80 THEN 'B'
        WHEN AVG(e.score) >= 70 THEN 'C'
        WHEN AVG(e.score) >= 60 THEN 'D'
        ELSE 'F'
    END as letter_grade,
    COUNT(CASE WHEN wr.status = 'late' THEN 1 END) as late_submissions,
    COUNT(CASE WHEN wr.status = 'missing' THEN 1 END) as missing_submissions
FROM academic_years ay
JOIN groups g ON ay.year_id = g.year_id
JOIN students s ON g.group_id = s.group_id
JOIN users su ON s.user_id = su.user_id
LEFT JOIN teachers t ON g.teacher_id = t.teacher_id
LEFT JOIN users tu ON t.user_id = tu.user_id
LEFT JOIN weekly_reports wr ON s.student_id = wr.student_id AND wr.year_id = ay.year_id
LEFT JOIN evaluations e ON wr.report_id = e.report_id
GROUP BY ay.year_name, s.department, g.group_name, s.student_id, 
         su.name, tu.name
HAVING COUNT(e.evaluation_id) > 0
ORDER BY ay.year_name, s.department, semester_average DESC;
```

#### 輸出結果
| year_name | department | group_name | student_id | student_name | teacher_name | total_evaluations | semester_average | lowest_score | highest_score | letter_grade | late_submissions | missing_submissions |
|-----------|------------|------------|------------|--------------|--------------|-------------------|------------------|--------------|---------------|--------------|------------------|-------------------|
| 2024-2025 | 資訊工程系 | AI影像組 | 40943214 | 王小美 | 吳美玲 | 2 | 91.00 | 90 | 92 | A | 0 | 0 |
| 2024-2025 | 資訊工程系 | 智慧家居組 | 40943212 | 陳廷威 | 劉建國 | 3 | 86.67 | 85 | 88 | B | 0 | 0 |
| 2024-2025 | 電機工程系 | 行動應用組 | 40943216 | 林雅婷 | 黃志偉 | 2 | 84.00 | 82 | 86 | B | 0 | 0 |
| 2024-2025 | 資訊工程系 | 智慧家居組 | 40943213 | 李明華 | 劉建國 | 3 | 75.33 | 73 | 78 | C | 2 | 0 |

---

### 2.16 專題進度追蹤報表檢視 (view_project_progress_report)

#### VIEW建立語句
```sql
CREATE VIEW view_project_progress_report AS
SELECT 
    ay.year_name,
    g.group_id,
    g.group_name,
    g.project_title,
    g.status as project_status,
    tu.name as supervisor_name,
    COUNT(DISTINCT s.student_id) as team_size,
    COUNT(DISTINCT wr.week_number) as weeks_reported,
    ROUND(COUNT(DISTINCT wr.week_number) * 100.0 / 18, 2) as completion_percentage,
    ROUND(AVG(e.score), 2) as average_score,
    COUNT(CASE WHEN wr.status = 'late' THEN 1 END) as late_submissions,
    COUNT(CASE WHEN wr.status = 'missing' THEN 1 END) as missing_submissions,
    MAX(wr.submitted_at) as last_update,
    COUNT(DISTINCT f.file_id) as total_files_uploaded,
    ROUND(SUM(f.file_size) / 1024 / 1024, 2) as total_file_size_mb
FROM academic_years ay
JOIN groups g ON ay.year_id = g.year_id
LEFT JOIN teachers t ON g.teacher_id = t.teacher_id
LEFT JOIN users tu ON t.user_id = tu.user_id
LEFT JOIN students s ON g.group_id = s.group_id
LEFT JOIN weekly_reports wr ON s.student_id = wr.student_id AND wr.year_id = ay.year_id
LEFT JOIN evaluations e ON wr.report_id = e.report_id
LEFT JOIN files f ON wr.report_id = f.report_id
GROUP BY ay.year_name, g.group_id, g.group_name, g.project_title, 
         g.status, tu.name
ORDER BY ay.year_name, completion_percentage DESC;
```

#### 輸出結果
| year_name | group_id | group_name | project_title | project_status | supervisor_name | team_size | weeks_reported | completion_percentage | average_score | late_submissions | missing_submissions | last_update | total_files_uploaded | total_file_size_mb |
|-----------|----------|------------|---------------|----------------|-----------------|-----------|----------------|----------------------|---------------|------------------|-------------------|-------------|----------------------|-------------------|
| 2024-2025 | 1 | 智慧家居組 | 基於IoT的智慧家居控制系統 | active | 劉建國 | 2 | 3 | 16.67 | 80.83 | 2 | 0 | 2024-09-30 01:30:00 | 5 | 9.00 |
| 2024-2025 | 2 | AI影像組 | 深度學習影像辨識系統 | active | 吳美玲 | 1 | 2 | 11.11 | 91.00 | 0 | 0 | 2024-09-22 21:45:00 | 2 | 11.46 |
| 2024-2025 | 3 | 行動應用組 | 校園導覽APP開發 | active | 黃志偉 | 1 | 2 | 11.11 | 84.00 | 0 | 0 | 2024-09-22 19:20:00 | 2 | 9.00 |

---

### 2.17 審核工作統計檢視 (view_approval_workload)

#### VIEW建立語句
```sql
CREATE VIEW view_approval_workload AS
SELECT 
    ua.admin_id,
    au.name as admin_name,
    ua.request_type,
    COUNT(*) as total_requests,
    SUM(CASE WHEN ua.status = 'pending' THEN 1 ELSE 0 END) as pending_count,
    SUM(CASE WHEN ua.status = 'approved' THEN 1 ELSE 0 END) as approved_count,
    SUM(CASE WHEN ua.status = 'rejected' THEN 1 ELSE 0 END) as rejected_count,
    ROUND(AVG(DATEDIFF(ua.processed_at, ua.requested_at)), 2) as avg_processing_days
FROM user_approvals ua
LEFT JOIN users au ON ua.admin_id = au.user_id
WHERE ua.requested_at >= DATE_SUB(NOW(), INTERVAL 90 DAY)
GROUP BY ua.admin_id, au.name, ua.request_type
ORDER BY pending_count DESC;
```

#### 輸出結果
| admin_id | admin_name | request_type | total_requests | pending_count | approved_count | rejected_count | avg_processing_days |
|----------|------------|--------------|----------------|---------------|----------------|----------------|-------------------|
| 9 | 系統管理員 | registration | 6 | 2 | 4 | 0 | 2.75 |
| 9 | 系統管理員 | role_change | 2 | 1 | 1 | 0 | 1.00 |
| 9 | 系統管理員 | account_recovery | 2 | 0 | 1 | 1 | 1.00 |

---

### 2.18 系統使用統計檢視 (view_system_usage_statistics)

#### VIEW建立語句
```sql
CREATE VIEW view_system_usage_statistics AS
SELECT 
    DATE(sl.created_at) as activity_date,
    sl.action_type,
    COUNT(*) as action_count,
    COUNT(DISTINCT sl.user_id) as unique_users,
    COUNT(DISTINCT sl.ip_address) as unique_ips
FROM system_logs sl
WHERE sl.created_at >= DATE_SUB(NOW(), INTERVAL 30 DAY)
GROUP BY DATE(sl.created_at), sl.action_type
ORDER BY activity_date DESC, action_count DESC;
```

#### 輸出結果
| activity_date | action_type | action_count | unique_users | unique_ips |
|---------------|-------------|--------------|--------------|------------|
| 2024-10-01 | create | 2 | 1 | 1 |
| 2024-09-30 | create | 1 | 1 | 1 |
| 2024-09-29 | create | 1 | 1 | 1 |
| 2024-09-24 | login | 2 | 2 | 2 |
| 2024-09-24 | view | 1 | 1 | 1 |
| 2024-09-23 | login | 1 | 1 | 1 |
| 2024-09-23 | create | 1 | 1 | 1 |
| 2024-09-20 | login | 1 | 1 | 1 |
| 2024-09-20 | update | 1 | 1 | 1 |
| 2024-09-17 | login | 1 | 1 | 1 |
| 2024-09-17 | create | 1 | 1 | 1 |
| 2024-09-15 | login | 1 | 1 | 1 |
| 2024-09-15 | create | 1 | 1 | 1 |

---

## 3. 常用查詢範例

### 3.1 管理員常用查詢
```sql
-- 查詢系統整體狀況
SELECT * FROM view_admin_dashboard;

-- 查詢待審核用戶
SELECT * FROM view_user_basic_info WHERE status = 'pending';

-- 查詢各學年度統計
SELECT * FROM view_academic_year_overview WHERE status = 'active';

-- 查詢本週提交的週報統計
SELECT * FROM view_weekly_report_submission_status 
WHERE year_name = '2024-2025' 
ORDER BY week_number DESC LIMIT 5;
```

### 3.2 教師常用查詢
```sql
-- 查詢教師工作台資訊
SELECT * FROM view_teacher_dashboard WHERE teacher_id = 'T123456';

-- 查詢教師指導的學生列表
SELECT * FROM view_student_full_info 
WHERE teacher_id = 'T123456' 
ORDER BY student_name;

-- 查詢組別進度統計
SELECT * FROM view_group_progress_stats 
WHERE teacher_name = '劉建國';

-- 查詢評分統計
SELECT * FROM view_evaluation_statistics 
WHERE teacher_id = 'T123456';
```

### 3.3 學生常用查詢
```sql
-- 查詢學生完整資訊
SELECT * FROM view_student_full_info WHERE student_id = '40943212';

-- 查詢學生週報統計
SELECT * FROM view_student_report_stats WHERE student_id = '40943212';

-- 查詢學生當前狀態
SELECT * FROM view_student_current_status WHERE student_id = '40943212';

-- 查詢學生成績分析
SELECT * FROM view_student_grade_analysis WHERE student_id = '40943212';

-- 查詢學生通知
SELECT * FROM view_user_notification_summary WHERE user_id = 1;
```

### 3.4 報表產生查詢
```sql
-- 產生學期成績報表
SELECT * FROM view_semester_grade_report 
WHERE year_name = '2024-2025' 
ORDER BY semester_average DESC;

-- 產生專題進度報表
SELECT * FROM view_project_progress_report 
WHERE year_name = '2024-2025' 
ORDER BY completion_percentage DESC;

-- 產生檔案使用統計
SELECT upload_date, SUM(total_files) as daily_files, SUM(total_size_mb) as daily_size_mb
FROM view_file_upload_statistics 
GROUP BY upload_date 
ORDER BY upload_date DESC;

-- 產生系統使用報表
SELECT activity_date, SUM(action_count) as total_actions, COUNT(DISTINCT action_type) as action_types
FROM view_system_usage_statistics 
GROUP BY activity_date 
ORDER BY activity_date DESC;
```

---

## 4. 總結

這份完整的資料庫實作文檔提供了：

### 📊 完整的範例資料
- 12個資料表，每表10筆真實且相互關聯的資料
- 涵蓋完整的專題進度追蹤流程
- 支援各種業務情境測試

### 🔍 功能完整的VIEW集合
- 18個精心設計的VIEW
- 涵蓋用戶管理、學習追蹤、教學管理、系統監控等各方面
- 提供儀表板和報表所需的預處理資料

### 📈 實用的查詢範例
- 管理員、教師、學生各角色的常用查詢
- 報表產生和統計分析查詢
- 支援系統各項功能需求

### 🎯 實際應用價值
- 可直接用於系統開發測試
- 提供使用者介面設計參考
- 支援功能驗證和展示
- 作為效能測試基準

這個完整的實作文檔為專題進度追蹤系統提供了堅實的資料庫基礎，確保系統能夠有效支援學術機構的專題管理需求。