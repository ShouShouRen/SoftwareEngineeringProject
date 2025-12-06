需求文件需要完整、正式的格式
其他文件不用

# 📄 需求文件（Requirements Document）

- **系統說明**
- **初步系統架構（高階）**
- **功能需求**
- **非功能需求**

---

# 🧩 設計文件（Design Document）

- **細部系統架構**
- **重要功能的系統塑模（需與需求文件對應）**
  - 活動圖（Activity Diagram）
  - 循序圖（Sequence Diagram）
  - 狀態圖（State Diagram）
- **選用項目（非必須）**
  - 資料庫設計
  - 系統介面設計
  - 使用者介面（UI）設計

---

# 🧪 測試文件（Test Document）

- **測試案例內容須包含：**
  - 進入條件（Preconditions）
  - 測試方法（Test Procedure）
  - 測試成功條件（Success Criteria）
- **測試案例必須能對應需求文件中的特定需求**

---

# 📋 檔案命名規範（File Naming Convention）

## 命名格式

為確保多人協作時的一致性與可維護性，所有文件請遵循以下命名規範：

```text
{Story編號}-{文件類型}-{功能描述}.md
```

### 參數說明

- **Story 編號**：使用 `S1`, `S2`, `S3` ... 格式（S = Story）
- **文件類型**：
  - `REQ` = 需求文件（Requirements Document）
  - `DESIGN` = 設計文件（Design Document）
  - `TEST` = 測試文件（Test Document）
- **功能描述**：簡短清楚的中文描述（建議 2-6 字）

### 命名範例

假設你負責 **Story 1: 學生選課功能** 和 **Story 2: 教室排程功能**：

#### Requirement_Document/ 資料夾

- `S1-REQ-學生選課.md`
- `S2-REQ-教室排程.md`

#### Design_Document/ 資料夾

- `S1-DESIGN-學生選課.md`
- `S2-DESIGN-教室排程.md`

#### Testing_Document/ 資料夾

- `S1-TEST-學生選課.md`
- `S2-TEST-教室排程.md`

### 注意事項

- ✅ 使用短橫線 `-` 分隔各部分
- ✅ 文件類型使用大寫英文縮寫
- ✅ 功能描述使用清楚易懂的中文
- ❌ 避免使用空格、冒號、特殊符號
- ❌ 避免過長的檔名

# 📢 初步設計報告（Presentation）

- **整合以上三份文件內容**
- **製作成 20 分鐘簡報**
