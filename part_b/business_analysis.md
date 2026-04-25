##### **Part B: Business Case Analysis**

###### **B1. Problem Formulation**

a).

1\. Target Variable (What we want to predict)



The goal is to maximize sales, so the natural target variable is:



items\_sold (per store per month)



This is a continuous numerical value, not a category.



2\. Input Features (What we use to make predictions)



These come from three main sources:



A. Promotion-related features

* Promotion type (Flat Discount, BOGO, Free Gift, etc.)
* Discount percentage (if applicable)
* Promotion duration (if variable)



B. Store characteristics

* Store size (square footage)
* Location type (urban / semi-urban / rural)
* Monthly footfall
* Local competition density
* Historical sales performance



C. Customer \& temporal features

* Customer demographics (age group distribution, income level, etc.)
* Month / seasonality (festive season, summer, etc.)
* Past promotion performance at that store
* Loyalty program participation rate





3\. Type of Machine Learning Problem



This is primarily a Supervised Learning - Regression problem.

We use Regression:

* The target (items\_sold) is continuous numeric.
* We want to predict a quantity, not a category.
* Models like Linear Regression, Random Forest Regressor, etc., are suitable.



4\. Business Interpretation

While the ML formulation is regression, the business objective is actually: Promotion optimization (decision-making problem).

Therefore, in practice, we would:



* Train a regression model to predict items\_sold
* For each store-month:

&#x20;   i). Try all 5 promotions as inputs.

&#x20;  ii). Predict sales for each.

&#x20; iii). **Select the promotion with the highest predicted sales.**



b).

Using items sold (sales volume) is a more reliable target variable for this specific problem, as using total sales revenue for this specific problem can actually be misleading when evaluating promotion effectiveness.



Total Sales Revenue is not ideal here, as:

Revenue is influenced by multiple factors beyond just how effective a promotion is:



* Price variations across promotions:

&#x20;  A Flat Discount reduces price, while a Free Gift or Loyalty Points scheme may keep prices unchanged. This means two promotions could generate very different revenues even if they sell the same number of items.



* Product mix differences:

&#x20;  Some promotions may push high-priced items, others may drive volume in low-priced categories. Revenue will fluctuate based on what gets sold—not just how much gets sold.



* Artificial inflation or suppression

&#x20;  A BOGO offer might double the number of items sold but reduce revenue per item. Revenue alone would understate its true impact on customer demand.

* Different promotions can shift customers toward cheaper or more expensive products.

&#x20;  Because of this, revenue becomes a mixed signal:



&#x20;  It reflects **price × quantity**, not purely customer response. So, if we train a model on revenue:



* It may wrongly conclude that a promotion is “bad” simply because it reduces price.
* It becomes difficult to compare promotions fairly.
* The model learns pricing effects rather than true promotion effectiveness.



Using items\_sold focuses directly on customer response to the promotion:



* It measures true demand uplift
* It is not distorted by price changes
* It allows fair comparison across different promotion types
* It better reflects the objective of increasing store activity and engagement



In short, it isolates the causal impact of the promotion more cleanly.



Broader Principle (General ML Insight)



We can choose a target variable that directly captures the outcome we want to optimize, and avoid targets that are confounded by multiple underlying factors.



In practice, a good target should be:



* Causally aligned with the decision (what you want to influence)
* Unambiguous (not mixing multiple effects like price + demand)
* Comparable across different scenarios or treatments



c).

Running one single global model is simple—but it assumes all stores behave the same way, which is rarely true here. Stores in urban vs rural areas, or with different customer profiles, can respond very differently to the same promotion. So a better approach is to explicitly model that variation.



As an effective alternative, we can use Hierarchical (Multi-level) or Segmented Modeling.

In Hierarchical / Mixed-Effects Model



We build a model that has:



* Global effects-overall impact of each promotion.
* Store-level (or region-level) effects-how each store deviates from the average.



