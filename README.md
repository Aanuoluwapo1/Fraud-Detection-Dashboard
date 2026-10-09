# Fraud Analytics Dashboard Using Splunk

## Project Overview

This project uses Splunk Search Processing Language (SPL) to explore financial transactions that are already labelled as fraudulent or legitimate. The searches compare transaction and fraud-labelled record counts across merchants, purchase categories, age groups, gender values, and months, with results presented through Splunk charts and tables.

The work demonstrates data exploration, filtering, aggregation, field transformation, visualisation, and evidence-based reporting in Splunk. It is a descriptive analytics project: the searches analyse existing fraud labels and do not predict or independently identify previously unknown fraud.

## Project objectives

- Inspect the available transaction fields in Splunk.
- Compare transaction volume across purchase categories.
- Summarise fraud-labelled records by merchant, category, age group, gender, and month.
- Examine fraud-labelled record counts across combinations of fields.
- Present results through readable Splunk visualisations.
- Explain the difference between fraud counts and fraud rates.

## Tools and technologies

- Splunk Enterprise
- Search Processing Language (SPL)
- CSV transaction data
- Splunk charts and tables

## Dataset and fields

The project documentation describes these fields:

| Field | Documented meaning |
|---|---|
| `step` | Month code mapped in the original analysis to May through August |
| `customer` | Customer identifier |
| `age` | Encoded age group |
| `gender` | Gender value |
| `postcodeOrigin` | Origin location |
| `merchant` | Merchant identifier |
| `category` | Purchase category |
| `amount` | Transaction value |
| `fraud` | `1` for fraud-labelled records and `0` for legitimate-labelled records |

### Data notes

The repository contains the SPL and nine screenshots produced for this analysis. The original CSV, dataset documentation, saved Splunk objects, and Splunk environment are not included, so independent reproduction is currently unavailable. Several chart labels contain trailing apostrophes, suggesting that some categorical values may need additional parsing or cleaning in a recreated environment.

### Sourcetype discrepancy

The original README queries use the misspelled sourcetype:

```text
fraud_dectection.csv
```

The raw-data screenshot visibly uses:

```text
fraud_detection.csv
```

The screenshot also shows `source=prepared_data.csv`. The original queries below retain the documented `fraud_dectection.csv` value. Determining which sourcetype was active would require access to the original Splunk configuration or a recreated environment.

## Analysis approach

The searches use a consistent process:

1. Select events from the documented index and sourcetype.
2. Filter on `fraud=1` where the question concerns fraud-labelled records.
3. Group matching records with `stats count`.
4. Sort or limit the aggregated results.
5. Present the output as a Splunk chart or table.

## Analysis

### Data preparation and field review

The original project began by previewing five events.

#### Original SPL

```spl
index="main" sourcetype="fraud_dectection.csv"
| head 5
```

