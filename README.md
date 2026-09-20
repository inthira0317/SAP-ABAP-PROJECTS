# SAP-ABAP-PROJECTS
It includes the developed RICEF.

Reports

# ZMM_MAT_ALV – Material Purchase Order ALV Report

## 📌 Overview

`ZMM_MAT_ALV` is a **Classical ALV Report** developed in SAP ABAP to display material-wise Purchase Order, vendor, quantity, and amount information.

The report accepts **Material Number(s)** as input and retrieves relevant data from standard SAP MM tables. The output is displayed using `REUSE_ALV_GRID_DISPLAY` with sorting, grand totals, conditional cell coloring, and interactive navigation to standard SAP transactions.

---

## 🎯 Objectives

The main objectives of this report are:

- Display Purchase Order details based on Material Number.
- Retrieve material descriptions and vendor information.
- Display PO quantity and net amount.
- Calculate and display a grand total for PO amount.
- Apply conditional cell coloring to the PO amount.
- Provide interactive navigation to standard SAP transactions.
- Demonstrate Classical ALV reporting concepts in ABAP.

---

## ✨ Features

- Material-based selection screen
- Open SQL with multiple table joins
- Classical ALV using `REUSE_ALV_GRID_DISPLAY`
- Custom field catalog
- Sorting by Material Number and PO Number
- Grand total for PO Amount
- Conditional cell coloring
- Zebra pattern for better readability
- Automatic column width optimization
- Interactive double-click navigation
- Navigation to:
  - `MM03` – Display Material
  - `ME23N` – Display Purchase Order
  - `XK03` – Display Vendor
- Modular programming using `FORM` and `PERFORM`

---

## 🗃️ SAP Tables Used

| Table | Description | Purpose |
|---|---|---|
| `EKPO` | Purchasing Document Item | Material, PO item, quantity, amount, short text |
| `EKKO` | Purchasing Document Header | PO header and vendor information |
| `MAKT` | Material Description | Material description |
| `LFA1` | Vendor Master | Vendor name and GST-related information |

### Table Relationships

```text
EKPO
 │
 ├── EBELN ──────> EKKO
 │                  │
 │                  └── LIFNR ──────> LFA1
 │
 └── MATNR ──────> MAKT
```

---

## 📊 Output Fields

The ALV output contains the following information:

| Field | Description |
|---|---|
| Material Number | SAP Material Number |
| Material Description | Description of the material |
| PO Number | Purchase Order Number |
| PO Item | Purchase Order Item |
| Short Text | PO Item Short Text |
| Quantity | Ordered Quantity |
| Net Amount | PO Item Net Amount |
| Vendor Code | Vendor Number |
| Vendor Name | Vendor Name |
| Vendor GST | Vendor GST-related information |

---

## 🔎 Selection Screen

The report uses:

```abap
SELECT-OPTIONS:
  s_matnr FOR ekpo-matnr.
```

This allows the user to enter:

- Single material
- Multiple materials
- Material ranges
- Multiple selection values

Example:

```text
Material Number:
10000001
10000002
10000010 - 10000020
```

---

## 🛠️ Open SQL Concepts Used

The report retrieves data using Open SQL and joins multiple SAP standard tables.

### Join Logic

```abap
FROM ekpo
INNER JOIN ekko
  ON ekpo~ebeln = ekko~ebeln

LEFT OUTER JOIN makt
  ON ekpo~matnr = makt~matnr
 AND makt~spras = @sy-langu

LEFT OUTER JOIN lfa1
  ON ekko~lifnr = lfa1~lifnr
```

### Selection Conditions

```abap
WHERE ekpo~matnr IN @s_matnr
  AND ekpo~menge > 0
  AND ekpo~netwr > 0
```

This ensures that only relevant material and purchasing records are displayed.

---

# 📋 ALV Implementation

The report uses **Classical ALV**:

```abap
REUSE_ALV_GRID_DISPLAY
```

The ALV is configured using:

- Field catalog
- Sort table
- Layout structure
- Callback user command
- Cell color table

---

## 🧾 Field Catalog

The field catalog defines how each column behaves in the ALV.

