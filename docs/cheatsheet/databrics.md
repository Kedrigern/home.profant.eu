# Databricks

## Základní koncepty

* **Data Warehouse (Datový sklad):** Centralizované úložiště pro **strukturovaná, vyčištěná data**. Optimalizováno pro business intelligence (BI) a reporting. Schéma je pevně dané (schema-on-write).
* **Data Lake (Datové jezero):** Centralizované úložiště pro **jakákoliv data v surové podobě** (strukturovaná, nestrukturovaná). Flexibilní, levné. Schéma se aplikuje až při čtení (schema-on-read).
* **Lakehouse:** **Kombinace Data Warehouse a Data Lake.** Přináší spolehlivost a výkon (jako Warehouse) přímo nad levné úložiště a surová data (jako Lake).

***

## Klíčové technologie

* **Apache Spark:** **Výpočetní engine** pro distribuované zpracování (OLAP). Klíčové vlastnosti: zpracování v paměti, líné vyhodnocování, paralelismus. Je to "motor" pod kapotou Databricks.
* **Parquet:** **Sloupcový formát souborů.** Optimalizován pro analytické dotazy (OLAP) a efektivní kompresi. Umožňuje číst jen ty sloupce, které jsou potřeba.
* **Delta Lake / Delta Table:** Vrstva nad Parquet soubory, která přidává vlastnosti databáze: **ACID transakce, časovou osu (time travel), a vynucování schématu.** Tvoří technický základ Lakehouse architektury.
* **Polars:** Ultrarychlá **knihovna pro zpracování dat na jednom stroji**. Využívá všechna jádra CPU a je paměťově efektivní. Ideální náhrada Pandas pro lokální analýzu dat, která se vejdou do RAM (typicky 1-100 GB).

***

## Databricks 🧱

Platforma postavená na filosofii **Lakehouse**. Je to jako flexibilní dílna s mocnými nástroji pro zpracování jakýchkoliv surovin.

* **Princip:** Jednotné webové rozhraní pro datové inženýrství, datovou vědu i machine learning.
* **Architektura (Medallion):** Postupné pročištění dat přes tři vrstvy:
    * **Bronze:** Surová data, archiv "tak, jak to přišlo".
    * **Silver:** Očištěná, validovaná, jednotná tabulka. "Jeden zdroj pravdy".
    * **Gold:** Agregované, byznys-orientované tabulky připravené pro reporting a analýzu.
* **Nejlepší pro:** Komplexní ETL/ELT pipelines, machine learning, zpracování nestrukturovaných dat. Ideální pro týmy, kde je silný **Python/Scala**.

***

## Snowflake ❄️

Moderní **Cloud Data Warehouse** dodávaný jako služba (SaaS). Je jako dokonale organizovaný sklad, optimalizovaný pro jednoduchost a výkon.

* **Princip:** Oddělení úložiště od výpočetních zdrojů. Více týmů může pracovat nad stejnými daty bez vzájemného ovlivňování.
* **Architektura:** Unikátní třívrstvá architektura (úložiště, výpočetní zdroje, cloudové služby).
* **Silné stránky:**
    * **SQL je král:** Vše je navrženo primárně pro SQL.
    * **Nativní podpora JSON:** Jednoduché dotazování nad polostrukturovanými daty.
    * **Data Sharing:** Bezpečné a okamžité sdílení živých dat s partnery.
* **Nejlepší pro:** Business Intelligence, reporting, ad-hoc analýzy. Ideální pro týmy, kde je silné **SQL**.
