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

## Conceptual model: PNG of the model plus a short English explanation of the entities & relationships.
<img width="797" height="796" alt="4610_project2_datamodel" src="https://github.com/user-attachments/assets/fea16c65-9da0-4628-a90e-d7634c0de087" />

## Data quality assessment: identify and explain the main data quality issues in the source file.

The following steps were taken to resolve the 14 data quality issues identified in the source file. Most of the cleaning was performed manually in Microsoft Excel using a combination of Find and Replace, Text to Columns, built-in functions, and manual corrections.

### Issue 1 – Inconsistent Date Formats (Sales_Dump.sale_date)
All date values were parsed using Python's dateutil.parser library, which handles natural language date formats automatically. Every value was converted to the ISO standard YYYY-MM-DD format (e.g., October 5 25, 10-11-2025, and 31/10/2025 all became 2025-10-05, 2025-10-11, and 2025-10-31 respectively). After cleaning, the column contained zero null values and one consistent format across all 200 rows.

### Issue 2 – Composite customer_info Field (Sales_Dump.customer_info)
The single customer_info field was split into four separate columns using Python string parsing. The script detected whichever delimiter was present in each row (semicolon, pipe, or forward slash) and extracted the relevant pieces. The four new columns are: customer_name (the customer's full name), loyalty_flag (Y if loyalty was noted, N otherwise), student_flag (Y if student was noted, N otherwise), and checkout_type (guest if a guest checkout was noted, blank otherwise). The original customer_info column was then dropped.

### Issue 3 – Inconsistent Payment Method Values (Sales_Dump.payment_method)
All payment method values were standardized to a consistent title case format using a lookup map. Specifically: VISA and visa became Visa, MC became Mastercard, DEBIT and debit became Debit, CASH became Cash, and AMEX, Apple Pay, and Interac were normalized similarly. After cleaning, all 200 rows contain one of 7 consistent values with no nulls.

### Issue 4 – Mixed Discount Formats (Sales_Dump.discount)
All discount values were converted to a uniform decimal format representing the discount rate (e.g., 0.10 for a 10% discount). The cleaning script used a regular expression to extract the numeric value from any format — whether it was a bare number (5), a percentage (10%), or a text code (promo5, student 10%). Any value greater than 1 was divided by 100 to convert it to a decimal. Null values and blanks were treated as no discount and filled with 0.0.

### Issue 5 – Mixed Tax Formats (Sales_Dump.tax)
All tax values were converted to a uniform decimal format using the same approach as discounts — a regular expression extracted the numeric portion from formats like 13%, HST 13%, and 0.13, and values greater than 1 were divided by 100. The 33 rows with no tax value were left as NULL rather than filled with zero, because a missing tax entry means the rate is unknown, which is different from a tax-exempt order.

### Issue 6 – Currency Code Embedded in Price Fields (Sales_Dump.unit_price, Product_Supplier_Master.cost, list_price)
In Sales_Dump, the unit_price column was split into two new columns: unit_price_amount (numeric) and unit_price_currency (USD or CAD), by stripping the currency prefix using a regular expression. The same approach was applied to the cost and list_price columns in Product_Supplier_Master, creating cost_amount, cost_currency, list_price_amount, and list_price_currency. The 5 rows in Product_Supplier_Master where the currency prefix was missing were recovered using the SKU prefix — SKUs containing -C- were assigned CAD and SKUs containing -U- were assigned USD.

### Issue 7 – Missing and Prefixed line_total Values (Sales_Dump.line_total)
Dollar sign prefixes were stripped from all line_total values and the column was converted to a numeric type. The 62 rows with a missing line_total were then recalculated using the formula: unit_price_amount × quantity × (1 − discount) × (1 + tax). For rows where tax was NULL, a tax value of 0 was assumed for the calculation only. After cleaning, all 200 rows have a numeric line_total value.

### Issue 8 – Inconsistent Country Values (Sales_Dump.ship_country)
All country values were standardized to a two-letter code: US and USA both became US, and CA and Canada both became CA. The 3 rows with a missing ship_country were recovered by reading the order_id prefix — orders beginning with UORD- were assigned US and orders beginning with CORD- were assigned CA. After cleaning, all 200 rows contain either US or CA with no nulls.

### Issue 9 – Student Flag Embedded in Category (Sales_Dump.category, Product_Supplier_Master.category)
In Sales_Dump, any category value containing a student annotation (e.g., Tech / Student, School / Student) was split so that the base category was kept in the category column and the student indicator was merged into the student_flag column already created in Issue 2. In Product_Supplier_Master, the same stripping was applied and additional inconsistencies were corrected — wrong separators (Lifestyle , Student), alternate connectors (Tech & Student), typos (Student and apparels), and self-referential entries (Accessories / Accessories) were all normalized. After cleaning, both sheets contain exactly 7 clean base categories: Accessories, Apparel, Audio, Desk Setup, Lifestyle, School, and Tech.

### Issue 10 – SKU Case Inconsistency (Both Sheets)
All SKU values in both sheets were converted to uppercase using a simple string operation. This ensures that joins between the Sales_Dump and Product_Supplier_Master sheets on the SKU column will work correctly.

### Issue 11 – Duplicate and Variant Rows in Product_Supplier_Master
After normalizing SKU case and product descriptions, exact duplicate rows (same SKU and same product description) were removed using pandas drop_duplicates. The remaining 53 rows represent legitimate distinct entries — base products, color or size variants, and student edition variants. Variant rows were linked back to their base product using the parent_sku column, which was already present in the source data.

### Issue 12 – Vendor Name Inconsistencies (Product_Supplier_Master.vendor_name)
Urban Sources was corrected to Urban Source to match the consistent vendor name used elsewhere. All vendor phone numbers were reformatted to a standard XXX-XXX-XXXX format by stripping all non-numeric characters and re-inserting dashes. The annotation / email missing that appeared in some vendor_rep entries was stripped out, leaving only the representative's name.

### Issue 13 – reorder_level Contains Text (Product_Supplier_Master.reorder_level)
The word ten was replaced with the integer 10. All values were then converted to integers. The 6 rows with no reorder_level were left as NULL because the correct reorder point for those products is unknown and should not be guessed.

### Issue 14 – Mixed Units in size_or_weight, weight, and length
In Sales_Dump, the single size_or_weight column was split into three new columns. Items recorded as a weight (ounces, grams, pounds, kilograms) were converted to grams and stored in weight_g. Items recorded as a length (inches, centimeters) were converted to centimeters and stored in length_cm. Items recorded as one size were given a value of Y in a new one_size_flag column, with all other rows filled as N. The original size_or_weight column was dropped. The same unit conversion was applied to the weight and length columns in Product_Supplier_Master, replacing the free-text fields with numeric weight_g and length_cm columns.

Queries: provide answers for 3 required queries and 3 additional queries designed by your group
