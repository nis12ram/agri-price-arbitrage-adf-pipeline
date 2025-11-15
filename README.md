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

### *parametrized datasets*
<img width="1466" height="640" alt="agri40" src="https://github.com/user-attachments/assets/b582d027-00f2-4dee-a360-4323ed36c861" />
<img width="1461" height="546" alt="agri41" src="https://github.com/user-attachments/assets/012c5f95-cbb1-43c0-b0fa-c0d8ecaa4204" />
<img width="1431" height="556" alt="agri42" src="https://github.com/user-attachments/assets/1ab2f74b-4f23-4127-bb0a-e25a6bebe173" />


### *mandi_ogd_api_ingestion*
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

- Using a Lookup Activity to retrieve the date when the `bronze_layer` pipeline last updated data.
  
<img width="986" height="697" alt="agri4" src="https://github.com/user-attachments/assets/a60fcaf8-be94-4479-aab9-a88f81d88375" />

- Using an If Condition activity to intelligently decide whether new data is their to load.

<img width="1706" height="633" alt="agri5" src="https://github.com/user-attachments/assets/e41080ab-498d-4ed5-bba3-7979ab47854e" />

- Using a Copy Activity to bring data from OGD API to bronze container of ADLS.
  
<img width="1700" height="737" alt="agri6" src="https://github.com/user-attachments/assets/961aee32-90c7-4053-81d8-33303a02e414" />

<img width="1742" height="658" alt="agri7" src="https://github.com/user-attachments/assets/73aca41b-809d-45bb-b2c7-2584f53e7898" />

<img width="1712" height="687" alt="agri8" src="https://github.com/user-attachments/assets/7da8bc27-fec8-4a67-b65f-6bf35f72dd88" />

**Surya's optimizations = ["Dynamic pagination rules", "Setting maximum number of retry attempts to 2", "Using parametrized datasets(dynamic_rest_api(params = [base_url ,relative_url]), dynamic_json(params = [container, folder, file])"]**

- Using a Copy Activity to update the date of the pipeline's last data refresh
  
<img width="1245" height="583" alt="agri9" src="https://github.com/user-attachments/assets/c3f877e8-ea44-478d-93e7-e0794ac565cc" />
<img width="1742" height="721" alt="agri10" src="https://github.com/user-attachments/assets/3816ea5d-e546-4ce7-9f42-2e87ea85b635" />
<img width="997" height="582" alt="agri11" src="https://github.com/user-attachments/assets/04c82f2f-0570-442d-9c3e-275b090ff376" />

### *bronze_layer*
<img width="1721" height="625" alt="agri12" src="https://github.com/user-attachments/assets/b7d381e6-e425-43e6-bf31-d26e89b48da2" />

A bronze-layer orchestration pipeline that triggers the mandi_ogd_api_ingestion pipeline, logs execution, monitors its status, and sends alerts if the ingestion process fails.

*Working*
- Using a Execute Pipeline Activity to execute mandi_ogdz_api_ingestion pipeline.

<img width="1670" height="660" alt="agri13" src="https://github.com/user-attachments/assets/96fec529-7d98-418e-9a0b-071c0dac23b9" />

- Using a Set variable Activity to set the value of `logs` pipeline variable.

<img width="1737" height="640" alt="agri14" src="https://github.com/user-attachments/assets/51650834-0662-4370-96fb-02049bdf981d" />

- Using a Set variable Activity to set the value of `status` pipeline variable.

<img width="1677" height="612" alt="agri15" src="https://github.com/user-attachments/assets/421f82b9-e644-4803-8c72-bf48eaf27157" />

- Using an If Condition activity to trigger an alert when `mandi_ogd_api_ingestion` fails.
<img width="1560" height="618" alt="agri16" src="https://github.com/user-attachments/assets/97f1bbfd-98df-467b-814f-646ee90cbe7f" />

