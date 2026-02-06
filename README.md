# 🚀 Glassdoor Information Scraping Procedure
This documentation outlines the automated workflow for scraping company overview data from Glassdoor. This project utilizes a multi-step process to bypass anti-scraping measures and ensure data accuracy across two primary datasets (EP and Transfer).

Note: The full datasets are not hosted on GitHub due to size constraints. To replicate this study, please contact the author via Dropbox for access.

--------------------------

## 📁 Project Structure

The project is organized into modular directories based on the execution stage:

```
my-project/
├── 📂 Basic Data/         # Basic Datasets           
│   ├── 📂 glassdoor-ep-revirew/                     
│   ├── 📂 Transfer/                       
├── 📂 Page-Source-Scraping/         # Codes and Storage after geting the Urls       
│   ├── 📂 Page_Scrape_Check_ep/                     
│   ├── 📂 Page_Scrape_Check_Trans/
│   ├── 📂 Page_Scrape_Formatting_ep/                     
│   ├── 📂 Page_Scrape_Formatting_Trans/
│   ├── 📂 Page_Url_Formatting_ep/                     
│   ├── 📂 Page_Url_Formatting_Trans/
│   ├── 📜 Page_Scrape_Check.ipynb/           # Step 3 - Use the url to scrap information
│   ├── 📜 Page_Scrape_Formatting.ipynb/      # Step 4 - Format the information to the structure we need 
│   └── 📜 Page-Url-Formatting.ipynb/         # Step 2 - Format the Url to overview page 
├── 📂 Url-Scrap-Method-1-Tansfer/                          
│   └── 📜 scrap_url_Method_1_full.ipynb/     # Step 1 for data in Transfer - Scrap Urls by company names
├── 📂 Url-Scrap-Method-2-Glassdoor-ep/                          
│   └── 📜 scrap_url_Method_2_full.ipynb/     # Step 1 for data in  EP - Scrap Urls by company names
├── 📜 filtered_companies_full_Transfer.csv      
├── 📜 Data_Full_Filter.ipynb          
└── 📜 README.md                              # Unique file in Github
```

We can learn from the structure that the main running logic shall be seperate into different files, and there are 2 dataset for you to deal with. The Method 1 and Method 2 are correlated with 2 dataset given in the Basic Data loaction. Please be aware that full data is not given in the github, and if you need to replicate the code, please contact the writer to download throgh Dropbox.

There should be 6 .ipynb files, which 5 of them (except Data_Full_Filter) is for the running procedure to generate the final outcome, and the exception is to generate the data from the initial dataset. The excecution order can be seen in notice above, and we will also make guideline blow using step notice as titles.

Please be awared that the empty file is not showed in this link here since the Github restriction, but it should be established for the running environmenr. The full structure should be found in the Onedrive link.

--------------------------

## 🛠 Environment Setup

### Prerequisites
- Python Version: 3.11.5 (Optimized for Jupyter Kernel compatibility).
- OS: Windows (Required for Step 3 remote debugger functionality).
- IDE: Visual Studio Code with Jupyter extension.

In this section we will introduce the environment settings you may need when running the code.
``` Python
import json
import time
import re
import os
import base64
import warnings
import glob
from urllib.parse import urlparse, parse_qs, urllib.parse

import pandas as pd
from tqdm import tqdm
from bs4 import BeautifulSoup

# Selenium-related
import undetected_chromedriver as uc
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

# WebDriver-manage
from webdriver_manager.chrome import ChromeDriverManager

# filter-warnings
warnings.filterwarnings('ignore', category=pd.errors.DtypeWarning)
```

Please check if you have all listed libraries in your Python environment installed, for the clear running. If not, please search in Chrome and use "pip install" to download. 
``` bash
pip install pandas tqdm beautifulsoup4 selenium undetected-chromedriver webdriver-manager
```
Please also be aware that all the test code is done in Visual Studio Code, and with Jupyter Notebook Kernal, and Python version is 3.11.5 (Please check the version so that Jupyter can support, normally now should be 3.10+).

-----------------------------------------------------------

## Step 1 Notice

This section and following shall be the explanation for each step and their notices, start from step 1.

In the code of step 1, both codes regarding different datasets share the same scraping logic. We will use the undetected chrome driver to generate the searching base on the Bing engine, since this will not meet the block as Chrome does. We will downlaod the page source in the first page to see whether there is a suitable url base on the "glassdoor". To be more specific, we will find the url containing "Overview" as the first choice and then "Reviews" as the second. This is to try getting urls as much as we can, since we have found that the revierw page can be transfered to the overview page under certain rules, which will be seen in step 2.

