# FinancialDashboard
The Bank Transaction Analyzer is a Python project that simplifies financial data management using CIBC Online. It employs data analysis, cleaning techniques, and Streamlit to create an interactive dashboard. This user-friendly tool enhances financial awareness and decision-making.

#### Downloading Transactions
Proceed to the CIBC website and visit the 'Download Transactions' section on the side menus. Select the desired account, and click 'All Available' at the display option. Scroll below to find Financial Management Software with the 'Spreadsheet(CSV)' option

#### Creating A New Folder
Creating a folder will assist in organization

```
import os

# Identify A Path To The Desktop or any location
desktop_path = os.path.expanduser("C:\\Users\\S00742997//Desktop")

# Name of the new folder
new_folder_name = "Bank_Transactions"

# Create the full path for the new folder
new_folder_path = os.path.join(desktop_path, new_folder_name)

# Create the new folder
os.makedirs(new_folder_path)

# A confirmation message
print(f"Folder '{new_folder_name}' created on the desktop.")
```

#### Transferring the file, from download to the new location
```
import shutil

# Path to the source folder. The file will be in the downloads
source_folder = "C:\\Users\\S00742997\\Downloads\\cibc.csv"

# Path to the destination folder. This is the file that we created earlier. 
destination_folder = "C:/Users/S00742997/Desktop/Bank_Transactions"

# Move the source folder to the destination folder
shutil.move(source_folder, destination_folder)

print(f"Folder moved from '{source_folder}' to '{destination_folder}'.")
```
### Working with the file
#### Adding a header section.

The file initially does not contain any headers, hence we must add ours in order to filter and sort the values

```
import csv

# New header (list of column names). These will now be in the first row of the sheet
new_header = ["Date", "Transaction", "Debited", "Credited", "Account"]  # Replace with your actual column names

# File Path created
csv_file_path = destination_folder +'\cibc.csv'

# Read the existing CSV data
existing_data = []
with open(csv_file_path, newline='') as csvfile:
    reader = csv.reader(csvfile)
    for row in reader:
        existing_data.append(row)

# Insert the new header at the beginning
updated_data = [new_header] + existing_data

# Write the updated data back to the CSV file
with open(csv_file_path, 'w', newline='') as csvfile:
    writer = csv.writer(csvfile)
    writer.writerows(updated_data)

# Confirmation message
print("New header added to the CSV file.")
```
### Data Cleansing
To clean the data, we must clear up inconsistencies in the data as well as improve the methods of organization

#### Importing the file
Importing the Data from the new location
```
import pandas as pd
df = pd.read_csv(csv_file_path)

from datetime import datetime

df['Date'] = pd.to_datetime(df['Date'])  # Convert 'Date' column to datetime

```
#### Creating a  Year & Month Column
```
# Creating a Function to obtain the 'Month':

def get_month_name(month_number):
    return datetime.strptime(str(month_number), "%m").strftime("%B")

df['Month'] = df['Date'].apply(lambda date: get_month_name(date.month))
df['Year'] = df['Date'].apply(lambda date: date.year)

# The new table consists of the following columns: 
df= df[['Year', 'Month', 'Transaction', 'Debited', 'Credited', 'Account']]

```
#### Fixing Transactions
```
# Convert the Transaction into lower case
df['Transaction'] = df['Transaction'].str.lower() 
# used to split the values in the 'Transaction' 
df['Transaction'] = df['Transaction'].str.split('#').str[0]

# This code removes the word "the" from each entry in the 'Transaction' column
df['Transaction'] = df['Transaction'].str.replace(r'\bthe\b', '', regex=True) 
```

Cleaning Null Values and replacing it with 0
```
df.fillna(0, inplace=True)  # This will fill NaN values with 0 in the original DataFrame
```
Spotify Transactions
```
# This code removes the random letters and numbers
df['Transaction'] = df['Transaction'].str.replace(r'spotify [a-z0-9]+', 'spotify', regex=True)
df['Transaction'] = df['Transaction'].str.replace(r'\d+', '', regex=True)
```
Fixing Presto Issues
```
def normalize_transaction(transaction):
    # Remove variations and standardize
    if 'presto' in transaction:
        return 'PRESTO'
    return transaction

# Apply the normalization function to the 'Transaction' column
df['Transaction'] = df['Transaction'].apply(normalize_transaction)

# Group by 'Transaction' and sum 'Debited'
x = df.groupby('Transaction')['Debited'].sum().reset_index() .sort_values('Debited', ascending = False) # Sorting the Transactions
```
Taking a look at our DataFrame to confirm there are no issues
```
df.head()
```
### Creating A Dashboard
#### Title & Home Page
```
import streamlit as st

# A page is created 
st.set_page_config(page_title='Transactions Dashboard', page_icon=':moneybag:', layout='wide')

# Title created
st.title(':moneybag: Finance Dashboard')
st.markdown('##')
```
#### Creating Filters
Creating filters for the page is important as it will allow the user to customize their views based off type of transactions, certain months and/or years

