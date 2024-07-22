# GDP-Data-ETL-Project
This project extracts, transforms, and loads GDP data from a Wikipedia page. Below is an explanation of the code used for this process.

## Prerequisites
Ensure you have the following Python libraries installed:

BeautifulSoup (bs4)
Requests
Pandas
Numpy
SQLite3 (if database integration is required)
Datetime
### You can install the required libraries using pip:

### bash Copy code:
pip install beautifulsoup4 requests pandas numpy
Code Overview


## The code performs the following steps:

Extract Data from Wikipedia
Transform the Data
Load Data to a CSV File
Log the Process
## 1. Extract Data from Wikipedia
The extract function fetches the HTML content of the Wikipedia page and extracts the GDP data.

### python Copy code:
url = 'https://web.archive.org/web/20230902185326/https://en.wikipedia.org/wiki/List_of_countries_by_GDP_%28nominal%29'
table_attribs = ["Country", "GDP_USD_millions"]
csv_path = "wikipedia_gdp.csv"

def extract(url, table_attribs):
    page = requests.get(url).text
    data = soup(page, 'html.parser')
    df = pd.DataFrame(columns=table_attribs)
    tables = data.find_all('tbody')
    rows = tables[2].find_all('tr')
    for row in rows:
        col = row.find_all('td')
        if len(col) != 0:
            if col[0].find('a') is not None and '—' not in col[2]:
                data_dict = {"Country": col[0].a.contents[0],
                             "GDP_USD_millions": col[2].contents[0]}
                df1 = pd.DataFrame(data_dict, index=[0])
                df = pd.concat([df, df1], ignore_index=True)
    return df
## 2. Transform the Data
The transform function processes the extracted data, converting GDP from millions to billions.

### python Copy code:
def transform(df):
    GDP_list = df["GDP_USD_millions"].tolist()
    GDP_list = [float("".join(x.split(','))) for x in GDP_list]
    GDP_list = [np.round(x / 1000, 2) for x in GDP_list]
    df["GDP_USD_millions"] = GDP_list
    df = df.rename(columns={"GDP_USD_millions": "GDP_USD_billions"})
    return df
## 3. Load Data to a CSV File
The load_to_csv function saves the transformed data to a CSV file.

### python Copy code:
def load_to_csv(df, csv_path):
    df.to_csv(csv_path)
## 4. Log the Process
The log_progress function logs the stages of the ETL process.

### python Copy code:
def log_progress(message):
    ''' This function logs the mentioned message at a given stage of the code execution to a log file. Function returns nothing'''
    timestamp_format = '%Y-%h-%d-%H:%M:%S' # Year-Monthname-Day-Hour-Minute-Second 
    now = datetime.now() # get current timestamp 
    timestamp = now.strftime(timestamp_format) 
    with open("./etl_project_log.txt","a") as f: 
        f.write(timestamp + ' : ' + message + '\n')
## Main Execution
The main part of the script orchestrates the ETL process.

### python Copy code:
log_progress('Preliminaries complete. Initiating ETL process')

df = extract(url, table_attribs)
log_progress('Data extraction complete. Initiating Transformation process')

df = transform(df)
log_progress('Data transformation complete. Initiating loading process')

load_to_csv(df, csv_path)
log_progress('Data saved to CSV file')

print("Done")
## Summary
This project extracts GDP data from a Wikipedia page, transforms it, and loads it into a CSV file while logging each step of the process. This script is a simple demonstration of the ETL (Extract, Transform, Load) process.
