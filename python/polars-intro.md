# An Introduction to Fast Data Analysis with the Polars Library in Python

Handling large datasets efficiently is a key challenge in data science, especially when working with traditional libraries like pandas that struggle with scalability. Enter Polars, a high-performance DataFrame library in Python designed for processing large datasets quickly and with low memory usage. In this blog, we’ll dive into Polars, showcase its main features, and demonstrate how to use it with a large dataset saved in Parquet format, a popular columnar data storage format optimized for high-speed analytics.

## Why Choose Polars for Big Data Analysis?

Polars is designed to address the limitations of traditional Python data libraries when handling large data volumes. Some reasons to consider Polars include:
- **Speed**: Polars is built in Rust, a highly performant language, making it significantly faster than pandas.
- **Low Memory Usage**: Polars uses a columnar format, which allows for efficient memory usage, especially beneficial when working with large datasets.
- **Lazy Evaluation**: Polars provides lazy evaluation, allowing users to defer computation until the final result is needed. This feature is crucial for optimizing complex query performance.
- **Multi-threaded**: Polars automatically takes advantage of multiple CPU cores, enabling parallel processing for faster computation.

These features make Polars an excellent choice when working with large datasets, especially those stored in Parquet format, which is also optimized for columnar storage.

## Installing Polars

You can install Polars via pip:
```bash
pip install polars
```

Make sure you also have pyarrow installed if you are working with Parquet files:
```bash
pip install pyarrow
```

## Loading Large Datasets from Parquet

Let’s start by creating a large dataset and saving it in Parquet format. The Parquet format’s columnar structure complements Polars’ design, allowing for fast reading and optimized storage.

```python
import numpy as np
import pandas as pd
import polars as pl

# Generate a large dataset
num_rows = 10_000_000
data = {
    "Column1": np.random.randint(1, 100, num_rows),
    "Column2": np.random.rand(num_rows),
    "Column3": np.random.choice(["A", "B", "C", "D", "E"], num_rows)
}

# Create a DataFrame
df = pl.DataFrame(data)

# Save the DataFrame as Parquet
df.write_parquet("large_dataset.parquet", compression="snappy")
```

Now let's open the Parquet file

```python
# Load a large Parquet file into a Polars DataFrame
file_path = "large_dataset.parquet"
df = pl.read_parquet(file_path)

# Display the first few rows and column names
print(df.head())
print("Number of rows:", df.shape[0])
print("Number of columns:", df.shape[1])
```

**Output**:
```plaintxt
| Column1 | Column2   | Column3 |
|---------|-----------|---------|
| 41      | 0.656287  | A       |
| 72      | 0.861548  | E       |
| 18      | 0.888267  | A       |
| 37      | 0.666067  | A       |
| 53      | 0.726323  | B       |

Number of rows: 10000000
Number of columns: 3
```

Polars can load data directly from Parquet files, which is particularly efficient for large datasets due to its optimized format. You’ll immediately notice that loading a Parquet file with Polars is fast and consumes less memory compared to other libraries like pandas.

## Data Manipulation with Polars

Polars provides an extensive set of methods for data manipulation. Let’s walk through some common operations with examples using our large dataset.

1. **Selecting Columns:**

Selecting specific columns from a large dataset can be done quickly, as Polars only loads the required columns into memory.

```python
# Select specific columns
selected_df = df.select(["Column1", "Column2"])
print(selected_df.head())
```

**Output**:
```plaintxt
| Column1 | Column2   |
|---------|-----------|
| 41      | 0.656287  |
| 72      | 0.861548  |
| 18      | 0.888267  |
| 37      | 0.666067  |
| 53      | 0.726323  |
```

2. **Filtering Rows:**

Polars allows you to filter rows based on conditions with its ```.filter()``` method, making it efficient for data wrangling.

```python
# Filter rows where Column1 is greater than a certain value
filtered_df = df.filter(pl.col("Column1") > 50)
print(filtered_df.head())
```

**Output**:
```plaintxt
| Column1 | Column2   | Column3 |
|---------|-----------|---------|
| 72      | 0.861548  | E       |
| 53      | 0.726323  | B       |
| 70      | 0.68528   | A       |
| 71      | 0.429749  | C       |
| 95      | 0.916151  | E       |
```

