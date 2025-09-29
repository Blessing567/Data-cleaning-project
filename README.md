# 🧹 Data Cleaning & Quality Utilities

This project provides a collection of reusable Python functions to detect, validate, and correct common data quality issues in datasets.

It is designed for data analysts, data scientists, and students who want to ensure their datasets are clean, consistent, and ready for analysis or machine learning tasks.


# 🚀 Project Motivation

* Data scientists spend 80–90% of their time cleaning data before analysis or modeling.
* This project was built to:

* Automate repetitive cleaning steps.

* Provide reusable, well-structured functions for both numerical and categorical data.

* Show practical computer science principles such as modular design, abstraction, and reusability.

# 📂 Project Structure
* Data/ → contains the raw or “dirty” dataset used for cleaning.

* Example: Dirty_Data.py holds the initial dataset used in the workflow.

* numerical_values/ → contains Python modules that clean and validate numeric-related columns.

* Remove_strings.py → removes unwanted characters and converts strings to numeric values.

* duplicates.py → identifies and removes duplicate rows.

* cross_field_validation.py → checks consistency between related fields (e.g., sum of components vs. total).

* detecting_outliers.py → detects and handles outliers using statistical methods (like IQR).

* out_of_range.py → caps values to fall within valid ranges (e.g., age between 0 and 100).

* phone.py → standardizes phone number formats.

* categorical_functions/ → contains modules for handling categorical data issues.

* honourifics.py → removes titles/honorifics (Mr., Dr., etc.) from names.

* membership_constraint.py → validates that values belong to a predefined set (e.g., tiers = Gold, Silver).

* numerical_to_category.py → converts numerical codes into category labels.

* functions/ → used for testing or utility scripts.

* full_cleaning_workflow.py → the main script that puts everything together.

* Loads the dirty dataset.

* Calls the different cleaning functions step by step.

* Outputs the final cleaned dataset into a data/cleaned/ folder.

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
* Out of Range Data set in numerical data

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
  def clean_phone_number(phone, keep_local=True, country="NG"):
    import re
    # remove spaces, dashes, brackets, etc.
    phone = re.sub(r"\D", "", str(phone)) if phone else ""

    # Nigeria
    if country == "NG":
        if phone.startswith("0") and len(phone) == 11:
            return phone if keep_local else "+234" + phone[1:]
        elif phone.startswith("234") and len(phone) == 13:
            return ("0" + phone[3:]) if keep_local else "+" + phone
        elif phone.startswith("+234"):
            return ("0" + phone[4:]) if keep_local else phone

    # US
    elif country == "US":
        if len(phone) == 10:
            return phone if keep_local else "+1" + phone
        elif phone.startswith("1") and len(phone) == 11:
            return phone[1:] if keep_local else "+" + phone
        elif phone.startswith("+1"):
            return phone[2:] if keep_local else phone

    # UK
    elif country == "UK":
        if phone.startswith("0") and len(phone) == 11:
            return phone if keep_local else "+44" + phone[1:]
        elif phone.startswith("44") and len(phone) == 12:
            return ("0" + phone[2:]) if keep_local else "+" + phone
        elif phone.startswith("+44"):
            return ("0" + phone[3:]) if keep_local else phone

    return "Invalid
   ~~~

Reusable Functions – Each process is wrapped in reusable Python functions, making data cleaning faster and easier for future datasets.

#📖 How to Use
## Data Set needed for cleaning
~~~python
import pandas as pd
import numpy as np

# Create a "dirty" dataset
data = {
    # Numeric issues: strings, extra symbols, out-of-range
    "age": ["25", "30yrs", "NaN", "-5", "200", "40", None],
    "income": ["$2000", "3000", "N/A", "4000", "5000", "unknown", "3500"],
    
    # Categorical issues: inconsistent case, invalid category, mapping needed
    "gender": ["Male", "female", "FEMALE", "M", "male", "f", None],
    "education_level": [1, 2, 3, 1, "2", 5, None],
    
    # Membership constraint issue
    "tier": ["Gold", "Bronze", "brnz", "Ultimate", None, "Silver", "gold"],
    
    # Cross-field validation: sum of components vs total
    "economy": [50, 60, 70, 80, 20, 30, 40],
    "business": [5, 10, 15, 5, 2, 3, 4],
    "first_class": [2, 5, 0, 1, 0, 1, 2],
    "total_passengers": [57, 75, 85, 86, 22, 34, 46],  # some totals are incorrect
    
    # Phone number issues: inconsistent formats, symbols, missing
    "phone": ["0812345678", "+234812345678", "234812345678", "0-812-345-678", None, "08012345678", "12345"],
    
    # Honorifics in names
    "name": ["Mr. John Doe", "Ms Jane Smith", "Dr. Alex Johnson", "Mrs. Emily Clark", None, "Prof. Mike Brown", "Miss Anna Lee"],
    
    # Duplicate row example
    "customer_id": [1, 2, 3, 4, 5, 3, 1]
}

