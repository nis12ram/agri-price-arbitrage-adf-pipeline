# **agri-price-arbitrage-adf-pipeline** (***PROJECT 1***)

Azure Data Factory (ADF) pipeline for AgriBridge Commodities Limited to ingest and process daily agricultural price data. The pipeline identifies in-state commodity price arbitrage opportunities (price differences for the same item within the same state) to generate actionable profit insights for the trading team.

## *Story Time*
AgriBridge Commodities Limited is an agricultural trading company based in Delhi, operating across India.

Their main goal is to find immediate, actionable profit opportunities. Their strategy is to identify price differences for the exact same item (same commodity, variety, and grade) within the same state.

They want to find a market where an item is cheap and another market in that same state where that exact item is expensive, allowing them to make quick, profitable decisions. They need this analysis for all states and all unique item combinations.

To do this, they hired Surya, a data engineer👨‍💻. His job is to get the daily commodity data every morning and transform it to highlight these specific, in-state profit opportunities for the trading team.

***And now in the remaining part, we will see how Surya***👨‍💻 ***approaches these problem...***

## *Architecture Overview*
To solve AgriBridge's problem, Surya decided to use ***medallion lakehouse architecture*** to build a clean, trustworthy, and durable data pipeline.

### **Bronze Layer** – Raw Ingestion
- ***Overview***: Ingest latest daily price of various commodities from various markets using [**OGD API**](https://www.data.gov.in/catalog/current-daily-price-various-commodities-various-markets-mandi) every morning.
- ***Source***: [**OGD API**](https://www.data.gov.in/catalog/current-daily-price-various-commodities-various-markets-mandi)
- ***Source Data Format***: JSON
- ***Sink***: The "bronze" container of "ADLS Gen2"
- ***Sink Data Format***: JSON *****(Reason: Surya found that the source API’s JSON response has a complex schema with many nested elements, which makes it naturally unsuitable for a tabular sink like Delta)*****

### **Silver Layer** – Data Cleaning & Standardization
- ***Overview***: Flattening nested JSON data, standardizing column names, data types, and formats, normalizing text, and handling missing, null, and corrupted values.
- ***Source***: JSON file from the "bronze" container of "ADLS Gen2"
- ***Source Data Format***: JSON
- ***Sink***: The "silver" container of "ADLS Gen2"
- ***Sink Data Format***: Delta Table *****(Reason: Surya transformed the raw JSON into simple clean JSON with proper schema, which makes it suitable for tabular sink like Delta)*****

### **Gold Layer** – Business Logic
- ***Overview***: Applying filters, joins & aggregation functions to create a business view that highlights in-state price arbitrage opportunities.
- ***Source***: Delta Table from the "silver" container of "ADLS Gen2"
- ***Source Data Format***: Delta Table
- ***Sink***: The "gold" container of "ADLS Gen2"
- ***Sink Data Format***: Delta Table

## *Tools*
From the initial analysis, Surya found that the daily data volume is only a few megabytes and the required transformations are moderately complex. Based on this, Surya chose ***Azure Data Factory*** for ******data orchestration, movement, and transformation*****. 

Since the transformations are not highly complex, Surya opted to use ADF Data Flows instead of Databricks.

## *Pipeline*
Surya designs a multi-layered data pipeline using Azure Data Factory (ADF)

### **Pipeline for bronze layer**
#### *mandi_ogd_api_ingestion*
<img width="1743" height="616" alt="adf_rest_api_ingestion_edited" src="https://github.com/user-attachments/assets/5a78e6a7-eed3-4431-b965-19d0137cc9c3" />

An ingestion pipeline that retrieves the latest daily commodity prices from the OGD API and loads only new or updated records.

*Working*
- Using a Web Activity to fetch the OGD API key from Azure Key Vault.
<img width="890" height="648" alt="agri1" src="https://github.com/user-attachments/assets/e4f411c6-3610-46b8-8869-0431e65629eb" />

**Surya's optimizations = ["Setting maximum number of retry attempts to 2", "Enabling Secure Input & Output options to prevent API key from being exposed in logs."]**

- Using a Set Activity to set the value of `ogd_api_key` pipeline variable.

<img width="747" height="527" alt="agri2" src="https://github.com/user-attachments/assets/e78aa949-67d9-4a04-8e13-2406391b5fea" />

- Using a Web Activity to fetch the metadata of the [**OGD API DATA**](https://www.data.gov.in/catalog/current-daily-price-various-commodities-various-markets-mandi).
  
<img width="735" height="687" alt="agri3" src="https://github.com/user-attachments/assets/08886892-15f1-42d8-a740-ff0ced793168" />

**Surya's optimizations = ["Setting maximum number of retry attempts to 2", "Enabling Secure Input to prevent API key from being exposed in logs."]**

- Using a Lookup Activity to retrieve the date of pipeline's last data refresh.
  
<img width="986" height="697" alt="agri4" src="https://github.com/user-attachments/assets/a60fcaf8-be94-4479-aab9-a88f81d88375" />


"description": "An ingestion pipeline that retrieves the latest daily commodity prices from the OGD API and loads only new or updated records into ADLS in JSON format. The pipeline uses Azure Key Vault to securely access the OGD API key and first collects API metadata to intelligently determine whether new data is available before performing any ingestion.",




