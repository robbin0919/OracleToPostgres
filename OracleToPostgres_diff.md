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

 
# 待處理的問題和限制，這些必須被其他替代方案手動重寫
1.  欄位約束功能 
2.  資料刪除語法 
3. 刪除資料庫物件
4. Dual table
5. 空字串和 NULL 值
6. NULL判斷函數
7. Oracle  Federation 與 Postgres 的外部表（Foreign Data Wrappers）
8. GRANT 語法
9. 階層式查詢
10. Join 語法搭配 (+)
11. 檢查 NOT NULL
12. 套件語法（Package）支援
13. PL/SQL 到 PL/PgSQL 的改寫
14. 遠端資料庫物件
15. ROWID、CTID 和識別欄位（Identity Columns）
16. 序號物件（Sequences）
17. SUBSTR 函數
18. 同義詞（Synonyms
19. SYSDATE、CURRENT_TIMESTAMP
20. TO_DATE 函數
21. 交易控制
22. 交易錯誤的例外處理
23. DECODE 
24. Subquery in FROM
25. to_number函数
26. instr函数
27. 字符串连接符( || )
28. substr
29. length
30. trim/ltrim/rtrim函数
31. NLSSORT
32. NLS_INITCAP/ NLS_LOWER/ NLS_UPPER
33. regexp_replace
34. regexp_substr
35. regexp_instr
36. regexp_like
37. NVL2
38. LNNVL
39. BITAND
40. REMAINDER
41. 

----

1.  欄位約束功能 
    *   Oracle 允許它們的用戶隨心所欲地停用和啟用欄位約束功能，但一般不建議這種操作關聯式資料庫的作法，如果操作不夠謹慎，可能會導致資料毀損。
    *  使用者能夠在 Postgres 中，透過 SET CONSTRAINTS 指令，指定延遲欄位約束功能（DEFERRED）。延遲欄位約束功能用來指定欄位約束的運作時機。如果在 Oracle 中欄位約束沒有預先啟用延遲功能，則必須先將該欄位約束刪除再重設啟用（部份情況下可以直接變更欄位約束啟用延遲功能設定）。
    *  建議：為避免潛在錯誤或資料損壞，建議將用於刪除或重建欄位約束的指令，以 BEGIN/COMMIT 程式區塊將其框列成一筆交易，以便在設定期間鎖定資料表。  
  
2.  資料刪除語法 
       * 在 Postgres 中，DELETE 語法必需搭配 FROM 關鍵字一起使用，但在 Oracle 中則不用。  
       * Oracle:
          ```
              delete table_name where column_name='AAAA';
          ```
       * Postgres:   
          ```
              delete  from table_name where column_name='AAAA';
          ```
3. 刪除資料庫物件
     *  在 Postgres 中，只有物件擁有者和「超級使用者」有刪除物件的權限。
     *  物件擁有者的群組能透過授權被賦予，但卻無法授權刪除資料庫物件的操作。
     *  如果原先搭配 Oracle 應用程式中仰賴非擁有者刪除物件的功能，會需要重新設計/設定相關功能。
  
4. Dual table
    * 在 Postgres 中， SELECT 語法不強制搭配 FROM 子句，因此 FROM DUAL 這個語法在 Postgres 中並不必要，一般情況下能直接能移除。
    * 如果使用 Postgres 時仍需要 Dual 表，可以創建視觀表代替。   
    * https://dba.stackexchange.com/questions/187800/dual-table-postgres-doesnt-return-timestamp
    * 
5. 空字串和 NULL 值
   *  空字串在 Oracle 中被視為 NULL ，在 Postgres 則否。在 Oracle 中，使用者能夠利用 IS NULL 條件式來檢查字串是否為空字串，但在 Postgres 中，IS NULL 對空字串的傳回值皆為 FALSE。（運算子為 NULL 時，則傳回 TRUE）
6. NULL判斷函數
   * Oracle
     * NULL判斷函數是 nvl(A, B) 和 coalesce 兩個函數。
     * nvl(A, B) 是判斷如果A不為NULL，則回傳A, 否則回傳B。參數需要是相同類型的，或者可以自動轉換成相同類型的, 否則需要明確轉換。 
     * coalese 參數可以有多個，傳回第一個不為NULL的參數。而參數必須為相同類型的 ，不會自動轉換。
   * PostgreSQL
     * 沒有nvl函數。但是有coalesce函數。用法和Oracle的一樣。
     * 可以使用coalesce來轉換Oracle的nvl和coalesce。參數需要使用相同的類型，或可以轉換成相同類型。否則需要手動轉換。