For example:

```abap
gs_fcat-fieldname = 'PO_AMOUNT'.
gs_fcat-seltext_m = 'PO Amount'.
gs_fcat-do_sum    = 'X'.
```

`DO_SUM = 'X'` enables the **grand total** for the PO Amount column.

---

## ➕ Grand Total

The report calculates the grand total of:

```text
PO Amount
```

The total is displayed automatically by ALV using:

```abap
DO_SUM = 'X'
```

Quantity is displayed without a grand total.

---

# 🎨 Conditional Cell Coloring

The report implements **cell-level coloring** using:

```abap
SLIS_T_SPECIALCOL_ALV
```

A separate field is maintained in the output structure:

```abap
CELL_COLOR TYPE SLIS_T_SPECIALCOL_ALV
```

The ALV layout is configured using:

```abap
gs_layout-coltab_fieldname = 'CELL_COLOR'.
```

### Example Logic

```abap
IF gs_output-po_amount > 5000.

  ls_color-fieldname = 'PO_AMOUNT'.
  ls_color-color-col = 6.
  ls_color-color-int = 1.
  ls_color-color-inv = 0.

ELSE.

  ls_color-fieldname = 'PO_AMOUNT'.
  ls_color-color-col = 5.
  ls_color-color-int = 1.
  ls_color-color-inv = 0.

ENDIF.
```

This demonstrates how ALV cells can be formatted dynamically based on business conditions.

---

# 🔀 Sorting

The ALV output is sorted using:

```abap
SLIS_T_SORTINFO_ALV
```

The report sorts by:

1. Material Number
2. Purchase Order Number

Example:

```text
Material 10000001
   PO 4500000010
   PO 4500000012

Material 10000002
   PO 4500000015
   PO 4500000018
```

---

# 🖱️ Interactive Navigation

One of the important features of this report is **interactive ALV navigation**.

The user can double-click specific fields to navigate directly to standard SAP transactions.

### Material Number → MM03

```abap
SET PARAMETER ID 'MAT'
  FIELD gs_output-matnr.

CALL TRANSACTION 'MM03'
  AND SKIP FIRST SCREEN.
```

Used for:

> Display Material

---

### Purchase Order → ME23N

```abap
SET PARAMETER ID 'BES'
  FIELD gs_output-po_no.

CALL TRANSACTION 'ME23N'
  AND SKIP FIRST SCREEN.
```

Used for:

> Display Purchase Order

---

### Vendor → XK03

```abap
SET PARAMETER ID 'LIF'
  FIELD gs_output-vendor_code.

CALL TRANSACTION 'XK03'
  AND SKIP FIRST SCREEN.
```

Used for:

> Display Vendor

> **Note:** In S/4HANA systems, vendor master processes may use **Business Partner (BP)** instead of the classic `XK03` transaction.

---

# 🔄 Program Flow

```text
                    START
                      │
                      ▼
              Selection Screen
                      │
                      ▼
                 GET_DATA
                      │
                      ▼
              Fetch SAP Data
                      │
                      ▼
          Build Field Catalog
                      │
                      ▼
          Build Sort Criteria
                      │
                      ▼
              Set ALV Layout
                      │
                      ▼
          Display Classical ALV
                      │
                      ▼
             User Double Click
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
        MM03        ME23N       XK03
       Material       PO        Vendor
```

---

# 🧩 Program Modularization

The report is divided into separate `FORM` routines.

### 1. `GET_DATA`

Responsible for:

- Reading database records
- Joining SAP tables
- Applying selection conditions
- Filling the output internal table
- Applying conditional cell colors

### 2. `BUILD_FIELDCAT`

Responsible for:

- Creating the ALV field catalog
- Defining column headings
- Defining totals
- Setting hotspots/key fields

### 3. `BUILD_SORT_CRITERIA`

Responsible for:

- Defining ALV sorting

### 4. `SET_LAYOUT`

Responsible for:

- Zebra pattern
- Automatic column width
- Cell color configuration

### 5. `DISPLAY_ALV`

Responsible for:

- Calling `REUSE_ALV_GRID_DISPLAY`
- Passing field catalog
- Passing layout
- Passing sorting information
- Registering the user-command callback

### 6. `USER_COMMAND`

Responsible for:

- Detecting double-click events
- Reading the selected ALV row
- Navigating to standard SAP transactions

---

# 🧠 ABAP Concepts Demonstrated

This project demonstrates several important SAP ABAP concepts.

### Core ABAP

- `REPORT`
- `TABLES`
- `TYPES`
- `DATA`
- Structures
- Internal Tables
- Work Areas
- `LOOP AT`
- `READ TABLE`
- `MODIFY`
- `APPEND`
- `SORT`
- `CLEAR`
- `IF / ELSE`
- `CASE`
- `FORM / PERFORM`

### Open SQL

- `SELECT`
- `INNER JOIN`
- `LEFT OUTER JOIN`
- `WHERE`
- `IN`
- Host variables using `@`
- `INTO CORRESPONDING FIELDS OF TABLE`

### Selection Screen

- `SELECT-OPTIONS`
- Range-based selection

### Classical ALV

- `REUSE_ALV_GRID_DISPLAY`
- Field Catalog
- Layout
- Sort Criteria
- Hotspot
- Key Field
- Grand Total
- Cell Coloring
- User Command

### SAP Integration

- `SET PARAMETER ID`
- `CALL TRANSACTION`
- `AND SKIP FIRST SCREEN`

---

# 🏗️ Technical Architecture

```text
                 SAP ABAP Report
                       │
                       ▼
              Selection Screen
                       │
                       ▼
                 Open SQL Query
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
      EKPO            EKKO            MAKT
        │              │
        │              ▼
        │             LFA1
        │
        └──────────────┬──────────────┘
                       ▼
                Internal Table
                       │
                       ▼
                Classical ALV
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
      MM03           ME23N           XK03
```

---

# ▶️ How to Execute

### Step 1 – Open SAP GUI

Go to:

```text
SE38
```

or

```text
SE80
```

### Step 2 – Enter Program

```text
ZMM_MAT_ALV
```

### Step 3 – Execute

Press:

```text
F8
```

### Step 4 – Enter Material

Enter one or more Material Numbers.

### Step 5 – Execute

The Classical ALV output will be displayed.

### Step 6 – Interactive Navigation

Double-click:

```text
Material Number → MM03
PO Number        → ME23N
Vendor Code      → XK03
```

---

# 📌 Sample Output

```text
Material    Description       PO Number     Item    Quantity    PO Amount
--------------------------------------------------------------------------------
10000001    Bearing            4500000010    10      25          4,500.00
10000001    Bearing            4500000012    20      50          7,500.00
10000002    Motor              4500000015    10      10          6,000.00
--------------------------------------------------------------------------------
                                                    Grand Total   18,000.00
```

The PO Amount column is conditionally highlighted based on the configured amount threshold.

---

# 📚 Key SAP MM Concepts

This report also provides practical exposure to SAP MM purchasing data.

### Purchase Order

A Purchase Order is represented through:

```text
EKKO → PO Header
EKPO → PO Item
```

### Vendor

Vendor information is retrieved using:

```text
EKKO-LIFNR
       │
       ▼
LFA1-LIFNR
```

### Material

Material information is retrieved using:

```text
EKPO-MATNR
       │
       ▼
MAKT-MATNR
```

---

# 💡 What I Learned From This Project

Through this report, I worked with:

- Classical ALV reporting
- SAP MM database tables
- Open SQL joins
- Internal table processing
- ALV field catalogs
- Dynamic cell formatting
- Sorting and aggregation
- Selection-screen programming
- Interactive ALV events
- Standard SAP transaction integration
- Modular ABAP programming
- Debugging and data validation

---

# 🚀 Future Enhancements

Possible enhancements include:

- Add Vendor and PO selection options
- Add date-based filtering
- Add Purchasing Organization filtering
- Add Plant filtering
- Add Excel export
- Add ALV layout variants
- Add authorization checks
- Add Business Partner navigation for S/4HANA
- Add dynamic user-configurable amount thresholds
- Add additional purchasing document information


