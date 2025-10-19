# Pandas Pivot & Reshape Guide

Concise reference for pandas DataFrame pivoting, reshaping, and transposition methods.

---

## Pivot Operations

### Basic Pivot

```python
df.pivot(index='date', columns='category', values='sales')              # Basic pivot
df.pivot(index=['region', 'date'], columns='product')                   # Multi-index pivot
df.pivot(columns='status', values=['amount', 'quantity'])               # Multiple values
df.pivot_table(index='date', columns='category', values='sales')        # Pivot table (handles duplicates)
```

### Pivot Table with Aggregation

```python
df.pivot_table(index='date', columns='team', values='sales', aggfunc='mean')  # Custom aggregation
df.pivot_table(index='date', columns='team', values='sales', fill_value=0)    # Fill missing values
df.pivot_table(index='date', columns='team', margins=True)           # Add totals (margins)
df.pivot_table(index='date', columns='team', margins_name='Total')   # Custom margin name
df.pivot_table(index='date', columns='team', aggfunc={'sales': 'sum', 'qty': 'mean'})  # Multiple agg functions
```

### Cross Tabulation

```python
pd.crosstab(df['category'], df['region'])                               # Basic crosstab
pd.crosstab(df['category'], df['region'], normalize='index')            # normalize = 'columns' ,'all' 
pd.crosstab([df['category'], df['type']], df['region'])                 # Multi-level crosstab
pd.crosstab(df['category'], df['region'], values=df['sales'], aggfunc='sum')  # With values
```

## Reshaping Operations

### Melt (Wide to Long)

```python
df.melt()                         # Melt all columns
df.melt(id_vars=['id', 'name'])   # Keep specific columns as identifiers
df.melt(value_vars=['sales_q1', 'sales_q2'])                           # Melt specific columns only
df.melt(id_vars=['id'], value_vars=['q1', 'q2'], var_name='quarter')   # Custom variable name
df.melt(id_vars=['id'], value_name='revenue')                          # Custom value name
pd.wide_to_long(df, stubnames='sales', i='id', j='quarter')            # Wide to long alternative
```

### Stack/Unstack

```python
df.stack()                     # Stack columns to rows
df.stack(level=0)              # Stack specific level
df.stack(dropna=False)         # Keep NaN values
df.unstack()                   # Unstack index to columns
df.unstack(level='category')   # Unstack specific level
df.unstack(fill_value=0)       # Fill with custom value
```

## Advanced Reshaping

### Multi-level Operations

```python
# Multi-level pivot
df.pivot_table(index=['region', 'date'], columns=['category', 'type'], values='sales')

# Unstack multiple levels
df.unstack(['category', 'type'])                                        # Unstack multiple levels
df.stack([0, 1])                                                        # Stack multiple levels

# Swaplevel and reorder
df.swaplevel(0, 1, axis=1)                                             # Swap column levels
df.reorder_levels([1, 0], axis=1)                                      # Reorder levels
```

### Complex Reshaping

```python
# Multiple value columns in pivot
df.pivot_table(index='date', columns='category', 
               values=['sales', 'quantity'], aggfunc={'sales': 'sum', 'quantity': 'mean'})

# Pivot with custom functions
df.pivot_table(index='date', columns='category', values='sales', 
               aggfunc=lambda x: x.max() - x.min())                    # Custom aggregation

# Conditional pivoting
df[df['status'] == 'active'].pivot_table(index='date', columns='type') # Filter before pivot
```

## Data Transformation Patterns

### Normalize/Denormalize

```python
# Normalize (expand related data)
df.join(df['metadata'].str.split(',', expand=True))                    # Split and expand
pd.json_normalize(df['json_col'])                                       # JSON to flat structure

# Denormalize (collapse related data)
df.groupby('id').agg({'values': list})                                 # Aggregate to lists
df.groupby('id')['category'].apply(lambda x: ','.join(x))              # Join values
```

### Frequency Tables

```python
df['category'].value_counts()                                           # Basic frequency
pd.crosstab(df['category'], columns='count')                           # Crosstab frequency
df.groupby(['category', 'region']).size().unstack(fill_value=0)        # 2D frequency table
df.pivot_table(index='category', aggfunc='size')                       # Pivot frequency
```

### Percentage Tables

```python
# Row percentages
df.div(df.sum(axis=1), axis=0) * 100                                   # Manual row percentages
pd.crosstab(df['cat'], df['region'], normalize='index') * 100          # Crosstab row %

# Column percentages  
df.div(df.sum(axis=0), axis=1) * 100                                   # Manual column percentages
pd.crosstab(df['cat'], df['region'], normalize='columns') * 100        # Crosstab column %
```

## Common Use Cases

```python
# Time series reshaping
df.set_index('date').resample('M').mean().T                            # Monthly aggregation + transpose
df.pivot(index='date', columns='metric').resample('D').interpolate()   # Daily interpolation

# Feature engineering
df.pivot_table(index='id', columns='feature_name', values='feature_value')  # Features to columns
df.melt(id_vars=['id'], var_name='feature', value_name='value')        # Long format for ML

# Reporting format
df.pivot_table(index='product', columns='quarter', values='sales', margins=True)  # Sales report
```