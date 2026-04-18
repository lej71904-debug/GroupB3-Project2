# MIST4610 Project 2 Group B3

## Team Name: 
47114 Group B3

## Team Members:
1. Landon Johnson: Data Cleaner
2. Yash Kirtane: 
3. Chris Ennis: 
4. Charan Nuthalapati: 
5. Lawrence Carpenter: 

## Case summary: brief explanation of the company and data challenges.

## Conceptual model: PNG of the model plus a short English explanation of the entities & relationships.

## Data quality assessment: identify and explain the main data quality issues in the source file.
The source data consists of two sheets exported from Northline Outfitters' Excel-based operations: Sales_Dump (200 rows, 21 columns) and Product_Supplier_Master (60 rows, 16 columns). Both sheets contain significant quality issues that prevent them from being used directly in a relational database.

Issue 1 – Inconsistent Date Formats (Sales_Dump.sale_date)
The sale_date column contains at least seven distinct date formats mixed across rows. Examples found in the data:
Raw ValueFormat10-11-2025MM-DD-YYYY (U.S. style)31/10/2025DD/MM/YYYY (Canadian/European style)Oct 17 25Abbreviated month, two-digit yearOctober 5 25Full month, two-digit yearOctober 10 2025Full month, four-digit year10 Sep 2025Day-first with abbreviated month10/14/2025U.S. slash format
The U.S. and Canadian formats are ambiguous when the day value is 12 or less (e.g., 10/11/2025 could be October 11 or November 10 depending on convention). Since Northline ships to both the U.S. and Canada, both formats appear legitimately in the data.

Issue 2 – Composite customer_info Field (Sales_Dump.customer_info)
Customer details were entered as a single free-text field using at least three different delimiters (;, |, /). The field may contain the customer's name alongside notes about loyalty status, student status, and checkout type. Examples:

Mason Rivera; Loyalty? Y
Grace Hall | Student | US
Zoe Garcia / guest order
Noah Smith | Student | CA

Additionally, the same customer appears with multiple different email addresses across orders (e.g., Emma Wilson has 5 distinct email addresses on file, Grace Hall has 3). There is no stable customer ID.

Issue 3 – Inconsistent Payment Method Values (Sales_Dump.payment_method)
The same payment types appear in different cases and abbreviations:

VISA and visa represent the same method
Debit and debit represent the same method
MC and Mastercard represent the same brand


Issue 4 – Mixed Discount Formats (Sales_Dump.discount)
Discounts are recorded in five distinct formats, and 32 of 200 rows (16%) have no discount value at all:
ValueMeaning10%10 percent with symbol5%5 percent with symbol55 percent as a bare number (no symbol)promo5A promotional code representing 5% offstudent 10%Text label plus percent value0No discountNULLMissing — assumed no discount

Issue 5 – Mixed Tax Formats (Sales_Dump.tax)
Tax is stored in at least five formats across 200 rows, and 33 rows have no tax value:
ValueMeaning8.25%Decimal percent with symbol0.0825Decimal rate7%Percent with symbol13%Canadian HST as percentHST 13%Labeled text with rate
The same tax rate of 13% appears as 13%, 0.13, and HST 13% in different rows.

Issue 6 – Currency Code Embedded in Price Fields (Sales_Dump.unit_price, Product_Supplier_Master.cost, list_price)
Unit prices in Sales_Dump are stored as strings with the currency code as a prefix (e.g., USD 18.99, CAD 46.99), making the column non-numeric. In Product_Supplier_Master, cost and list_price are inconsistent — some rows have the currency prefix and others are bare numeric values (e.g., 31.4, 27.8, 21.75), with no currency indicated.

Issue 7 – Missing and Prefixed line_total Values (Sales_Dump.line_total)
62 of 200 rows (31%) have a NULL line_total. Among the rows that do have a value, some include a $ prefix (e.g., $19.43, $64.78) while others are plain numeric strings (e.g., 97.19, 161.97). The column is stored as text rather than a numeric type.

Issue 8 – Inconsistent Country Values (Sales_Dump.ship_country)
Four distinct strings represent what should be only two countries, and 3 rows have no value at all:

US and USA → United States
CA and Canada → Canada
3 NULL rows

The order_id column encodes the country in its prefix (UORD- = U.S. order, CORD- = Canada order), which can be used to recover the 3 missing values.

Issue 9 – Student Flag Embedded in Category (Sales_Dump.category, Product_Supplier_Master.category)
In Sales_Dump, category values like Tech / Student and Accessories / Student blend the actual product category with a student-discount flag. The 14 distinct values in Sales_Dump reduce to 7 base categories once the student annotation is stripped.
In Product_Supplier_Master the problem is worse — 20 distinct category strings exist due to additional inconsistencies including wrong separators (Lifestyle , Student), different connector words (Tech & Student), typos (Student and apparels), and self-referential entries (Accessories / Accessories).

Issue 10 – SKU Case Inconsistency (Both Sheets)
The same product SKU appears in both uppercase and lowercase across rows in both sheets. For example, SKU-C-1012 and sku-c-1012 refer to the same product. This breaks any join between Sales_Dump and Product_Supplier_Master.

Issue 11 – Duplicate and Variant Rows in Product_Supplier_Master
The Product_Supplier_Master sheet has 60 rows but only 20 distinct base products. The rows fall into three patterns:

True duplicates — same SKU (after case normalization), same product, with minor field differences (e.g., conflicting discontinued flag, phone format variants for the same vendor)
Color/size variants — a product like "Commuter Backpack / Black" shares the same base SKU number as "Commuter Backpack" but is a distinct item
Student edition variants — a product like "Commuter Backpack Student Edition" is a separate offering under the same base SKU


Issue 12 – Vendor Name Inconsistencies (Product_Supplier_Master.vendor_name)
Urban Source and Urban Sources appear as two distinct vendor names but are clearly the same supplier (same phone number, same rep). Phone numbers for the same vendor are also formatted inconsistently: 404-555-0181, (404)555-0181. Vendor rep entries sometimes append / email missing as a note directly in the name field (e.g., Jason Wu / email missing).

Issue 13 – reorder_level Contains Text (Product_Supplier_Master.reorder_level)
Most reorder level values are integers (5, 10, 12, 15) but some rows contain the word 'ten' instead of the number 10. 6 rows have no value.

Issue 14 – Mixed Units in size_or_weight, weight, and length
Weight and size information in both sheets is stored as free text in a mixture of imperial and metric units:

Weight examples: 8 ounces, 272 grams, 0.6 pound, 0.22 kilograms, 272g, 0.30 kg
Length examples: 11", 11 inches, 11 in, 28 cm, 38cm, 38 centimetres

In Sales_Dump, size_or_weight conflates weight and size into a single field with no consistent type. 23 rows have no value.

Issue 15 – Missing Values Summary
ColumnSheetNull Countcustomer_emailSales_Dump50line_totalSales_Dump62discountSales_Dump32taxSales_Dump33return_flagSales_Dump40ship_countrySales_Dump3size_or_weightSales_Dump23discontinuedPSM17reorder_levelPSM6weightPSM11lengthPSM13

## Data cleaning process: explain how you resolved the data quality issues identified previously. Include any SQL statements used to standardize, split, convert, or update the imported data.

## Queries: provide answers for 3 required queries and 3 additional queries designed by your group
