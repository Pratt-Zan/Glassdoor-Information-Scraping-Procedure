# Glassdoor-Information-Scraping-Procedure
This is the code and data structure for other ends to done the overview scraping, please read the instruction carefully before you actually need to done the procedure.

--------------------------

The files shall follow the listed order

## 📁 File Structure

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
└── 📜 README.md 
```

We can learn from the structure that the main running logic shall be seperate into different files, and there are 2 dataset for you to deal with. The Method 1 and Method 2 are correlated with 2 dataset given in the Basic Data loaction. Please be aware that full data is not given in the github, and if you need to replicate the code, please contact the writer to download throgh Dropbox.

There should be 6 .ipynb files, which 5 of them (except Data_Full_Filter) is for the running procedure to generate the final outcome, and the exception is to generate the data from the initial dataset. The excecution order can be seen in notice above.

--------------------------

In this section we will introduce the environment settings you may need when running the code.
```
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

Please check if you have all listed libraries in your Python environment installed, for the clear running. If not, please search in Chrome and use "pip install" to download. This procedure is quite easy and then I will leave it.
Please also be aware that all the test code is done in Visual Studio Code, and with Jupyter Notebook Kernal, and Python version is 3.11.5 (Please check the version so that Jupyter can support).

-----------------------------------------------------------












