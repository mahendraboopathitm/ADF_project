# 📘 Azure Data Factory Pipeline  
## **Pipeline 1: File Classification Based on File Size**

---

## 📝 Overview

This Azure Data Factory pipeline is designed to **classify incoming files based on their size** and route them into different folders or processing paths.

The pipeline supports three categories:

- **Category A:** Files **≤ 1 KB**  
- **Category B:** Files **> 1 KB and ≤ 2 KB**  
- **Category C:** Files **> 2 KB**


# 📘 Azure Data Factory Pipeline  
## **Pipeline 1: File Classification Based on File Size**

---

## 🏗️ Pipeline Architecture

```
flowchart TD
    A[Get Metadata (List of Files)] --> B[ForEach - Loop Through Each File]
    B --> C[Get Metadata (File Size)]
    C --> D{Check File Size}
    
    D -->|<=1 KB| E[Copy to 1KB Folder]
    D -->|>1 KB and <=2 KB| F[Copy to 2KB Folder]
    D -->|>2 KB| G[Copy to Above2KB Folder]
````



## 🔧 Activities Used

### **1️⃣ Get Metadata (Folder Level)**

**Purpose:** Retrieve a list of all files in the source container
**Metadata retrieved:** `childItems`

---

### **2️⃣ ForEach Activity**

* Iterates through each file returned by Get Metadata
* Passes each file name dynamically to inner activities

---

### **3️⃣ Get Metadata (File Level)**

**Purpose:** Retrieve the size of each file
**Metadata retrieved:** `size`

---

### **4️⃣ If Condition Activity**

Used to evaluate file size and classify it into the correct category.

#### **Condition 1: Files ≤ 1 KB**

```json
@lessOrEquals(item().size, 1024)
```

#### **Condition 2: Files > 1 KB and ≤ 2 KB**

```json
@and(greater(item().size, 1024), lessOrEquals(item().size, 2048))
```

#### **Condition 3: Files > 2 KB**

```json
@greater(item().size, 2048)
```

---

### **5️⃣ Copy Data Activity**

Files are copied into their respective folders based on size.

| Category | Condition       | Target Folder |
| -------- | --------------- | ------------- |
| L1       | ≤ 1 KB          | `/1kb/`       |
| L2       | >1 KB and ≤2 KB | `/2kb/`       |
| L3       | >2 KB           | `/above2kb/`  |

---

## 📁 Folder Structure Example

```
/source-files
    file1.csv
    file2.csv
    file3.csv

/classified-files
    /1kb
    /2kb
    /above2kb
```

---

## ⭐ Summary

This pipeline automates file organization by classifying files based on their size.
It uses a metadata-driven approach with:

**Get Metadata → ForEach → Size Check → Copy**

This allows files to be routed into their respective folders for downstream processing.

---

```


.

