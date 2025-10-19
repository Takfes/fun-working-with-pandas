# 🐼 Complete pandas GroupBy Aggregation Guide

## 📋 Table of Contents

| Section | Patterns | Performance | Best For |
|---------|----------|-------------|----------|
| [**Aggregation Patterns**](#-aggregation-patterns) | Simple agg, Named agg, Dictionary-based | 🚀⚡ | Basic groupby operations, clean column names |
| [**Custom Logic**](#-custom--advanced-logic) | Lambda functions, Custom functions | 🐌⚡ | Complex calculations |
| [**Apply Pattern**](#-the-apply-aggregation-pattern-) | apply() + lambda, apply() + pd.Series | 🐌 | Full group access, correlations |
| [**Special Aggregations**](#-specialized-aggregations) | String ops, Quantiles, Ranking, Rolling, Conditionals | 🚀⚡ | Text data, percentiles, time-series |
| [**Transform & Filtering**](#-transform--filter-operations) | transform(), filter(), Ad-hoc grouping | 🚀⚡ | Window functions, group filtering |
| [**Post-Aggregation**](#-post-aggregation-operations) | pipe(), assign() | ⚡ | Result transformation |
| [**Putting It All Together**](#-examples) | Complete examples | - | Real-world usage |

**Performance Legend:** 🚀 Fast | ⚡ Medium | 🐌 Slower (but flexible)

---

## 📌 Aggregation Patterns

### Single Function GroupBy 🚀
```python
df.groupby('region')['sales'].mean()
df.groupby('category').agg({'revenue': 'sum'})
df.groupby('region')['customer_id'].nunique()    # Count distinct
df.groupby('region')['sales'].agg(['min', 'max', 'mean', 'std'])
```
👉 **When:** Simple set of aggregation functions per group

### Dictionary-Based Aggregation ⚡
```python
df.groupby('region').agg({
    'sales': ['sum', 'mean'], 
    'profit': 'max',
    'customer_id': 'nunique'
})
```
👉 **When:** Different functions per column

### Named Aggregations (Preferred Modern Style) 🚀
```python
df.groupby('region').agg(
    total_sales=('sales', 'sum'),
    avg_profit=('profit', 'mean'),
    unique_customers=('customer_id', 'nunique')
)
```
👉 **When:** Clean output column names needed

### Reusing GroupBy Objects ⚡
```python
# Store groupby object for multiple operations (avoid recomputation)
g = df.groupby('region')

# Add multiple transformations to original DataFrame
df['sales_mean'] = g['sales'].transform('mean')
df['sales_rank'] = g['sales'].rank(ascending=False)
df['profit_cumsum'] = g['profit'].cumsum()

# Multiple aggregations
summary_stats = g['sales'].agg(['count', 'mean', 'std'])
profit_analysis = g['profit'].agg(['sum', 'min', 'max'])
```
👉 **When:** Multiple operations on the same grouping (performance optimization)

---

## 🔧 Custom & Advanced Logic

### Lambda Functions ⚡
```python
df.groupby('region')['sales'].agg(lambda x: x.max() - x.min())
```
👉 **When:** Simple custom calculations

### Custom Functions with Complex Logic 🐌
```python
def custom_mean_diff(group_data):
    """Calculate mean and subtract from first 5 values"""
    mean_val = group_data.mean()
    first_5 = group_data.head(5)
    return (first_5 - mean_val).sum()

df.groupby('region')['sales'].agg(custom_mean_diff)
```
👉 **When:** Complex per-group calculations needed

---

## 🪡 The Apply (Aggregation) Pattern 🐌

### Single-metric (per group):
```python
# Use apply + lambda for a single custom metric per group
df.groupby('region')['sales'].apply(lambda x: x.max() - x.min())  # Range per group
df.groupby('region').apply(lambda g: g['sales'].corr(g['profit']))  # Correlation per group
```

### Multi-metric (per group):
```python
# Use apply + lambda + pd.Series for multiple metrics per group
df.groupby('region').apply(
    lambda g: pd.Series({
        'sales_range': g['sales'].max() - g['sales'].min(),
        'profit_margin': g['profit'].sum() / g['sales'].sum()
    })
)
```

👉 **When:**
- Use `apply` when you need full access to each group (e.g., filtering, custom logic, or metrics not possible with `agg`).
- For a single custom value per group, use `apply` with a lambda.
- For multiple metrics per group, return a `pd.Series` from your lambda.
- Prefer `agg` for simple aggregations (faster), and use `apply` only when necessary.

---

## 📈 Specialized Aggregations

### String Aggregations 🚀
```python
# Join strings
df.groupby('region')['product_name'].agg(', '.join)
# Create lists
df.groupby('region')['product_name'].agg(list)
# String stats
df.groupby('region')['description'].agg(['count', lambda x: x.str.len().mean()])
```

### Quantiles & Percentiles ⚡
```python
df.groupby('region')['sales'].quantile([0.25, 0.5, 0.75])
df.groupby('region')['sales'].agg(lambda x: x.quantile(0.95))
```

### Ranking Within Groups 🚀
```python
df['sales_rank'] = df.groupby('region')['sales'].rank(method='dense', ascending=False)
df['profit_rank'] = df.groupby('region')['profit'].rank(method='min')
```

### Rolling Aggregations with GroupBy ⚡
```python
df.groupby('stock_symbol')['price'].rolling(window=7).mean()
df.groupby('customer_id')['purchase_amount'].expanding().sum()
```

### Conditional Aggregations (Advanced) ⚡
```python
import numpy as np
df.groupby('region').agg({
    # Sum only above-average sales
    'sales': lambda x: np.where(x > x.mean(), x, 0).sum(),
    # Count profitable transactions  
    'profit': lambda x: (x > 0).sum(),
    # Weighted average (using quantity as weight)
    'price': lambda x: np.average(x, weights=df.loc[x.index, 'quantity']),
    # Sum of top 20% performers
    'performance': lambda x: x[x >= x.quantile(0.8)].sum(),
    # Conditional mean (exclude outliers)
    'clean_mean': lambda x: x[(x >= x.quantile(0.05)) & (x <= x.quantile(0.95))].mean()
})
```

---

## 🔄 Transform & Filter Operations

### Transform (Same Shape as Input) 🚀
```python
# Add group statistics back to original DataFrame
df['sales_group_mean'] = df.groupby('region')['sales'].transform('mean')
df['profit_pct_of_group'] = df['profit'] / df.groupby('region')['profit'].transform('sum')
```
👉 **When:** Add group stats to each row (SQL window functions)

### Filter Groups (SQL WHERE/HAVING) ⚡
```python
# Filter ROWS within groups (like SQL WHERE - before aggregation)
high_value_sales = df[df['sales'] > 1000].groupby('region')['sales'].sum()

# Filter ENTIRE GROUPS (like SQL HAVING - after aggregation)  
df.groupby('region').filter(lambda x: x['sales'].mean() > 1000)       # High average sales
df.groupby('region').filter(lambda x: len(x) > 5)                     # Group size > 5
df.groupby('region').filter(lambda x: x['sales'].quantile(0.9) > 5000) # Top 10% > threshold
```
👉 **When:** for `WHERE` kind of operations, filter individual rows before grouping; for `HAVING`, filter entire groups after aggregation

### Ad-Hoc Column Creation During Aggregation 🚀
```python
# Group by derived/calculated columns (like SQL date functions)
df.groupby(df['timestamp'].dt.date)['sales'].sum()                    # Group by date part
df.groupby(df['timestamp'].dt.hour)['traffic'].mean()                 # Group by hour
df.groupby(df['sales'] // 1000)['customer_id'].nunique()              # Group by sales brackets
df.groupby(pd.cut(df['age'], bins=[0, 25, 45, 65, 100]))['income'].mean()  # Group by age ranges

# Complex derived grouping
df.groupby([
    df['region'],
    df['sales'].apply(lambda x: 'high' if x > 1000 else 'low'),       # Custom categorization
    df['date'].dt.quarter                                              # Quarter extraction
])['profit'].sum()
```
👉 **When:** Need to group by calculated/derived columns not in original data

---

## 🩹 Post-Aggregation Operations

After groupby aggregation, you often need to transform the results further. Here are the key patterns: `pipe()` vs `assign()` vs `pipe() + assign()`

### example with pipe() + assign()
```python
def advanced_metrics(df):
    """Complex transformations that can't be done with assign()"""
    df = df.copy()
    # Complex logic requiring multiple steps
    df['sales_zscore'] = (df['total_sales'] - df['total_sales'].mean()) / df['total_sales'].std()
    df['profit_zscore'] = (df['avg_profit'] - df['avg_profit'].mean()) / df['avg_profit'].std()
    return df

result = (df
    .groupby('region')
    .agg(total_sales=('sales', 'sum'), avg_profit=('profit', 'mean'))
    .pipe(advanced_metrics)                    # Complex transformations
    .assign(                                   # Simple additions
        combined_score=lambda x: x['sales_zscore'] + x['profit_zscore'],
        performance_tier=lambda x: pd.qcut(x['combined_score'], 3, labels=['Bronze', 'Silver', 'Gold'])
    )
)
```

👉 **When:** `assign()` for simple calculations, `pipe()` for complex logic, multiple steps, custom functions

---

## 🎯 Examples

### Comprehensive Aggregation in One Step
```python
# Custom function for complex metrics
def sales_momentum(series):
    """Calculate growth momentum over last 3 periods"""
    if len(series) < 3:
        return 0
    recent = series.tail(3)
    return (recent.iloc[-1] - recent.iloc[0]) / recent.iloc[0]

# Combine all aggregation patterns in a single .agg() call
result = df.groupby('region').agg(
    # Basic aggregation
    total_sales=('sales', 'sum'),
    # Count unique
    unique_customers=('customer_id', 'nunique'),
    # Quantile with lambda
    sales_p95=('sales', lambda x: x.quantile(0.95)),
    # String operation (join top 3 products)
    top_products=('product_name', lambda x: ', '.join(x.value_counts().head(3).index)),
    # Custom function
    sales_momentum=('sales', sales_momentum),
    # Conditional aggregation (percent of high-value sales)
    high_value_sales_pct=('sales', lambda x: (x > x.quantile(0.8)).mean() * 100)
)
```

### Advanced apply() Examples
```python
# Conditional aggregations based on group characteristics
def smart_metric(group):
    if len(group) > 100:  # Large groups
        return group['sales'].median()  # Use median for stability
    else:  # Small groups  
        return group['sales'].mean()    # Use mean for sensitivity

df.groupby('region').apply(smart_metric)

# Time-based conditional aggregations
df.groupby('product').apply(
    lambda x: pd.Series({
        'recent_sales': x[x['date'] >= '2024-01-01']['sales'].sum(),
        'historical_sales': x[x['date'] < '2024-01-01']['sales'].sum(),
        'growth_rate': (
            x[x['date'] >= '2024-01-01']['sales'].mean() / 
            x[x['date'] < '2024-01-01']['sales'].mean() - 1
        ) if len(x[x['date'] < '2024-01-01']) > 0 else 0
    })
)
```

---