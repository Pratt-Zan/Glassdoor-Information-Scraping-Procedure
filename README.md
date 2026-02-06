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

There should be 6 .ipynb files, which 5 of them (except Data_Full_Filter) is for the running procedure to generate the final outcome, and the exception is to generate the data from the initial dataset。 Since we have already generated the filtered_companies_full_Transfer.csv dataset, we will not execute this code here. The excecution order can be seen in notice above, and we will also make guideline blow using step notice as titles.

Please be awared that the empty file is not showed in this link here since the Github restriction, but it should be established for the running environment(please add these files after downloading the full project). The full structure should be found in the Onedrive link.

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

In this step we will use the urls generated in the last step to find the source in the overview page. By the help of Beautifulsoup library, we can store the page source and locate the text we want by the following codes.

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

In this section we shall here use the former json to done the formatting to change the info-x to the specified item. They includes "website" "" "location" "employees" "type" "revenue" "industry", here we can find that only "location" and "industry" shall be hard to position, while else can be found based on the key strings they have, such as ".com" "www." for website, and "employee" "Type:" " Revenue:" for "employees" "type" "revenue". To loacate the remaining, we found some other ways.
 
For "location" we find that it only appear in the first 3 info lines, and also contain a comma，and for "industry" we found it must appear in the last 2 info line. After we filtered out the keys for those in other line, the remaining line in the last 2 position must be the line for "industry". Then after the formatting, the final outcome will still be stored in a json file, and it can be final result for our need. The results will be stored in the Page_Scrape_Formatting_xxx folder accordingly.

-----------------------------------------------------------

## The last tip

In the running code, I found that the total time for the running should at least be 500+ (300+ for step 1 and 200+ for step 3) hours. So the running should be seperate for several parts in different ends. Also, the idea is done in Windows for sure since we need the powershell to run the debugger browser in step 3. And lastly, I do recommend running 1000 company one time at one end, and store the json name as urls-1.x.json / urls.2.x.json to avoid any traceback that made the progame to kill running. Lastly, the running is based on the HK web environment, if you do need to run these codes on your end, please use HK IP or use VPN to ensure stable and clear running.

🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥
🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥🟥

# 🚀 Glassdoor 信息抓取流程
本文档概述了从Glassdoor抓取公司概览数据的自动化工作流程。该项目采用多步骤流程来规避反抓取措施，确保两个主要数据集（EP和Transfer）的数据准确性。

注意：完整数据集因大小限制未托管在GitHub上。如需复现本研究，请通过Dropbox联系作者获取访问权限。

--------------------------

## 📁 项目结构

项目根据执行阶段按模块化目录组织：

```
my-project/
├── 📂 Basic Data/         # 基础数据集            
│   ├── 📂 glassdoor-ep-revirew/                     
│   ├── 📂 Transfer/                       
├── 📂 Page-Source-Scraping/         # 获取URL后的代码和存储        
│   ├── 📂 Page_Scrape_Check_ep/                     
│   ├── 📂 Page_Scrape_Check_Trans/
│   ├── 📂 Page_Scrape_Formatting_ep/                     
│   ├── 📂 Page_Scrape_Formatting_Trans/
│   ├── 📂 Page_Url_Formatting_ep/                     
│   ├── 📂 Page_Url_Formatting_Trans/
│   ├── 📜 Page_Scrape_Check.ipynb/           # 步骤3 - 使用URL抓取信息
│   ├── 📜 Page_Scrape_Formatting.ipynb/      # 步骤4 - 将信息格式化为所需结构
│   └── 📜 Page-Url-Formatting.ipynb/         # 步骤2 - 将URL格式化为概览页面
├── 📂 Url-Scrap-Method-1-Tansfer/                          
│   └── 📜 scrap_url_Method_1_full.ipynb/     # Transfer数据的步骤1 - 按公司名称抓取URL
├── 📂 Url-Scrap-Method-2-Glassdoor-ep/                          
│   └── 📜 scrap_url_Method_2_full.ipynb/     # EP数据的步骤1 - 按公司名称抓取URL
├── 📜 filtered_companies_full_Transfer.csv      
├── 📜 Data_Full_Filter.ipynb          
└── 📜 README.md                              # GitHub上的文字指引文件
```

从上述的结构可以看出，主要运行逻辑分为不同的Python文件，并且此处我们处理了处理两个数据集。Method1和Method2与Basic Data位置中给出的两个数据集分别对应相关。请注意，完整数据未在GitHub中提供，如果您需要复现代码，请通过Dropbox联系作者下载。

此处总共应该有6个.ipynb文件，其中5个（除了Data_Full_Filter）是用于生成最终结果的运行过程，而Data_Full_Filter是从初始数据集生成数据，由于我们已经生成好了filtered_companies_full_Transfer.csv数据故这里我们不会执行这个代码。执行顺序可以在上面的注释中看到，我们还将使用步骤说明作为标题在下方提供指南。

请注意，空文件在此处未显示，因为GitHub限制，但应为运行环境建立完整的架构（需要下载后手动添加）。完整结构应在OneDrive链接中找到。

--------------------------

## 🛠 环境设置

### 先决条件

- Python版本：3.11.5（优化了Jupyter内核兼容性）
- 操作系统：Windows（步骤3远程调试器功能必需）
- IDE：带有Jupyter扩展的Visual Studio Code

在本节中，我们将介绍运行代码时可能需要的环境设置。
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

