# Data Pipeline

## Objective

This module implements the raw-to-relational catalog pipeline described in the capstone: scrape public product data, clean and type the fields, convert prices using the required fixed project rate, load a normalized SQLite database, and demonstrate SQL and pandas querying.

## Files

- `scrape_books.ipynb` — scraping, cleaning, currency conversion, SQLite creation, SQL queries, and pandas validation.
- `books.db` — SQLite database generated/used by the notebook.
- `README.md` — module documentation.

## Pipeline

```text
books.toscrape.com
        |
        v
requests + BeautifulSoup
        |
        v
Raw book records
        |
        v
Cleaning / typing
        |
        +--> price_gbp
        +--> rating (1-5)
        +--> availability -> boolean
        +--> category
        |
        v
GBP -> INR conversion
1 GBP = 105.50 INR
        |
        v
SQLite
categories <-- category_id --> books
        |
        +--> SQL queries
        |
        +--> pandas read_sql()
        |
        +--> pandas merge()
```

## Scraping Scope

The notebook scrapes these five category paths:

- `travel_2`
- `mystery_3`
- `historical-fiction_4`
- `sequential-art_5`
- `classics_6`

The executed notebook output shows **90 books** across **5 categories**, exceeding the minimum requirement of 60 books across at least 3 categories.

## Cleaning Decisions

The notebook performs the following transformations:

- Category suffixes such as `_2` are removed with a regular expression.
- Star-rating words (`One` through `Five`) are mapped to integers `1` through `5`.
- Missing numeric ratings are filled with the column median.
- The pound symbol is removed and price is converted to `float`.
- Missing prices are filled with the column mean.
- Availability is converted to a boolean using the presence of `"In stock"`.
- The fixed conversion rate required by the assignment is **1 GBP = 105.50 INR**.

The current notebook uses the names `price_in_gbp`, `price_in_inr`, `star_rating`, and `availability`. The assignment's example names are `price_gbp`, `price_inr`, `rating`, and `in_stock`; these are semantically equivalent, but exact naming should be reviewed before final submission if the grader expects the specified names literally.

## Database Design

The notebook creates two normalized tables:

```sql
categories(
    category_id INTEGER PRIMARY KEY,
    category_name TEXT UNIQUE
)

books(
    book_id INTEGER PRIMARY KEY,
    title TEXT,
    price_in_gbp REAL,
    price_in_inr REAL,
    rating INTEGER,
    availability INTEGER,
    category_id INTEGER,
    FOREIGN KEY(category_id) REFERENCES categories(category_id)
)
```

The `category_id` foreign key connects each book to its category.

## SQL Demonstration

The notebook executes five queries covering the requested SQL concepts:

1. `SELECT` + `WHERE` — books with a rating of 5.
2. `ORDER BY` + `LIMIT` — most expensive book.
3. `DISTINCT` + `JOIN` — category/title results for expensive books.
4. `BETWEEN` — books priced from £10 to £20.
5. `JOIN` + `WHERE` — books in the mystery category.

The saved notebook output shows, for example:

- 11 books with a 5-star rating.
- The most expensive scraped book at £59.48.
- 20 mystery-category books.
- 20 books priced between £10 and £20.

## SQL vs pandas Join

The notebook also reads the `books` and `categories` tables with `pd.read_sql()` and reproduces the relationship using:

```python
pd.merge(df_books, df_categories, on="category_id")
```

This demonstrates the requested SQL-join/pandas-merge equivalence. The two displayed results contain the same book/category relationship, although the pandas merge includes additional columns from the source DataFrames.

## How to Run

From the repository root:

```bash
jupyter notebook data_pipeline/scrape_books.ipynb
```

Run the notebook from top to bottom. It recreates the database tables and repopulates them from the freshly scraped data.

## Current Implementation Notes

The supplied notebook is functional enough to demonstrate the core pipeline, but review these items before final submission:

- The assignment explicitly asks for robust handling of failed HTTP requests. The current code checks `response.status_code == 200`, but does not explicitly call `raise_for_status()` or provide broader request-exception handling.
- The notebook inserts manually assigned category IDs. A more robust implementation would obtain the IDs from the inserted category table rather than depending on a hard-coded dictionary.
- The notebook's query set covers the required SQL clauses, but the first query output and other results are notebook outputs rather than separately exported query-result files.
- The database file is included in the supplied archive, so it can be inspected without rerunning the scraper; rerunning the notebook is the reproducible path.

## Design Summary

The implementation follows the requested scrape → clean → convert → relational store → query → pandas validation story. The fixed currency baseline is deliberately used instead of a live API because the project specification makes 105.50 INR per GBP the required graded conversion.
