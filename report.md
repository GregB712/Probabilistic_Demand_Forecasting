# 📊 Demand Forecasting Challenge Report

## 1. 🧩 Problem Description

The task was to forecast the **full probability distribution** of **daily demand** for each **item-store combination** from **September 12 to 18, 2022**.

* **Objective**: Produce probabilistic forecasts rather than point estimates, capturing uncertainty around demand to better support inventory and operations planning.
* 
* 📌 **Assumptions**:

  * All stores are open every day during the forecast period.
  * There are no stockouts during the prediction window.
* 🧠 **Why probabilistic forecasting?**

  * Captures uncertainty, improves decision-making in inventory, pricing, and logistics.

---

## 2. 🔍 Exploratory Data Analysis (EDA)

### 📅 Dataset Overview

* **Time Range**: January 2019 to September 2022 (\~3.5 years)
* **Granularity**: Daily sales per item-store combination
* **Entities**: 6 items × 4 stores = 24 combinations
* **Total Observations**: 12,118 rows

This time span is long enough to capture seasonal effects, promotion cycles, and external disruptions (e.g., COVID-19). The item-store structure supports hierarchical modeling approaches, and the compact size enables detailed exploration at each level.

---

### 📊 Summary Statistics

| Variable        | Mean   | Std Dev | Median | Max    |
| --------------- | ------ | ------- | ------ | ------ |
| `sales_value`   | 520.02 | 1071.97 | 19.40  | 20,434 |
| `regular_price` | 175.37 | 130.07  | —      | 330.40 |
| `promo_price`   | 186.53 | 114.57  | —      | 296.00 |

Sales values are heavily right-skewed with high variance, suggesting the presence of demand spikes. This supports the use of quantile regression rather than mean-based methods.

`promo_price` is only present in \~2% of rows, so I chose to rely primarily on a derived binary `promo_flag` feature for modeling promotional effects.

* **Sales Value**
  * Mean: 520.02 | Std: 1071.97 | Max: 20,434 | Median: 19.40
  * High variance and skewness indicate presence of outliers and long-tail behavior.

* **Regular Price**
  * Mean: 175.37 | Std: 130.07
  * Stable distribution with meaningful item-level variability.

* **Promo Price**
  * Present in only \~2% of rows (253/12118)
  * Always lower than regular price when present.
  * Promotional periods show **higher variance and uplift** in sales.

---

### ❓ Missingness & Temporal Coverage

| Column        | Missing Values |
| ------------- | -------------- |
| `promo_price` | 11,865 (\~98%) |
| Other columns | 0              |

Most item-store time series were nearly complete, with over 99% coverage. A few combinations (e.g., item5-store3) had significant gaps (coverage ≈ 81%). To handle this, I created a complete daily index for each pair and used linear interpolation when necessary.

![Missing Days & Coverage](assets/Notebook1%20-%20Distribution%20of%20Missing%20Days%20and%20Date%20Coverage%20Percentage.png)

---

### 🗓️ Seasonality, Trend & Promotion Patterns

I visualized aggregate sales over time to understand trends and seasonal behaviors:

#### 📆 Monthly Trends

![Monthly Sales - All Stores/Items](assets/Notebook2%20-%20Total%20Sales%20Per%20Month%20All%20Stores%20and%20Items.png)

* **Monthly seasonality** is evident, with peaks in some months and dips during others.
* **COVID-era dips** are visible around early 2020.

#### 📅 Day-of-Week Effects

![Day-of-week Breakdown](assets/Notebook2%20-%20Total%20Sales%20Per%20dayofweek.png)

* Suggests weekly seasonality; Sunday dips or Friday peaks in some combos
* **Day-of-week effects** (e.g., stronger sales on weekends) suggest including calendar-based features.

![Day of Week Patterns](assets/Notebook2%20-%20Total%20Sales%20Per%20dayofweek.png)

These insights guided the creation of time-based features that help the model capture structured, repeated patterns in demand.

#### 📊 Promotion Analysis

* Promotional periods show **higher variance and uplift** in sales.

---

## 3. 🧹 Data Cleaning & Outlier Handling

### Outlier and Anomaly Detection

I applied the following logic:

* **IQR-based filtering** for non-promo, non-missing sales to detect extreme values.
* **Abnormal zero-sales runs** (>3 consecutive days) were flagged as potential anomalies.

This approach helps reduce noise and prevents the model from learning misleading patterns from rare or abnormal events.

### Imputation Strategy

| Issue               | Strategy                                        |
| ------------------- | ----------------------------------------------- |
| Missing days        | Synthetic rows + interpolation                  |
| Missing prices      | Forward-filled `regular_price`                  |
| Promo effect        | Estimated and subtracted to get `sales_depromo` |
| Outliers (IQR rule) | Interpolated if flagged                         |
| Zero-sale anomalies | Flagged + interpolated                          |