Besides there is one thing needing attention, which is that the result in the Bing search is encrypted, so the following python code for search is in need. However, it is not always happening but do exist, so when learning the code, the second judgement of whether url has "bing" in its line is crucial, which means that the code should not be changed. Also, since the result may change every time running, here we added a retry system to fully check the ability for the urls. The retry time is set to be 5, and too small will lose its effectiveness, while to large will reduce the efficiency of the code, which will make the running time to long.

Please also be aware that the code uploaded here only use 10 rows as a test. If you are in need for a larger sample with certain slice, please find the notice in the code to do the selection. We recommend to use 1000 rows per time per day on an end, which will be not too long but can make full use of the time. More samples in a single time may cause larger time waste if meeting problems in running.

``` Python
u_param = parse_qs(parsed_url.query).get('u', [None])[0]
if not u_param:
    continue
try:
    b64_str = u_param[2:]
    padding = len(b64_str) % 4
    if padding:
        b64_str += "=" * (4 - padding)
    decoded_bytes = base64.urlsafe_b64decode(b64_str)
    decoded_url = decoded_bytes.decode('utf-8')
```

The final output in this step will be a json file stored in the belonging folder(you can also make you own storage by changing the location in the last step), containing the company name and the corresponding url.

-----------------------------------------------------------

## Step 2 Notice

In this step, we deal with the code to transform the url belongs to review page to overview page. We have found that the url share a basic structure and a high correlation between the review page and the ioverview page, by just chaging the suffix and adding several words can we make the transformation clear. This code is not difficult, and the result will also be in json style and be stored in the folder: Page_Url_Formatting_xxx. Here by notice, the name of jsons should be the same in order to check, but just stored in different places to show the difference between them. There is no need to make long names for each json which has been achieved by their parent folders.

-----------------------------------------------------------

## Step 3 Notice

In this step we will use the urls generated in the last step to find the source in the overview page. By the help of Beautifulsoup library, we can store the page source and locate the text we want by the following codes

``` Python
pagesource = driver.page_source
soup = BeautifulSoup(pagesource, 'html.parser')
details_list = soup.find('ul', attrs={'data-test': 'companyDetails'})
if details_list:
    for j, li in enumerate(details_list.find_all('li'), 1):
        text = li.get_text(strip=True)
        if text:
            item[f'info-{j}'] = text
```

It is clear that we just add few information lines to the former json. Since the information for different company varies, the numnber we add is not certain. 

Besides this, the connection to the glassdoor is needing a special way. When using the code, please read the markdone part in the code to follow the instrcution as below:

``` 
"1. close all Chrome drivers"
"2. Open (Win+R，输入cmd)，copy and paste to run the command:"
"C:\\Program Files\\Google\\Chrome\\Application\\chrome.exe" --remote-debugging-port=9222 --user-data-dir="C:\\chrome_temp"
Then use the connection function to connect the code, and Please login Glassdoor by your account first before the scraping starts.
```

This step must be done to ensure the block won't happen in Glassdoor website. After running the code, we can see that the json data is stored in the Page_Scrape_Check_xxx folders.

-----------------------------------------------------------

## Step 4 Notice

In this section we shall here use the former json to done the formatting to change the info-x to the specified item. They includes "website" "" "location" "employees" "type" "revenue" "industry", here we can find that only "location" and "industry" shall be hard to position, while else can be found based on the key strings they have, such as "com." "www." for website, and "employee" "Type:" " Revenue:" for "employees" "type" "revenue". To loacate the remaining, we found some other ways.
 
For "location" we find that it only appear in the first 3 info lines, and also contain a comma，and for "industry" we found it must appear in the last 2 info line. After we filtered out the keys for those in other line, the remaining line in the last 2 position must be the line for "industry". Then after the formatting, the final outcome will still be stored in a json file, and it can be final result for our need.

-----------------------------------------------------------

## The last tip

In the running code, I found that the total time for the running should at least be 500+ (300+ for step 1 and 200+ for step 3) hours. So the running should be seperate for several parts in different ends. Also, the idea is done in Windows for sure since we need the powershell to run the debugger browser in step 3. And lastly, I do recommend running 1000 company one time at one end, and store the json name as urls-1.x.json / urls.2.x.json to avoid any traceback that made the progame to kill running. Lastly, the running is based on the HK web environment, if you do need to run these codes on your end, please use HK IP or use VPN to ensure stable and clear running.

PRATT ZAN, 2026 Feb 06

🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥



