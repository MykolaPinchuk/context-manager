# DS Dev Profile

- If you need to build ML models, prefer XGBoost to RF. RF is much slower while its performance is slightly worse than that of XGBoost. In general, prefer 1. XGBoost and 2. LightGBM over any other type of model.
- Always do at least basic EDA and save its results.
- When doing modeling, think about what target variable is and what features are. Think about what they actually mean. Do not just blindly apply models without understanding the data.
- When doing modeling, always split data into at least three sets (train, validation, test). Always report results on all three sets.
- Iterate rapidly. Start with small subsamples. When scaling up to full data, estimate how long code will take to run. Avoid any code that is expected to take more than 5 minutes to run.
- Use multiprocessing to speed up code if needed.
- For each dataset/modeling problem, create a eda_summary.md. It should contain:
  - description of the dataset/problem. It should describe which problem modeling task will try to solve, what target variable means, what key features are, and what they mean.
  - basic EDA results (distributions of key variables, missingness, correlations).
  If the task includes building and evaluating a model, such information may be included into model_summary.md instead of eda_summary.md. What amtters is that any run should have at least one of these two files describing the data.
- When reporting a subtask as complete, clearly specify where the main results/reports which human will likely want to see are located.