![Raw dataset preview in Splunk](https://github.com/user-attachments/assets/ea8d2818-3300-4d86-83ce-230372d480f8)

*Figure 1. Raw-data preview showing five CSV events, `source=prepared_data.csv`, and `sourcetype=fraud_detection.csv`. The screenshot does not display each extracted field, although the later visualisations indicate that fields such as `category`, `merchant`, and `fraud` were available for analysis.*

### 1. Transaction volume by category

**Question:** How many transaction records appear in each category?

#### Original SPL

```
index="main" sourcetype="fraud_dectection.csv"
| stats count by category
| sort -count
```

![Transaction volume by category chart](https://github.com/user-attachments/assets/cbbfb126-0e0d-445c-8e37-806382be062c)

*Figure 2. Category distribution chart showing transaction-record counts.*

**Interpretation:** This query counts all matching transaction records by category and orders the largest counts first. It identifies high-volume categories, but it does not measure fraud risk because it does not filter on `fraud=1` or calculate a fraud rate.

### 2. Fraud-labelled transactions by merchant

**Question:** Which merchants appear most frequently in records labelled as fraudulent?

#### Original SPL

```
index="main" sourcetype="fraud_dectection.csv" fraud=1
| stats count as fraud_count by merchant
| sort -fraud_count
```

![Fraud-labelled transaction count by merchant](https://github.com/user-attachments/assets/39cc360d-2816-4989-b0f5-7d43f49100b4)

*Figure 3. Merchant chart showing counts of records selected by `fraud=1`.*

**Interpretation:** This query ranks merchants by their raw number of fraud-labelled records. It does not show that a merchant was deliberately targeted or has a higher fraud rate. A merchant with more total transactions may naturally have more fraud-labelled transactions.

### 3. Fraud-labelled transactions by age group

**Question:** How are fraud-labelled records distributed across the documented age groups?

#### Original SPL

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

![Fraud-labelled transaction count by age group](https://github.com/user-attachments/assets/f17d9b24-23bf-4c24-9875-359ce4ae466e)

*Figure 4. Age-group chart based on the mapping used in the project search.*

**Interpretation:** The query maps six encoded values to readable labels and counts fraud-labelled records in each resulting group. It does not establish that one age group has a higher probability of fraud, because total transaction volume per group is not included. The mapping and treatment of unexpected or missing age codes require source documentation to verify.

### 4. Fraud-labelled transactions by month code

**Question:** How are fraud-labelled records distributed across the four documented month codes?

#### Original SPL

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

![Fraud-labelled transaction count by mapped month](https://github.com/user-attachments/assets/e9ab4d99-0101-4844-bd42-e81f79de5838)

*Figure 5. Fraud-labelled transaction counts by mapped month. The chart displays the month names alphabetically: August, July, June, and May.*

**Interpretation:** This analysis uses SPL to group fraud-labelled transactions by month and compare their distribution across the four months represented in the dataset. The query transforms numeric month codes into readable labels, counts the matching records, and displays the results in Splunk. Sorting by the numeric month code would present the results in chronological order. The analysis provides a descriptive view of monthly fraud-labelled transaction counts; comparing fraud rates would require the total transaction volume for each month.

### 5. Fraud-labelled transactions by category

**Question:** Which categories contain the most records labelled as fraudulent?

#### Original SPL

```
index="main" sourcetype="fraud_dectection.csv" fraud=1
| stats count by category
| sort -count
```

![Fraud-labelled transaction count by category](https://github.com/user-attachments/assets/5446d1fc-f266-4263-bdfe-40820641b34b)

*Figure 6. Category chart limited to records selected by `fraud=1`.*

**Interpretation:** The query identifies categories with larger raw counts of fraud-labelled records. It does not establish that those categories are more vulnerable. Comparing vulnerability or relative risk would require total transaction counts and a fraud rate for each category.

### 6. Fraud-labelled transactions by gender

**Question:** How are fraud-labelled records distributed across the available gender values?

#### Original SPL

```
index="main" sourcetype="fraud_dectection.csv"  fraud=1
| stats count by gender 
```

![Fraud-labelled transaction distribution by gender](https://github.com/user-attachments/assets/cc61a968-1580-4ddb-b50c-17ecbcf63ef6)

*Figure 7. Pie chart showing raw counts of fraud-labelled records by the available gender values.*

**Interpretation:** The query compares raw fraud-labelled record counts. It does not measure relative fraud risk because the total number of transactions for each gender value is unavailable in the result.

### 7. Gender and category distribution

**Question:** How are fraud-labelled records distributed across gender and category combinations?

#### Original SPL

```
index="main" sourcetype="fraud_dectection.csv"  fraud=1
| stats count by gender category
| sort -count
```

![Gender and category analysis](https://github.com/user-attachments/assets/65a238f9-3dbd-4252-b079-1a6528f7637f)

*Figure 8. Visualisation associated with grouping fraud-labelled records by gender and category. The chart does not present the two-dimensional relationship clearly.*

**Interpretation:** The query counts fraud-labelled records for each gender/category pair. The `gender` value is a transaction or customer attribute; it does not identify who committed fraud. A table, stacked bar chart, or heatmap would communicate this relationship more clearly.

### 8. Top age-group and merchant combinations

**Question:** Which age-group and merchant pairs have the largest fraud-labelled record counts?

#### Original SPL

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

![Top age-group and merchant combinations](https://github.com/user-attachments/assets/a9e5088b-441a-4a60-bb64-331c068dd660)

*Figure 9. Visualisation associated with the ten largest age-group and merchant counts.*

**Interpretation:** The query returns the ten age-group/merchant pairs with the largest raw counts of fraud-labelled records. It does not establish causation, merchant risk, or age-group risk. The chart repeats age labels without clearly presenting the paired merchant, so a table would better preserve the relationship.

## Fraud counts and fraud rates

Most original searches calculate a **fraud count**: the number of records matching `fraud=1` in each group.

A **fraud rate** would compare that count with all transactions in the same group:

```text
fraud rate = fraud-labelled transactions / total transactions
```

Counts answer, “Where are the most labelled fraud records?” Rates answer, “What proportion of a group's transactions are labelled as fraud?” Both can be useful, but they support different conclusions. This repository does not contain verified fraud-rate results, so none are claimed.

## Future improvements

The following enhancements would extend the analysis in a recreated environment. They have not been tested against the original dataset.

### Proposed chronological month ordering

The following example preserves the numeric `step` value for chronological sorting. It uses the sourcetype visible in the raw-data screenshot rather than the misspelling in the documented project queries.

```spl
index="main" sourcetype="fraud_detection.csv" fraud=1
| eval month=case(
    step=0,"May",
    step=1,"June",
    step=2,"July",
    step=3,"August"
)
| stats count by step month
| sort 0 step
```

This proposal still produces counts, not fraud rates, and it depends on the unverified month mapping.

### Additional improvements

- Confirm the dataset source, licence, schema, and label definitions.
- Validate the age and month mappings against source documentation.
- Correct and test the sourcetype in a recreated environment.
- Clean categorical values if the visible trailing apostrophes are present in the extracted fields.
- Calculate both total transactions and fraud-labelled transactions before deriving rates.
- Use clearer tables or two-dimensional charts for combined-field analyses.
- Save and export a complete dashboard in a future recreated lab.
- Document future recreated-lab results separately from the work shown here.

## Results and interpretation

The searches and screenshots show that:

- Transaction records were explored and aggregated by category.
- Fraud-labelled records were grouped by merchant, category, encoded age group, gender, mapped month, and selected field combinations.
- `stats`, `eval`, `sort`, and `head` were used to transform and summarise the records.
- The resulting group counts were presented through Splunk charts and tables.

These results describe the distribution of records in the labelled data. They do not establish fraud rates, statistical significance, causation, or predictive detection. The repository also does not verify a saved dashboard or alerting implementation.

## Skills demonstrated

- SPL filtering and aggregation
- Field transformation with `eval` and `case()`
- Sorting and limiting statistical results
- Comparing transaction and fraud-labelled record counts across dimensions
- Explaining the analytical difference between fraud counts and fraud rates
- Presenting search results with Splunk charts and tables
- Identifying data-quality and visualisation limitations
- Reporting findings within the limits of the available evidence

## Technical references

- [Splunk `stats` command](https://help.splunk.com/en/splunk-enterprise/spl-search-reference/10.4/search-commands/stats)
- [Splunk `eval` command](https://help.splunk.com/en/splunk-enterprise/search/search-manual/10.4/calculate-and-format-values/evaluate-and-manipulate-fields-with-multiple-values)
- [Splunk evaluation functions](https://help.splunk.com/en/splunk-enterprise/spl-search-reference/10.4/evaluation-functions/evaluation-functions)