```
# A header on the sidebar has been created for 'Filters'
st.sidebar.header('Filters: ')

# Filter created for the 'Month', 'Year', 'Transaction'
month = st.sidebar.multiselect('Select the Month:', options=df['Month'].unique(), default=df['Month'].unique().tolist())
year = st.sidebar.multiselect('Select the Year:', options=df['Year'].unique(), default=df['Year'].unique().tolist())
transaction = st.sidebar.multiselect('Select the Transaction', options=df['Transaction'].unique(), default=df['Transaction'].unique().tolist())

# The effects of the 'Month', 'Year', 'Transaction' button will impact the results of graphs
df= df.query('Month == @month & Year == @year & Transaction == @transaction')
```
#### Total Spendings, Average and First Impressions
```
total_cost = int(df['Debited'].sum()) # The sum of the Debited column is the total spending 
spending =df[df['Debited']>0] # This creates a DataFrame for debit values over 0
transaction_count = len(spending) # This will determine the amount of transactions
average = round((total_cost/transaction_count),2) # The average is determined by total cost over number of transaction

# Placing values on the page
left_column, right_column = st.columns(2)  # Use two columns, but only use the first one
with left_column:
    st.subheader('Total Debited :')
    st.subheader(f"CAD $ {total_cost:,}") # Total Cost
with right_column:
    st.subheader('Average Cost Per Transaction: ')
    st.subheader(f'CAD $ {average:,}') # Average

st.markdown('---')
```
### Adding Graphs
#### Top 10 Costly Transactions & Costs per Year
This will display the transactions that have costed the user the most money. These transactions are grouped
```
# Import the approrpriate libraries
import plotly.express as px

top_10 = x.head(10) # The 'top_10' variable, contains the variable x
# variable x is a DF that is grouped by the debited amounts in order
# By taking the first 10 values, we are able to see the top 10 most costly amounts

# The result_year varibale is sorted by Year with the sum of the total debited amounts for the corresponding year
result_year = df.groupby(['Year'])['Debited'].sum().reset_index().sort_values('Year', ascending = False)

# The two variables for graphs is created 
fig_10 = px.bar(top_10, x='Transaction', y='Debited', title='Top 10 Costly Transactions', color_discrete_sequence = ['red'])
fig_year = px.bar(result_year, x='Year', y='Debited', title='Debited Amount Over Years', color_discrete_sequence=['green'])

col1, col2 = st.columns(2)  # Create two columns

# The graphs are placed into each column
col1.plotly_chart(fig_10, use_container_width=True)
col2.plotly_chart(fig_year, use_container_width=True)

```
#### Top 10 Most Recurring Transactions
```
# The data frame of the top 10 transactions that have occured the most
transaction_count = df['Transaction'].value_counts().head(10)

# Creating the graph
fig = px.bar(
    x=transaction_count.values, 
    y=transaction_count.index,
    orientation='h',  # Horizontal bar chart
    labels={'x': 'Number of Occurrences', 'y': 'Transaction'},
    title='Top 10 Most Recurring Transactions',
    color_discrete_sequence=['red'],  # Set the bar color to red

)
st.plotly_chart(fig)

```

#### Transaction Trend

```
# This variable was created in order to keep a track of the months in order
month_order = [
    'January', 'February', 'March', 'April', 'May', 'June',
    'July', 'August', 'September', 'October', 'November', 'December'
]

# Convert the 'Month' column to a categorical variable with custom sorting
df['Month'] = pd.Categorical(df['Month'], categories=month_order, ordered=True)

# Group and aggregate the data by year and month
monthly_data = df.groupby(['Year', 'Month'])['Debited'].sum().reset_index()

# Create line graph using Plotly Express (px)
fig = px.line(
    monthly_data, x='Month', y='Debited', color='Year',
    labels={'Debited': 'Debited'},
    title='Monthly Debited Over Years'
)

# Customize the plot layout
fig.update_xaxes(categoryorder='array', categoryarray=month_order)  # Sort months
fig.update_layout(width=1000, height=500)  # Adjust plot size

# Annotations are added to the graph for eaiser reading
for i, row in monthly_data.iterrows():
    fig.add_annotation(
        text=f'{row["Debited"]:.2f}', x=row["Month"], y=row["Debited"],
        showarrow=True, arrowhead=2, font=dict(size=10)
    )


# Streamlit app
    # A title is created
st.title('Monthly Debited Analysis')
st.plotly_chart(fig)

```
```
# Display the dataframe
st.dataframe(df)
```
























