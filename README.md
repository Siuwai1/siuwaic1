# Freddie Mac Personal Project
---
Business Problem
---
We, Cheng Mortgage Services, want to improve how we handle home loans. We're facing more customers having trouble repaying their mortgages. To tackle this, we need to make sense of a big dataset from Freddie Mac. But it's tricky because the data is huge and complicated. Cheng Mortgage Services needs a solution to quickly figure out which loans are risky, predict if someone might not be able to pay, and take steps to avoid losing money

---
Data Sourcing
---
We first acquired the data sets from Freddie Mac's official website, https://www.freddiemac.com/research/datasets/sf-loanlevel-dataset. We specifically picked Loan-Level Dataset Files which are Standard and Annual. Then we chose the sample data sets for the years 2020, 2021, and 2022 from the Originating Files. The sample dataset is a simple random sample of 50,000 loans selected from each full vintage year and a proportionate number of loans from each partial vintage year of the Standard Dataset. Included in the file is the User Guide of this data set, which serves as our Data Dictionary. You can access it in Github or through this link:

Data Dictionary: https://cis4400pp.blob.core.windows.net/datadictionaries/data_dictionary.pdf

The links for the Raw Data Sets in .txt format are attached below: 

sample origination data for the Year 2020: https://cis4400pp.blob.core.windows.net/rawdata/sample_orig_2020.txt

sample origination data for the Year 2021: https://cis4400pp.blob.core.windows.net/rawdata/sample_orig_2021.txt

sample origination data for the Year 2022: https://cis4400pp.blob.core.windows.net/rawdata/sample_orig_2022.txt

Below are the Python scripts that downloaded the .txt files which were uploaded to Microsoft Azure Blob Storage, merged them into one Excel file for better viewing and analyzing, and uploaded it back to Azure Blob Storage. 

```python
pip install pandas openpyxl azure-storage-blob
import pandas as pd
from io import StringIO
import os
from azure.storage.blob import BlobServiceClient, BlobClient, ContainerClient

# Connect to Azure Storage Account
connection_string = "DefaultEndpointsProtocol=https;AccountName=cis4400pp;AccountKey=v5P+piRvmQQjp+PYMyBTR6oV8JSjJIYxUZCpEnDrTdsKqduDPFGOAhN/vLo2r8H3ye873+A2IG7K+AStmNDHsQ==;EndpointSuffix=core.windows.net"
blob_service_client = BlobServiceClient.from_connection_string(connection_string)

# Container and Blob Names
container_name = "rawdata"
output_blob_name = "combined_data.xlsx"

# Define constants
input_folder = 'C:/Users/User/Desktop/School/Baruch Stuffs/FALL 2023/CIS 4400/raw_data'
output_excel_path = 'C:/Users/User/Desktop/School/Baruch Stuffs/FALL 2023/CIS 4400/combined_data/combined_data.xlsx'

common_headers = ['CREDIT_SCORE',
                  'FIRST_PAYMENT_DATE',
                  'FIRST_TIME_HOMEBUYER_FLAG',
                  'MATURITY_DATE',
                  'MSA',
                  'MORTGAGE_INSURANCE_PERCENTAGE',
                  'NUMBER_OF_UNITS',
                  'OCCUPANCY_STATUS',
                  'ORIGINAL_CLTV',
                  'ORIGINAL_DTI_RATIO',
                  'ORIGINAL_UPB',
                  'ORIGINAL_LTV',
                  'ORIGINAL_INTEREST_RATE',
                  'CHANNEL',
                  'PPM_FLAG',
                  'AMORTIZATION_TYPE',
                  'PROPERTY_STATE',
                  'PROPERTY_TYPE',
                  'POSTAL_CODE',
                  'LOAN_SEQUENCE_NUMBER',
                  'LOAN_PURPOSE',
                  'ORIGINAL_LOAN_TERM',
                  'NUMBER_OF_BORROWERS',
                  'SELLER_NAME',
                  'SERVICER_NAME',
                  'SUPER_CONFORMING_FLAG',
                  'PRE-RELIEF_REFINANCE_LOAN_SEQUENCE_NUMBER',
                  'PROGRAM_INDICATOR',
                  'RELIEF_REFINANCE_INDICATOR',
                  'PROPERTY_VALUATION_METHOD',
                  'INTEREST_ONLY_INDICATOR',
                  'MI_CANCELLATION_INDICATOR']

# Initialize an empty DataFrame
combined_data = pd.DataFrame()

# Iterate through text files
for filename in os.listdir(input_folder):
    if filename.endswith(".txt"):
        file_path = os.path.join(input_folder, filename)

        # Read the text content into a DataFrame
        df = pd.read_csv(file_path, sep='|', header=None)
        
        # Add headers to the DataFrame
        df.columns = common_headers

        # Cleanse and preprocess the data as needed
        df = df.drop_duplicates()

        # Fill in 0 for empty values
        df = df.fillna(0)
        
        # Append the data to the combined DataFrame
        combined_data = combined_data.append(df, ignore_index=True)

# Convert the combined DataFrame to Excel format
combined_data.to_excel(output_excel_path, index=False, engine='openpyxl')

# Upload the Excel file back to Blob Storage
with open(output_excel_path, "rb") as data:
    blob_client = blob_service_client.get_blob_client(container=container_name, blob=output_blob_name)
    blob_client.upload_blob(data, overwrite=True)


