# ETL Project for Largest Banks Data
This project `extracts`, `transforms`, and `loads` data on the largest banks by market capitalization from a Wikipedia page. Below is an explanation of the code used for this process.

## Prerequisites
Ensure you have the following Python libraries installed:

- `beautifulsoup4`
`requests`
`pandas`
`numpy`
`sqlite3 (if database integration is required)`

###You can install the required libraries using pip:

```bash
pip install beautifulsoup4 requests pandas numpy sqlite3 apache-airflow
```
## Code Overview
The code performs the following steps:
- Extract Data from Wikipedia
- Transform the Data
- Load Data to a CSV File
- Load Data to SQLite Database
- Run SQL Queries
- Log the Process
  
### 1. Extract Data from Wikipedia
The extract function fetches the HTML content of the Wikipedia page and extracts the data on the largest banks.

```python
url = 'https://web.archive.org/web/20230908091635/https://en.wikipedia.org/wiki/List_of_largest_banks'
csv_path = r"C:\Users\Abdo\Desktop\Projects\ETL_Preoject_Bank\largest_banks.csv"

def extract(url):
    page = requests.get(url).text
    data = soup(page, 'html.parser')
    heading = data.find('span', {'id': 'By_market_capitalization'})
    table = heading.find_next('table', {'class': 'wikitable'})
    df = pd.read_html(StringIO(str(table)))[0]
    df['Market cap (US$ billion)'] = df['Market cap (US$ billion)'].apply(lambda x: float(str(x).strip()[:-1]))
    return df
```
### 2. Transform the Data
The transform function processes the extracted data, converting market capitalization to other currencies using exchange rates.

```python
def transform(df, exchange_rate):
    df['MC_GBP_Billion'] = [np.round(x * exchange_rate['GBP'], 2) for x in df['Market cap (US$ billion)']]
    df['MC_EUR_Billion'] = [np.round(x * exchange_rate['EUR'], 2) for x in df['Market cap (US$ billion)']]
    df['MC_INR_Billion'] = [np.round(x * exchange_rate['INR'], 2) for x in df['Market cap (US$ billion)']]
    return df
```
### 3. Load Data to a CSV File
The load_to_csv function saves the transformed data to a CSV file.

```python
def load_to_csv(df, csv_path):
    df.to_csv(csv_path, index=False)
    print(f"Data saved to {csv_path}")
```
### 4. Load Data to SQLite Database
The load_to_db function loads the transformed data into an SQLite database.

```python
def load_to_db(conn, table_name, df):
    df.to_sql(table_name, conn, if_exists='replace', index=False)
    print(f"Data loaded into table {table_name} in database.")
```
### 5. Run SQL Queries
The run_queries function executes SQL queries on the SQLite database and logs the output.

```python
def run_queries(query, conn):
    """
    Executes the given SQL query and prints the results.
    
    Parameters:
    query (str): The SQL query to execute.
    conn (sqlite3.Connection): The SQLite3 database connection object.
    """
    print(f"Executing Query: {query}")
    try:
        df = pd.read_sql_query(query, conn)
        print("Query Output:")
        print(df)
    except Exception as e:
        print(f"Error: {e}")
    
    log_progress(f"Executed Query: {query}\nOutput: {df.to_string(index=False)}", log_path)
```
### 6. Log the Process
The log_progress function logs the stages of the ETL process.

```python
def log_progress(message, log_path):
    ''' Logs the mentioned message at a given stage of the code execution to a log file. '''
    timestamp_format = '%Y-%b-%d-%H:%M:%S'
    now = datetime.now()
    timestamp = now.strftime(timestamp_format)
    os.makedirs(os.path.dirname(log_path), exist_ok=True)
    with open(log_path, "a") as f:
        f.write(timestamp + ' : ' + message + '\n')
```
### Main Execution
The main part of the script orchestrates the ETL process.

```python
log_progress('Preliminaries complete. Initiating ETL process', log_path)

df = extract(url)
log_progress('Data extraction complete.', log_path)

exchange_rate_df = pd.read_csv(exchange_rate_path)
exchange_rate = exchange_rate_df.set_index('Currency').to_dict()['Rate']

log_progress('Exchange rate data loaded.', log_path)

df = transform(df, exchange_rate)
log_progress('Data transformation complete.', log_path)

load_to_csv(df, csv_path)
log_progress('Transformed data saved to CSV file.', log_path)

conn = sqlite3.connect(db_name)
load_to_db(conn, table_name, df)
log_progress(f'Data loaded into table {table_name} in database.', log_path)
conn.close()

conn = sqlite3.connect(db_name)
queries = [
    "SELECT * FROM Largest_banks",
    "SELECT AVG(MC_GBP_Billion) FROM Largest_banks",
    "SELECT `Bank name` FROM Largest_banks LIMIT 5"
]
for query in queries:
    run_queries(query, conn)
conn.close()
```
## Summary
- This project `extracts` data on the largest banks from a Wikipedia page, `transforms` it, and `loads` it into both a CSV file and an SQLite database while logging each step of the process.
- This script demonstrates the `ETL` (`Extract`, `Transform`, `Load`) process effectively.
