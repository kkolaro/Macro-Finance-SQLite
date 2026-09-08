# Macro-Finance SQLite Practice Project

This educational project demonstrates how to combine corporate financial statements with macroeconomic indicators using Python, SQLite, SQL, and pandas.

## Technologies

* Python 3.10+
* SQLite, through Python's built-in `sqlite3` module
* pandas

## Database structure

The database contains two tables.

### `macro\_indicators`

Stores macroeconomic observations for a country and reporting date.

|Column|Type|Description|
|-|-|-|
|`record\_id`|INTEGER|Auto-incrementing primary key|
|`country\_code`|TEXT|Country or economic-area code|
|`date`|TEXT|Observation date in `YYYY-MM-DD` format|
|`gdp\_growth\_pct`|REAL|GDP growth rate|
|`cpi\_inflation\_pct`|REAL|CPI inflation rate|
|`central\_bank\_rate`|REAL|Central-bank policy rate|

The following constraint prevents duplicate macro observations for the same country and date:

```sql
UNIQUE (country\_code, date)
```

### `corporate\_financials`

Stores quarterly corporate financial statements.

|Column|Type|Description|
|-|-|-|
|`statement\_id`|INTEGER|Auto-incrementing primary key|
|`ticker`|TEXT|Company ticker|
|`country\_code`|TEXT|Country or economic-area code|
|`statement\_date`|TEXT|Financial-statement date|
|`fiscal\_year`|INTEGER|Fiscal year|
|`fiscal\_quarter`|INTEGER|Fiscal quarter|
|`revenue\_usd`|REAL|Revenue in US dollars|
|`net\_income\_usd`|REAL|Net income in US dollars|
|`total\_assets\_usd`|REAL|Total assets in US dollars|

A company cannot have two records for the same fiscal period:

```sql
UNIQUE (ticker, fiscal\_year, fiscal\_quarter)
```

## Composite relationship

The corporate table is connected to the macroeconomic table through two columns:

```sql
FOREIGN KEY (country\_code, statement\_date)
    REFERENCES macro\_indicators (country\_code, date)
```

Both columns are required because a country has many macroeconomic observations over time. `country\_code` alone cannot identify the correct period.

The corresponding join is:

```sql
ON f.country\_code = m.country\_code
AND f.statement\_date = m.date
```

This does not create hierarchical groups. The pair of values identifies the macroeconomic record that corresponds to a financial statement.

## SQLite foreign-key validation

Foreign-key enforcement is enabled for the current SQLite connection with:

```python
conn.execute("PRAGMA foreign\_keys = ON")
```

Without this setting, SQLite may allow a corporate record whose country and date do not exist in `macro\_indicators`.

The setting must be enabled for every new SQLite connection.

## Installation

Clone the repository and enter its directory:

```bash
git clone <repository-url>
cd <repository-directory>
```

Create and activate a virtual environment.

Windows:

```bash
python -m venv .venv
.venv\\Scripts\\activate
```

macOS or Linux:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Install pandas:

```bash
python -m pip install pandas
```

The `sqlite3` module is included with Python and normally does not require a separate installation.

## Running the project

Run the Python script from the project directory:

```bash
python financial\_database.py
```

The program will:

1. open or create the SQLite database;
2. enable foreign-key validation;
3. create the two tables;
4. insert sample data;
5. execute the SQL queries;
6. display the results as pandas DataFrames;
7. close the database connection.

```

## Notes

* Dates are stored in ISO format: `YYYY-MM-DD`.
* `JOIN` without an explicit type means `INNER JOIN`.
* `INSERT OR IGNORE` makes repeated insertion of the sample data convenient, but it should be used carefully because it may conceal data-quality problems.
* The database connection is closed inside a `finally` block, even if an error occurs.
* If an older database has a different schema, use a new database filename or remove the old test database before running the updated script.

## 

