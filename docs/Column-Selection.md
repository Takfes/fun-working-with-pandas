# Pandas Column Selection Guide

---

## Column Selection Methods

### Basic Selection

```python
df[['col1', 'col2']]                                     # Multiple columns by name
df.col1                                                  # Single column (attribute access)
df['col1']                                               # Single column (bracket access)
df.loc[:, 'col1':'col3']                                 # Column slice by label
df.iloc[:, 0:3]                                          # Column slice by position
```

### Boolean Masks

```python
# By column names
mask = df.columns.str.contains('col')                   # Pattern matching
df.loc[:, mask]                                         # Apply mask
df.loc[:, df.columns.str.endswith('_total')]            # Suffix matching
df.loc[:, df.columns.str.startswith('sales_')]          # Prefix matching

# By data properties
numeric_mask = df.dtypes.apply(lambda x: x.kind in 'biufc')  # Numeric columns
df.loc[:, numeric_mask]                                      # Select numeric columns
df.loc[:, df.dtypes != 'bool']                               # Exclude boolean columns
```

### Regex Filtering

```python
df.filter(regex=r'.*col$')                             # Columns ending with 'col'
df.filter(regex=r'^(sales|revenue)')                   # Starting with sales or revenue
df.filter(regex=r'_q[1-4]$')                           # Quarterly columns
df.filter(regex=r'^\d{4}_')                            # Starting with year
df.columns[df.columns.str.contains(r'(int|str)_col')]  # Complex pattern matching
```

### Selection by Data Type

```python
df.select_dtypes(include=['int64'])                     # Integer columns only
df.select_dtypes(include=['float64'])                   # Float columns only
df.select_dtypes(include=['object'])                    # Object columns only
df.select_dtypes(include=[np.number])                   # All numeric types
df.select_dtypes(include=['datetime64'])                # Datetime columns
df.select_dtypes(exclude=['object'])                    # Exclude object columns
df.select_dtypes(include=['int64', 'float64'])          # Multiple specific types
df.select_dtypes(include=[np.number, 'bool'])           # Numeric and boolean
```

### Advanced Selection Patterns

```python
# Conditional selection
cols_with_nulls = df.columns[df.isnull().any()]         # Columns with null values
high_variance_cols = df.select_dtypes(include=[np.number]).columns[df.var() > 1.0]  # High variance
unique_cols = df.columns[df.nunique() == len(df)]       # All unique values

# Dynamic patterns
time_cols = [col for col in df.columns if 'time' in col.lower()]  # Time-related columns
id_cols = [col for col in df.columns if col.endswith('_id')]      # ID columns
metric_cols = df.filter(regex=r'(count|rate|pct|avg)').columns    # Metric columns
```

### Column Manipulation

```python
# Reorder columns
df[['important_col'] + [col for col in df.columns if col != 'important_col']]  # Move to front
df.reindex(columns=sorted(df.columns))  # Alphabetical order
df[df.columns[::-1]]                    # Reverse order

# Column subsets
feature_cols = [col for col in df.columns if col.startswith('feature_')]  # Feature columns
target_col = [col for col in df.columns if col.startswith('target_')]     # Target column
df[feature_cols + target_col]                           # Features + target

# Exclude columns
df.drop(columns=['unwanted_col'])                 # Drop specific columns
df.loc[:, ~df.columns.isin(['col1', 'col2'])]     # Exclude multiple columns
```

### Performance Optimizations

```python
# Column name operations
cols = df.columns.tolist()                              # Convert to list once
selected = [col for col in cols if condition(col)]      # List comprehension

# Type-based operations
numeric_cols = df.select_dtypes(include=[np.number]).columns.tolist()  # Cache numeric columns
df[numeric_cols].mean()                                 # Operations on subset

# Pattern caching
import re
pattern = re.compile(r'^sales_.*_q[1-4]$')             # Compile regex once
matching_cols = [col for col in df.columns if pattern.match(col)]  # Reuse pattern
```

### Common Use Cases

```python
# Data preprocessing
categorical_cols = df.select_dtypes(include=['object', 'category']).columns
numeric_cols = df.select_dtypes(include=[np.number]).columns
date_cols = df.select_dtypes(include=['datetime64']).columns

# Feature engineering
text_cols = df.select_dtypes(include=['object']).columns
df[text_cols] = df[text_cols].fillna('')               # Handle text nulls

# Analysis preparation
analysis_cols = df.columns[df.dtypes != 'object']      # Exclude text for correlation
df_analysis = df[analysis_cols]                        # Analysis-ready dataset
```