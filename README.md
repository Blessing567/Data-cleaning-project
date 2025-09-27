# 🧹 Data Cleaning & Quality Utilities

This project provides a collection of reusable Python functions to detect, validate, and correct common data quality issues in datasets.

It is designed for data analysts, data scientists, and students who want to ensure their datasets are clean, consistent, and ready for analysis or machine learning tasks.


🚀 Project Motivation

As a data analyst, I’ve realized that 80–90% of the work is data preparation, not the “cool” modeling or visualization. Tasks like handling missing values, fixing outliers, validating categories, and cleaning text are repetitive and time-consuming.

This project was created to solve that challenge by developing a set of reusable, modular data cleaning functions. Instead of rewriting the same cleaning logic for every dataset, I can now simply import the right function and apply it.

Why this matters:

⏳ Time Efficiency → Saves hours of repetitive work by automating common cleaning tasks.

🛠️ Reusability → Functions are modular and can be applied across projects.

✅ Consistency → Ensures standardized, high-quality cleaning every time.

📈 Industry Relevance → Matches real-world workflows where analysts constantly prepare raw data for decision-making.

By reducing the time spent on repetitive cleaning, I can focus more on analysis and insights, which is the ultimate goal of a data analyst.

# 📂 Data Cleaning Modules

The toolkit is organized into separate modules for different tasks:

numeric_cleaning.py → Functions for handling numeric issues (outliers, missing values, type conversion).

categorical_cleaning.py → Functions for handling categorical data (honourifics, membership constraints, normalization).

regex_utils.py → Regex-based functions for cleaning formatting and unwanted strings.

data_quality_utils.py → General utilities (missing detection, duplicate handling, cross-field validation).

tests/ → Contains test scripts to validate each function.

# 📌 Features
## 🧹 Data Cleaning Process

The data cleaning process in this project focuses on ensuring data quality, consistency, and reliability before analysis. It involves:

* Outliers: Data points that deviate significantly from the rest of the dataset, which can distort analysis; they are detected (e.g., via IQR or Z-score) and handled by removal, capping, or replacement for cleaner, reliable data.
~~~ python
import pandas as pd

def handle_missing_values(df, strategy="mean", columns=None, fill_value=None):
    """
    Handles missing values in a DataFrame.
    
    Parameters:
        df (pd.DataFrame): Input DataFrame
        strategy (str): "mean", "median", "mode", "constant"
        columns (list): Columns to apply strategy on
        fill_value: Value for 'constant' strategy
    
    Returns:
        pd.DataFrame: Cleaned DataFrame
    """
    df_copy = df.copy()
    cols = columns if columns else df_copy.columns
    
    for col in cols:
        if strategy == "mean":
            df_copy[col].fillna(df_copy[col].mean(), inplace=True)
        elif strategy == "median":
            df_copy[col].fillna(df_copy[col].median(), inplace=True)
        elif strategy == "mode":
            df_copy[col].fillna(df_copy[col].mode()[0], inplace=True)
        elif strategy == "constant":
            df_copy[col].fillna(fill_value, inplace=True)
    return df_copy


Outliers – Identifying unusual values using methods like Interquartile Range (IQR) and correcting them with robust approaches such as trimmed mean or capping.

    import pandas as pd
  """

    Detect and handle outliers in a numerical column using the IQR method.
    
    Parameters
    ----------
    df : pandas.DataFrame
        The input dataframe.
    column : str
        The column name to process.
    method : str, optional (default="trim")
        The method to handle outliers:
        - "trim" : Remove rows with outliers
        - "cap" : Cap outliers to the boundary values
        - "nan" : Replace outliers with NaN (so they can be imputed later)
    factor : float, optional (default=1.5)
        The multiplier for the IQR to determine outlier boundaries.
    
    Returns
    -------
    pandas.DataFrame
        DataFrame with outliers handled in the specified column.
    """
    def handle_outliers_iqr(df, column, method="trim", factor=1.5):
    
    if column not in df.columns:
        raise ValueError(f"Column '{column}' not found in dataframe.")
    
    # Calculate Q1, Q3 and IQR
    Q1 = df[column].quantile(0.25)
    Q3 = df[column].quantile(0.75)
    IQR = Q3 - Q1
    
    lower_bound = Q1 - factor * IQR
    upper_bound = Q3 + factor * IQR
    
    # Detect outliers
    outliers = (df[column] < lower_bound) | (df[column] > upper_bound)
    
    if method == "trim":
        # Drop rows with outliers
        return df[~outliers].reset_index(drop=True)
    elif method == "cap":
        # Cap outliers to the boundary values
        df[column] = df[column].clip(lower=lower_bound, upper=upper_bound)
        return df
    elif method == "nan":
        # Replace outliers with NaN
        df.loc[outliers, column] = pd.NA
        return df
    else:
        raise ValueError("Method must be one of: 'trim', 'cap', or 'nan'")
    if __name__ == "__main__":
    print("This script contains only the handle_outliers_iqr function. Pass a DataFrame to it externally.")