### *silver_layer*
<img width="1605" height="577" alt="agri17" src="https://github.com/user-attachments/assets/711fab80-009f-48d9-96cb-c02e5ab60668" />

A silver-layer pipeline that intelligently transforms only new raw JSON data from the bronze layer in ADLS using a data flow named data_transformation. The processed data is then stored in ADLS in Delta format.

*Working*
- Using a Lookup activity to retrieve the date when the `silver_layer` pipeline last processed data.

<img width="946" height="637" alt="agri18" src="https://github.com/user-attachments/assets/038ae160-7c2e-4455-96b6-e535e5a9b798" />

- Using a Lookup Activity to retrieve the date when the `bronze_layer` pipeline last updated data.

<img width="942" height="635" alt="agri19" src="https://github.com/user-attachments/assets/8ae063c8-e2f7-42dc-a218-79ec826d1082" />

- Using an If Condition activity to intelligently decide whether new data is their to process.

<img width="1712" height="678" alt="agri20" src="https://github.com/user-attachments/assets/ff876d9d-b749-4f71-9b35-37a14f54f3ba" />

- Using a Data Flow Activity to execute `data_transformation` data flow.

<img width="1663" height="727" alt="agri21" src="https://github.com/user-attachments/assets/b150ac4d-ed94-4fd5-86c6-23df45c6fa94" />

<img width="1833" height="565" alt="agri22" src="https://github.com/user-attachments/assets/1abb7dbb-3b41-4c09-b927-30ce62862813" />

- Using a Copy activity to update the pipeline’s last processed date..
  
<img width="1087" height="582" alt="agri23" src="https://github.com/user-attachments/assets/a5fd9b42-de62-4a4d-962d-244aecdc9fc2" />

<img width="1687" height="727" alt="agri24" src="https://github.com/user-attachments/assets/2b75b3b1-0b71-410a-b184-adec55dace0a" />

<img width="1122" height="571" alt="agri25" src="https://github.com/user-attachments/assets/2f6f8611-e105-46ef-a261-c8c2958cd81b" />

### *gold_layer*
<img width="1715" height="598" alt="agri26" src="https://github.com/user-attachments/assets/c3e46e69-daad-4582-9e31-b442329c7a8e" />

A gold-layer pipeline that intelligently processes only new data from the silver layer, using a data flow named `data_serving` to generate a business-ready view that highlights in-state commodity price arbitrage opportunities. The resulting curated dataset is stored in ADLS in Delta format.

*Working*
- Using a Lookup activity to retrieve the date when the `gold_layer` pipeline last processed data.

<img width="1172" height="598" alt="agri27" src="https://github.com/user-attachments/assets/51b46bd6-9ab1-46bf-aa98-6d4f0fa2bb14" />

- Using a Lookup activity to retrieve the date when the `silver_layer` pipeline last processed data.

<img width="1103" height="612" alt="agri28" src="https://github.com/user-attachments/assets/ed64d2b7-5a2b-4419-940f-1031e445c64f" />

- Using an If Condition activity to intelligently decide whether new data is their to process.

<img width="1736" height="730" alt="agri29" src="https://github.com/user-attachments/assets/6d4ecbce-7f74-4e3b-8cac-d5f55e5cda06" />

- Using a Data Flow Activity to execute `data_serving` data flow.

<img width="1753" height="722" alt="agri30" src="https://github.com/user-attachments/assets/bb1a1cb4-1eea-42db-b1d8-c3ea7a0c2dd5" />

<img width="820" height="506" alt="agri31" src="https://github.com/user-attachments/assets/01fd1af8-7236-4673-babf-28d40ed1e73f" />

*sql implementation of gold layer logic*

