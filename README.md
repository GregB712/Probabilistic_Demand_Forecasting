# 📊 Probabilistic Demand Forecasting Challenge

This repository contains my solution to a demand forecasting challenge involving the prediction of **daily probabilistic demand distributions** for item-store combinations. The solution is structured across two Jupyter notebooks and is designed to be easy to run end-to-end.

---

## 📁 Project Structure

Place the following files in the **same directory**:

```
├── Basic_Data_Exploration_and_Merging.ipynb       # EDA, cleaning, feature engineering
├── Probabilistic_Forecast_Generation.ipynb        # Modeling, forecasting, PMF construction
├── sales.csv                                      # Historical sales data / not included
├── promo_price.csv                                # Promotional pricing data / not included
├── regular_price.csv                              # Regular pricing data / not included
├── requirements.txt                               # Python dependencies
```

---

## 🚀 How to Run

### 1. Install Dependencies

```bash
pip install -r requirements.txt
```

### 2. Execute Notebooks

Run the notebooks **in order**, without skipping cells:

1. **`Basic_Data_Exploration_and_Merging.ipynb`**

   * Loads and merges raw data
   * Handles missingness, outliers, and promo adjustment
   * Performs exploratory data analysis and feature engineering

2. **`Probabilistic_Forecast_Generation.ipynb`**

   * Trains LightGBM quantile regression models
   * Generates forecasts for the target week (Sept 12–18, 2022)
   * Constructs a discrete probability mass function (PMF) per forecasted demand

> ✅ Both notebooks assume all files are in the same folder.
> ✅ No additional configuration is needed.

---

## 📌 Key Technologies

* Python, Jupyter Notebooks
* LightGBM (quantile regression)
* Pandas, NumPy, Scikit-learn
* Plotly, Matplotlib (interactive and static visualizations)

---

## 📄 Output

The final output includes:

* Forecasted **quantiles (q10, q50, q90)** for each item-store-date
* A **discrete probability mass function (PMF)** over demand values
* Visualizations of historical trends, forecast intervals, and features' importance

---

## 📬 Questions?

Feel free to reach out if you’d like to discuss the approach, design decisions, or potential extensions.

---

