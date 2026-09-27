# FWC2022 Player Performance Analysis

A personal football analytics project using event-level match data to explore player performance through Python data processing and Power BI visualization.

## Project Overview

This project analyzes player performance during the **2022 FIFA World Cup** using detailed match event data. The current version focuses on transforming event-level data into a player-match dataset and building interactive Power BI dashboards.

The project currently includes:

* Data preprocessing and aggregation from event-level data
* Player-level match performance statistics
* Tournament-level performance overview
* Interactive player selection and stats display
* Weight-based dynamic scoring system and ranking

## Data Source

The original event data comes from **StatsBomb Open Data**, which provides publicly available football event data for selected competitions.

For this project, the dataset used is the **2022 FIFA World Cup** event data.

Source: [StatsBomb Open Data](https://github.com/statsbomb/open-data)

## Current Dashboard

### 1. Tournament Overview

`FWC2022_Abstract.pbix` provides an overview of player and tournament performance.

### 2. Player Performance Explorer

`FWC2022_PlayerCard.pbix` allows users to select an individual player and examine their stats across the tournament.

### 3. Dynamic Scoring System

## Dynamic Scoring System

This stage of the project introduces a dynamic player scoring system that combines two perspectives of player performance:

* **Per Match:** The player's performance percentile compared with other players in the same match.
* **Global:** The player's performance percentile compared with all player-match performances across the tournament.

A weighted score is calculated dynamically as:

```text
Dynamic Score =
    Match Weight × Per Match Rank
    + (1 − Match Weight) × Global Rank
```

The **Per Match vs Global** weight can be adjusted interactively in `FWC2022_DynamicScore.pbix` using a slider. This allows users to explore how the player ranking changes when placing more emphasis on performance in individual matches versus consistency relative to the overall tournament.

'Positive' and 'negative', 'offensive' and 'defensive' statistics are treated separately based on domain knowledge.

## Data Processing

The original StatsBomb event data contains individual match events such as passes, shots, carries, pressures, duels, and other actions.

Python was used to:

* Filter and clean event-level data
* Exclude penalty shootout events from the current analysis
* Aggregate event-level data into a player-match statistics dataset
* Calculate passing, shooting, possession, defensive, and other player performance metrics
* Handle missing values according to the meaning of each metric
* Calculate percentile ranks relative to players in the same match
* Calculate percentile ranks across all player-match observations in the tournament
* Prepare the ranking data for a weighted player scoring system
* Separate positive and negative, offensive and defensive statistics
* Export the processed data for use in Power BI

## Repository Contents

* `player_match_stats_v1.xls` — Excel version of processed player-match dataset used by Power BI
* `dataset_preprocessing.ipynb` — Python code used to process and aggregate the event data
* `FWC2022_Abstract.pbix` — Power BI tournament overview dashboard
* `FWC2022_PlayerCard.pbix` — Power BI player-level performance dashboard
* `Dynamic_Scoring_System.ipynb` — Python notebook for calculating player performance percentile ranks and preparing the scoring data.
* `player_stats_rank.xlsx` — Processed player-match dataset containing the percentile ranking metrics used by the scoring system.
* `FWC2022_DynamicScore.pbix` — Power BI report implementing the interactive weighting and dynamic ranking.

## Tools

* Python
* Power BI
* Pandas
* Excel
* StatsBomb Open Data