```sql
WITH daily_price_data AS (
  SELECT *
  FROM workspace.default.ogd_mandi
  WHERE arrival_date = '2025-11-06'
), avg_price_data (
  SELECT commodity, grade, variety, state, ROUND(AVG(modal_price_in_rs_per_quintal), 2) AS avg_price_in_state
  FROM daily_price_data
  GROUP BY commodity, grade, variety, state
), high_low_price AS (
  SELECT high_price_data.arrival_date,
    high_price_data.commodity, 
    high_price_data.grade,
    high_price_data.variety, 
    high_price_data.state, 
    ROUND(((high_price_data.modal_price_in_rs_per_quintal - low_price_data.modal_price_in_rs_per_quintal) / avg_price_data.avg_price_in_state)* 100, 2) AS price_diff_pct,
    high_price_data.district AS district_with_high_price,
    high_price_data.market AS market_with_high_price,
    high_price_data.modal_price_in_rs_per_quintal AS high_price,
    low_price_data.district AS district_with_low_price,
    low_price_data.market AS market_with_low_price,
    low_price_data.modal_price_in_rs_per_quintal AS low_price,
    avg_price_data.avg_price_in_state
  FROM daily_price_data high_price_data
  INNER JOIN daily_price_data low_price_data
  ON high_price_data.commodity = low_price_data.commodity
    AND high_price_data.grade = low_price_data.grade
    AND high_price_data.variety = low_price_data.variety
    AND high_price_data.state = low_price_data.state
    AND high_price_data.market != low_price_data.market 
  INNER JOIN avg_price_data
  ON high_price_data.commodity = avg_price_data.commodity
    AND high_price_data.grade = avg_price_data.grade
    AND high_price_data.variety = avg_price_data.variety
    AND high_price_data.state = avg_price_data.state
  WHERE((high_price_data.modal_price_in_rs_per_quintal - low_price_data.modal_price_in_rs_per_quintal) / avg_price_data.avg_price_in_state) > 0.08
  ORDER BY commodity, grade, variety, state, price_diff_pct DESC
)

SELECT *
FROM high_low_price
```

*data flow implementation of gold layer logic*

<img width="1792" height="623" alt="agri32" src="https://github.com/user-attachments/assets/4f64dfe2-ffcd-4315-a98f-baf2fe4bf467" />

- Using a Copy activity to update the pipeline’s last processed date..

<img width="1182" height="580" alt="agri33" src="https://github.com/user-attachments/assets/45498462-aa62-429d-8760-0836b8d76d4f" />

<img width="1835" height="778" alt="agri34" src="https://github.com/user-attachments/assets/096a5669-f035-4f87-84d1-0d0bb53feb22" />

<img width="1147" height="583" alt="agri35" src="https://github.com/user-attachments/assets/6bf9db62-0bb7-49f3-9de9-2c5f4800fa04" />

### *main*

<img width="1672" height="643" alt="agri36" src="https://github.com/user-attachments/assets/6a522561-f7d7-4d8c-9d49-2dc1c9721a1c" />

The main orchestration pipeline that sequentially connects and executes the bronze, silver, and gold layers.

*Working*
- Using a Execute Pipeline Activity to execute `bronze_layer` pipeline.

<img width="1451" height="605" alt="agri37" src="https://github.com/user-attachments/assets/2f7c6b4f-90f3-4c62-bd38-dfbb9089f7f9" />

- Using a Execute Pipeline Activity to execute `silver_layer` pipeline.

<img width="1271" height="532" alt="agri38" src="https://github.com/user-attachments/assets/11c6c528-44fc-4c89-a410-496bdf553efe" />

- Using a Execute Pipeline Activity to execute `gold_layer` pipeline.

<img width="1016" height="553" alt="agri39" src="https://github.com/user-attachments/assets/17c7198e-8096-4fbf-8f72-eeb4327e8b32" />

## *Conclusion*
By building this ADF-driven automated pipeline, Surya enables AgriBridge to:

- Detect profitable opportunities instantly  
- Compare markets within states accurately  
- Generate actionable insights every morning  
- Standardize and store clean historical data(silver layer data).  