~~~


* Duplicates – Detecting and removing duplicate rows or entries to avoid bias in results.
  ~~~ python
  import pandas as pd
   def handle_duplicates(df, subset=None, action="drop", keep="first"):
    """
    Detects and handles duplicate rows in a DataFrame.
    
    Parameters
    ----------
    df : pandas.DataFrame
        Input dataframe.
    subset : list or None, optional (default=None)
        Columns to consider for detecting duplicates. 
        If None, all columns are used.
    action : str, optional (default="drop")
        How to handle duplicates:
        - "drop": remove duplicates
        - "mark": add a flag column 'is_duplicate'
        - "count": return a summary of duplicate counts
    keep : {"first", "last", False}, optional (default="first")
        Determines which duplicates (if any) to keep:
        - "first": Keep the first occurrence, drop others
        - "last": Keep the last occurrence, drop others
        - False: Drop all duplicates
    
    Returns
    -------
    pandas.DataFrame
        DataFrame with duplicates handled according to action.
    """
    
    df = df.copy()  # Work on a copy so original DataFrame isn’t changed
    
    # Detect duplicates based on chosen subset
    duplicates = df.duplicated(subset=subset, keep=keep)
    
    if action == "drop":
        # Drop duplicate rows
        return df[~duplicates]
    
    elif action == "mark":
        # Add a boolean column marking duplicates
        df["is_duplicate"] = duplicates
        return df
    
    elif action == "count":
        # Return counts of duplicates by chosen subset
        if subset is None:
            subset = df.columns.tolist()
        return df.groupby(subset).size().reset_index(name="count")
    
    else:
        raise ValueError("Invalid action. Choose from 'drop', 'mark', or 'count'.")
    
    if __name__ == "__main__":
    print("This script contains only the handle_duplicates function. Pass a DataFrame to it externally.")
  ~~~
* Out of Range Data set

  ~~~ python
  mport pandas as pd

  def cap_out_of_range(df, column, min_val=None, max_val=None, inplace=False):

    if column not in df.columns:
        raise ValueError(f"Column '{column}' not found in DataFrame")

    if inplace:
        if min_val is not None:
            df[column] = df[column].clip(lower=min_val)
        if max_val is not None:
            df[column] = df[column].clip(upper=max_val)
        return df
    else:
        df_copy = df.copy()
        if min_val is not None:
            df_copy[column] = df_copy[column].clip(lower=min_val)
        if max_val is not None:
            df_copy[column] = df_copy[column].clip(upper=max_val)
        return df_copy
    if __name__ == "__main__":
    print("This script contains only the cap_out_of_range function. Pass a DataFrame to it externally.")

  ~~~

Formatting Issues – Cleaning strings, removing unwanted characters, and standardizing values.
~~~ python
import re
import pandas as pd
def Remove_strings(df, columns):
    """
    Remove non-numeric characters from specified columns and convert them to numeric type.

    Parameters:
    df (pd.DataFrame): The input DataFrame.
    columns (list): List of column names where string cleaning should be applied.

    Returns:
    pd.DataFrame: DataFrame with cleaned numeric values in specified columns.
    """

    for col in columns:
        if col in df.columns:
            # Convert the column to string and remove anything that is not:
            #   - digit (0-9)
            #   - decimal point (.)
            #   - minus sign (-)
            df[col] = df[col].astype(str).apply(
                lambda x: re.sub(r"[^0-9.\-]", "", x)  
            )

            # Convert the cleaned string values to numeric (floats or ints)
            # Invalid conversions are set to NaN (errors='coerce')
            df[col] = pd.to_numeric(df[col], errors="coerce")

    return df
if __name__ == "__main__":
    print("This script contains only the Remove_strings function. Pass a DataFrame to it externally.")
~~~

Cross-Field Validation – Ensuring logical consistency across fields (e.g., sum of components equals total).
 ~~~ python
 import pandas as pd
import numpy as np

