# ROUTE 代碼分析
多模態多任務 text2sql 處理
## 代碼目錄
### 1. **生成 SQL 查詢的提示欄位模板**
- **`prompt_temp`**
- **`prompt_sl_temp_sft`**
- **`prompt_sl_temp`**

### 2. **SQL 查詢正確性檢查**
- **`prompt_nc_temp_sft`**
- **`prompt_nc_temp`**

### 3. **SQL 查詢補全（繼續寫作）**
- **`prompt_cw_temp_sft`**
- **`prompt_cw_temp`**

### 4. **數據處理**
- **`read_json_file`**

### 5. **LLM 模型**
- **`LLM_Model` 類**
  - **`generate_response`**

- **`LLM_Online` 類**
  - **`generate_response`**

### 6. **資料庫操作**
- **`convert_fk_index`**
- **`dump_db_json_schema`**
- **`get_schema_dict`**
- **`get_example_str`**
- **`get_schmea_str_and_examples`**
- **`parse_sql_from_string`**
- **`replace_multiple_spaces`**
- **`filter_dict_by_sql`**
- **`filter_dict_by_sl`**

### 7. **SQL 查詢執行**
- **`execute_query_limit`**
- **`execute_query`**

### 8. **語句替換**
- **`replace_syn`**

### 9. **GPU 資源管理**
- **`usegpu`**

### 10. **主程序執行**
- **`eval_all`**
  
### 11. **主函數**
- **`__main__`**

---


# 具體語法細節
## 一、載入套件

1. **數據處理**：
   - `json`：用於讀寫 JSON 格式數據。
   - `copy`：用於創建對象的深拷貝。
   - `csv`：用於讀寫 CSV 文件。
   - `re`（正則表達式庫）：用於文本模式匹配、查找、替換等操作。
   - `sqlite3`：用於與 SQLite 資料庫交互。

2. **錯誤處理**：
   - `traceback`：用於捕獲並顯示錯誤信息，幫助調試和處理異常。

3. **資源管理**：
   - `os`：提供操作系統相關功能，如設置環境變量、管理文件系統。

4. **模型生成**：
   - `vllm`：用於運行語言模型並生成相應的回應，支持多種模型結構。

5. **計算超時**：
   - `func_timeout`：用於設置超時限制，防止函數長時間無回應。

6. **進度顯示**：
   - `tqdm`：用於顯示進度條，幫助跟蹤任務的進度。

## 二、 生成前準備：SQL 語料處理  
### 1. 生成 SQL 查詢的提示欄位模板

- **`prompt_temp`**：
  - **功能**：生成 SQL 查詢提示，包含資料庫架構、範例資料、問題描述和提示語，讓模型生成有效的 SQL 查詢。
  
- **`prompt_sl_temp_sft`**：
  - **功能**：指示模型從資料庫架構和問題中提取相關表格和欄位，而不是直接生成 SQL 查詢，並且要求模型只輸出所需的表格和欄位。

- **`prompt_sl_temp`**：
  - **功能**：提供模型範例，展示如何根據問題選擇相關表格和欄位，並以 JSON 格式輸出映射，無需生成 SQL 語句。

### 2. SQL 查詢正確性檢查

- **`prompt_nc_temp_sft`**：
  - **功能**：檢查生成的 SQL 查詢是否能正確回答問題，如果不能，請提供正確的 SQL 查詢。

- **`prompt_nc_temp`**：
  - **功能**：同上，並提供範例，展示正確和錯誤的 SQL 查詢，幫助模型理解如何修正不正確的查詢。

### 3. SQL 查詢補全（繼續寫作）

- **`prompt_cw_temp_sft`**：
  - **功能**：將不完整的 SQL 查詢補全為完整的查詢，確保它能正確回答問題。

- **`prompt_cw_temp`**：
  - **功能**：同上，並提供範例，展示如何將不完整的 SQL 查詢補全為正確的查詢。

## 三、生成前準備：功能函式、類別定義
### 1. 數據處理與模型運行

- **`read_json_file`**：
  - **功能**：讀取 JSON 文件並將其轉換為 Python 數據結構（如字典或列表）。

