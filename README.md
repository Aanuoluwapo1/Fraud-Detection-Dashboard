# Fraud Analytics Dashboard Using Splunk

This project uses Splunk to analyse financial transactions that are already labelled as fraudulent or legitimate. The analysis examines transaction patterns across merchants, categories, age groups, gender values, and months, with results presented through Splunk charts and tables.

The project demonstrates practical skills relevant to SOC and cybersecurity analyst roles, including writing SPL searches, reviewing available fields, aggregating events with `stats`, transforming values with `eval`, examining relationships between fields, and communicating analytical results clearly.

The project provides descriptive fraud analytics based on existing labels; it does not independently predict or identify previously unknown fraud.

## Project objectives

- Inspect the available transaction fields in Splunk.
- Compare transaction volume across purchase categories.
- Summarise fraud-labelled records by merchant, category, age group, gender, and month.
- Explore combinations of fields that may provide useful investigation leads.
- Present results through readable Splunk visualisations.
- Distinguish fraud counts from fraud rates.

## Tools and technologies

- Splunk Enterprise
- Search Processing Language (SPL)
- CSV transaction data
- Splunk charts and tables

## Dataset and Fields

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

### Project limitations

The README preserves the SPL and nine screenshots produced for this analysis. The source CSV, dataset documentation, saved Splunk objects, and original Splunk environment are unavailable, so the dataset provenance, field mappings, and results cannot currently be independently rerun or verified. The screenshots remain the available evidence of the searches and visualisations.

Several visible chart labels retain trailing apostrophes, which may indicate that some categorical values required additional parsing or cleaning.

### Sourcetype discrepancy

The original README queries use the misspelled sourcetype:

```text
fraud_dectection.csv
```

The raw-data screenshot visibly uses:

```text
fraud_detection.csv
```

The screenshot also shows `source=prepared_data.csv`. The queries below preserve the documented `fraud_dectection.csv` value; the discrepancy has not been retested.

## Analysis approach

The project searches follow a consistent process:

1. Select events from the documented index and sourcetype.
2. Filter on `fraud=1` where the question concerns fraud-labelled records.
3. Group matching records with `stats count`.
4. Sort or limit the aggregated results.
5. Display the output as a Splunk chart or table.

The searches count records carrying an existing label. They do not explain how that label was assigned or determine whether an unlabelled transaction is fraudulent.

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

## Future Improvements

The following enhancements are proposed and have not been tested against the original dataset.

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
- Export a saved dashboard or include a complete dashboard screenshot if one is recreated.
- Distinguish any future recreated-lab results from the screenshots and results documented here.

## Results and Interpretation

The available evidence supports the following conclusions:

- Transaction records were explored and aggregated in Splunk.
- Fraud-labelled records were grouped by several transaction attributes.
- SPL commands including `stats`, `eval`, `sort`, and `head` were used in the documented analysis.
- The project screenshots document multiple Splunk charts and tables.
- The results identify differences in raw counts across groups.

The project demonstrates descriptive Splunk analysis of labelled transaction data. It does not claim predictive fraud detection, fraud-rate results, statistical significance, causation, a complete saved dashboard, or an alerting implementation.

## Skills demonstrated

- SPL filtering and aggregation
- Field transformation with `eval` and `case()`
- Sorting and limiting statistical results
- Comparing grouped event counts
- Creating Splunk visualisations
- Reviewing analytical limitations and data quality
- Communicating evidence without overstating conclusions

## Technical references

- [Splunk `stats` command](https://help.splunk.com/en/splunk-enterprise/spl-search-reference/10.4/search-commands/stats)
- [Splunk `eval` command](https://help.splunk.com/en/splunk-enterprise/search/search-manual/10.4/calculate-and-format-values/evaluate-and-manipulate-fields-with-multiple-values)
- [Splunk evaluation functions](https://help.splunk.com/en/splunk-enterprise/spl-search-reference/10.4/evaluation-functions/evaluation-functions)
