Oracle 和 Postgres 之間的以下差異:  
* 欄位結構轉換  
  * 在 Postgres 12 版以前，Postgres 沒有支援任何虛擬欄位功能，建議將這些虛擬列改寫為視觀表（view）。當前版本 Postgres 已具備生成欄位（generated columns）功能，並與 Oracle 的虛擬欄位有許多相同之處。
* 欄位約束功能（Constraints）  
  * 這兩種資料庫系統，主鍵（Primary Key）、外鍵（ Foreign Key）、空值檢查（Check Not-Null）和唯一性約束的運作模式都大同小異。
* 物件名稱差異  
  * Oracle 預設將 Schema、表格、欄位和函數等物件名字轉為大寫（如果名字被雙引號標記則不受影響）。
  * Postgres 物件名稱預設轉換為小寫 (如果名字被雙引號標記則不受影響)。
  * 只要應用程式中的 SQL 物件一致地套用雙引號，或是完全未使用雙引號標記，基本上不會有太大問題。
  * 加註:
    * 對即有程式對進行白箱測試，函蓋率儘可提高
*  索引支援
   *  Postgres 同 Oracle 支援功能相同的二元樹索引（B-tree index）和降冪排序索引（descending index）  
   *  Postgres 尚未支援
      *  返向索引（Reverse key index）
      *  點陣索引（Bitmap index）
      *  結合索引（Join index）
      *  跨分割表全域索引（Global index）
   * 加註:
     *  SQL執行效能與執行計畫
* 分區表（Partioning）類型支援  
  * Postgres 支援等同 Oracle 的 Hash、List 和 Range 的分區表功能
* Table 差異
  * CREATE TABLE 指令在多數情況下語法都是相容的，但些許例外：
    * Postgres 不支援 Oracle 全局暫存表（global temporary tables），須改用一般臨時表（LOCAL TEMP）。
    * 分區表（Partioning）：可使用繼承（Inheritance）、觸發器（Triggers）、和檢查限制式（CHECK Constraints ）功能實現。
      * （譯注：Postgres 10 之後支援原生的分區表語法，非常建議採用）
    * Oracle 特有的儲存參數（INITRANS, MAXEXTENTS）在 Postgres 中無法識別，因此需要移除
    * 使用 Postgres 的 fillfactor 取代在 Oracle 使用的 PCTFREE 參數
* Tablespace 差異  
  * Oracle 和 Postgres 的表空間功能有所差異，但都可以支援相同的用途。（如：分散資料儲存到不同磁碟區）
* 資料型態 差異 ，列出了 Oracle 和 Postgres 資料型態的顯著差異

| Oracle | Postgres | EDB  Postgres Advanced Server | 建議 |  
| :-- | :-- |:--|:--|
| VARCHAR2(n) | VARCHAR(n) | VARCHAR2(n),VARCHAR(n) | 請注意不要被 Oracle 和 Postgres 資料類型的 ‘n’混淆。在 Oracle 中，它代表 bytes 的大小，在 Postgres 中，它代表資料字元數量 |
| NVARCHAR, NVARCHAR2 | VARCHAR or TEXT | NVARCHAR, NVARCHAR2,VARCHAR or TEXT | 
| CHAR(n), NCHAR(n) | CHAR(n) | CHAR(n),NCHAR(n), | 不要被 Oracle 和 Postgres 資料類型的 ‘n’混淆。 在 Oracle 中，它代表 bytes 的大小，在 Postgres 中，它代表資料字元數量 |
| NUMBER(n, m)  | NUMERIC(n,m) | NUMERIC(n,m) , NUMBER(n, m)  | NUMBER 資料型態可以轉換為 NUMERIC 數值型態。儘管 NUMERIC 的數值範圍不受限制，但 SMALLINT、INT、 BIGINT、 REAL 和 DOUBLE PRECISION 資料類型卻提供更好的效能。 |
| NUMBER(4) | SMALLINT  | NUMBER(4),SMALLINT  | 
| NUMBER(9)  | INT | NUMBER(9) , INT | 
| NUMBER(18) | BIGINT | NUMBER(18),BIGINT | 
| NUMBER(n) | NUMERIC(n) | NUMBER(n),NUMERIC(n) | 當 n>=19 | 
| BINARY_INTEGER, BINARY_FLOAT  | INTEGER, FLOAT  | BINARY_INTEGER,INTEGER, FLOAT | 
| DATE | TIMESTAMP(0) | DATE,TIMESTAMP(0) | 在 Oracle 中 DATE 資料類型會同時回傳日期和時間，但在 Postgres 中，DATE 資料類型只返回日期不含時間。 |
| TIMESTAMP WITH LOCAL TIME ZONE | TIMESTAMPTZ | TIMESTAMP WITH LOCAL TIME ZONE,TIMESTAMPTZ | Oracle 擁有 TIMESTAMP WITH TIME ZONE 和 TIMESTAMP WITH LOCAL TIME ZONE 這兩種資料類型。Postgres 的 TIMESTAMPTZ 與 Oracle 的 TIMESTAMP WITH LOCAL TIME ZONE 相互對應。 |
| CLOB, LONG | TEXT | CLOB, LONG , TEXT |Postgres 的 TEXT 資料類型能夠儲存 1 GB 大小的文字格式資料。|
| BLOB,RAW(n),LONG RAW | BYTEA(1 GB limit),Large object | BLOB,RAW(n),LONG RAW,BYTEA(1 GB limit) , Large Object | 在 Oracle 中，BLOB 資料類型適用於儲存非結構化二進位資料，在儲存上幾乎沒有大小限制（能夠儲存將近 128 TB 的二進制資料），而 Postgres 的 BYTEA 資料類型能儲存將近 1GB 的二進位資料。如果您的資料量超過儲存上限，可以考慮  Large Object（他們被存於不同的表格中）。 |
| NLS_DATE_FORMAT | DateStyle | DateStyle | 這些是設置顯示日期資訊格式的參數。 Postgres 的預設日期格式 DateStyle 參數為 ISO。而 Oracle 的預設日期格式則是由NLS_TERRITORY 參數繼承而來。 |

 
*  挑戰和限制，這些必須被其他替代方案手動重寫
   *  
 

 
 