7. Oracle  Federation 與 Postgres 的外部表（Foreign Data Wrappers）
8. GRANT 語法
9. 階層式查詢
10. Join 語法搭配 (+)
11. 檢查 NOT NULL
    * Oracle 18 以前的版本， NULL 是 NULL、空字串是 NULL，但是 PostgreSQL 並不接受空字串是 NULL
    * 在 Oracle 18 已經改了  https://docs.oracle.com/en/database/oracle/oracle-database/18/sqlrf/Nulls.html#GUID-B0BA4751-9D88-426A-84AD-BCDBD5584071
    * 
12. 套件語法（Package）支援
    * Postgres 不支援 packages 語法，
    * 但能利用 Schema 特性，將函數（user-defined function）和程序（stored procedure）組織起來。
    * 也能夠使用 Orafce 相容套件，提供部份的 Oracle 內建 package，
    * 或使用 EDB Postgres Advanced Server ，內建相容 package 與 package 語法支援。
13. PL/SQL 到 PL/PgSQL 的改寫
14. 遠端資料庫物件
15. ROWID、CTID 和識別欄位（Identity Columns）
16. 序號物件（Sequences）
    * Oracle:  
      ```
      Sequence_name.nextval
      ```
    * PostgreSQL:
      ````
      Nextval(‘sequence_name’)
      ````
17. SUBSTR 函數  
18. 同義詞（Synonyms  
19. SYSDATE  
20. TO_DATE 函數  
21. 交易控制  
22. 交易錯誤的例外處理  
23. DECODE   
24. Subquery in FROM   
    * a query for Oracle:  
          ```
               SELECT * FROM (SELECT * FROM table_a);
          ``` 
    * a query for PostgreSQL :  
          ```
                SELECT * FROM (SELECT * FROM table_a) AS foo
          ```           
25. to_number函数  
      * to_number用於將字元類型轉換成數字。  
      * 主要使用在顯示格式的控制以及排序等地方。  
      * 特別是排序的時候，因為按照字元排序和按照數字排序，結果是不同的。  

    * Oracle的to_number函數接受兩個參數。
      * 第一個參數是需要轉換的字串，
      * 第二個參數是要轉換的格式。
      * 實際使用的時候，除非以特殊的格式顯示，否則一個參數就已經足夠了。

    * PostgreSQL的to_number函數也接受兩個參數。
      * 使用的時候，不能只使用一個參數。
      * 排序的時候， 格式字串中需要設定轉換的位數為該欄位的最大位數。否則，只使用格式提供的轉換位數來排序。例如格式提供了兩位，那麼就轉換最前面的兩位字元為數字，然後再用它來排序。
26. instr函数
27. 字符串连接符( || )
    * PostgresQL 中的 || 用 法與其他資料庫不同：
      ````
      select a|| b from table1;
      ````
      當a 或b 其中一個為null 時， 查詢傳回null 
      必需改用CONCAT()

    * Since version 9.1, PostgreSQL has introduced a built-in string function called CONCAT() to concatenate two or more strings into one.  
    * https://www.linkedin.com/pulse/tutorial-you-learn-how-use-postgresql-concat-function-akash-kumar-ukefc  
    * https://www.postgresqltutorial.com/postgresql-string-functions/postgresql-concat-function/  
  
28. substr  
29. length  
30. trim/ltrim/rtrim函数  
31. NLSSORT  
32. NLS_INITCAP/ NLS_LOWER/ NLS_UPPER  
33. regexp_replace  
34. regexp_substr  
35. regexp_instr  
36. regexp_like  
37. NVL2  
38. LNNVL  
39. BITAND  
40. REMAINDER  
41.     
 


參考資料來源： 
* https://severalnines.com/blog/migrating-oracle-postgresql-what-you-should-know/
* https://www.omniwaresoft.com.tw/product-news/edb-news/oracle-to-postgresql-5-steps-3/
* https://www.cs.cmu.edu/~pmerson/docs/OracleToPostgres.pdf
* https://wiki.postgresql.org/wiki/Oracle_to_Postgres_Conversion
* https://developer.aliyun.com/search?k=PostgreSQL%E5%92%8COracle%E7%9A%84SQL%E5%B7%AE%E5%BC%82%E5%88%86%E6%9E%90%E4%B9%8B%E4%BA%94&scene=community&page=1
* https://rojerchen.blogspot.com/2019/09/postgresql-oraclefdw.html
* https://www.cnblogs.com/ios9/p/15489894.html
* 