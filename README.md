# FWC2022 Player Performance and Ranking System

A personal football analytics project based on StatsBomb Open Data for the 2022 FIFA World Cup.

The project combines event-level match data, player-match performance statistics, weakly supervised POTM modelling, relative performance features, ranking models, and Power BI visualization.

## Project Overview

The project evaluates player performance at match level and produces an interpretable player ranking system.

Main components:

- Event-level data preprocessing
- Player-match statistical aggregation
- Percentile-based dynamic scoring
- Weakly supervised MVP modelling
- Repeated grouped cross-validation
- Relative performance feature engineering
- XGBoost classification and ranking models
- Winner-only post-processing for POTM selection
- Interactive Power BI visualization

The current system is designed for post-match analysis. It is not intended to predict POTM before a match takes place.

## Data Source

The original event data comes from [StatsBomb Open Data](https://github.com/statsbomb/open-data).

This project uses data from the 2022 FIFA World Cup.

The event data contains actions such as passes, shots, carries, pressures, ball recoveries, duels, etc.

The event-level data is aggregated into player-match observations before modelling.

## Weakly Supervised POTM Label

The original dataset does not provide a unified numerical player rating or a directly usable POTM label.

A manually added MVP column is therefore used as a weakly supervised target:

    MVP = 1  -> manually labelled match MVP
    MVP = 0  -> non-MVP player

This label is subjective and should be interpreted as a noisy target rather than an absolute ground truth.

The models learn patterns associated with the manually assigned MVP labels. They do not automatically discover an objective definition of football performance.

## Dynamic Scoring Baseline

The project includes a percentile-based baseline score.

For each performance statistic, two rankings are calculated:

1. Per-match percentile: the player's percentile relative to other players in the same match.
2. Global percentile: the player's percentile relative to all player-match observations in the tournament.

Positive and negative statistics are treated differently:

- Higher values are preferred for positive statistics (e.g., goals).
- Lower values are preferred for negative statistics (e.g., fouls committed).

The dynamic score is calculated as:

    Dynamic Score =
        Per-Match vs Global Weight × Per-Match Percentile
        + (100 − Per-Match vs Global Weight) × Global Percentile

The baseline uses equal weight:

    Per-Match vs Global Weight = 50

The Dynamic Score is scaled to a 0–100 range for visualization.

## Machine Learning Models

The notebook evaluates several model families.

### Logistic Regression

A standardized logistic regression model with class balancing.

### Random Forest

A random forest classifier with 500 trees, maximum depth of 6, minimum leaf size of 5, and class balancing.

### XGBoost Classifier

An XGBoost binary classifier using 300 estimators, maximum depth of 3, learning rate of 0.03, row and column subsampling, and log-loss evaluation.

### Neural Network

A small feed-forward neural network with the following structure:

    Input layer
    → 32-unit hidden layer
    → Dropout
    → 16-unit hidden layer
    → Output layer

The neural network uses standardized features and weighted binary cross-entropy loss.

### XGBRanker

An XGBoost pairwise ranking model using:

    objective = rank:pairwise
    eval_metric = ndcg@5

Unlike a classifier, XGBRanker is trained to order players within match-level groups rather than independently estimating a binary MVP probability for each player.

XGBoost performed best among all models listed above and thus is chosen to be further modified as the final model.

## Relative Features

The ranking experiments create additional contextual features from the original player-match statistics.

For each statistic, the following features are calculated:

- Match percentile
- Match share
- Team percentile
- Team share
- Difference from team mean

For example, for goals:

    goals__match_pct
    goals__match_share
    goals__team_pct
    goals__team_share
    goals__minus_team_mean

These features provide context that raw statistics cannot capture. For example, 30 passes may represent very different performances depending on match tempo, team style, and player role.

## Winner-Only MVP Reranking

The winner-only rule is applied only during the final MVP selection stage.

The underlying player performance score remains unmasked for all players. This is important because the project is intended to provide a performance rating, not only a winner prediction.

The final POTM selection process is:

1. Generate the model-based performance score.
2. Rank all players within the match.
3. Use the match result as a post-processing rule.
4. Restrict the final MVP candidate selection to the winning team when the match has a winner.

This produces two different concepts:

    Performance Rank
        Unmasked ranking of player performance across both teams.

    MVP Rank
        Final MVP selection ranking after applying the winner-only rule.

The winner-only rule uses post-match information and should not be interpreted as a pre-match predictive feature.

## Validation Strategy

Players from the same match are highly dependent observations. Ordinary random cross-validation could therefore cause data leakage.

The notebook uses repeated grouped cross-validation:

- All players from one match remain in the same fold.
- No match appears in both training and validation data for a fold.
- Five folds are used.
- The procedure is repeated five times.
- The same grouped folds are used for model comparisons.

This provides a more realistic estimate of how well the models generalize to unseen matches.

## Evaluation Metrics

### PR-AUC

Precision-recall area under the curve. This is particularly useful because the MVP label is highly imbalanced.

### ROC-AUC

Measures the ability to distinguish MVP and non-MVP observations. Because of class imbalance, ROC-AUC should not be interpreted alone.

### Top-1 Accuracy

The proportion of matches where the highest-ranked player is the manually labelled MVP.

### Top-3 Accuracy

The proportion of matches where the manually labelled MVP appears among the top three ranked players.

### Top-5 Accuracy

The proportion of matches where the manually labelled MVP appears among the top five ranked players.

## Model Comparison

The notebook compares the following variants:

1. Base XGBoost Classifier
2. Base XGBoost Classifier + Relative Features
3. Base XGBoost Classifier + Winner-Only Reranking
4. XGBRanker + Raw Features
5. XGBRanker + Relative Features
6. XGBRanker + Winner-Only Reranking
7. XGBRanker + Relative Features + Winner-Only Reranking

All variants use the same repeated grouped cross-validation procedure.

Main conclusions:

- Relative features improve performance compared with raw player statistics.
- Winner-only reranking improves POTM selection accuracy.
- XGBRanker is appropriate because POTM selection is fundamentally a within-match ranking problem.
- The combined XGBRanker, relative feature, and winner-only approach is effective for final POTM selection.
- Winner-only masking should not be applied to the underlying performance score, because it would assign the same floor value to all losing-team players.
- The exported performance score therefore remains unmasked, while winner-only logic is applied only to the final MVP rank.

## Final Output Dataset

The final Power BI dataset is:

    player_performance_rank.xlsx

The notebook first writes:

    player_performance_rank.csv

The CSV was converted to Excel for use in Power BI.

Important columns include:

| Column | Description |
|---|---|
| match_id | Match identifier |
| player_id | Player identifier |
| player | Player name |
| team | Player's team |
| MVP | Manually assigned MVP label |
| goals | Player goals |
| xG | Expected goals |
| team_goals | Player's team goals |
| opponent_goals | Opponent goals |
| match_outcome | Match outcome from the player's team perspective |
| winner_eligible | Whether the player's team is eligible for winner-only MVP selection |
| model_name | Model used for the exported score |
| model_rank_score | Unmasked model-based performance score |
| MVP_score | Match-relative percentile score from the model |
| performance_rank | Unmasked within-match performance rank |
| winner_only_rank_score | Score after winner-only post-processing |
| MVP_rank | Final MVP selection rank |
| model_top1_candidate | Whether the player is the final top-1 MVP candidate |
| oof_score_tie_count | Number of players sharing the same out-of-fold score |
| dynamic_score_0_100 | Percentile-based baseline score scaled to 0–100 |

## Power BI Report

The Power BI report is:

    FWC2022_RankingSystem.pbix

The report allows users to:

- Select a match
- Filter by team
- View the matchup and match result
- Compare model scores with the Dynamic Score baseline
- Examine player performance rankings
- Compare model rank and baseline rank
- Identify the manually labelled MVP
- Inspect player-level match statistics

The player table includes an Actual MVP field. Rows with MVP = 1 are highlighted with a gold background.

## Repository Contents

## Repository Contents

* `player_match_stats_v1.xls` — Excel version of processed player-match dataset used by Power BI
* `dataset_preprocessing.ipynb` — Python code used to process and aggregate the event data
* `FWC2022_Abstract.pbix` — Power BI tournament overview dashboard
* `FWC2022_PlayerCard.pbix` — Power BI player-level performance dashboard
* `Dynamic_Scoring_System.ipynb` — Python notebook for calculating player performance percentile ranks and preparing the scoring data
* `player_stats_rank.xlsx` — Processed player-match dataset containing the percentile ranking metrics used by the scoring system
* `FWC2022_DynamicScore.pbix` — Power BI report implementing the interactive weighting and dynamic ranking
* `model.ipynb` — The main modelling notebook containing data loading, baseline experiments, neural network experiments, repeated grouped cross-validation, relative feature engineering, XGBoost evaluation, XGBRanker evaluation, winner-only MVP reranking, final score generation, and export logic
* `player_performance_rank.xlsx` — The final player-match dataset containing player statistics, match result information, model scores, performance ranks, MVP ranks, and Dynamic Score baseline values
* `FWC2022_RankingSystem.pbix` — Power BI report for interactive exploration of the final player performance ranking system

## Reproducing the Analysis

The notebooks expect the processed player-match input dataset to be available in the same directory.

## Required Python Packages

    pip install pandas numpy scikit-learn xgboost openpyxl torch jupyter

The neural network section requires PyTorch.

## Important Interpretation Notes

This project is an exploratory football analytics system rather than an official player rating provider.

Important limitations include:

- The MVP label is manually assigned and subjective.
- The dataset is relatively small for machine learning.
- Players from the same match are not independent observations.
- Features such as playing time, player position, and assists are not currently included.
- The model score is not a universal measure of football ability.
- Winner-only reranking uses post-match information.
- The final model is intended for post-match evaluation rather than pre-match prediction.

## Future Improvements

Potential future work includes:

- Adding features such as playing time, player roles, and assists
- Developing position-specific models
- Testing temporal validation across tournaments and generalization to other competitions
- Comparing against external player rating systems

## License and Data Attribution

This project uses StatsBomb Open Data.

Please refer to the [StatsBomb Open Data repository](https://github.com/statsbomb/open-data) for the applicable data license and attribution requirements.

This repository is intended for educational and exploratory football analytics purposes.
