# Pandas Row Filtering Guide

---

## Row Filtering Methods

### .loc - Label-based Selection

```python
df.loc[df['age'] > 30]                            # Boolean condition
df.loc[df['department'] == 'IT']                  # Equality filter
df.loc[(df['age'] > 25) & (df['salary'] > 55000)] # Multiple conditions
df.loc[df['age'] > 30, ['name', 'salary']]        # Rows + specific columns
df.loc[lambda x: x['age'].between(25, 30)]        # Callable filtering
```

### .iloc - Integer Position Selection

```python
df.iloc[0:3]         # First 3 rows
df.iloc[[0, 2, 4]]   # Specific positions
df.iloc[0:3, 1:4]    # Rows 0-2, columns 1-3
df.iloc[:, [0, 2]]   # All rows, columns 0 and 2
df.iloc[-3:]         # Last 3 rows
```

### .at - Single Value Access

```python
value = df.at[0, 'name']        # Get single value by label
df.at[0, 'age'] = 26            # Set single value
df.at[df.index[0], 'salary']    # Using index reference
```

### .iat - Single Value by Position

```python
value = df.iat[0, 1]     # Get value at position (0, 1)
df.iat[0, 1] = 27        # Set value at position
df.iat[-1, -1]           # Last row, last column
```

### Boolean Masks

```python
mask = df['age'] > 30                                    # Simple condition
df[(df['age'] > 25) & (df['salary'] < 65000)]            # Multiple conditions (&, |, ~)
df[df['department'].isin(['IT', 'HR'])]                  # Multiple values
df[~df['department'].isin(['Finance'])]                  # Negation
df[df['age'].between(25, 35)]                            # Range filtering
df[(df['age'].between(25, 35)) & (df['salary'] > 50000) | (df['dept'] == 'IT')] # Complex
```

### Query Method

```python
df.query('age > 30')                                     # Simple condition
df.query('age > 25 and salary > 55000')                  # Multiple conditions
df.query('department == "IT" or department == "HR"')     # OR conditions
df.query('age >= @min_age')                              # External variables
df.query('department in @dept_list')                     # List membership
df.query('city.str.contains("o")', engine='python')      # String operations
df.query('age > salary / 2000')                          # Complex expressions
df.query('25 <= age <= 32')                              # Range conditions
```

### String Filtering

```python
df[df['name'].str.contains('a', case=False)]             # Case-insensitive
df[df['name'].str.startswith('A')]                       # Starts with
df[df['city'].str.endswith('n')]                         # Ends with
df[df['department'].str.match('IT')]                     # Match from start
df[df['city'].str.fullmatch('New York')]                 # Exact full match
df[df['name'].str.len() > 4]                             # Length filtering
df[df['name'].str.contains(r'^[A-C]')]                   # Regex patterns
df[df['city'].str.contains(r'on$')]                      # Regex end match
df[df['name'].str.lower().str.contains('alice')]         # Case operations
```

### Finding Nulls and Empty Values

```python
df[df['A'].isnull()]                          # Find null values (isna() equivalent)
df[df['A'].notnull()]                         # Find non-null values (notna() equivalent)
df[df.isnull().any(axis=1)]                   # Rows with any null
df[df.notnull().all(axis=1)]                  # Rows with all non-null
df[df['B'] == '']                             # Empty strings
df[(df['B'].notnull()) & (df['B'] != '')]     # Non-null and non-empty
df.query('A.isnull()', engine='python')       # Find nulls with query method
df.query('A.notnull()', engine='python')      # Find non-nulls with query method
df[pd.notna(df['timestamp_col'])]             # Filter for valid timestamps (not NaT)
df[pd.isna(df['timestamp_col'])]              # Filter for NaT (Not a Timestamp)
df.isnull().sum()                             # Count nulls per column
df.dropna()                                   # Drop rows with any null
df.dropna(subset=['A', 'B'])                  # Drop if specific columns null
```

---

## Performance Tips

```python
# Vectorized operations (avoid loops)
df[df['age'] > 30]                                      # Good: Vectorized
df.query('age > 25 and salary > 55000')                 # Alternative for complex queries
df.loc[df['age'] > 30, ['name', 'salary']]              # Explicit row/column selection

# Categorical data for repeated strings
df['department'] = df['department'].astype('category')   # Faster filtering

# Defensive filtering
df[df['city'].str.contains('York', na=False)]            # Handle NaN values
```