

# 📘 Azure Data Factory Pipeline  
## **Pipeline 2: Union of Existing and Incoming Data With Deduplication**

---

## 📝 Overview

This Azure Data Factory Mapping Data Flow is designed to:

1. Read **existing full dataset**
2. Read **incoming new dataset**
3. **Union** both datasets
4. **Remove duplicates based on ID**
5. **Keep the incoming/latest record when duplicates occur**
6. Write the final **deduplicated master dataset** to the sink

This ensures that your final table always contains the **most recent version** of each record, based on the unique `ID`.

---

## 🏗️ Pipeline Architecture

```
flowchart TD
    A[Existing Data Source - allempsource] --> B[Aggregate Existing Data]
    H[Incoming Data Source - upcomedata] --> C[Aggregate Incoming Data]

    B --> D[Select Required Columns]
    C --> D

    D --> E[Join & Compare]
    E --> F[Derived Column - prepare metadata]

    F --> G[Exists - Identify Duplicates]
    G --> I[Surrogate Key]

    I --> J[CrossJoin - Combine matching logic]
    J --> K[Solution - Apply Deduplication Rules]
    K --> L[Select Final Columns]
    L --> M[Sink - Final Master Table]
```



## 🔧 Activities Used and What They Do

### **1️⃣ Existing Data Source (allempsource)**

Contains your **historical records**.

### **2️⃣ Incoming Data Source (upcomedata)**

Contains **new records** that need to be merged.

---

## **3️⃣ Aggregate Transformations (aggregate1 & aggregate2)**

Purpose:

* Group by **ID**
* Ensure duplicates within each dataset are cleaned
* Produce a unique list of IDs before union

---

## **4️⃣ Select Transformation**

Filters and selects only required columns for merging.

---

## **5️⃣ Derived Column (derivedColumn1)**

Used for:

* Creating hash keys (optional)
* Setting update flags
* Adding metadata (load date, source flag, etc.)

---

## **6️⃣ Exists Transformation (exists1)**

This step identifies:

* **Duplicate IDs** between incoming and existing data
* **Unique new IDs** (forward insert)
* **Records that should be replaced**

⭐ **Key Logic:**
If an ID exists in both datasets → **incoming record is kept**, existing record is removed.

This ensures the final table always holds the **latest version** of the record.

---

## **7️⃣ Surrogate Key Transformation**

Generates a new unique identifier for each row (if needed for dimension tables).

---

## **8️⃣ CrossJoin Transformation**

Used to combine:

* Deduped incoming records
* Aggregated metadata from existing records

This helps build the final merge logic.

---

## **9️⃣ Solution Transformation**

This is the core of your logic:

### It decides:

* Which record to keep
* Whether to override an existing record
* Which records should move to output

It implements:

```
Union(existing + incoming)
Remove duplicates using ID
Keep incoming record when ID matches
```

---

## **🔟 Final Select → Sink**

Only cleaned, deduplicated, latest records are written to the final sink table.

---

## 📁 Input / Output Example

### **Existing Data (allempsource)**

| ID  | Name | Salary |
| --- | ---- | ------ |
| 101 | Maha | 25000  |
| 103 | John | 40000  |

### **Incoming Data (upcomedata)**

| ID  | Name     | Salary |
| --- | -------- | ------ |
| 101 | Mahendra | 30000  |
| 102 | Prabhu   | 35000  |

---

### **Union Result (before dedupe)**

| ID  | Name     |
| --- | -------- |
| 101 | Maha     |
| 103 | John     |
| 101 | Mahendra |
| 102 | Prabhu   |

---

### **Final Output (after dedupe)**

Incoming record wins when duplicate ID exists.

| ID  | Name     | Salary |
| --- | -------- | ------ |
| 101 | Mahendra |        |
| 102 | Prabhu   |        |
| 103 | John     |        |

---

## ⭐ Summary (What This Pipeline Does)

* Combines **existing + new** datasets
* Removes all duplicates using **ID** as the primary key
* Keeps **incoming/latest record** when conflicts exist
* Ensures master table always stays updated and clean
* Prepares data for analytical or operational consumption

This pipeline is important for:

* Incremental loads
* Master data maintenance
* Dimension table preparation
* Removing duplicates
* Keeping the latest record automatically

---

```

---


