# Module 1: Data Pipeline

This module implements the complete catalog pipeline:

- Scrape book data from `http://books.toscrape.com/`.
- Clean and type the raw fields.
- Convert GBP prices to INR using the required fixed rate.
- Load the data into a normalized SQLite database.
- Query the database with SQL and pandas.


## Installation and Run Instructions

1. Open `data_preparation.ipynb` in VS Code or Jupyter with a Python kernel.
2. Install the required packages:

    ```text
    pip install requests beautifulsoup4 pandas
    ```

3. Run the notebook cells from top to bottom.
4. The notebook will recreate `books.db` and generate `query_outputs.json`.

## Requirement 1: Scraping

- Uses `requests` and `BeautifulSoup`.
- Scrapes the first five paginated catalogue pages.
- Follows each book's detail-page link to capture its category.
- Captures the following raw fields:
   - `title`
   - `price`
   - `rating` as text such as `One` or `Five`
   - `availability`
   - `category`
- The validated run produced **100 books across 29 categories**, exceeding the requirement of at least 60 books across at least 3 categories.

## Requirement 2: Cleaning and Data Types

- Removes the GBP symbol from `price` and converts it to numeric `price_gbp` with type `float`.
- Maps text ratings from `One` through `Five` to integer `rating` values from 1 through 5.
- Converts availability into boolean `in_stock` values:
   - `In stock` becomes `True`.
   - `Out of stock` becomes `False`.
- Drops rows with unrecognized availability text instead of silently classifying them.
- Uses median imputation for numeric price or rating parse failures so valid records are retained.
- Rounds an imputed rating to the nearest integer before converting it to the integer type.
- The validated cleaned columns are typed as:
   - `price_gbp`: `float64`
   - `rating`: `int64`
   - `in_stock`: `bool`

## Requirement 3: Fixed Currency Conversion

- Uses the required project-defined baseline rate:

   **1 GBP = 105.50 INR**

- Computes `price_inr` as:

   `price_inr = price_gbp * 105.50`

- Does not call an exchange-rate API because the assignment requires the fixed baseline rate.

## Requirement 4: Normalized SQLite Schema

- Creates two related tables:

   **`categories`**
   - `category_id INTEGER PRIMARY KEY AUTOINCREMENT`
   - `category_name TEXT UNIQUE NOT NULL`

   **`books`**
   - `book_id INTEGER PRIMARY KEY AUTOINCREMENT`
   - `title TEXT NOT NULL`
   - `price_gbp REAL`
   - `price_inr REAL`
   - `rating INTEGER`
   - `in_stock INTEGER`
   - `category_id INTEGER`
   - `FOREIGN KEY (category_id) REFERENCES categories(category_id)`

- Enables SQLite foreign-key enforcement with `PRAGMA foreign_keys = ON`.
- Inserts each category once and stores the category relationship through `category_id`.

## Requirement 5: SQL Queries and Saved Outputs

The notebook executes and saves five SQL queries in `query_outputs.json`:

- **Query 1:** `SELECT` and `WHERE`, with `ORDER BY rating` and `LIMIT 10`.
- **Query 2:** `DISTINCT` category IDs for books currently in stock.
- **Query 3:** `IN`, filtering books with rating 1 or 5.
- **Query 4:** `BETWEEN`, filtering prices from GBP 20 to GBP 30.
- **Query 5:** `JOIN`, combining books with category names and returning the top 10 rated books.

The saved output file contains both the query text and tabular output for every query.

## Requirement 6: Pandas Validation

- Reads multiple SQL results into pandas DataFrames using `pd.read_sql(...)`.
- Recreates the category JOIN with `pd.merge(...)` using the in-memory book and category DataFrames, without SQL.
- Displays the SQL JOIN result and the pandas merge result side by side.
- Confirms equivalent results with:

   ```text
   JOIN outputs equivalent: True
   ```

## Requirements Checklist

- At least 60 books: **100 scraped**.
- At least 3 categories: **29 categories found**.
- Required cleaned columns: **present and correctly typed**.
- Fixed conversion: **1 GBP = 105.50 INR**.
- Normalized SQLite schema: **two tables with a primary/foreign-key relationship**.
- SQL coverage: **five queries covering all required clauses and a JOIN**.
- Saved query evidence: **available in `query_outputs.json`**.
- SQL-to-pandas JOIN comparison: **equivalent**.
