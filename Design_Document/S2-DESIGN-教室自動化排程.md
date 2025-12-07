# **選課系統設計文件 (Design Document)：教室自動化排程**

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


