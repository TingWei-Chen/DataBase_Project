# 🎯 專題進度追蹤系統

![MariaDB](https://img.shields.io/badge/MariaDB-003545?style=for-the-badge&logo=mariadb&logoColor=white)
![PHP](https://img.shields.io/badge/PHP-777BB4?style=for-the-badge&logo=php&logoColor=white)
![Laravel](https://img.shields.io/badge/Laravel-FF2D20?style=for-the-badge&logo=laravel&logoColor=white)
![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)
![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)

## 👨‍💻 開發者

### 陳廷威 40943212

我是「專題進度追蹤系統」的開發者，致力於資料庫設計與實作、系統架構規劃及前後端整合。
正在對於資料庫正規化設計與ACID特性實踐方面進行深入研究。

#### 🧰 技術棧

**資料庫技術**
![MariaDB](https://img.shields.io/badge/MariaDB-003545?style=for-the-badge&logo=mariadb&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)

**後端/前端技術**
![PHP](https://img.shields.io/badge/PHP-777BB4?style=for-the-badge&logo=php&logoColor=white)
![Java](https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)
![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)

#### 🔗 聯絡方式

[![Email](https://img.shields.io/badge/Email-40943212%40gm.nfu.edu.tw-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:40943212@gm.nfu.edu.tw)
[![GitHub](https://img.shields.io/badge/GitHub-TingWei--Chen-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/TingWei-Chen)

## 📚 課堂練習與作業

### **[📄 05/07 Appendix E Common data models](./homework/DB_Exercise_0507.md)**

## ✨ 題目定義

專題進度追蹤系統是一個多用戶網頁應用平台，旨在幫助教師有效地追蹤和記錄學生的專題進展。系統支持三種用戶角色（學生、教師和管理員），提供專題管理、進度追蹤、評分回饋和記錄查看等功能。

## 👥 應用情境與使用案例

### 情境一：學生週報繳交與進度追蹤

**參與者**：資工系大三學生

**情境描述**：
某資工系大三學生，正在進行畢業專題。每週五是他們小組繳交週報的截止日。今天是週四晚上，學生需要提交本週的進度報告。

**使用案例**：
1. 學生登入系統進入「進度繳交」頁面
2. 系統顯示表單，包含「本週進度報告」與「下週工作計畫」欄位
3. 學生填寫本週已完成的程式模組測試結果，並上傳測試程式碼檔案
4. 學生填寫下週將進行系統整合測試的計畫
5. 學生提交週報，系統記錄提交時間
6. 系統通知學生週報提交成功，可在「瀏覽模式」查看
7. 學生進入「瀏覽模式」，透過日曆界面查看本週的提交記錄
8. 學生查看上週填寫的「下週工作計畫」與本週實際完成的工作，進行自我檢視

### 情境二：教師評閱週報與提供回饋

**參與者**：資工系教授

**情境描述**：
教授指導多組畢業專題，每週需要審閱學生提交的週報並給予回饋。週報截止日的隔天（週六），教授安排時間檢視並評分各組的週報。

**使用案例**：
1. 教授登入系統後，進入「進度評分」頁面
2. 系統顯示日曆視圖，標記出已有學生提交週報的日期
3. 教授點選昨天（週五）的日期，系統顯示所有已提交週報的組別
4. 教授選擇「智慧家居系統」專題組，查看該組所有成員的週報
5. 系統顯示該組每位成員的週報內容、上傳檔案及歷史進度比對
6. 教授發現學生的程式測試方法有改進空間，於評論區提供具體建議
7. 教授為學生的週報評分並提交，系統記錄評分時間
8. 教授繼續評閱其他組員的週報，完成本週所有評分工作
9. 教授進入「組別管理」，根據週報表現調整某些學生的分組

### 情境三：系統管理與專題分配調整

**參與者**：系統管理員

**情境描述**：
新學年開始，專題指導老師選擇時間已截止，同時系上來了新教授和轉學生。此外，系主任需要查看上一學年度的所有專題記錄作為參考。管理員需要處理這些系統管理任務。

**使用案例**：
1. 管理員登入系統，進入「審核介面」頁面
2. 系統顯示待審核的新用戶申請列表，包含新教授和轉學生
3. 管理員審核新進教授的帳號申請，並分配教師權限
4. 管理員審核轉學生的帳號申請，並分配學生權限
5. 審核完成後，管理員進入學生「組別管理」頁面
6. 根據學生的專長和老師需求，管理員調整部分學生的專題組別分配
7. 管理員確認每個專題組別的人數平衡
8. 接著，管理員進入「統計功能」頁面
9. 管理員選擇上一學年度的時間範圍，查詢所有專題記錄
10. 系統生成視覺化報表，顯示各組學生的進度完成情況與評分情況
11. 管理員匯出這些統計數據，準備提供給系主任作為本學年度專題規劃參考

## 🖥️ 系統需求說明

<table>
  <tr>
    <th>功能類別</th>
    <th>功能描述</th>
  </tr>
  <tr>
    <td><b>📊 用戶管理</b></td>
    <td>
      • <b>註冊與登入</b>：所有用戶需要註冊帳號並登入系統才能使用功能<br>
      • <b>權限管理</b>：系統需區分學生、教師和管理員三種權限<br>
      • <b>帳號審核</b>：新用戶註冊須經管理員審核後方可使用<br>
      • <b>密碼管理</b>：提供密碼重設和修改功能
    </td>
  </tr>
  <tr>
    <td><b>👨‍🎓 學生功能</b></td>
    <td>
      • <b>個人資料管理</b>：查看個人基本資料<br>
      • <b>週報提交</b>：填寫本周專題報告項目和下周工作計畫<br>
      • <b>檔案上傳</b>：上傳專題相關檔案（設計圖、程式碼、文件等）<br>
      • <b>進度查詢</b>：以日曆方式查看自己和組員的週報提交情況<br>
      • <b>回饋查看</b>：查看教師對週報的評語和評分
    </td>
  </tr>
  <tr>
    <td><b>👨‍🏫 教師功能</b></td>
    <td>
      • <b>組別管理</b>：建立、編輯專題組別，分配和調整組員<br>
      • <b>進度評核</b>：查看、評分學生提交的週報，提供評語<br>
      • <b>評分記錄</b>：查看過去的評分的紀錄<br>
      • <b>歷年專題查詢</b>：查看負責的學生過往學年的專題資料
    </td>
  </tr>
  <tr>
    <td><b>👨‍💼 管理員功能</b></td>
    <td>
      • <b>用戶審核</b>：審核新用戶註冊申請，分配適當權限<br>
      • <b>用戶調整</b>：根據需求調整學生所屬的教授和實驗室<br>
      • <b>檔案管理</b>：查看往年所有學生的專題<br>
      • <b>系統監控</b>：監控系統運作狀況，查看系統日誌
    </td>
  </tr>
  <tr>
    <td><b>🔄 系統共通功能</b></td>
    <td>
      • <b>日曆視圖</b>：以日曆形式顯示週報提交與評分情況<br>
      • <b>檔案管理</b>：支援多種格式檔案上傳、下載與預覽<br>
      • <b>搜尋功能</b>：按年份、組別、關鍵字等條件搜尋專題資料<br>
      • <b>通知功能</b>：週報提交、評分完成等事件的通知提醒
    </td>
  </tr>
</table>

## 📝 完整性限制

### **[📄 完整性限制詳細說明](docs/database/database-details.md#完整性限制)**

此連結包含完整的資料庫完整性限制機制說明，涵蓋實體完整性、參照完整性、值域完整性和用戶定義完整性的詳細實作。

## 📊 ER Diagram

![er_diagram](docs/database/er_diagram.png)

### ER Diagram 實體說明

#### 主要實體 (Entities)

1. **用戶 (User)**
   - 屬性：用戶ID、姓名、電子郵件、密碼、用戶類型、建立日期、狀態
   - 說明：系統中所有用戶的基本資訊

2. **學生 (Student)**
   - 屬性：學號、科系、入學年份
   - 說明：學生特有的學術資訊

3. **教師 (Teacher)**
   - 屬性：教師編號、科系、職稱
   - 說明：教師特有的職業資訊

4. **組別 (Group)**
   - 屬性：組別ID、組別名稱、專題名稱、建立日期
   - 說明：專題組別的基本資訊

5. **週報 (WeeklyReport)**
   - 屬性：週報ID、本週工作、下週計畫、提交日期、週次、年份
   - 說明：學生提交的週報內容

6. **評分 (Evaluation)**
   - 屬性：評分ID、分數、評語、評分日期
   - 說明：教師對週報的評分記錄

7. **學年度 (Academic_Year)**
   - 屬性：學年ID、學年名稱、開始日期、結束日期、狀態
   - 說明：管理不同學年度的資訊

8. **評分標準 (Evaluation_Criteria)**
   - 屬性：標準ID、名稱、描述、權重
   - 說明：評分的具體標準和權重

9. **檔案 (File)**
   - 屬性：檔案ID、檔案名稱、檔案路徑、檔案大小、檔案類型、上傳日期
   - 說明：系統中的檔案管理

10. **通知 (Notification)**
    - 屬性：通知ID、內容、類型、已讀狀態、建立日期
    - 說明：系統通知訊息

11. **系統日誌 (System_Log)**
    - 屬性：日誌ID、操作類型、實體類型、實體ID、描述、IP位址、記錄時間
    - 說明：系統操作記錄

#### 主要關係 (Relationships)

1. **用戶-學生**：一對一關係，一個用戶帳號對應一個學生身份
2. **用戶-教師**：一對一關係，一個用戶帳號對應一個教師身份
3. **教師-組別**：一對多關係，一位教師可指導多個組別
4. **學生-組別**：多對一關係，多位學生屬於一個組別
5. **學生-週報**：一對多關係，一位學生可提交多份週報
6. **週報-評分**：一對一關係，一份週報對應一個評分記錄
7. **教師-評分**：一對多關係，一位教師可評分多份週報
8. **週報-檔案**：一對多關係，一份週報可包含多個檔案
9. **學年度-組別**：一對多關係，一個學年度包含多個組別
10. **學年度-評分標準**：一對多關係，一個學年度有多個評分標準

## 📊 完整資料庫設計

### **[📄 詳細資料庫設計文檔](docs/database/database-details.md)**

此連結包含：
- 12個核心資料表的完整SQL建立語句
- 各表的範例測試資料（每表至少10筆資料）
- 資料庫用戶與權限設計
- 典型查詢示例
- 交易管理範例
- VIEW設計與SQL語法

## 📋 專案進度與完成狀況

| **Final Project 階段** | **項目內容** | **狀態** | **備註** |
|------------------------|-------------|----------|---------|
| **Part I** | | | |
| | 題目定義 | ✅ 已完成 | 專題進度追蹤系統 |
| | 應用情境與使用案例 | ✅ 已完成 | 三個主要使用情境 |
| | 系統需求說明 | ✅ 已完成 | 功能需求分析 |
| | 完整性限制 | ✅ 已完成 | 四種完整性機制 |
| | ER Diagram及詳細說明 | ✅ 已完成 | 11個主要實體與關係 |
| **Part II** | | | |
| | 修正系統需求說明 | ✅ 已完成 | 根據回饋調整內容 |
| | 修正 ER Diagram 與說明 | ✅ 已完成 | 優化實體關係設計 |
| | 完整資料庫 Schema（SQL語法） | ✅ 已完成 | 12個核心資料表 |
| | 範例資料說明 | ✅ 已完成 | 每表至少10筆測試資料 |
| | 資料庫使用者與權限設計 | ✅ 已完成 | 角色權限控制機制 |
| **Part III** | | | |
| | 題目與需求整合 | ✅ 已完成 | 整合前兩階段內容 |
| | ER Diagram 最終版 | ✅ 已完成 | 完整實體關係圖 |
| | 完整 DB Schema 與說明 | ✅ 已完成 | 含VIEW與SQL語法 |
| | 各資料表真實資料 | ✅ 已完成 | 每表10筆以上資料 |
| | 系統實作成果 | ⚠️ 進行中 | 後端開發進行中 |
| | 展示影片 | 🕒 待開始 | 預計系統完成後製作 |
| **開發實作** | | | |
| | 資料庫建置 | ✅ 已完成 | MariaDB環境建置 |
| | 後端API開發 | ⚠️ 進行中 | 評估Laravel框架 |
| | 前端介面開發 | 🕒 待開始 | 預計使用React |
| | 系統整合 | 🕒 待開始 | 前後端整合測試 |

## 🛠️ 開發技術與工具

### 使用語言
- **後端**：PHP 8.1 / Laravel 9
- **前端**：HTML5, CSS3, JavaScript (ES6+), React
- **資料庫**：SQL (MariaDB 10.6)

### 開發工具
- **IDE**：PhpStorm 2023.1, WebStorm 2023.1
- **資料庫管理**：DataGrip 2023.1, phpMyAdmin
- **版本控制**：Git, GitHub
- **API 測試**：Postman
- **前端框架**：React, Bootstrap 5
- **後端框架**：Laravel 9
