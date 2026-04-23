# MIST4610 Project 2 Group B3

## Team Name: 
47114 Group B3

## Team Members:
1. Landon Johnson: Data Cleaner
2. Yash Kirtane: Group Leader
3. Chris Ennis: Conceptual Modeler
4. Charan Nuthalapati: SQL Writer
5. Lawrence Carpenter: Data Quality

## Case Summary:

Northline Outfitters is a small online retailer selling student-friendly lifestyle and tech accessories to customers in the United States and Canada. The company sources products from outside vendors and currently manages all records in Excel rather than a proper database.
The data presented significant challenges including inconsistent date and price formats, customer information stored in a single text field, mixed discount and tax formats, duplicate product rows, SKU capitalization inconsistencies, and measurements recorded in a mix of imperial and metric units. These issues required extensive cleaning and restructuring before the data could be loaded into a relational database.

## Conceptual Model:
<img width="797" height="796" alt="4610_project2_datamodel" src="https://github.com/user-attachments/assets/fea16c65-9da0-4628-a90e-d7634c0de087" />

## Data Quality Assessment

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

## Data Cleaning Proccess:

All data cleaning was performed manually in Microsoft Excel prior to importing the data into the database. No SQL cleaning statements were required.

### Issue 1 – Inconsistent Date Formats (Sales_Dump.sale_date)
All date values were manually reviewed and reformatted to the ISO standard YYYY-MM-DD format. Excel's Find and Replace was used to standardize common patterns, and remaining edge cases such as abbreviated month names and two-digit years were corrected by hand. After cleaning, the column contained zero null values and one consistent format across all 200 rows.

### Issue 2 – Composite customer_info Field (Sales_Dump.customer_info)
The single customer_info field was split into four separate columns using Excel's Text to Columns feature, applied once for each delimiter type. The extracted pieces were organized into four new columns: customer_name, loyalty_flag, student_flag, and checkout_type. The original customer_info column was then deleted.

### Issue 3 – Inconsistent Payment Method Values (Sales_Dump.payment_method)
All payment method values were standardized using Find and Replace. VISA and visa were replaced with Visa, MC was replaced with Mastercard, DEBIT and debit were replaced with Debit, and CASH was replaced with Cash. After cleaning all 200 rows contain one of 7 consistent values with no nulls.

### Issue 4 – Mixed Discount Formats (Sales_Dump.discount)
All discount values were converted to a uniform decimal format representing the discount rate. Find and Replace was used to remove percent symbols and text prefixes such as promo and student. Excel formulas were then written to divide any remaining whole number values by 100 to produce the correct decimal. Null values and blanks were treated as no discount and filled with 0.

### Issue 5 – Mixed Tax Formats (Sales_Dump.tax)
All tax values were converted to a uniform decimal format using the same approach as discounts. Find and Replace was used to remove labels such as HST and percent symbols, and Excel formulas were used to convert percentage values greater than 1 to their decimal equivalent by dividing by 100. The 33 rows with no tax value were left as NULL because a missing tax entry means the rate is unknown, which is different from a tax-exempt order.

### Issue 6 – Currency Code Embedded in Price Fields (Sales_Dump.unit_price, Product_Supplier_Master.cost, list_price)
In Sales_Dump the unit_price column was split into two new columns, unit_price_amount and unit_price_currency, by using Text to Columns with a space delimiter. The same approach was applied to cost and list_price in Product_Supplier_Master. The 5 rows where the currency was missing were filled in manually by checking the SKU prefix — SKUs containing -C- were assigned CAD and SKUs containing -U- were assigned USD.

### Issue 7 – Missing and Prefixed line_total Values (Sales_Dump.line_total)
Find and Replace was used to remove the dollar sign prefix from all line_total values and the column was formatted as a number. For the 62 rows with a missing line_total an Excel formula was written to calculate the correct value using unit_price_amount multiplied by quantity multiplied by (1 minus discount) multiplied by (1 plus tax). After cleaning all 200 rows have a numeric line_total value.