> The model learns a general pattern across all stores.

> Then adjusts predictions for each store (or group of stores).



This is better because:



* It captures store-specific behavior without needing 50 completely separate models.
* It handles data imbalance (some stores may have less historical data).
* It shares learning across stores while still allowing customization.



Segmented Models (Cluster-Based Approach)



Instead of one global model:



Group stores based on similarity:

* Location type (urban / rural)
* Footfall
* Customer demographics
* Train one model per segment



This works because:

Stores in the same segment likely respond similarly to promotions.

Simpler than full hierarchical modeling.

Improves accuracy over a single global model.



Justification (Core Idea)



Different stores have:



* Different customer behavior
* Different competition levels
* Different sensitivity to promotions



A single global model averages out these differences, which can lead to:



> Poor predictions for certain stores

> Suboptimal promotion decisions



Broader Insight:

When data comes from heterogeneous groups, models should explicitly account for that structure - either by sharing information intelligently (hierarchical models) or separating groups (segmentation).



###### **B2. Data and EDA Strategy**



a). To build a reliable model, the first step is to combine all four tables into a single, consistent dataset with the right level of detail (grain).

To Join the Tables:



We start with the transactions table as the base because it contains the core sales activity.



Step-by-step joins:

Transactions + Store Attributes

Join on: store\_id

Additions: store size, location type, competition density, etc.

Transactions + Promotion Details

Join on: (store\_id, month) or (store\_id, date) depending on how promotions are recorded

Additions: promotion type, discount level, etc.

Transactions + Calendar Table

Join on: transaction\_date

Additions: weekend flag, festival flag, seasonality indicators



Grain of the Final Dataset:-



After joining and aggregation, the final dataset should have:



One row = one store × one month



* Promotions are run monthly per store.
* The business decision is: which promotion to assign to each store each month.
* Aggregating to this level aligns perfectly with the decision-making unit.



Aggregations to Perform: Since transactions are typically at a per-purchase level, we need to aggregate them to the store-month level.



Target Variable

items\_sold - sum of quantity sold per store per month.



**Feature Aggregations:**

**Sales \& Activity Metrics**

Total transactions (count of transactions)

Total revenue (optional, for analysis)

Average basket size (items per transaction)

**Customer Behavior**

Average items per transaction

Repeat purchase indicators (if customer IDs available)

**Footfall Proxy**

Number of unique transactions (proxy for store traffic)

**Calendar Features**

Number of weekends in the month

Number of festival days

Binary: is\_festival\_month

**Promotion Features**

Promotion type (categorical)

Discount intensity (if available)

**Store Features (no aggregation needed)**

Store size

Location type

Competition density



b). Before modelling, the goal of EDA is to understand patterns, detect biases, and uncover relationships that will directly shape features and model choice. Here are key analyses:



**1. Promotion Type vs Items Sold (Boxplot / Bar Chart)**

Plot average (or distribution of) items\_sold for each promotion type.



What to look for:



> Which promotions generally perform better

> Variability within each promotion (consistent vs risky promotions)



How it will help:



It identifies strong baseline predictors

It may suggest:

Encoding promotion as categorical

Creating promotion ranking features



**2. Store Segmentation Analysis (Grouped Comparison)**

Compare items\_sold across:

* Location type (urban / rural)
* Store size
* Possibly use grouped bar charts or facet plots



What to look for:



> Do different store types respond differently?

> Are some promotions only effective in certain segments?



How it will help:



It motivates:

> Segmentation or hierarchical models

> Interaction features like:

&#x20; promotion × location\_type

&#x20; promotion × store\_size



**3. Time Series Analysis (Line Plot over Months)**

Plot monthly items\_sold trends per store or overall.



What to look for:



* Seasonality (festivals, holidays)
* Trends (growth/decline)
* Sudden spikes (promotion-driven or anomalies)



How it will help:



* It adds features like:

