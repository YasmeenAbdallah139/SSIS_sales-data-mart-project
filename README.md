# SSIS_sales-data-mart-project-using-
# Telecom ETL Project (SSIS)

## 📌 Project Overview
This project implements an **ETL (Extract, Transform, Load)** pipeline for a telecom company using **SQL Server Integration Services (SSIS)**.  

The company generates a **CSV file every 5 minutes** that contains customer activity data. The goal of this ETL solution is to process, validate, and transform this raw data before storing it in a **SQL Server Data Warehouse**.

Additionally:
- **Valid records** are loaded into the target database.  
- **Rejected records** (due to validation or business rules) are stored in a separate table along with the **source file name**.  
- After processing, the source CSV file is archived in a different folder.  

---

## 📂 Source Data
The input is a **CSV file** with the following structure:

| Column Name | Data Type  | Nullable | Example            |
|-------------|------------|----------|--------------------|
| ID          | INT        | No       | 156                |
| IMSI        | STRING(9)  | No       | 310120265          |
| IMEI        | STRING(14) | Yes      | 490154203237518    |
| CELL        | INT        | No       | 123                |
| LAC         | INT        | No       | 12                 |
| EVENT_TYPE  | STRING(1)  | Yes      | 1                  |
| EVENT_TS    | TIMESTAMP  | No       | 16/1/2020 7:45:43  |

---

## 🔄 Transformation Rules

| Source Column | Transformation Rule | Target Column |
|---------------|---------------------|---------------|
| **ID**        | As-is | `transaction_id` |
| **IMSI**      | Reject if NULL | `IMSI` |
| **IMSI**      | Lookup `subscriber_id` from IMSI reference table. If not found → `-99999` | `subscriber_id` |
| **IMEI**      | First 8 chars. If NULL or length < 14 → `-99999` | `TAC` |
| **IMEI**      | Last 6 chars. If NULL or length < 14 → `-99999` | `SNR` |
| **IMEI**      | As-is | `IMEI` |
| **CELL**      | As-is | `CELL` |
| **LAC**       | Reject if NULL | `LAC` |
| **EVENT_TYPE**| As-is | `EVENT_TYPE` |
| **EVENT_TS**  | Validate timestamp format. Reject if invalid or NULL | `EVENT_TS` |

---

## 🚫 Rejected Records
- Stored in a **separate table** with:
  - All source columns  
  - Reason for rejection  
  - Source file name  

---

## 📦 ETL Workflow (SSIS)
1. **Extract**  
   - Read CSV files from the landing folder.  

2. **Transform**  
   - Apply all validation and transformation rules.  
   - Split records into **valid** and **rejected** outputs.  

3. **Load**  
   - Insert valid records into the **fact/transaction table**.  
   - Insert rejected records into the **rejected table** with filename.  

4. **Post-Processing**  
   - Move processed CSV files into an **archive folder**.  

---

## 🛠 Tools & Technologies
- **SQL Server Integration Services (SSIS)**  
- **SQL Server (DWH & reference tables)**  
- **CSV as source system**  
- **GitHub for version control**  

---

## 📁 Repository Structure
```
Telecom-ETL-SSIS/
│
├── ETL_Solution/           # SSIS project & packages
├── SQL_Scripts/            # DWH table creation & reference scripts
├── Sample_Files/           # Example CSV files
├── Archive_Files/          # Processed files (post-execution)
└── README.md               # Project documentation
```

---

## 🚀 How to Run
1. Clone the repo:
   ```bash
   git clone https://github.com/<your-username>/Telecom-ETL-SSIS.git
   ```
2. Open the **SSIS project** in **SQL Server Data Tools (SSDT) / Visual Studio**.  
3. Configure connection managers (point to your SQL Server & file directories).  
4. Deploy and execute the SSIS package.  
5. Monitor database tables (`transactions`, `rejected_records`) and archived files.  

---

## 📖 Notes
- Make sure reference table for **IMSI → subscriber_id** is preloaded.  
- Adjust folder paths in **File System Task** for your environment.  
- Scheduling can be done using **SQL Server Agent** to run every 5 minutes.  
