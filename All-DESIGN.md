# **設計文件**

# **一、選課系統設計文件(Design Document)：新增課程**

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
<img width="4250" height="3885" alt="順序圖" src="https://github.com/user-attachments/assets/36af8407-41cb-44a1-ae67-60c0ca16f030" />

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

# **二、選課系統設計文件 (Design Document)：教室自動化排程**

**文件版本:** V1.0  
**日期:** 2025/12/7
**文件對應:** 本設計文件對應 S2
**專案作者：** 凌彣碩, B1243003

## **1\. 細部系統架構 (Detailed System Architecture)**

本系統採用 **分層式架構 (Layered Architecture)**，結合 **RESTful API** 設計，以確保各模組的低耦合與高內聚。

### **1.1 架構分層設計**

| 分層 (Layer) | 元件名稱 | 職責描述 | 技術選型範例 |
| :---- | :---- | :---- | :---- |
| **表現層 (Presentation)** | **教師前端 SPA** | 負責 UI 渲染、使用者互動、發送 AJAX 請求。包含「我的課表」與「更換教室」模組。 | React / Vue.js |
| **API 閘道層 (Controller)** | **ScheduleController** | 接收 HTTP 請求，驗證 Token (Auth)，參數檢核，並轉發至服務層。 | Spring Boot Web / Express |
| **業務邏輯層 (Service)** | **SchedulingService** | **核心層**。實作「自動排程演算法」、「衝突檢測」、「交易控制 (@Transactional)」。 | Java / Node.js |
| **資料存取層 (Repository)** | **ClassroomRepo CourseRepo** | 負責與資料庫進行 CRUD 操作，執行複雜的 SQL 查詢（如尋找可用教室）。 | JPA / Hibernate / Sequelize |
| **資料儲存層 (Database)** | **PostgreSQL/MySQL** | 儲存關聯式資料，利用 DB 鎖機制確保 ACID 交易一致性。 | PostgreSQL (推薦) |

### **1.2 核心模組交互**

1. **衝突檢測器 (Collision Detector):** 獨立元件，輸入時間區間與教室 ID，輸出 Boolean (是否衝突)。  
2. **容量過濾器 (Capacity Filter):** 輸入課程人數，輸出符合容量的教室清單。

## **2\. 重要功能的系統塑模 (System Modeling)**

本節針對需求文件中定義的 **任務 1 (自動排程)** 與 **任務 3 (手動修改)** 進行邏輯建模。

### **2.1 活動圖 (Activity Diagram)**

**描述:** 說明「自動排程演算法」的執行流程。對應需求 AC1 (時間衝突)、AC2 (容量)。
<img width="956" height="444" alt="排課" src="https://github.com/user-attachments/assets/eeeb92f2-66b4-4075-9f13-cf9cda406d23" />

### **2.2 循序圖 (Sequence Diagram)**

**描述:** 說明「教師手動更換教室」的互動流程。對應需求 AC5 (手動更換) 與 AC3 (防止重複預訂)。
<img width="3620" height="3102" alt="手動排程循序圖" src="https://github.com/user-attachments/assets/333ce68c-9de7-4ee4-bda1-7f10f8e937d5" />

### **2.3 狀態圖 (State Diagram)**

**描述:** 描述一門課程在排程系統中的生命週期狀態變化。
<img width="2008" height="1008" alt="自動排課狀態圖" src="https://github.com/user-attachments/assets/44344ac1-b3de-4926-8645-b94fe32cd741" />

## **3\. 資料庫設計 (Database Design) \-**

為了滿足 **AC1\~AC3** 的資料驗證需求，資料庫設計如下。

### **3.1 實體關聯圖 (ER Diagram 概念)**

* **Teachers (教師表):** 儲存教師基本資料。  
* **Classrooms (教室表):** 儲存靜態資源，關鍵欄位為 Capacity (容量)。  
* **Courses (課程表):** 核心表，儲存上課時間與分配到的教室。

### **3.2 資料表結構 (Schema)**

Table: Classrooms  
| 欄位名 | 類型 | 描述 | 備註 |  
| :--- | :--- | :--- | :--- |  
| id | VARCHAR(10) | 教室編號 | PK, e.g., "E-201" |  
| name | VARCHAR(50) | 教室名稱 | |  
| capacity | INT | 容納人數 | 用於 AC2 檢查 |  
| equipment | JSON | 設備列表 | e.g., \["Projector", "PC"\] | 


Table: Courses  
| 欄位名 | 類型 | 描述 | 備註 |  
| :--- | :--- | :--- | :--- |  
| id | BIGINT | 課程 ID | PK |  
| teacher\_id | BIGINT | 授課教師 | FK \-\> Teachers.id |  
| subject\_name| VARCHAR(100)| 課程名稱 | |  
| student\_count| INT | 預計修課人數| 用於 AC2 檢查 |  
| day\_of\_week | INT | 星期幾 | 1=週一, ..., 5=週五 |  
| start\_period| INT | 開始節次 | e.g., 3 (第3節) |  
| end\_period | INT | 結束節次 | e.g., 4 (第4節) |  
| classroom\_id| VARCHAR(10) | 分配教室 | FK \-\> Classrooms.id (Null表示未排) |  
| status | VARCHAR(20) | 排程狀態 | PENDING, SCHEDULED, FAILED |