3. **Grouping and Aggregation:**

Polars excels at grouping and aggregation, a common bottleneck for larger datasets. Here’s an example of grouping by a categorical column (Column3) and calculating the mean of another column (Column2).

```python
# Group by "Column3" and calculate the mean of "Column2"
grouped_df = df.group_by("Column3").agg(
    pl.col("Column2").mean().alias("Average_Column2")
)
print(grouped_df)
```

**Output**:
```plaintxt
| Column3 | Average_Column2 |
|---------|------------------|
| E       | 0.499956        |
| A       | 0.499943        |
| D       | 0.500039        |
| C       | 0.499648        |
| B       | 0.499614        |
```


4. **Adding and Modifying Columns:**

Polars also supports adding new columns or modifying existing ones. Let’s say we want to add a new column that’s the result of an arithmetic operation.

```python
# Add a new column that is the square of "NumericColumn"
df = df.with_columns((pl.col("Column2") ** 2).alias("Column2_squared"))
print(df.head())
```

**Output**:
```plaintxt
| Column1 | Column2  | Column3 | Column2_squared |
|---------|----------|---------|-----------------|
| 41      | 0.656287 | A       | 0.430712       |
| 72      | 0.861548 | E       | 0.742265       |
| 18      | 0.888267 | A       | 0.789018       |
| 37      | 0.666067 | A       | 0.443645       |
| 53      | 0.726323 | B       | 0.527546       |
```

5. **Lazy Evaluation for Complex Pipelines:**

Lazy evaluation defers computation until the final result is needed, optimizing the query for efficiency. Here’s an example of building a complex query with lazy evaluation.

```python
# Enable lazy evaluation
lazy_df = df.lazy()

# Perform a series of transformations
result = (
    lazy_df
    .filter(pl.col("Column1") > 50)
    .with_columns((pl.col("Column2") * 100).alias("Column2_percentage"))
    .group_by("Column3")
    .agg(pl.col("Column2_percentage").mean().alias("Avg_Percentage"))
)

# Execute the lazy query
final_result = result.collect()
print(final_result)
```

**Output**:
```plaintxt
| Column3 | Avg_Percentage |
|---------|-----------------|
| A       | 49.979991      |
| D       | 50.016197      |
| B       | 49.934186      |
| E       | 49.993116      |
| C       | 49.957459      |
```

This method builds the pipeline without immediate execution, allowing Polars to optimize the query plan and execute it efficiently in one go when .collect() is called. For those who are familiar with Xarray this is similar to what that library does. 

## Performance Comparison: Polars vs Pandas

Let’s compare Polars and Pandas to understand the performance benefits. We’ll measure the time to read a large Parquet file and filter rows based on a condition.

```python
import pandas as pd
import polars as pl
import time

# Measure Polars read and filter performance
start_time = time.time()
polars_df = pl.read_parquet("large_dataset.parquet")
filtered_polars = polars_df.filter(pl.col("Column1") > 50)
print("Polars Time:", time.time() - start_time, "seconds")

# Measure Pandas read and filter performance
start_time = time.time()
pandas_df = pd.read_parquet("large_dataset.parquet")
filtered_pandas = pandas_df[pandas_df["Column1"] > 50]
print("Pandas Time:", time.time() - start_time, "seconds")
```

**Output**:
```plaintxt
Polars Time: 0.0274350643157959 seconds
Pandas Time: 0.20871400833129883 seconds
```
That's a `x10` speedup!

In most cases, you should see that Polars is significantly faster than Pandas, especially for larger datasets. Polars’ performance benefits become even more pronounced with more complex data transformations and aggregations.

## Conclusion

Polars is a powerful library that shines when working with large datasets. It’s optimized for performance, using multi-threading, lazy evaluation, and low memory usage to handle data-intensive tasks quickly and efficiently. When combined with Parquet files, Polars offers a highly performant solution for reading, analyzing, and transforming big data in Python.

For data scientists working with big data, Polars is a game-changer that can make your data pipelines faster and more efficient. Whether you’re performing simple filtering or complex transformations, Polars provides the tools you need for effective large-scale data manipulation.