# -----------------------------------------------------------
# Cross-field validation function
# Ensures that the sum of component columns equals the total column.
# Adds helper columns to show discrepancies and validity flags.
# -----------------------------------------------------------
def crossfield_validation(df, component_cols, total_col, tolerance=0):
    df = df.copy()  # Work on a copy of the DataFrame to avoid modifying the original
    
    # Step 1: Calculate the sum of the component columns for each row
    df["component_sum"] = df[component_cols].sum(axis=1)
    
    # Step 2: Compute the difference between reported total and calculated component sum
    df["difference"] = df[total_col] - df["component_sum"]
    
    # Step 3: Mark rows as valid if the difference is within tolerance (default: exact match)
    df["is_valid"] = df["difference"].abs() <= tolerance
    
    return df


# -----------------------------------------------------------
# Handle cross-field issues by applying different correction strategies:
# - drop: remove inconsistent rows
# - set_na: replace invalid totals with NaN
# - impute: replace invalid totals with calculated component sum
# - mark: keep everything (flag inconsistencies for manual review)
# -----------------------------------------------------------
def handle_crossfield_issues(df, component_cols, total_col, action="mark", tolerance=0):
    df = df.copy()
    
    # Step 1: Recalculate component sums
    df["component_sum"] = df[component_cols].sum(axis=1)
    
    # Step 2: Calculate the difference from the reported total
    df["difference"] = df[total_col] - df["component_sum"]
    
    # Step 3: Identify rows where totals match component sums (within tolerance)
    df["is_valid"] = df["difference"].abs() <= tolerance
    
    # Step 4: Apply the chosen correction action
    if action == "drop":
        # Remove rows where totals are inconsistent
        return df[df["is_valid"]].drop(columns=["component_sum", "difference", "is_valid"])
    
    elif action == "set_na":
        # Replace inconsistent totals with NaN
        df.loc[~df["is_valid"], total_col] = np.nan
        return df.drop(columns=["component_sum", "difference", "is_valid"])
    
    elif action == "impute":
        # Replace inconsistent totals with the calculated component sum
        df.loc[~df["is_valid"], total_col] = df.loc[~df["is_valid"], "component_sum"]
        return df.drop(columns=["component_sum", "difference", "is_valid"])
    
    elif action == "mark":
        # Keep helper columns for further inspection
        return df  
    
    else:
        raise ValueError("Invalid action. Choose from 'drop', 'set_na', 'impute', or 'mark'.")
if __name__ == "__main__":
    print("This script contains only the crossfield_validation and handle_crossfield_issues functions. Pass a DataFrame to them externally.")
 ~~~~

* the function takes messy phone numbers and standardizes them into either local or international

   ~~~ python
   import pandas as pd
  import re

  # Function to normalize phone numbers for NG, US, and UK
  def clean_phone_number(phone, keep_local=True, country="NG"):
    """
    Standardizes phone numbers to either local format or international format.
    
    Parameters:
    - phone: str, the phone number to clean
    - keep_local: bool, True to return local format (e.g., 08123456789), 
                  False to return international format (e.g., +2348123456789)
    - country: str, country code ('NG', 'US', 'UK') to determine formatting rules
    
    Returns:
    - str: normalized phone number, or "Invalid" if it cannot be parsed
    """
    
    # Remove all non-digit characters (spaces, dashes, parentheses)
    phone = re.sub(r"\D", "", phone)
    
    # Nigeria phone number rules
    if country == "NG":
        if phone.startswith("0") and len(phone) == 11:
            # Already local format
            return phone if keep_local else "+234" + phone[1:]
        elif phone.startswith("234") and len(phone) == 13:
            # International without plus sign
            return ("0" + phone[3:]) if keep_local else "+" + phone
        elif phone.startswith("+234"):
            # International with plus sign
            return ("0" + phone[4:]) if keep_local else phone
    
    # US phone number rules
    elif country == "US":
        if len(phone) == 10:
            # Local format
            return phone if keep_local else "+1" + phone
        elif phone.startswith("1") and len(phone) == 11:
            # International without plus
            return phone[1:] if keep_local else "+" + phone
        elif phone.startswith("+1"):
            # International with plus
            return phone[2:] if keep_local else phone
    
    # UK phone number rules
    elif country == "UK":
        if phone.startswith("0") and len(phone) == 11:
            # Local format
            return phone if keep_local else "+44" + phone[1:]
        elif phone.startswith("44") and len(phone) == 12:
            # International without plus
            return ("0" + phone[2:]) if keep_local else "+" + phone
        elif phone.startswith("+44"):
            # International with plus
            return ("0" + phone[3:]) if keep_local else phone
    
    # If it doesn’t match any known pattern

   ~~~

Reusable Functions – Each process is wrapped in reusable Python functions, making data cleaning faster and easier for future datasets.

📖 How to Use