Additionally, I estimated the **average uplift from promotions** and created a `sales_depromo` column by adjusting promotional-day sales. This provided a cleaner target for modeling baseline demand behavior.

![Sales & Promo Adjustments](assets/Notebook2%20-%20Sales%20and%20Prices.png)

This adjustment ensures that the model focuses on demand drivers beyond promotions, with promotional effects handled as an additive component later in the pipeline.

---

## 4. 🏗️ Feature Engineering

### Feature Set Overview

I designed features that capture:

* **Seasonality**: Month, week of year, day of week
* **Trends**: Lagged and rolling averages of historical sales
* **Price sensitivity**: Regular price, promo flag
* **Data reliability**: Flags for missing data, interpolations, and outliers

These features allow the model to learn patterns across multiple temporal scales and account for known promotional effects.

### 🔑 Core Features Used

| Category                | Examples                                           |
| ----------------------- | -------------------------------------------------- |
| **Calendar**            | `month`, `weekofyear`, `dayofweek`                 |
| **Lag/Rolling**         | `monthly_avg_sales_lag1`, `weekly_avg_sales_roll3` |
| **Price/Promo**         | `regular_price`, `promo_flag`                      |
| **Preprocessing Flags** | `interpolated_flag`, `outlier_flag`                |

> Inclusion of `sales_depromo` was in order to isolate organic demand from promo-induced spikes.

### 🧠 Additional Suggested Features

If available, the following could further enhance accuracy:

* **Weather data** (e.g., rain, temperature)
* **Holiday indicators**
* **Regional events or sports matches**
* **Social media buzz**
* **Competitor pricing/promotions**
* **Foot traffic or store-level capacity**
* **Marketing campaigns**

These could further enhance the model's responsiveness to external demand drivers.

---

### 📊 Feature Importance Insights

![Feature Importance](assets/Notebook2%20-%20Feature%20Importances.png)

Interestingly, `promo_flag` ranked among the least important features. This supports the decision to model **de-promo-adjusted sales (`sales_depromo`)** as the target variable — by removing the direct promotional uplift, the model could focus more on organic demand patterns that generalize better over time.

The top contributors were:

* `regular_price`: indicating strong price sensitivity in demand
* `weekly_avg_sales_y`: reflecting short-term sales trends
* `weekofyear` and `monthly_avg_sales_y`: capturing seasonal behavior at different granularities

These results validate the value of including calendar-based and rolling sales features to model recurring demand cycles effectively.


---

## 5. 🤖 Modeling Approach

### 🧮 Problem Setup

* Framed as a **supervised learning** task.
* Target variable: `sales_depromo` (sales with promo effect removed).

### Model Selection

I chose **LightGBM with quantile regression** because:

* It efficiently handles large feature sets with non-linear interactions.
* Its native support for **quantile objectives** allows direct modeling of predictive intervals.
* It provides features' importance, improving model interpretability.

Certainly! Here’s a clear explanation of **quantiles** and **pinball loss**, written in the same professional tone as your report:

---

#### 📐 What Are Quantiles?

Quantiles are values that divide a probability distribution into intervals with equal probabilities. In forecasting, they are used to model different points of a predicted distribution instead of a single average value.

For example:

* The **10th percentile (q10)** means there's a 10% chance the actual demand will fall below this value — a *low estimate*.
* The **50th percentile (q50)** is the **median** — a *central estimate* where there's a 50% chance demand will be above or below.
* The **90th percentile (q90)** indicates there's a 90% chance demand will fall below this value — a *high estimate*.

##### Why use quantiles?

Quantile forecasts provide a **distributional view** of future demand rather than a single point forecast. This is especially useful in retail, where the cost of overestimating (e.g., overstock) and underestimating (e.g., stockouts) is not symmetric.

---

### Evaluation Metric

I used **Quantile Loss (Pinball Loss)**, which is appropriate for assessing accuracy across different quantiles of the forecast distribution. It penalizes underestimation and overestimation asymmetrically, aligning well with inventory management goals.

#### 📉 What Is Pinball Loss?

The **pinball loss** (also called quantile loss) is the evaluation metric used in quantile regression. It measures how well a predicted quantile matches the actual observed value. It is **asymmetric**, meaning it penalizes over- and under-predictions differently based on the quantile.

The formula for pinball loss for a quantile `q` is:

$$
L_q(y, \hat{y}) = 
\begin{cases}
q \cdot (y - \hat{y}) & \text{if } y \geq \hat{y} \\
(1 - q) \cdot (\hat{y} - y) & \text{if } y < \hat{y}
\end{cases}
$$

Where:

* $y$ is the actual value
* $\hat{y}$ is the predicted quantile value
* $q \in (0, 1)$ is the target quantile (e.g., 0.1, 0.5, 0.9)

##### Why use pinball loss?

Unlike MSE or MAE, which evaluate accuracy of a mean prediction, pinball loss is **tailored for quantile forecasts**. It encourages the model to learn the correct conditional quantile of the target distribution, making it ideal for **probabilistic forecasting**.

### Data Splitting

* Used **TimeSeriesSplit** to ensure validation folds respected the chronological order of data.
* Avoided leakage of future information, which is critical for forecasting accuracy.

### 🔧 Grid Search and Final Training

* Performed **grid search** for each quantile: **q10, q50, and q90**
* Selected best models based on cross-validated pinball loss.
* Trained final models on the full training set using best parameters per quantile

---

## 6. 🔮 Forecasting & Output

### Prediction Strategy

For the forecast period (Sept 12–18), I:

1. Created a full item-store-date grid for all 7 days.
2. Merged the latest historical rolling/lagged features and price/promo data.
3. Ran all three quantile models to generate:

   * 10th percentile forecast (`pred_q10`)
   * 50th percentile forecast (`pred_q50`)
   * 90th percentile forecast (`pred_q90`)

### Forecast Visuals

![Forecast with history](assets/Notebook2%20-%20Forecast%20with%20History%20Item1%20Store1%20v2.png)

This plot overlays historical demand with the forecasted distribution interval:

* The shaded region between `q10` and `q90` shows the forecast uncertainty.
* The median line (`q50`) represents the most likely outcome.
* Captures both expected demand and potential volatility.

---

## 7. 📐 Generating Discrete Demand Distribution (PMF)

To fulfill the requirement of producing a **discrete probability distribution** for daily item-store demand, I used the quantile forecasts (q10, q50, q90) to construct an approximate **probability mass function (PMF)** over integer demand values.

### 🧮 Methodology

* For each `(item_id, store_id, date)` combination, I interpolated a **triangular-like distribution** using the predicted quantiles:

  * **q10** (lower bound)
  * **q50** (most likely / mode)
  * **q90** (upper bound)
* I discretized this continuous distribution by rounding to the nearest integer values within a reasonable range (e.g., 0–21).
* Probability weights were assigned to each integer demand level such that the resulting distribution:

  * Approximates the spread implied by the quantiles
  * Sums to 1 (valid PMF)

### 🔍 Why This Approach?

This approach allows us to represent uncertainty in a format suitable for downstream use cases, such as:

* **Inventory simulation**
* **Stochastic optimization**
* **Risk-aware decision making**

### ✅ Final Output Format

For each item-store-date triple in the forecast window, the final output was a **discrete probability distribution** over demand values:

| demand | prob |
| ------ | ---- |
| 0      | 0.05 |
| 1      | 0.12 |
| 2      | 0.17 |
| ...    | ...  |
| 10     | 0.01 |

This transformation from quantiles to PMF aligns with the **probabilistic forecasting goal** and ensures compatibility with systems that require categorical demand probabilities rather than continuous estimates.

---

## ✅ Reflections on Design Choices

### ✔ Why LightGBM + Quantile Regression?

* Efficient and scalable for tabular, time-aware data
* Allows explicit modeling of uncertainty — essential in retail contexts where over/under-forecasting have asymmetric costs

### ✔ Why Quantile Loss?

* Unlike MSE/MAE, quantile loss aligns with business needs (e.g., planning for 90th percentile to avoid stockouts)
* Provides a richer view of forecast uncertainty

### ✔ Why Use `sales_depromo`?

* By removing promotional effects from the target, the model focuses on **organic demand** patterns.
* This avoids overfitting to short-term promotional spikes and improves generalizability.

### ✔ Why Calendar Features?

* Sales patterns are highly seasonal — using `month`, `weekofyear`, `dayofweek` allows the model to learn these patterns explicitly.

## ⚠️ Opportunities for Improvement

* 🔄 **Promo modeling** could be further enhanced with imputation or uplift modeling techniques.
* 📉 Consider **evaluating CRPS** in future versions for richer distributional accuracy.
* 📅 A **holiday calendar** or external events could boost forecast performance.

---

## 📦 Summary

| Component            | Method Used                          |
| -------------------- | ------------------------------------ |
| **Model**            | LightGBM Quantile Regression         |
| **Evaluation**       | Quantile Loss (Pinball Loss)         |
| **Features**         | Calendar, Promo, Lagged/Rolling      |
| **Target Variable**  | `sales_depromo` (promotion-adjusted) |
| **Forecast Output**  | q10, q50, q90 quantiles              |
| **Forecast Horizon** | Sept 12–18, 2022                     |

---

