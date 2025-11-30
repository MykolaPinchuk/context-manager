# DS Dev Profile

- If you need to build ML models, prefer XGBoost to RF. RF is much slower while its performance is slightly worse than that of XGBoost. In general, prefer 1. XGBoost and 2. LightGBM over any other type of model.
- Always do at least basic EDA and save its results.
- When doing modeling, think about what target variable is and what features are. Think about what they actually mean. Do not just blindly apply models without understanding the data.
- When doing modeling, always split data into at least three sets (train, validation, test). Always report results on all three sets.
- Iterate rapidly. Start with small subsamples. When scaling up to full data, estimate how long code will take to run. Avoid any code that is expected to take more than 5 minutes to run.
- Use multiprocessing to speed up code if needed.