data = pd.DataFrame(data)
print(data)


~~~
# full_cleaning_workflow.py
* This script serves as the main workflow that ties together all individual data-cleaning modules. Instead of writing long cleaning logic inside one file, the project organizes cleaning tasks into modular Python files. Each module handles a specific type of cleaning (numerical, categorical, or text-based).
  * pandas – used to load, transform, and save tabular data.
  * os – provides utilities for handling file paths, making the workflow more portable across operating systems.
~~~ python
import pandas as pd
import os
~~~

# Module Importation in full_cleaning_workflow.py

📂 Module Importation

* This project is structured into modular components. Each cleaning function is stored in its own script, organized by type:

* numerical_values/ → functions for handling numeric data (e.g., removing strings, detecting outliers, range capping, cross-field validation).

* categorical_functions/ → functions for handling categorical data (e.g., removing honorifics from names, validating membership tiers, converting numbers to categories).

* In the main workflow (full_cleaning_workflow.py), we import only the necessary functions:
~~~ python
# Import cleaning modules
from numerical_values.cross_field_validation import handle_crossfield_issues
from numerical_values.Remove_strings import Remove_strings
from numerical_values.duplicates import handle_duplicates
from numerical_values.phone import clean_phone_number
from numerical_values.detecting_outliers import handle_outliers_iqr
from numerical_values.out_of_range import cap_out_of_range
from categorical_functions.membership_constraint import membership_constraint
from categorical_functions.honourifics import remove_honorifics
from categorical_functions.numerical_to_category import numerical_column_to_category
~~~
# import Dataset and Call cleaning functions
~~~ python
# Step 1: Load the "dirty" dataset
from Data.Dirty_Data import data

dirty_df = data.copy()  # always work on a copy

# Step 2: Clean numeric columns (remove strings, convert to numeric)
dirty_df = Remove_strings(dirty_df, columns=["age", "income"])

# Step 3: Fix outliers using Interquartile Range (IQR)
dirty_df = handle_outliers_iqr(dirty_df, columns="age")
dirty_df = handle_outliers_iqr(dirty_df, columns="income")

# Step 4: Cap numeric values to valid ranges (example: age 0–100)
dirty_df = cap_out_of_range(dirty_df, column="age", min_val=0, max_val=100)

# Step 5: Handle duplicates based on customer_id
dirty_df = handle_duplicates(dirty_df, subset=["customer_id"])

# Step 6: Normalize phone numbers and remove honorifics from names
dirty_df["phone"] = dirty_df["phone"].apply(lambda x: clean_phone_number(x))
dirty_df = remove_honorifics(dirty_df, column="name")

# Step 7: Handle membership constraints for "tier"
valid_tiers = ["Bronze", "Silver", "Gold", "Platinum"]
dirty_df = membership_constraint(
    dirty_df,
    column="tier",
    valid_values=valid_tiers,
    action="replace",
    replace_value="Unknown"
)

# Step 8: Cross-field validation (e.g., passengers)
dirty_df = handle_crossfield_issues(
    dirty_df,
    component_cols=["economy", "business", "first_class"],
    total_col="total_passengers",
    action="impute"  # Fill incorrect totals
)

# Step 9: Convert numeric-like columns to category if needed
dirty_df = numerical_column_to_category(dirty_df, "education_level")
~~~
* Save output as an excel file in an output folder
~~~python
# Step 10: Save the cleaned dataset to the output folder
output_folder = "data/cleaned"
os.makedirs(output_folder, exist_ok=True)

# Save as CSV
csv_path = os.path.join(output_folder, "cleaned_dataset.csv")
dirty_df.to_csv(csv_path, index=False)

# Save as Excel
excel_path = os.path.join(output_folder, "cleaned_dataset.xlsx")
dirty_df.to_excel(excel_path, index=False)

print(f"✅ Cleaned dataset saved to:\n- {csv_path}\n- {excel_path}")
~~~
# Why Functions Are Important in This Project

* Reusability: Instead of rewriting the same cleaning logic again and again (e.g., removing outliers or fixing phone numbers), you only write it once as a function and reuse it anywhere.

* Time-Saving: Functions save hours of repetitive coding because common cleaning steps (like removing honorifics or handling duplicates) are just one function call away.

* Maintainability: If you ever need to update the cleaning logic (for example, add new honorifics like “Engr.”), you only update it once in the function instead of searching through multiple scripts.

* Scalability: As datasets grow larger or more complex, functions keep your code organized, modular, and scalable.

* Professionalism: Functions reflect software engineering best practices. They make your code more readable and easier to understand for teammates, recruiters, or potential employers reviewing your portfolio.

👉 So, instead of hardcoding cleaning steps into  workflow every time, we can  built a toolbox of reusable cleaning functions that can handle many different datasets with minimal effort.




