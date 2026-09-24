# FWC2022 Player Performance Analysis

A personal football analytics project using event-level match data to explore player performance through Python data processing and Power BI visualization.

## Project Overview

This project analyzes player performance during the **2022 FIFA World Cup** using detailed match event data. The current version focuses on transforming event-level data into a player-match dataset and building interactive Power BI dashboards.

The project currently includes:

* Data preprocessing and aggregation from event-level data
* Player-level match performance statistics
* Tournament-level performance overview
* Interactive player selection and stats display

## Data Source

The original event data comes from **StatsBomb Open Data**, which provides publicly available football event data for selected competitions.

For this project, the dataset used is the **2022 FIFA World Cup** event data.

Source: [StatsBomb Open Data](https://github.com/statsbomb/open-data)

## Current Dashboard

### 1. Tournament Overview

The first Power BI report provides an overview of player and tournament performance.

### 2. Player Performance Explorer

The second Power BI report allows users to select an individual player and examine their stats across the tournament.

The current version focuses on match-level statistics rather than a single overall player rating or impact score.

## Data Processing

The original StatsBomb event data contains individual match events such as passes, shots, carries, pressures, duels, and other actions.

Python was used to:

* Filter and clean event-level data
* Exclude penalty shootout events from the current analysis
* Aggregate events into a player-stats dataset
* Calculate passing and shooting statistics
* Handle missing values according to the meaning of each metric
* Export the processed data for use in Power BI

## Repository Contents

* `player_match_stats_v1.xls` — Excel version of processed player-match dataset used by Power BI
* `dataset_preprocessing.ipynb` — Python code used to process and aggregate the event data
* `FWC2022_Abstract.pbix` — Power BI tournament overview dashboard
* `FWC2022_PlayerCard.pbix` — Power BI player-level performance dashboard

## Tools

* Python
* Pandas
* Power BI
* Excel
* StatsBomb Open Data
