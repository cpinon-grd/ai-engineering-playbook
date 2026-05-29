---
name: pandas-analysis-guidelines
description: Use when writing, reviewing, refactoring, debugging, testing, documenting, or explaining Python data analysis code that uses pandas, numpy, matplotlib, seaborn, Jupyter notebooks, scikit-learn, data validation, visualization, exploratory analysis, feature engineering, or reproducible analytical workflows.
---

# Pandas Analysis Guidelines

Apply these rules when generating or reviewing Python data analysis, visualization, notebook, or scikit-learn workflow code. Favor concise, technical, reproducible examples that are readable and easy to rerun.

## Core Principles

- Use pandas for tabular data manipulation and analysis.
- Use numpy for numerical operations and array-oriented calculations.
- Prefer vectorized pandas or numpy operations over explicit Python loops.
- Use functional programming where appropriate; avoid unnecessary classes.
- Use descriptive variable names that reflect the data they contain.
- Follow PEP 8 style guidelines.
- Keep examples concise and technically accurate.

## Data Analysis and Manipulation

- Begin analysis with data exploration and summary statistics.
- Inspect shape, columns, dtypes, missingness, ranges, and representative rows.
- Prefer method chaining for readable transformation pipelines when it remains clear.
- Use `.loc` and `.iloc` for explicit row and column selection.
- Use `groupby` operations for efficient aggregation.
- Preserve reproducibility by making filtering, cleaning, and transformation assumptions explicit.

```python
summary = (
    sales.loc[:, ["region", "revenue", "order_date"]]
    .assign(order_date=lambda df: pd.to_datetime(df["order_date"]))
    .groupby("region", as_index=False)
    .agg(total_revenue=("revenue", "sum"), orders=("revenue", "size"))
)
```

## Data Validation and Error Handling

- Run data quality checks at the beginning of analysis.
- Validate required columns, data types, value ranges, uniqueness, and missingness.
- Handle missing data intentionally through imputation, removal, or explicit flags.
- Use targeted `try`/`except` blocks for error-prone external data reads.
- Fail early with clear messages when required data contracts are not met.

```python
required_columns = {"customer_id", "order_date", "revenue"}
missing_columns = required_columns - set(orders.columns)
if missing_columns:
    raise ValueError(f"Missing required columns: {sorted(missing_columns)}")
```

## Visualization

- Use matplotlib for low-level plot control and customization.
- Use seaborn for statistical plots and strong defaults.
- Always add informative labels, titles, and legends when useful.
- Choose chart types that match the comparison or distribution being shown.
- Use accessible color schemes and consider color-blindness.
- Create reusable plotting functions when consistent visual style is needed.

```python
def plot_revenue_by_region(summary: pd.DataFrame) -> None:
    """Plot total revenue by region."""
    ax = sns.barplot(data=summary, x="region", y="total_revenue", color="steelblue")
    ax.set_title("Total Revenue by Region")
    ax.set_xlabel("Region")
    ax.set_ylabel("Total revenue")
```

## Jupyter Notebook Practices

- Structure notebooks with clear markdown sections.
- Keep cell execution order meaningful and reproducible.
- Include explanatory markdown that documents analysis steps, sources, assumptions, and methodology.
- Keep code cells focused and modular.
- Use `%matplotlib inline` when inline plotting is needed.
- Avoid hidden state by rerunning notebooks from a clean kernel before treating results as final.
- Keep notebooks under version control when they are part of the project.

## Performance

- Prefer vectorized pandas and numpy operations.
- Use efficient dtypes, such as `category` for low-cardinality string columns.
- Avoid row-wise `.apply()` when a vectorized alternative exists.
- Consider Dask for larger-than-memory datasets.
- Profile before optimizing when performance bottlenecks are unclear.
- Avoid materializing unnecessary intermediate copies in large workflows.

## Scikit-Learn Workflows

- Keep preprocessing, feature engineering, and models in explicit, reproducible pipelines.
- Split training and validation data before fitting transformations that learn from data.
- Use `random_state` for reproducible stochastic models and splits.
- Use suitable metrics for the task and class balance.
- Avoid data leakage by fitting preprocessing only on training data.
- Document target definitions, feature assumptions, and evaluation methodology.

```python
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import OneHotEncoder, StandardScaler

preprocess = ColumnTransformer(
    transformers=[
        ("categorical", OneHotEncoder(handle_unknown="ignore"), ["region"]),
        ("numeric", StandardScaler(), ["tenure_months"]),
    ]
)

model = Pipeline(
    steps=[
        ("preprocess", preprocess),
        ("classifier", classifier),
    ]
)
```

## Dependencies

Use these libraries when they fit the task and project conventions:

- `pandas`
- `numpy`
- `matplotlib`
- `seaborn`
- `jupyter`
- `scikit-learn` for machine learning tasks
- `dask` for larger-than-memory data when needed

## Documentation and Reproducibility

- Document data sources, assumptions, and methodologies clearly.
- Keep analysis scripts and notebooks reproducible from a clean environment.
- Prefer deterministic examples and fixed random seeds where appropriate.
- Track notebooks and analysis scripts with version control.
- Refer to official pandas, matplotlib, seaborn, Jupyter, numpy, and scikit-learn documentation when API details are uncertain.