# **三、選課系統設計文件 (Design Document)：查看學生選課狀況模組**

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

# **四、選課系統設計文件 (Design Document)：登入系統身分驗證**

**文件版本:** V1.0  
**日期:** 2025/12/07
**文件對應:** 本設計文件對應 S4
**專案作者：** 施穎禾, B1243021

## **1\. 細部系統架構 (Detailed System Architecture)**

本登入模組採用 **分層式架構 (Layered Architecture)**，核心在於 **身份驗證服務 (Authentication Service)**，並特別強調安全性與 Token/Session 管理。

### **1.1 架構分層設計**

| 分層 (Layer) | 元件名稱 | 職責描述 | 技術選型範例 |
| :---- | :---- | :---- | :---- |
| **表現層 (Presentation)** | **Login UI** | 負責登入表單的渲染、前端輸入檢查 (AC3)，發送登入請求。 | React / Vue.js |
| **API 閘道層 (Controller)** | **AuthController** | 接收學號/密碼 (HTTP POST)，執行基本參數檢核，並轉發至服務層。 | Spring Boot Web / Express |
| **業務邏輯層 (Service)** | **AuthService** | **核心層**。執行密碼雜湊比對、錯誤登入計數/鎖定邏輯 (AC5)，並生成 Session/Token (AC4)。 | Java / Node.js |
| **資料存取層 (Repository)** | **StudentRepo** | 負責讀取資料庫中學生的**學號**與**加密密碼**。 | JPA / Hibernate / Sequelize |
| **資料儲存層 (Database)** | **PostgreSQL/MySQL** | 儲存學生帳號資料，密碼需使用 bcrypt/Argon2 雜湊儲存 (NFR 4.1.1)。 | PostgreSQL (推薦) |

### **1.2 核心安全機制**

1.  **密碼儲存:** 密碼必須使用加鹽 (Salted) 的雜湊演算法儲存，確保即使資料庫外洩，密碼也不可逆讀取。
2.  **暴力破解防護 (AC5):** 登入失敗時，系統應記錄失敗 IP 或學號，超過 5 次失敗則暫時鎖定帳號（例如 30 分鐘）。
3.  **Session/Token:** 登入成功後，發放 JWT (JSON Web Token) 或 Session ID，並設置 `HttpOnly` 及 `Secure` 屬性 Cookie，以防 XSS 攻擊 (NFR 4.1.4)。

## **2\. 重要功能的系統塑模 (System Modeling)**

本節針對需求文件中定義的 **T1 (輸入與驗證)** 與 **T2 (成功導向)** 進行邏輯建模。

### **2.1 活動圖 (Activity Diagram)**
**<img width="1511" height="337" alt="登入活動圖" src="https://github.com/user-attachments/assets/89c6954a-df09-4808-9697-b2c25c44c925" />
描述:** 說明「學生登入與身份驗證」的完整流程。對應需求 AC1, AC2, AC3。

### **2.2 循序圖 (Sequence Diagram)**

**描述:** 說明「身份驗證成功並建立授權」的互動流程。對應需求 AC2, AC4, T2。
<img width="3620" height="3102" alt="登入成功循序圖" src="" />

### **2.3 狀態圖 (State Diagram)**

**描述:** 描述一個**使用者身份**在系統中的生命週期狀態變化。
<img width="2008" height="1008" alt="使用者身份狀態圖" src="" />

## **3\. 資料庫設計 (Database Design)**

為了滿足 **AC1** 的身份驗證與 **NFR 4.1.1** 的安全性要求，資料庫設計如下。

### **3.1 實體關聯圖 (ER Diagram 概念)**

* **Students (學生表):** 儲存登入所需的學號、姓名、加密密碼。
* **Security_Log (安全日誌表):** 儲存登入失敗的嘗試，用於防護暴力破解 (AC5)。

### **3.2 資料表結構 (Schema)**

Table: Students  
| 欄位名 | 類型 | 描述 | 備註 |
| :--- | :--- | :--- | :--- |
| student\_id | VARCHAR(10) | 學號 | PK, 用於 AC1 |
| full\_name | VARCHAR(50) | 學生姓名 | |
| password\_hash | VARCHAR(255) | 加鹽的加密密碼 | NFR 4.1.1 密碼儲存標準 |
| salt | VARCHAR(64) | 密碼雜湊使用的 Salt | |
| last\_login\_at | TIMESTAMP | 上次登入時間 | |

