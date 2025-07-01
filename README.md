---
layout: default
title: "League of Legends Role Impact Statistical Analysis"
---

# League of Legends Role Impact Statistical Analysis

League of Legends Role Impact Analysis is a data science project conducted at UCSD. The project includes exploratory data analysis, data cleaning, an analysis of missingness on the dataset, hypothesis testing, a baseline model, final model, and a fairness analysis.

---

## Introduction

### General Introduction and Purpose

League of Legends is a popular online multiplayer video game created by Riot Games, featuring one of the largest esports scenes for over a decade. The dataset used in this project is a professional dataset developed by Oracle’s Elixir from professional League of Legends esports matches throughout 2025.

This dataset includes a wide variety of statistics from the matches played, making it useful for examining player tendencies, team interactions, and game results. It includes details on individual performance, strategic decisions, in-game metrics, and overall match progression.

In any given match, there are two teams that go against each other (red side and blue side), each team has 5 members, and each member is assigned to a specific role which they play throughout the game. The roles include top lane, jungle, mid lane, bot lane, and support. A key topic debated within League of Legends and its community is the balancing of these 5 roles. In other words, community members debate the impact of each role on the game's outcome and whether it is equal or not.

The question I'm aiming to explore is: **Which role in League of Legends has the greatest impact on game outcomes?** To answer this, I'll apply data analysis techniques to evaluate each role by examining key in-game metrics such as early gold and XP advantages, damage dealt by players, and kill participation.

By analyzing these factors, we seek to determine whether there is a singular role that has a greater impact on match results than other positions. Additionally, we aim to explore the extent to which early-game advantages translate into victories and whether certain statistical patterns about each role can be leveraged to predict game outcomes. This analysis allows for the ability to further understand the impact of roles and their contribution to the final result.

---

## Introduction of Columns

The dataset contains 19,068 rows and 161 columns featuring various statistical outcomes from professional matches. While each column provides key intel on gameplay, here are the columns relevant to our central question:

- **gameid**: A distinct label assigned to each match played.
- **side**: Represents the team affiliation of each player or team—red or blue.
- **position**: The role of each individual player within their team—top, mid, jng (for jungler), bot, sup, or team for team rows.
- **gamelength**: The length of each match played.
- **result**: The outcome of a match for a team or player—1 representing a win and 0 representing a loss.
- **kills**: The total number of enemy champions a player eliminates.
- **deaths**: The number of times each player is eliminated by an enemy champion.
- **assists**: The number of instances where a player helped an ally teammate eliminate an enemy champion without killing them.
- **damagetochampions**: The total amount of damage a player or team deals within a game.
- **visionscore**: Measures how effectively a player controls vision on the map by counting actions such as placing wards, clearing enemy wards, and revealing hidden areas.
- **golddiffat15**: The gold difference at 15 minutes between a player in a specific role and their corresponding opponent in the same role or team.
- **xpdiffat15**: The XP difference at 15 minutes between a player in a specific role and their corresponding opponent in the same role or team.

---

# Data Cleaning and Exploratory Data Analysis

## Data Cleaning

For simplicity let's only keep the relevant columns introduced in the introduction. Each game completes 12 given rows in the dataset, 5 for each member of each team, and the remaining 2 containing gameplay metrics of each team different from individual players. For future analysis I will mainly include player rows only since many of the team rows are sums of the player roles so they would just be repeated information. This dataframe is what will be used in our hypothesis testing and prediction model.

While the dataset included columns such as goldat10, xpat10 for each feature all the way until 25 minutes into the game, I chose to analyze the data at the 15-minute mark because the average game length was approximately 19 minutes and 39 seconds (19:39). Using multiple timestamps would introduce redundancy, and analyzing metrics at later points, such as 20 or 25 minutes, would not be meaningful, given that many matches end before reaching those times.

Below is the head of the dataframe df_cleaned.

![Raw Dataset Sample](screenshot-2025-03-14-233605.png)

## Univariate and Bivariate Analysis

This horizontal histogram shows the sum of gold at 15 minutes per role. We can see that bot has the most by just a little bit while support has much less than the rest. This makes sense and introduces bias against the support role because strategies, especially by professional players, do not funnel gold into support players.

![Gold at 15 Minutes by Role](assets/screenshot-2025-03-14-222802.png)

This bivariate analysis on gold difference at 15 minutes and results visualizes the difference between how much gold the winning team has vs how much gold the losing team has. From this, we can see that winning teams do have more gold and the gold difference between bot is the most significant.

![Gold Difference at 15 Minutes by Role (Winners vs Losers)](assets/screenshot-2025-03-14-222749.png)

## Interesting Aggregates

These aggregates summarize gameplay metrics for each role, highlighting differences in gold, experience, kills, assists, and damage dealt. Bot lane has the highest damage output and assists, reinforcing its role in team fights. Jungle shows high variance in kills, reflecting its impact variability. Support leads in assists but deals the least damage, aligning with its playstyle. Despite these differences, win rates remain balanced, suggesting fair matchmaking.


![Aggregated Role Statistics](assets/screenshot-2025-03-14-233556.png)

---

# Assessment of Missingness

