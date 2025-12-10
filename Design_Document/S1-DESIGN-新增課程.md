# **選課系統設計文件（Design Document）：新增課程**

**文件版本:** V1.0  
**日期:** 2025/12/10  
**對應需求:** Story1
**作者：** 張育鴻, B1243008

---

## **1\. 細部系統架構 (Detailed System Architecture)**

登入系統採用 **三層式架構（Three-Tier Architecture）**，包含：

- **前端（Frontend）**
- **後端 API（Backend Application）**
- **資料庫（Database）**

系統負責完成帳號密碼驗證（Authentication）與 Session 建立（Session Management）。

---

### 1.1 系統架構層次

| 層級 | 元件名稱 | 功能說明 |
|------|-----------|-----------|
| **前端 UI 層（Presentation Layer）** | Login Page | 顯示登入表單，讓使用者輸入帳號與密碼，並發送登入請求 |
| **應用層（Application Layer, Controller）** | AuthController | 接收 POST /auth/login 請求、驗證輸入格式 |
| **服務層（Business Service Layer）** | AuthService | 進行帳號查詢、密碼 Hash 驗證、產生 Session Token |
| **資料存取層（Data Access Layer）** | StudentRepository | 查詢 Students Table，確認帳密是否正確 |
| **資料庫層（Database Layer）** | Students Table | 儲存使用者帳號、密碼 Hash 與身分（teacher/student） |

---

## 2.系統塑模（System Modeling）

---

### **2.1 活動圖 (Activity Diagram)**
<img width="6088" height="1057" alt="活動圖" src="https://github.com/user-attachments/assets/e2738dbf-9161-4442-a8ab-ff9fdffb0ea9" />

### **2.2 循序圖 (Sequence Diagram)**
<img width="5785" height="4925" alt="順序圖" src="https://github.com/user-attachments/assets/69e52e60-dd50-4dcf-aad9-39c69b3e5540" />

### **2.3 狀態圖 (State Diagram)**
<img width="5051" height="787" alt="狀態圖" src="https://github.com/user-attachments/assets/0a041600-11c9-4fee-a613-23649fbe817e" />

----------------------------------------
## **3. Database Schema**
Table:Courses表
| 欄位名           | 類型           | 描述     | 備註                       |
| :------------ | :----------- | :----- | :----------------------- |
| course_id     | BIGINT       | 課程 ID  | PK, 自動生成                 |
| teacher_id    | VARCHAR(20)  | 建立者 ID | FK → Students.student_id |
| name          | VARCHAR(100) | 課程名稱   | 不可為空                     |
| student_limit | INT          | 人數上限   | 不可為空                     |
| day_of_week   | INT          | 星期幾    | 1=週一 … 7=週日              |
| start_period  | INT          | 節次開始   | e.g., 3                  |
| end_period    | INT          | 節次結束   | e.g., 4                  |
| created_at    | DATETIME     | 建立時間   | 預設 CURRENT_TIMESTAMP     |
