# CSV vs TXT vs Parquet: Choosing the Right File Format in Python for Tabular Data

The choice of file format can significantly impact performance, storage, and usability. Python offers seamless tools for handling different formats, but the challenge lies in selecting the most suitable one for your needs. This blog post explores three popular file formats - CSV, TXT, and Parquet — comparing their characteristics, advantages, and limitations to help you make an informed decision.

## Overview of CSV, TXT, and Parquet Formats

- CSV (Comma-Separated Values): One of the most commonly used formats, especially in data science, because it is human-readable and works well with both small and large datasets. CSV files contain plain-text data organized in rows, with each row representing an entry and each entry separated by commas or another delimiter.

- TXT (Text): The TXT format is highly flexible and can store any text data, often in a row-by-row format. While it doesn’t have a standard structure for tabular data, it’s versatile for storing unstructured data or custom structures, such as logs or custom-separated values.

- Parquet: A columnar storage file format optimized for big data processing. Developed by Apache, it’s efficient in terms of storage and performance, making it ideal for analytics workloads that require reading and writing large datasets quickly. Parquet files support schema definitions and advanced compression techniques, adding another layer of efficiency.

## Performance and Storage Efficiency

Performance and storage requirements vary depending on the file format. Let’s dive into how each format stacks up.

#### CSV:

- **Size**: CSV files generally occupy more storage space compared to Parquet due to lack of compression.
- **Read/Write Speed**: Moderate performance; not as fast as Parquet for large datasets, but efficient for smaller ones.
- **Memory Usage**: CSV files require loading the entire file into memory, which can be limiting for very large files.
- **Readability**: Easy to read and edit manually; widely supported by various tools and platforms.
- **Structure**: Requires consistency in column and row structure to avoid data misalignment.
- **Compatibility**: Supported by almost all data processing libraries in Python, such as pandas, and commonly used in data science.

#### TXT:

- **Size**: TXT files can vary in size depending on the structure and content, but like CSV files, they typically lack compression.
- **Read/Write Speed**: TXT files can be read relatively quickly for smaller datasets; however, performance decreases significantly for larger files.
- **Memory Usage**: Similar to CSV, TXT files require considerable memory for large datasets, with no inherent optimization for tabular data.
- **Readability**: Also human-readable, though structure depends on how data is organized, so it may not be as easy to interpret.
- **Structure**: Flexible, but lacks the rigid structure of tabular formats. Custom delimiters or data organization are often required.
- **Compatibility**: Universally compatible, as it’s essentially a simple text file; Python offers native and library-based tools to read and manipulate TXT files.

#### Parquet:

- **Size**: Parquet files are highly compressed, often achieving reductions of 70-90% compared to CSV.
- **Read/Write Speed**: Parquet’s columnar structure allows fast reading of specific columns without loading the entire dataset, making it highly efficient.
- **Memory Usage**: Optimized for large-scale data and can handle significant datasets with lower memory overhead by reading data in chunks.
- **Readability**: Not directly human-readable due to binary format.
- **Structure**: Enforces a consistent schema with efficient compression. Well-suited for structured data.
- **Compatibility**: Supported by data-centric libraries like pandas and polars. Less universal than CSV but commonly used in data science and analytics.

## Python Libraries for File Handling

Python provides several libraries to work with these file formats efficiently. Here are some common options:

- Pandas: Provides convenient functions to read and write CSV and Parquet files (pd.read_csv() and pd.to_parquet()).

- CSV Module: Python’s native library for reading and writing CSV files. Great for small to medium-sized files but lacks the processing power for very large datasets.

- Polars: An optimized library for reading and writing Parquet files. It’s ideal for working with Parquet as it’s optimized for high-performance data processing.

- Text File Handling: Python’s built-in open() function can read and write TXT files. The flexibility of handling data this way is offset by the need to manually parse and structure data.

## When to Use Each Format

Here’s a quick guide on when to use each format based on different scenarios:
- CSV:
    - When data is smaller and needs to be human-readable.
	- For interoperability with tools like Excel or other non-Python applications.
	- When performing straightforward data analysis in Python or transferring data between applications.
- TXT:
	- When data is unstructured or requires a custom format.
	- For storing simple log files, raw text, or other data not necessarily in tabular form.
	- When flexibility is required in terms of delimiters and structure.
- Parquet:
	- When working with large datasets where performance and storage are critical.
	- For data engineering tasks, big data analytics, and columnar-based processing.
	- When using a data warehouse or data lake for storing large-scale analytics data.

Choosing the right file format—CSV, TXT, or Parquet—can dramatically affect your data processing workflow in Python. For small to medium-sized datasets that require human readability, CSV is a practical choice. TXT files, though flexible, are best suited for unstructured or custom-formatted data. Parquet, with its advanced compression and optimized storage, is ideal for large-scale datasets and big data environments.

## Code example: Using each Format

###  Working with CSV Files

CSV files are easy to read and write with libraries like pandas and polars. Here’s how to handle CSV files efficiently in Python.

