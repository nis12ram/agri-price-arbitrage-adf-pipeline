# **agri-price-arbitrage-adf-pipeline** (***PROJECT 1***)

Azure Data Factory (ADF) pipeline for AgriBridge Commodities Limited to ingest and process daily agricultural price data. The pipeline identifies in-state commodity price arbitrage opportunities (price differences for the same item within the same state) to generate actionable profit insights for the trading team.

## *Story Time*
AgriBridge Commodities Limited is an agricultural trading company based in Delhi, operating across India.

Their main goal is to find immediate, actionable profit opportunities. Their strategy is to identify price differences for the exact same item (same commodity, variety, and grade) within the same state.

They want to find a market where an item is cheap and another market in that same state where that exact item is expensive, allowing them to make quick, profitable decisions. They need this analysis for all states and all unique item combinations.

To do this, they hired Surya, a data engineer 👨‍💻. His job is to get the daily commodity data every morning and transform it to highlight these specific, in-state profit opportunities for the trading team.

***And now in the remaining part, we will see how Surya***👨‍💻 ***does his work....***

## *Project Overview*
To solve AgriBridge's problem, Surya decided to use ***medallion lakehouse architecture*** to build a clean, trustworthy, and durable data pipeline.

The pipeline ingests the daily raw commodity price data using the [**OGD API**](https://www.data.gov.in/catalog/current-daily-price-various-commodities-various-markets-mandi) every morning, validates and cleans the data, applies business transformations, and finally produces a gold-level Delta table that highlights all in-state price arbitrage opportunities.

On early analysis, Surya found the daily data volume as small as few mb's  & transformation complexity is also moderate




