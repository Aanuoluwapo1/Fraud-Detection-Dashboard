# Fraud Detection Dashboard using Splunk

## Overview

This project analyzes simulated financial transactions to detect fraudulent activity using Splunk. The dataset contains customer transactions across multiple months, with attributes such as age group, gender, merchant, category, and transaction amount.
The goal was to identify fraud patterns across different dimensions like time, demographics, and merchants, and present insights through a Splunk dashboard.

### Dataset Description
- Step: Month of transaction (May–August) 
- Customer: Unique customer ID
- Age: Age group encoded (0–5)
- Gender: M / F
- PostcodeOrigin: Origin location
- Merchant: Merchant ID
- Category: Purchase category
- Amount: Transaction value
- Fraud: 1 = Fraud, 0 = Legitimate

## Data Preparation in Splunk

After uploading the dataset into Splunk, the first step was validating field extraction.
```spl
index="main" sourcetype="fraud_dectection.csv"
| head 5
```
<img width="1350" height="668" alt="Screenshot (81)" src="https://github.com/user-attachments/assets/ea8d2818-3300-4d86-83ce-230372d480f8" />
This confirmed that fields like age, gender, merchant, category, and fraud were correctly parsed.


*Screenshot 1: Raw dataset in Splunk search results*

## Dashboard Visualizations & SPL Queries

### 1. Transaction Volume by Category
This shows which product categories have the highest transaction counts.
```
index="main" sourcetype="fraud_dectection.csv"
| stats count by category
| sort -count
```
<img width="1106" height="299" alt="Screenshot (80)" src="https://github.com/user-attachments/assets/cbbfb126-0e0d-445c-8e37-806382be062c" />
*Screenshot 2: Category distribution chart*

What this tells us:
It helps identify high-activity categories that may require closer fraud monitoring.

### 2. Fraudulent Transactions by Merchant
```
index="main" sourcetype="fraud_dectection.csv" fraud=1
| stats count as fraud_count by merchant
| sort -fraud_count
```
<img width="1097" height="290" alt="Screenshot (82)" src="https://github.com/user-attachments/assets/39cc360d-2816-4989-b0f5-7d43f49100b4" />
*Screenshot 3: Fraud by merchant visualization*


Insight:
Shows which merchants are most targeted by fraudulent transactions.

### 3. Fraud Distribution by Age Group

First, we convert encoded age values into readable groups.

```
index="main" sourcetype="fraud_dectection.csv" fraud=1
| eval age_group=case(
    age=0,"<=18",
    age=1,"19-25",
    age=2,"26-35",
    age=3,"36-45",
    age=4,"46-55",
    age=5,"56-65"
)
| stats count by age_group
| sort -count
```
<img width="1101" height="307" alt="Screenshot (83)" src="https://github.com/user-attachments/assets/f17d9b24-23bf-4c24-9875-359ce4ae466e" />
*Screenshot 4: Fraud by age group chart*


Insight:
Helps identify which demographic is most associated with fraud activity.

### 4. Fraud by Month (Step)
```
index="main" sourcetype="fraud_dectection.csv" fraud=1 
| eval month=case(
    step=0,"May",
    step=1,"June",
    step=2,"July",
    step=3,"August"
)
| stats count by month
| sort month
```
<img width="1098" height="301" alt="Screenshot (84)" src="https://github.com/user-attachments/assets/e9ab4d99-0101-4844-bd42-e81f79de5838" />

*Screenshot 5: Fraud trend over time*

Insight:
Helps detect seasonal or monthly fraud spikes.

### 5. Fraud by Category
```
index="main" sourcetype="fraud_dectection.csv" fraud=1
| stats count by category
| sort -count
```
<img width="1095" height="296" alt="Screenshot (86)" src="https://github.com/user-attachments/assets/5446d1fc-f266-4263-bdfe-40820641b34b" />
*Screenshot 6: Fraud by category chart*

Insight:
Shows which purchase categories are most vulnerable to fraud.

### 6. Fraud by Gender
```
index="main" sourcetype="fraud_dectection.csv"  fraud=1
| stats count by gender 
```
<img width="1123" height="302" alt="Screenshot (87)" src="https://github.com/user-attachments/assets/cc61a968-1580-4ddb-b50c-17ecbcf63ef6" />

*Screenshot 7: Gender distribution pie chart*

Insight:
Shows whether fraud is concentrated in a specific gender group.

### 7. Gender vs Category (Fraud Behavior)
```
index="main" sourcetype="fraud_dectection.csv"  fraud=1
| stats count by gender category
| sort -count
```
<img width="1101" height="312" alt="Screenshot (88)" src="https://github.com/user-attachments/assets/65a238f9-3dbd-4252-b079-1a6528f7637f" />

*Screenshot 8: Gender-category table*


Insight:
Reveals behavioral patterns, for instance, which gender commits fraud in which category.


### 8. Age Group vs Merchant (Top Fraud Pattern)
```
index="main" sourcetype="fraud_dectection.csv" fraud=1
| eval age_group=case(
    age=0,"<=18",
    age=1,"19-25",
    age=2,"26-35",
    age=3,"36-45",
    age=4,"46-55",
    age=5,"56-65"
)
| stats count by age_group merchant
| sort -count
| head 10
```
<img width="1100" height="297" alt="Screenshot (89)" src="https://github.com/user-attachments/assets/a9e5088b-441a-4a60-bb64-331c068dd660" />

*Screenshot 9: Age vs Merchant fraud table*


Insight:
Identifies which age groups are linked to fraud at specific merchants.