### Issue 8 – Inconsistent Country Values (Sales_Dump.ship_country)
Find and Replace was used to standardize all country values — USA was replaced with US and Canada was replaced with CA. The 3 rows with a missing ship_country were filled in manually by checking the order_id prefix.

### Issue 9 – Student Flag Embedded in Category (Sales_Dump.category, Product_Supplier_Master.category)
In Sales_Dump, Find and Replace was used to remove student annotations from category values leaving only the base category name. The student indicator was recorded in the student_flag column. In Product_Supplier_Master the same stripping was applied and additional inconsistencies were corrected manually. After cleaning both sheets contain exactly 7 clean base categories: Accessories, Apparel, Audio, Desk Setup, Lifestyle, School, and Tech.

### Issue 10 – SKU Case Inconsistency (Both Sheets)
Excel's UPPER() function was applied to the SKU column in both sheets to convert all values to uppercase. The formula results were then pasted as values to replace the original column.

### Issue 11 – Duplicate and Variant Rows in Product_Supplier_Master
Excel's built-in Remove Duplicates feature was used to identify and remove exact duplicate rows based on the SKU and product description columns. The remaining 53 rows represent legitimate distinct entries. Variant rows were already linked to their base product via the parent_sku column.

### Issue 12 – Vendor Name Inconsistencies (Product_Supplier_Master.vendor_name)
Find and Replace was used to correct Urban Sources to Urban Source. All vendor phone numbers were manually reformatted to a standard XXX-XXX-XXXX format. The annotation / email missing was removed from vendor_rep entries using Find and Replace.

### Issue 13 – reorder_level Contains Text (Product_Supplier_Master.reorder_level)
Find and Replace was used to replace the word ten with the number 10. The column was then formatted as a number. The 6 rows with no reorder_level were left as NULL because the correct reorder point is unknown.

### Issue 14 – Mixed Units in size_or_weight, weight, and length
In Sales_Dump the size_or_weight column was split into three new columns. Weight values were converted to grams using Excel formulas and stored in weight_g. Length values were converted to centimeters and stored in length_cm. Rows with a one size note were marked Y in a new one_size_flag column with all remaining rows filled as N. The original size_or_weight column was deleted. The same unit conversion formulas were applied to the weight and length columns in Product_Supplier_Master.

## Queries:

## Required Query 1 — Which products generated the highest total sales revenue, by country?
<img width="753" height="771" alt="image" src="https://github.com/user-attachments/assets/d785651d-e4d4-4d13-8cd8-ac2f99de9a07" />

## Required Query 2 — Which employees handled the largest number of orders, and how do their results compare with other employees under the same manager?
<img width="997" height="737" alt="image" src="https://github.com/user-attachments/assets/b1489c5a-21e5-4bcc-a2ad-48f15a2d0e26" />

## Required Query 3 — Which vendors supply products that appear in more than one category?
<img width="973" height="602" alt="image" src="https://github.com/user-attachments/assets/c628c3c6-f73e-4422-b4a4-e0222e70883b" />

## Additional Query 1 — Which customers have the highest lifetime spending and are they loyalty members?
Business justification: Identifying top spending customers helps Northline target their loyalty program more effectively and decide which customers to prioritize for promotions and retention efforts.
<img width="917" height="747" alt="image" src="https://github.com/user-attachments/assets/908b074a-3a94-4673-8b2d-d91a95c678c8" />

## Additional Query 2 — Which products have the highest return rates?
Business justification: Understanding which products are being returned most frequently helps Northline identify potential quality issues, misleading product descriptions, or sizing problems that could be costing the business money.
<img width="1048" height="761" alt="image" src="https://github.com/user-attachments/assets/d9ee87ec-fa35-4b60-9eae-08aecdc1094f" />

## Additional Query 3 — Which product categories generate the most revenue from student customers compared to non-student customers?
Business justification: Since Northline targets student customers with special discounts, this query helps the business understand which categories are most popular among students versus regular customers, which can guide future product selection and marketing decisions.
<img width="972" height="705" alt="image" src="https://github.com/user-attachments/assets/2f62d076-cac0-4bbf-ab88-2698c7199923" />
