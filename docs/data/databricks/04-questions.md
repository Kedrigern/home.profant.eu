---
title: DBX questions
---

## Links

https://www.testpreplab.com/Databricks-Certified-Data-Engineer-Associate-free-practice-test
https://www.examcatalog.com/exam/databricks/certified-data-engineer-associate/

## Questions

#### Compute for...

> An analyst runs frequent ad hoc queries on a large Delta Lake dataset using Databricks. However, the analyst is experiencing slow query performance. They need to achieve quick, interactive responses for exploratory analysis by relying on cached data.
>
> Which instance type is best suited for this workload?
>
> a) Storage Optimized  
> b) Memory Optimized  
> c) General Purpose  
> d) Compute Optimized

/// details | Answer

**a) Storage optimized**

Ad hoc and interactive analysis benefits greatly from Storage Optimized instances, which can leverage Delta caching to accelerate performance. These instance types are tailored for high disk throughput, enabling faster reads and reduced latency for exploratory queries.

///

#### Warehouse solution

> Which of the following services provides a data warehousing solution in the Databricks Intelligence Platform?
>
> a) SQL warehouse  
> b) Databricks SQL  
> c) Unity Catalog  
> d) Delta Lives Tables (DLT)

/// details | Answer

**b) Databricks SQL**

Otázka se ptá na službu (service) / řešení, které poskytuje funkcionalitu datového skladu (Data Warehousing).

Databricks SQL: Toto je název celého produktu (služby) v rámci Databricks platformy.

SQL warehouse: Toto je výpočetní zdroj (compute), který běží uvnitř služby Databricks SQL.
///

#### Data Skew & Join Optimization

> A data engineer is analyzing a dataset of clickstream events from a high-traffic website. The dataset includes fields such as user_id, timestamp, event_type, and page_url. During a join operation between the clickstream logs and a user profile dataset (joined on user_id), the job’s performance is significantly hindered due to uneven data distribution. Further analysis confirms a data skew caused by a small subset of users generating a disproportionately large number of events.
>
> Which of the following approaches is NOT an appropriate solution to mitigate the skew in this scenario?
>
> a) Broadcast the skewed keys to all worker nodes to avoid shuffle during the join.  
> b) Separate processing of skewed keys by handling high-frequency users in a dedicated job.  
> c) Repartition the clickstream dataset to increase the number of partitions before the join.  
> d) Use salting by appending a random prefix to skewed user_id values to distribute the load across partitions.

/// details | Answer

**a) Broadcast the skewed keys to all worker nodes to avoid shuffle during the join.**

Broadcast join je skvělý pro *malé* tabulky. Tato možnost ale navrhuje broadcastovat *skewed keys* (což je ta obrovská, problematická část dat) na všechny workery. To by okamžitě zahltilo paměť (RAM) na workerech a způsobilo pád (Out Of Memory).

Ostatní možnosti (Salting, oddělené zpracování) jsou validní řešení skew. Repartitioning sice skew přímo neřeší (data jednoho uživatele stále končí spolu), ale na rozdíl od možnosti A nezpůsobí pád clusteru, proto je v kontextu otázky "akceptovatelný".
///

#### DLT Execution & Deployment Modes

> The data engineer team has a DLT pipeline that updates all the tables at defined intervals until manually stopped. The compute resources terminate when the pipeline is stopped.
>
> Which of the following best describes the execution modes of this DLT pipeline?
>
> a) The DLT pipeline executes in Continuous Pipeline mode under Development mode.  
> b) The DLT pipeline executes in Triggered Pipeline mode under Development mode.  
> c) The DLT pipeline executes in Continuous Pipeline mode under Production mode.  
> d) The DLT pipeline executes in Triggered Pipeline mode under Production mode.

/// details | Answer

**c) The DLT pipeline executes in Continuous Pipeline mode under Production mode.**

**Continuous:** Poznáme podle "updates... until manually stopped". Triggered by se spustil jednou a vypnul.
**Production:** Poznáme podle chování clusteru ("resources terminate when stopped"). V Development módu by cluster často zůstal běžet "naprázdno" (idle) pro rychlejší restart při ladění. Production mód šetří zdroje a vypíná je hned, jak nejsou potřeba (zde po manuálním stopnutí pipeline).
///

#### Data Layout: Liquid Clustering vs. Partitioning

> A data engineering team is working on a user activity events table stored in Unity Catalog. Queries often involve filters on multiple columns like `user_id` and `event_date`.
>
> Which data layout technique should the team implement to avoid expensive table scans?
>
> a) Use liquid clustering on the combination of `user_id` and `event_date`
> b) Use partitioning on the `event_date` column.  
> c) Use Z-order indexing on the `user_id`  
> d) Use partitioning on the `user_id` column, along with Z-order indexing on the `event_date` column.  

/// details | Answer

**a) Use liquid clustering on the combination of user_id and event_date **

In this scenario, using liquid clustering on the combination of user_id and event_date is the best choice to avoid expensive scans. This technique incrementally optimizes data layout based on both columns, efficiently supporting filters on these columns and avoiding costly table scans.

Partitioning only on event_date helps queries filtering by date but doesn’t optimize filtering by user_id, leading to potential full scans within partitions. Z-order indexing on user_id optimizes queries filtering on user_id but ignores event_date filtering, resulting in inefficient scans when filtering by date. Lastly, partitioning on user_id + Z-order on event_date supports filtering on both columns but can create many small partitions (if users are numerous), causing management and performance issues.

**Proč je to správně:**
Liquid Clustering je moderní náhrada za Partitioning a Z-Order (od Databricks Runtime 13.3+). Je to **doporučená metoda** pro nové tabulky. Umí efektivně clusterovat data podle více sloupců najednou, aniž by trpěla problémy s "High Cardinality" (velké množství unikátních hodnot).

**Proč je možnost D ("Partitioning on user_id") nebezpečný chyták:**
Partitioning (adresářové dělení) je vhodné jen pro sloupce s nízkou kardinalitou (např. datum, země). Pokud uděláš partition podle `user_id` (kde jsou miliony uživatelů), vytvoříš miliony malých složek a souborů (**Small Files Problem**). To drasticky *zpomalí* výkon, místo aby ho to zrychlilo. Liquid Clustering tento problém nemá.
///