- **`LLM_Model` 類**：
  - **功能**：用於初始化語言模型，設置生成參數並生成回應（SQL 查詢）、設置分詞器。

- **`generate_response`**：
  - **功能**：根據給定的提示（`prompts`）生成回應，並根據參數（如 `temperature`, `top_p`）控制生成的隨機性。

### 2. 資料庫操作

- **`get_schema_dict`**：
  - **功能**：從 SQLite 資料庫中提取結構信息（表格名、欄位名、資料類型等）並整理成字典。

- **`filter_dict_by_sql`**、**`filter_dict_by_sl`**：
  - **功能**：根據 SQL 查詢過濾資料庫結構，保留與查詢相關的表格和欄位。

- **`execute_query`**：
  - **功能**：執行 SQL 查詢並檢查結果，根據執行是否成功進行後續處理。

## 四、主訓練

- **`eval_all`**：
  - **功能**：解析數據集、設置訓練參數、生成 SQL 查詢提示、執行 SQL 查詢生成，並對查詢進行後處理（如噪聲修正、查詢補全等）。

- **`usegpu`**：
  - **功能**：根據 GPU 使用情況自動選擇空閒的 GPU 並設置環境變量。

---

## 程式運行流程

### 1. **`eval_all` 函數**

`eval_all` 是程序的核心邏輯，負責處理整體的數據集處理、SQL 查詢生成等任務。具體過程包括：

- **解析數據集**：加載指定的數據集（例如 `spider`, `bird` 等），並根據模式 (`dev`, `test`) 進行處理。
- **設定批次大小**：設置每次處理的數據批次大小，以便有效控制內存使用和運行效率。
- **基於配置的標誌選擇任務**：
  - 根據配置的標誌（如 `flg1`, `flg2`），決定是否執行 SQL 查詢生成、正確性檢查、查詢補全等。
- **使用語言模型生成 SQL 查詢**：利用預先訓練的語言模型來生成 SQL 查詢，並根據生成的結果進行後續處理。

這部分代碼會調用多個子函數來完成具體的任務，例如：
- **`generate_response`**：生成 SQL 查詢的回應。
- **`filter_dict_by_sql`**：過濾資料庫結構，根據生成的 SQL 查詢選擇相關表格和欄位。
- **`execute_query`**：執行 SQL 查詢並檢查結果是否正確。

### 2. **`main` 函數**

`main` 函數主要負責設置程序的運行環境，解析命令行參數，設置 GPU 資源，並最終調用 `eval_all` 來開始執行 SQL 查詢生成和模型訓練的任務。

具體過程如下：

- **解析命令行參數**：使用 `argparse` 解析命令行中的參數，如數據集名稱（`--dataset`）、GPU 數量（`--gpus`）等配置。
- **根據 `args.gpus` 設置 GPU 資源**：選擇並設置可用的 GPU，確保模型在正確的硬件上運行。
- **調用 `eval_all` 開始執行任務**：根據解析的參數，啟動 `eval_all` 函數來執行 SQL 查詢生成或模型訓練等任務。

---

## 二、整體運行流程

1. 用戶通過命令行傳遞參數來設置數據集、模型、GPU 資源等。
2. `main` 函數解析參數並設置 GPU 資源。
3. `main` 函數調用 `eval_all` 函數來開始 SQL 查詢生成、模型訓練等任務。
4. `eval_all` 函數負責整個任務的運行，從數據集處理、SQL 查詢生成到最終結果的處理。


# DPO + CoT pipeline
1. CoT synthesis : 有現成 dataset 可以用（人大團隊） 
2. supervised fine-tuning (SFT) 4/22
   - 代碼
     - 參考 ROUTE 代碼
   - 模型
     - 先使用deepseek 6.7b LLM（ ROUTE 沒用，所以要另外寫入?)
   - 資料集
     - Vanilla
     - CoT-enhanced dataset(先拿 BIRD?)
4. direct preference optimization (DPO) 4/29
   - 代碼
     - 參考 ROUTE 代碼
     - 找 DPO－SQL 代碼（待處理）
   - 模型
     - 先使用 deepseek 6.7b LLM（ ROUTE 沒用，所以要另外寫入
   - 資料集
     - Vanilla
     - CoT-enhanced dataset(先拿 BIRD)