Among our relevant columns, there were no columns that were Not Missing at Random (NMAR). Only a few of our key features contained missing values, including goldat10, xpat10, etc. I performed a permutation test to see if the missingness of the column goldat10 depended on any of the other columns. One column which I believed the missingness depended on was the URL and a column whose missingness did not depend on was damagetochampions. I chose a 0.05 significance level using the difference of means as the test statistic.

### Lets begin with the damagetochampions:

**Null Hypothesis:** The missingness of goldat10 is independent of damagetochampions.  
**Alternative Hypothesis:** The missingness of goldat10 is dependent on damagetochampions.

From performing the permutation test, we got a p-value of 0.29 meaning we fail to reject the null hypothesis, concluding that goldat10 is not dependent on damagetochampions.

Now let's perform the permutation test with the URL column:

**Null Hypothesis:** The missingness of goldat10 is independent of the URL.  
**Alternative Hypothesis:** The missingness of goldat10 is dependent on the URL.

From performing the permutation test, we got a p-value of 0 meaning we reject the null hypothesis. This means that the column goldat10 was entirely dependent on whether the URL was present or not. This is likely because the URL contains stats on the game, hence the creators of the dataset thought it would be unnecessary to include it again.

# Interesting Aggregates

These aggregates summarize gameplay metrics for each role, highlighting differences in gold, experience, kills, assists, and damage dealt. Bot lane has the highest damage output and assists, reinforcing its role in team fights. Jungle shows high variance in kills, reflecting its impact variability. Support leads in assists but deals the least damage, aligning with its playstyle. Despite these differences, win rates remain balanced, suggesting fair matchmaking.

---

# Hypothesis Testing

The purpose of this hypothesis test is to determine whether there is a significant difference in gold difference at 15 across the roles. I performed a two-tailed experiment utilizing Welch’s t-test at a 0.05 significance level.

**Null Hypothesis:** There is no significant difference in gold difference at 15 minutes between winners and losers for a given role. Any observed difference is due to random chance.  
**Alternative Hypothesis:** There is a significant difference in gold difference at 15 minutes between winners and losers for at least one role.

The results of the experiment concluded in extremely low p-values, close to 0, for all roles. Hence, we reject the null hypothesis. This means that the gold difference at 15 minutes is significantly different between winners and losers for every role. Since the t-statistics are all positive, winners generally have higher gold differences at 15 minutes compared to losers.

---

# Framing a Prediction Problem

Our hypothesis testing concluded that gold difference plays a significant difference across all roles in the outcome of winning or losing a game. Our exploratory data analysis showed us that the statistics differ for each role. Since we saw that the bot role had the greatest amount of both gold at 10 minutes and damage dealt to enemy champions at 10 minutes, we can frame our prediction problem around the bot role.

Our prediction problem will be to try to predict the outcome of a game based on the stats of the bot player. In order to do this, we need to construct a model that takes in early-game statistics such as gold difference at 15 minutes (golddiffat15) and XP difference at 15 minutes (xpdiffat15) as input features and outputs whether the bot player’s team will win or lose the game.

---

# Baseline Model

### Model Description

The model used in this analysis is a logistic regression classifier, which is a linear model designed for binary classification tasks. In this case, it predicts whether the bot lane player's team will win or lose based on early-game statistics.

### Model Performance

The logistic regression model achieved an accuracy of 67.8%, indicating moderate predictive capability. The F1-scores for each class were 0.684 (loss) and 0.671 (win), with a weighted average of 0.678.

### Evaluation

While the model provides a reasonable baseline, its accuracy and F1-scores indicate that it is not highly reliable. The results suggest that gold and XP differences at 15 minutes are useful indicators of match outcomes, but they are likely not the only determining factors.

---

# Final Model

### Feature Engineering & Selection

In refining our model, we explored different feature combinations to improve predictive performance. One key feature we added was kill participation (kill_participation), which is an engineered feature. Kill participation measures the proportion of team kills a player is involved in and serves as a strong indicator of player impact on the game.

### Model Performance Comparison

The logistic regression model, which served as our baseline, achieved an accuracy of 67.8% and a weighted F1-score of 0.678. In contrast, the final random forest classifier achieved an accuracy of 80.8% and a weighted F1-score of 0.808.

---

# Fairness Analysis

For our hypothesis test, we define Group X as players who had a gold advantage at 15 minutes (gold_advantaged) and Group Y as players who were at a gold disadvantage at 15 minutes (gold_disadvantaged). The goal of this test is to determine whether having a gold advantage at 15 minutes leads to significantly better model performance when predicting match outcomes.

**Null Hypothesis:** There is no significant difference in model performance between gold-advantaged and gold-disadvantaged players.  
**Alternative Hypothesis:** There is a significant difference in model performance between gold-advantaged and gold-disadvantaged players.

Both p-values for the permutation test were 0.024 for precision and 0.019 for F1-score, which are below the threshold of 0.05. Therefore, we reject the null hypothesis. This result confirms that having a gold advantage at 15 minutes is associated with significantly better model performance in predicting match outcomes. These findings suggest that early-game economic advantages translate into more predictable game results, reinforcing the importance of gold leads in competitive League of Legends play.

---