&#x20;   month, season, festival\_flag

&#x20;   Lag features (previous month sales)

* Ensures proper temporal split for modelling



**4. Correlation Analysis (Heatmap / Pair Plot)**



Compute correlation between numerical features and items\_sold



What to look for:



> Strong predictors (e.g., footfall, transactions)

> Multicollinearity (features highly correlated with each other)



How it will help:



* Feature selection (remove redundant variables)
* Better model stability
* It helps interpret feature importance later



c). Handling Imbalance:  80% of transactions in the dataset occurred without any promotion.



1. How it Affects the Model:



* If 80% of historical observations have no promotion active, a model trained naively on this data will have seen very few examples of promotional conditions.
* It will underfit the promoted scenarios and may systematically under-predict items sold during promotions.
* In other words, it could learn that "no promotion" is the **baseline** and attribute any lift to confounding features- like a festival month or a high-footfall store - rather than the promotion itself. This produces a model that is poorly calibrated precisely in the decision space that matters most.
* It has potential negative effects like biased predictions toward non-promotion scenarios.



2\. How to Address It



(a) Resampling the Data:

&#x20;   > Oversample promotion cases or undersample non-promotion cases.

&#x20;   > This helps balance representation.



(b) Apply case-based weighting:

&#x20;   > Assign higher weights to rows with promotions during training.

&#x20;   > This forces the model to pay more attention to promotion effects.



(c) Separate Modelling Approach (Two-Stage Model)

&#x20;   > Model baseline demand (no promotion)

&#x20;   > Model uplift due to promotion

&#x20;   > This isolates the true effect of promotions



(d) Stratified Evaluation

&#x20;   > Evaluate model performance separately for:

&#x20;     Promotion months

&#x20;     Non-promotion months

&#x20;   > This ensures the model works well where it matters



(e) Feature Engineering

&#x20;   > Create: is\_promotion (binary flag)

&#x20;   > Interaction terms with promotion

&#x20;   > This helps the model explicitly distinguish cases



###### **B3. Model Evaluation and Deployment**



**(a). Train-Test Split and Evaluation**

A Random Split is inappropriate as:

This dataset has an inherent temporal ordering - each row represents a store-month observation, and future months are causally downstream of past ones. A random split would allow data from, say, Month 30 to appear in the training set while Month 28 appears in the test set. This constitutes data leakage-the model would have implicitly seen the future data during training, producing optimistically inflated evaluation metrics that will not hold in deployment. Additionally, any time-based features (seasonal trends, festival cycles, evolving customer behaviour) would be corrupted because the model would learn patterns from future periods and apply them to past ones. Hence, mixing past and future data.



Recommended Approach: Temporal (Walk-Forward) Split

With 36 months of data across 50 stores, the correct strategy is a chronological split:



* Training set: Months 1–24 (first two years, all 50 stores)
* Validation set: Months 25–30 (used for hyperparameter tuning)
* Test set: Months 31–36 (final six months, held out entirely until model is finalised)



Evaluation Metrics and Business Interpretation:

Since the core task is selecting the best promotion per store per month (a regression problem), metrics should reflect both prediction accuracy and decision quality:



1\. RMSE (Root Mean Squared Error)

* Penalizes large errors heavily.
* Useful when big mistakes (wrong promotion choice) are costly.



Interpretation:

We can question, “How far off (in units sold) are our predictions, especially large misses?”



2\. MAE (Mean Absolute Error)

* Measures average absolute deviation.
* More robust to outliers.



Interpretation:

We put the question, “On average, how many units are we off per store-month?”



* MAE is preferred here because it is interpretable in the same units as the target (items) and is less sensitive to outlier months such as festival spikes.



**(b). Explaining Different Recommendations (Feature Importance)**

The model gives:

December- Loyalty Points Bonus

March- Flat Discount



Explaining why the Model recommends differently for the same store:



