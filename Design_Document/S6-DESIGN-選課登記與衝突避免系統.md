# 📐 系統設計文件 (Design Document) - Story 6：選課登記與衝突避免

## 1. 文件概覽 (Document Summary)

本文件描述了「選課登記與衝突避免」功能的設計細節，著重於選課流程中的**資料一致性**和**多重檢查邏輯**。所有系統塑模圖均使用 Mermaid 語法呈現，以確保在 GitHub 上的渲染穩定性。

| 項目 | 描述 |
| :--- | :--- |
| **文件標題** | Story 6：選課登記與衝突避免系統設計 |
| **文件版本** | V1.0 (設計初始版本) |
| **設計目標** | 確保選課流程是原子性 (Atomic) 的，並能在交易啟動前阻擋所有不符合資格的操作。 |

---

## 2. 系統塑模圖 (UML Models)

本章節整合了選課功能的系統設計視圖，包含：**活動圖** (流程決策)、**循序圖** (交易時序) 與 **狀態圖** (生命週期)。

### I. 活動圖 (Activity Diagram) - 選課決策流程
描述學生點擊「選課」後的完整判斷路徑。

```mermaid
flowchart TD
    %% 修正：所有包含特殊符號的文字都加上了雙引號
    start(("開始: 學生點擊選課")) --> A["系統: 執行前置檢查 (FR-S6.2)"];
    
    A -- "資格/額滿檢查失敗" --> F1{"顯示錯誤訊息: 額滿/資格不符"};
    A -- "檢查通過" --> B{"檢查學分是否超限？"};
    
    B -- "超限" --> F2{"顯示錯誤訊息: 超過學分上限"};
    B -- "未超限" --> C["系統: 取出既有課表時間 (FR-S6.6)"];
    
    C --> D{"時間是否衝突？ (FR-S6.7)"};
    D -- "衝突" --> F3{"顯示錯誤訊息: 時間衝突"};
    D -- "無衝突" --> E["Database: START TRANSACTION"];
    
    E --> G["Database: 寫入選課紀錄 (FR-S6.3)"];
    G --> H["Database: 課程名額 -1 (FR-S6.4)"];
    H --> I{"顯示: 選課成功 (FR-S6.5)"};
    
    F1 --> stop(("結束"));
    F2 --> stop;
    F3 --> stop;
    I --> stop;
sequenceDiagram
    participant S as Student
    participant UI as Course Reg UI
    participant RC as RegController
    participant CE as CheckEngine
    participant DB as Database
    
    S->>UI: 點擊選課按鈕
    UI->>RC: handleRegistration(courseID, studentID)
    
    RC->>CE: 1. checkEligibility(studentID, courseID) (FR-S6.2)
    alt 檢查不通過 (學分/資格)
        CE-->>RC: 返回失敗
        RC-->>UI: 返回錯誤訊息
    else 檢查通過
        RC->>DB: 2. START TRANSACTION
        
        RC->>DB: 3. 檢查剩餘名額 (FR-S6.2)
        DB-->>RC: 返回名額狀態
        
        alt 名額已滿
            RC->>DB: ROLLBACK
            RC-->>UI: 返回錯誤 (額滿)
        else 名額足夠
            RC->>CE: 4. checkTimeConflict(courseID, studentSchedule) (FR-S6.7)
            alt 時間衝突
                CE-->>RC: 返回失敗
                RC->>DB: ROLLBACK
                RC-->>UI: 返回錯誤 (時間衝突)
            else 無衝突
                RC->>DB: 5. INSERT 紀錄 (FR-S6.3)
                RC->>DB: 6. UPDATE 名額 -1 (FR-S6.4)
                RC->>DB: COMMIT
                DB-->>RC: 交易成功
                RC-->>UI: 顯示成功訊息 (FR-S6.5)
            end
        end
    end
stateDiagram-v2
    [*] --> Initial: 未選課
    
    Initial --> Pending_Reg: 點擊選課按鈕
    
    Pending_Reg --> Initial: 檢查失敗/取消 (FR-S6.2, FR-S6.8)
    Pending_Reg --> Enrolled: 交易成功提交 (FR-S6.3/S6.4)
    
    Enrolled --> Dropped: 點擊退選按鈕 (FR-S6.9)
    
    Dropped --> Initial: 退選完成 (FR-S6.10/S6.11)
    
    state Dropped {
        Dropped_Start : 名額已+1
        Dropped_End : 紀錄已刪除
        Dropped_Start --> Dropped_End: 資料庫操作完成
    }