请检查您的Python环境中是否安装了所有列出的库，以确保顺利运行。如果没有，请在Chrome中搜索并使用"pip install"下载。
``` bash
pip install pandas tqdm beautifulsoup4 selenium undetected-chromedriver webdriver-manager
```
另请注意，所有测试代码均在Visual Studio Code中使用Jupyter Notebook内核完成，Python版本为3.11.5（请检查版本以便Jupyter支持，现在通常应为3.10+）。

-----------------------------------------------------------

## 步骤1说明

本节及以下内容将是每个步骤及其注意事项的说明，从步骤1开始。

在步骤1的代码中，关于不同数据集的代码共享相同的抓取逻辑。我们将使用undetected chrome驱动基于Bing搜索引擎生成搜索，因为这不会像Chrome那样遇到封锁。我们将下载第一页的页面源代码，以查看是否存在基于"glassdoor"的合适URL。更具体地说，我们将优先查找包含"Overview"的URL，然后才是"Reviews"。这是为了尽可能多地获取URL，因为我们发现Review页面可以在特定规则下转换为Overview页面，这将在步骤2中看到。

此外，有一点需要注意：Bing搜索的结果有时是加密的，因此需要以下Python代码进行判断并解密。然而，这种情况并不总是发生，但确实存在，因此在运行代码时，判断URL是否包含"bing"在其行中是至关重要的，这意味着此处的代码代码是否需要更正。另外，由于每次运行搜索的结果可能不同，我们在此添加了重试系统以全面检查URL的获取能力。重试次数设置为5次，太小会失去效果，而太大会降低代码效率，导致运行时间过长。

另请注意，此处上传的代码仅使用10行作为测试。如果您需要具有特定切片的大样本，请依靠代码中查找注释进行选择。我们建议每天每次运行1000行，这样既不会太长，又能充分利用时间。需要注意的是，单次更多样本可能在导致运行时遇到问题时造成更大的时间浪费。

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

此步骤的最终输出将是存储在所属文件夹中的JSON文件（您也可以通过更改最后步骤中的位置来创建自己的存储），Json中会包含包含公司名称和相应的URL。

-----------------------------------------------------------

## 步骤2说明

在此步骤中，我们处理将属于评论页面的URL转换为概览页面URL的代码。我们发现URL共享一个基本结构，并且评论页面和概览页面之间存在高度相关性，只需更改后缀并添加几个词即可实现转换。此代码不难，结果也将以JSON格式存储并保存在文件夹Page_Url_Formatting_xxx中。请注意，为了便于检查，JSON的名称应相同，但只是存储在不同位置以显示它们之间的差异。不需要为每个JSON取长名称，存储区别已通过它们的父文件夹实现。

-----------------------------------------------------------

## 步骤3说明

在此步骤中，我们将使用上一步生成的URL查找概览页面的源代码。借助Beautifulsoup库，我们可以存储页面源数据并通过以下代码定位所需的文本。

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

很明显，我们只是向前面的JSON添加了几行信息。由于不同公司的信息不同，我们添加的数量是不确定的。

除此之外，连接到Glassdoor需要特殊方式。使用代码时，请阅读代码中的markdown部分，按照以下说明操作：

``` 
"1. close all Chrome drivers"
"2. Open (Win+R，输入cmd)，copy and paste to run the command:"
"C:\\Program Files\\Google\\Chrome\\Application\\chrome.exe" --remote-debugging-port=9222 --user-data-dir="C:\\chrome_temp"
Then use the connection function to connect the code, and Please login Glassdoor by your account first before the scraping starts.
```

必须完成此步骤之后继续运行代码（这也是我们选用Juypter Notebook来运行而不是直接Python源代码的原因），以确保在Glassdoor网站上不会发生封锁。运行代码后，我们可以看到JSON数据存储在Page_Scrape_Check_xxx文件夹中。

-----------------------------------------------------------

## 步骤4说明

在本节中，我们将使用之前的JSON完成格式化，将info-x更改为指定项。它们包括"website" "" "location" "employees" "type" "revenue" "industry"，这里我们可以发现只有"location"和"industry"难以定位，而其他项可以根据它们具有的关键字符串找到，例如".com" "www." 用于"website"，"employee" "Type:" " Revenue:"用于"employees" "type" "revenue"。为了定位剩余的项，我们找到了其他方法。

对于"location"，我们发现它只出现在前3个信息行中，并且包含逗号；对于"industry"，我们发现它必须出现在最后2个信息行中。过滤掉其他行中的这些项后，最后2个位置中剩余的行必是"industry"行。格式化后，最终结果仍将存储在JSON文件中，这可以作为我们需要的最终结果。结果会相应的存储在Page_Scrape_Formatting_xxx文件夹之中。

-----------------------------------------------------------

## 最后提示

在运行代码时，我发现总运行时间至少应为500+小时（步骤1 300+小时，步骤3 200+小时）。因此，应在不同的终端上分成几个部分运行。此外，此方法必须在Windows上完成，因为我们需要在步骤3中使用PowerShell运行调试器浏览器。最后，我建议每次在每个终端上运行1000家公司，并将JSON名称存储为urls-1.x.json / urls.2.x.json，以避免任何导致程序终止运行的导致的数据失效。最后，测试运行是基于香港网络环境的，如果您需要在您的终端上运行这些代码，请使用香港IP或使用VPN以确保稳定和清晰的运行。

PRATT ZAN, 2026 Feb 06