* The model sees Store 12 identically in both months in terms of static attributes (size, location type, competition density), but the time-varying features differ significantly between December and March.
* December carries high festival-day counts, elevated footfall, and strong seasonal demand.
* March is a relatively flat month with standard footfall and no major calendar events.
* The model has learned, from historical data, that Loyalty Points Bonus yields the highest items\_sold in high-footfall, high-engagement periods - because customers are already motivated to spend and the loyalty mechanic rewards repeat visits within a concentrated window.
* Flat Discount, by contrast, drives volume most effectively in low-motivation periods by lowering the price barrier to purchase.



Checking via Feature Importance

Using a tree-based model (e.g., XGBoost or Random Forest), extract SHAP values for each store-month prediction. SHAP values decompose each prediction into the additive contribution of every individual feature, making them ideal for explaining why two predictions differ.

We look at: Promotion type importance, seasonal features (month, festival\_flag) and store characteristics, which identifies key drivers overall.

In December case, likely drivers are:

* Festival/holiday season.
* High footfall.
* Customers responsive to loyalty incentives.

In March case, likely drivers are:

* Lower demand period.
* Need stronger incentive.
* Thus, flat discount may stimulate demand more effectively.



Communicating to the Marketing Team:

The model makes different recommendations for the same store (Store 12) in different months as:

In December, demand is already high due to seasonal factors, so offering loyalty points helps maximise value without cutting prices. In March, demand is lower, so a direct discount is more effective in driving purchases.



**(c) End-to-End Deployment Process**

The goal is to make monthly recommendations without retraining every time.

1\. Save the Trained Model

* Use: joblib or pickle (for scikit-learn pipelines)
* And save: Model + preprocessing pipeline together
* This ensures consistency between training and inference



Preparing and Feeding Monthly Data

At the start of each month:

An automated pipeline runs the following steps: pull the previous month's transaction data from the data warehouse and aggregate it to store-month grain; join with current store attributes, the deployed promotion's details, and the upcoming month's calendar features (month, festivals). Apply the saved preprocessing pipeline to produce a feature vector for each of the 50 stores; and pass all 50 feature vectors through the saved model in a single batch inference call. 

Feature engineering should have same transformations as training:

> Aggregations

> Encoding

> Scaling

The steps must be identical to training pipeline. The entire pipeline should include data validation checks at each join step to catch missing stores, null footfall values, or unrecognised promotion codes before inference runs.



Generate Recommendations - for each store:

> Simulate all promotion types

> Predict items\_sold for each

> Select promotion with highest predicted sales

The output- a ranked list of promotions per store - is written to a recommendations table accessible by the marketing dashboard.

Table: store\_id, recommended\_promotion, expected\_items\_sold



Monitoring for Degradation and Retraining Strategy

Three layers of monitoring should be in place:

(a) Data Drift Monitoring

Check if input distributions change: (Footfall, Customer behavior, Promotion usage). If drift is detected, the model may become unreliable.



(b) Performance Monitoring

> Compare: Predicted vs actual sales (monthly)

> Track: RMSE / MAE over time

> This detects degradation.

> Compute rolling MAE and Hit Rate over the past three months. Set threshold alerts: if Hit Rate drops below, say, 60% for two consecutive months or MAE increases by more than 20% above the baseline test-set MAE, trigger a retraining job automatically.



(c) Business-logic sanity checks 

* Flag any month in which the model recommends the same promotion for more than 40 of 50 stores, as this likely indicates a degenerate model that has collapsed to a single dominant pattern. Similarly, alert if predicted items sold are implausibly high or low relative to historical store ranges.
* Retraining should use the walk-forward approach on all available data up to the current month, re-evaluate on the most recent six months as a holdout, or when performance drops beyond threshold, and only promote the new model to production if its metrics are equal to or better than the incumbent.
* This prevents hasty replacement of a working model with one that merely overfits to a recent anomaly.