**Using Pandas:**
```python
import pandas as pd
import numpy as np

# Generate sample data with numpy
data = {
    "Column1": np.random.randint(1, 100, 10),
    "Column2": np.random.rand(10),
    "Column3": ["A", "B", "C", "D", "E", "F", "G", "H", "I", "J"]
}

# Convert data to a DataFrame
df = pd.DataFrame(data)

# Write to CSV
df.to_csv("sample.csv", index=False)
print("CSV file written successfully.")

# Read from CSV
df_from_csv = pd.read_csv("sample.csv")
print(df_from_csv)
```

###  Working with TXT Files

TXT files are versatile, especially when storing unstructured or loosely structured data. You can read and write TXT files with standard Python functions or libraries like numpy.

**Writing and Reading Structured TXT Files with Numpy:**
```python
import numpy as np

# Generate structured data
data_array = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])

# Save data as TXT
np.savetxt("sample.txt", data_array, delimiter=",", fmt="%d")
print("Structured TXT file written successfully.")

# Load data from TXT
loaded_data = np.loadtxt("sample.txt", delimiter=",")
print(loaded_data)
```

###  Working with Parquet Files

Parquet files are ideal for handling large datasets and are commonly used in big data environments. Libraries like pandas and polars make it easy to work with Parquet files in Python.

**Using Polars with Parquet:**
```python
import polars as pl

# Convert pandas DataFrame to Polars DataFrame
pl_df = pl.DataFrame(df)

# Write to Parquet
pl_df.write_parquet("sample_polars.parquet")

# Read from Parquet
pl_df_from_parquet = pl.read_parquet("sample_polars.parquet")
print(pl_df_from_parquet)
```

### Summary of Examples

| Format    | Library | File Type | Code Snippet |
| --------- | ------- | --------- | ------------ |
| CSV       |  Pandas |  .csv     | pd.to_csv() and pd.read_csv() |
| CSV       |  Polars |  .csv     | pl_df.write_csv() and pl.read_csv() |
| TXT       |  Python |  .txt     | open() with .write() and .read() |
| TXT       |  Numpy  |  .txt     | np.savetxt() and np.loadtxt() |
| Parquet   |  Pandas | .parquet  | pd.to_parquet() and pd.read_parquet() |
| Parquet   |  Polars | .parquet  | pl_df.write_parquet() and pl.read_parquet() |

## Code Example: Comparing Parquet and CSV Read Times for a large dataset

To illustrate the speed advantages of Parquet over CSV when working with large datasets, let’s create a test comparing read times for both formats. This example will show the time it takes to load a large dataset in both CSV and Parquet formats, highlighting how Parquet’s columnar storage and compression boost performance.

```python
import pandas as pd
import numpy as np
import time
import pyarrow as pa
import pyarrow.parquet as pq

# Generate a large dataset
num_rows = 10_000_000
data = {
    "Column1": np.random.randint(1, 100, num_rows),
    "Column2": np.random.rand(num_rows),
    "Column3": np.random.choice(["A", "B", "C", "D", "E"], num_rows)
}

# Create a DataFrame
df = pd.DataFrame(data)

# Save the DataFrame as CSV and Parquet
df.to_csv("large_dataset.csv", index=False)
df.to_parquet("large_dataset.parquet", engine="pyarrow", compression="snappy")

# Function to measure load times
def measure_load_time(file_path, format_type="csv"):
    start_time = time.time()
    if format_type == "csv":
        _ = pd.read_csv(file_path)
    elif format_type == "parquet":
        _ = pd.read_parquet(file_path, engine="pyarrow")
    end_time = time.time()
    return end_time - start_time

# Measure load time for CSV
csv_load_time = measure_load_time("large_dataset.csv", format_type="csv")
print(f"CSV Load Time: {csv_load_time:.2f} seconds")

# Measure load time for Parquet
parquet_load_time = measure_load_time("large_dataset.parquet", format_type="parquet")
print(f"Parquet Load Time: {parquet_load_time:.2f} seconds")
```

**Explanation of the Code**

1.	**Data Generation**: We create a DataFrame with 10 million rows to simulate a large dataset, ensuring that the load times for CSV and Parquet will be noticeable.
2.	**File Saving**: The dataset is saved as both a CSV (large_dataset.csv) and a Parquet file (large_dataset.parquet). For Parquet, we specify compression (snappy), which is a common choice to further reduce file size and improve read performance.
3.	**Load Time Measurement**: The measure_load_time function reads the specified file and measures the time it takes to complete the operation. It supports both csv and parquet formats.
4.	**Performance Comparison**: We print the time it takes to load each file format, and typically, you should observe that Parquet loads much faster than CSV due to its efficient columnar storage and compression.

**Output:**

```plaintext
CSV Load Time: 12.34 seconds
Parquet Load Time: 2.56 seconds
```

Note: The exact results will depend on your machine and dataset. In most cases, Parquet will be significantly faster, especially with large datasets and selective column loading.