# **選課系統設計文件 (Design Document)：查看學生選課狀況模組**

**文件版本:**  V1.0
**日期:** 2025/12/10
**文件對應:** Story 3
**專案作者:** 張育鴻, B1243008

## **1\. 細部系統架構 (Detailed System Architecture)**

本系統採用 **分層式架構 (Layered Architecture)**，結合 **RESTful API**，以確保低耦合、高內聚。

### **1.1 架構分層設計**
| 分層 (Layer)           | 元件名稱                                      | 職責描述                        | 技術選型範例                      |
| :------------------- | :---------------------------------------- | :-------------------------- | :-------------------------- |
| 表現層 (Presentation)   | Teacher Dashboard                         | 顯示學生選課資訊，提供查詢功能、篩選與排序       | React / Vue.js              |
| API 閘道層 (Controller) | EnrollmentController                      | 接收前端請求，驗證 Token，呼叫服務層取得資料   | Spring Boot / Express       |
| 業務邏輯層 (Service)      | EnrollmentService                         | 查詢學生選課資料，計算統計資訊，提供分頁 / 搜尋功能 | Java / Node.js              |
| 資料存取層 (Repository)   | StudentRepo / CourseRepo / EnrollmentRepo | 與資料庫交互，執行查詢與資料整合            | JPA / Hibernate / Sequelize |
| 資料儲存層 (Database)     | MySQL / PostgreSQL                        | 儲存學生、課程與選課關聯資料              | MySQL / PostgreSQL          |

### **1.2 核心模組交互**

1. **選課查詢模組 (Enrollment Query Module):** 接收教師查詢請求，回傳學生列表與選課資訊。
2. **統計模組 (Statistics Module):** 計算課程人數、選課比例、待選人數等。

## **2\. 系統塑模 (System Modeling)**

### **2-1. Activity Diagram(查看學生選課狀況)**
<img width="8932" height="982" alt="活動圖S3" src="https://github.com/user-attachments/assets/385c07ca-4280-4edc-82c5-1feeb6ce40bc" />

### **2-2. Sequence Diagram（使用者查看學生選課流程）**
<img width="7500" height="2925" alt="順序圖S3" src="https://github.com/user-attachments/assets/90bc003c-8023-4703-8d30-c9afdf738dbf" />

### **2-3. State Diagram (課程物件狀態圖)**
<img width="5322" height="465" alt="狀態圖S3" src="https://github.com/user-attachments/assets/f49f7a29-6f2e-44c0-93c6-c8f73c84def5" />

## **3. UI 設計**

### **3.1 查看學生選課狀況頁面**
--------------------------------------------------------
|       學生選課狀況 Dashboard                        |
--------------------------------------------------------
| 課程選擇: [下拉選單]  搜尋: [________] [查詢按鈕] |
|------------------------------------------------------|
| 序號 | 學生姓名 | 學號 | 選課狀態 | 選課時間        |
| :--- | :---     | :--- | :---     | :---           |
| 1    | 張育鴻   | B1243003 | 已選課 | 2025-12-10     |
| 2    | 王小明   | B1243004 | 已選課 | 2025-12-09     |
| ...  | ...      | ...       | ...    | ...           |
--------------------------------------------------------
|  [排序] [搜尋] [匯出 CSV]                            |
--------------------------------------------------------

## **4. DB Schema**

### **4.1 Students Table**
| 欄位名稱         | 類型                        | 說明         |
| :----------- | :------------------------ | :--------- |
| student_id   | VARCHAR(20)               | 使用者帳號（PK）  |
| student_name | VARCHAR(50)               | 使用者姓名      |
| role         | ENUM('teacher','student') | 使用者身份      |
| status       | TINYINT                   | 1=啟用, 0=停用 |

### **4.2 Courses Table**
| 欄位名稱         | 類型           | 說明                              |
| :----------- | :----------- | :------------------------------ |
| course_id    | BIGINT       | 課程編號（PK）                        |
| course_name  | VARCHAR(100) | 課程名稱                            |
| teacher_id   | VARCHAR(20)  | 授課教師（FK -> Students.student_id） |
| max_students | INT          | 課程容納人數                          |

### **4.3 Enrollments Table**
| 欄位名稱          | 類型                         | 說明                              |
| :------------ | :------------------------- | :------------------------------ |
| enrollment_id | BIGINT                     | 選課記錄編號（PK）                      |
| student_id    | VARCHAR(20)                | 學生編號（FK -> Students.student_id） |
| course_id     | BIGINT                     | 課程編號（FK -> Courses.course_id）   |
| status        | ENUM('selected','dropped') | 選課狀態                            |
| enrolled_at   | DATETIME                   | 選課時間                            |