Table: Security\_Log
| 欄位名 | 類型 | 描述 | 備註 |
| :--- | :--- | :--- | :--- |
| id | BIGINT | 日誌 ID | PK |
| student\_id | VARCHAR(10) | 嘗試登入學號 | |
| attempted\_at | TIMESTAMP | 嘗試登入時間 | |
| result | VARCHAR(10) | 嘗試結果 | SUCCESS / FAILURE |
| source\_ip | VARCHAR(40) | 來源 IP | NFR 4.3.2 稽核與 AC5 鎖定機制 |

# **五、選課系統設計文件 (Design Document)：課程課程瀏覽與高效搜尋**

**文件版本:** V1.0  
**日期:** 2025/12/08
**文件對應:** 本設計文件對應 S5
**專案作者：** 楊立安

# 📐 系統設計文件 (Design Document) - Story 5：課程瀏覽與高效搜尋

## 1. 文件概覽 (Document Summary)

本文件描述了「課程瀏覽與高效搜尋」功能的設計細節，包含系統的核心靜態結構和動態流程。

| 項目 | 描述 |
| :--- | :--- |
| **文件標題** | Story 5：課程瀏覽與高效搜尋系統設計 |
| **文件版本** | V1.0 (設計初始版本) |
| **設計目標** | 確保課程列表的資料結構清晰，並定義多條件查詢和篩選的內部運作流程。 |

### **活動圖 (Activity Diagram)**
<img width="845" height="623" alt="S5-活動圖" src="https://github.com/user-attachments/assets/053f86c7-09c1-490c-b751-d2e158bdb208" />

### **循序圖 (Sequence Diagram)**
<img width="758" height="432" alt="S5-順序圖" src="https://github.com/user-attachments/assets/562ffd6b-1ee4-491b-8c00-42974b481f9c" />
 
### **狀態圖 (State Diagram)**
<img width="688" height="650" alt="S5-狀態圖" src="https://github.com/user-attachments/assets/02a3e649-3e5c-4912-b79b-b779e5b2bc05" />

# **六、選課系統設計文件 (Design Document)：選課登記與衝突避免**

**文件版本:** V1.0  
**日期:** 2025/12/08
**文件對應:** 本設計文件對應 S6
**專案作者：** 楊立安

# 📐 系統設計文件 (Design Document) - Story 6：選課登記與衝突避免(V1.0)

## 1. 文件概覽 (Document Summary)

本文件描述了「選課登記與衝突避免」功能的設計細節，著重於選課流程中的**資料一致性**和**多重檢查邏輯**。所有系統塑模圖均使用 Mermaid 語法呈現，以確保在 GitHub 上的渲染穩定性。

| 項目 | 描述 |
| :--- | :--- |
| **文件標題** | Story 6：選課登記與衝突避免系統設計 |
| **文件版本** | V1.0 (設計初始版本) |
| **設計目標** | 確保選課流程是原子性 (Atomic) 的，並能在交易啟動前阻擋所有不符合資格的操作。 |

---

### **活動圖 (Activity Diagram)**
![605232823_860400106633157_2847693983377031434_n](https://github.com/user-attachments/assets/55244d31-1644-47ec-adad-7fc3a03b45a1)


### **循序圖 (Sequence Diagram)**
<img width="857" height="618" alt="S6順序圖" src="https://github.com/user-attachments/assets/eada5347-6283-40f4-9490-d6d719afe1a4" />


### **狀態圖 (State Diagram)**
<img width="1048" height="642" alt="S6狀態圖" src="https://github.com/user-attachments/assets/417dbe5a-b097-423c-9f7c-d955153e5b14" />

# **七、選課系統設計文件 (Design Document)：查看課表與匯出課表**

**文件版本:** V1.0  
**日期:** 2025/12/09
**文件對應:** 本設計文件對應 S7
**專案作者：** 陳暐仁, B1243013

### **流程圖 (flow diagram)**
<img width="992" height="269" alt="S7流程圖" src="https://github.com/user-attachments/assets/d8b8eab6-6304-48ac-b877-4d051cafbd2f" />

# **八、選課系統設計文件 (Design Document)：排課管理與衝突偵測系統**

**文件版本:** V1.0  
**日期:** 2025/12/09
**文件對應:** 本設計文件對應 S8
**專案作者：** 蔡昀辰,B1243037

### **流程圖 (flow diagram)-修改選課規則活動**
<img width="961" height="470" alt="修改選課規則活動流程圖" src="https://github.com/user-attachments/assets/18e63326-e6d6-41fe-92d1-34fcd4a73594" />

### **流程圖 (flow diagram)-調解衝突活動**
<img width="1503" height="335" alt="調解衝突活動流程圖" src="https://github.com/user-attachments/assets/12faa136-3c2b-447f-b724-77bbf7e3d363" />

### **狀態圖 (State Diagram)-管理介面系統**
<img width="671" height="630" alt="管理介面系統流程圖" src="https://github.com/user-attachments/assets/3f66238d-fe34-47a5-bc17-d03fd810fc7b" />

