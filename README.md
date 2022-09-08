# NHSBSA-open-data-ingestion-example
Importing and cleaning open patient list size data (RF1.Patient List Size) from catalyst.services.nhsbsa.nhs.uk

```Python
# import libs
import glob
import pandas as pd
import openpyxl
from datetime import datetime
 
# specifying the path to excel files
path = "..."
 
# grab all excel files in the path
file_list = glob.glob(path + "/*.xlsx")
```


```Python
# extract each sheet into one file

excl_list = []
skipList= list(range(0,8)) # Skip reading first 9 rows of bad data

for file in file_list:
    # Get data
    data = pd.read_excel(file, skiprows= skipList, usecols= "A:AA", na_values= "N/A")
    # Get date
    wb = openpyxl.load_workbook(file)
    sheet = wb.active
    # covert 'date' cell to datetime-string
    dateValue = sheet['A6'].value
    dateValue = dateValue.replace('For ', '')
    dateValue = datetime.strptime(dateValue, '%b-%y')
    dateValue = str(dateValue.date())
    data.insert(0, 'Date', dateValue)
    # drop bad data rows
    data.dropna(inplace=True)
    excl_list.append(data)

# concatenate all DataFrames in the list into a single DataFrame, returns new DataFrame.
excl_merged = pd.concat(excl_list, ignore_index=True)
```


```Python
# Data cleaning

excl_merged.rename(columns = {'Code':'Region_Code', 'Code.1':'STP_Code', 'Comm./Prov.':'ICB', 'Code.2':'ICB_Code', 'Code.3':'Practice_Code'}, inplace = True)
excl_pivot = excl_merged.melt(id_vars=['Date', 'Region', 'Region_Code', 'STP', 'STP_Code', 'ICB', 'ICB_Code', 'Practice', 'Practice_Code'], var_name="Count")
```


```Python
# Export to csv

excl_pivot.to_csv('.../excl_merged.csv', index=False)
```
