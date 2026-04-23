# MIST4610 Project 2 Group B3

## Team Name: 
47114 Group B3

## Team Members:
1. Landon Johnson: Data Cleaner
2. Yash Kirtane: Group Leader
3. Chris Ennis: Conceptual Modeler
4. Charan Nuthalapati: SQL Writer
5. Lawrence Carpenter: Data Quality

## Case summary: brief explanation of the company and data challenges.

Northline Outfitters is a small online retailer selling student-friendly lifestyle and tech accessories to customers in the United States and Canada. The company sources products from outside vendors and currently manages all records in Excel rather than a proper database.
The data presented significant challenges including inconsistent date and price formats, customer information stored in a single text field, mixed discount and tax formats, duplicate product rows, SKU capitalization inconsistencies, and measurements recorded in a mix of imperial and metric units. These issues required extensive cleaning and restructuring before the data could be loaded into a relational database.

## Conceptual model: PNG of the model plus a short English explanation of the entities & relationships.
<img width="797" height="796" alt="4610_project2_datamodel" src="https://github.com/user-attachments/assets/fea16c65-9da0-4628-a90e-d7634c0de087" />

## Data quality assessment: identify and explain the main data quality issues in the source file.

The source data consists of two sheets exported from Northline Outfitters' Excel-based operations: Sales_Dump (200 rows, 21 columns) and Product_Supplier_Master (60 rows, 16 columns). Both sheets contain significant quality issues that prevent them from being used directly in a relational database.

### Issue 1 – Inconsistent Date Formats (Sales_Dump.sale_date)
The sale_date column contains at least seven distinct date formats mixed across rows, reflecting inconsistent data entry over time. Examples found in the data include 10-11-2025 (U.S. MM-DD-YYYY), 31/10/2025 (Canadian DD/MM/YYYY), Oct 17 25 (abbreviated month with two-digit year), October 5 25 (full month with two-digit year), October 10 2025 (full month with four-digit year), 10 Sep 2025 (day-first with abbreviated month), and 10/14/2025 (U.S. slash format). Because Northline ships to both the U.S. and Canada, both MM/DD/YYYY and DD/MM/YYYY conventions appear legitimately in the data, making ambiguous values like 10/11/2025 particularly problematic. Without standardization, sorting, filtering, and any date-based reporting such as monthly revenue trends would be unreliable.

### Issue 2 – Composite customer_info Field (Sales_Dump.customer_info)
Customer details were entered as a single free-text field using at least three different delimiters (semicolon, pipe, and forward slash). The field may contain the customer's name alongside notes about loyalty status, student status, and checkout type. Additionally, the same customer appears with multiple different email addresses across orders and there is no stable customer ID.

### Issue 3 – Inconsistent Payment Method Values (Sales_Dump.payment_method)
The same payment types appear in different cases and abbreviations. VISA and visa represent the same method, Debit and debit represent the same method, and MC and Mastercard represent the same brand.

### Issue 4 – Mixed Discount Formats (Sales_Dump.discount)
The discount column contains five distinct formats across the dataset, with 32 of 200 rows having no value at all. Some entries use a percent symbol, others store the value as a bare number, and others use text-based codes such as promo5 or student 10% that require interpretation before they can be used in any calculation. Zero-discount rows are recorded as 0 in some cases and left NULL in others.

### Issue 5 – Mixed Tax Formats (Sales_Dump.tax)
The tax column contains at least five distinct formats across the dataset, with 33 rows having no value at all. Some entries use a percent symbol, others store the equivalent decimal rate, and others include a label alongside the value such as HST 13%. The same underlying tax rate can appear in multiple forms within the same column.

### Issue 6 – Currency Code Embedded in Price Fields (Sales_Dump.unit_price, Product_Supplier_Master.cost, list_price)
Unit prices in Sales_Dump are stored as strings with the currency code as a prefix such as USD 18.99 or CAD 46.99, making the column non-numeric. In Product_Supplier_Master, cost and list_price are inconsistent — some rows have the currency prefix and others are bare numeric values with no currency indicated.

### Issue 7 – Missing and Prefixed line_total Values (Sales_Dump.line_total)
62 of 200 rows have a NULL line_total. Among the rows that do have a value, some include a dollar sign prefix while others are plain numeric strings. The column is stored as text rather than a numeric type.

### Issue 8 – Inconsistent Country Values (Sales_Dump.ship_country)
Four distinct strings represent what should be only two countries, and 3 rows have no value at all. US and USA both represent the United States and CA and Canada both represent Canada. The order_id column encodes the country in its prefix which can be used to recover the 3 missing values.

### Issue 9 – Student Flag Embedded in Category (Sales_Dump.category, Product_Supplier_Master.category)
In Sales_Dump, category values like Tech / Student and Accessories / Student blend the actual product category with a student discount flag. The 14 distinct values in Sales_Dump reduce to 7 base categories once the student annotation is stripped. In Product_Supplier_Master the problem is worse with 20 distinct category strings due to additional inconsistencies including wrong separators, different connector words, typos, and self-referential entries such as Accessories / Accessories.

### Issue 10 – SKU Case Inconsistency (Both Sheets)
The same product SKU appears in both uppercase and lowercase across rows in both sheets. For example SKU-C-1012 and sku-c-1012 refer to the same product. This breaks any join between Sales_Dump and Product_Supplier_Master.

### Issue 11 – Duplicate and Variant Rows in Product_Supplier_Master
The Product_Supplier_Master sheet has 60 rows but only 20 distinct base products. The rows fall into three patterns: true duplicates with minor field differences, color or size variants sharing the same base SKU number, and student edition variants as separate offerings under the same base SKU.

### Issue 12 – Vendor Name Inconsistencies (Product_Supplier_Master.vendor_name)
Urban Source and Urban Sources appear as two distinct vendor names but are clearly the same supplier with the same phone number and same rep. Phone numbers for the same vendor are also formatted inconsistently and vendor rep entries sometimes append notes directly in the name field.

### Issue 13 – reorder_level Contains Text (Product_Supplier_Master.reorder_level)
Most reorder level values are integers but some rows contain the word ten instead of the number 10. Six rows have no value.

### Issue 14 – Mixed Units in size_or_weight, weight, and length
Weight and size information in both sheets is stored as free text in a mixture of imperial and metric units. In Sales_Dump, size_or_weight conflates weight and size into a single field with no consistent type and 23 rows have no value.
