---
layout: post
title: "Python Google Scholar"
subtitle: "Web Scraping with Selenium"
date: 2021-09-07 14:52:50
header-style: text
catalog: true
author: "Yuan"
tags: 
    - Google Scholar
    - Web Scaping
    - Python
    - selenium
    - BeautifulSoup
    - argparse
---

> Life is hard, and ignorance makes it harder

I was trying to analyze my google citations. After testing several github repositories, I found two of them are helpful for my task.

One is [gscholar-citations-crawler](https://github.com/thu-pacman/gscholar-citations-crawler), the code is simple, including all necessary elements for a basic webpage scraping task, including parameters input (argparse), BeautifulSoup html parsing, and result output. However, it will fail after several rounds of running because of the "CAPTCHA" problem. Also, the original code failed catching citation ids and thus failed to save them into bib file.

#### Config argparse
```python
import argparse

parser = argparse.ArgumentParser(description='To retrieve all of your citations from Google Scholar.')
parser.add_argument('google_scholar_uri', type=str, help='Your google scholar homepage')
parser.add_argument('--request-interval', action='store', type=int, default=100,
                    help='Interval (in seconds) between requests to google scholar')
parser.add_argument('--should-download', action='store_const', const=True, default=False,
                    help='Download PS/PDF files of all citations iff True')
parser.add_argument('--citation-name', action='store', default="citation.bib",
                    help='File name for all your citations in BibTex format')
opts = parser.parse_args()

#using the input parameters
time.sleep(opts.request_interval)
soup = create_soup_by_url(opts.google_scholar_uri)

```

Usage: 
```bash
python *.py -h
python *.py 'url' --should-download False
```

Another repository called [**etudier**](https://github.com/edsu/etudier) bypass this blocking problem through selenium. Also, it saved all citation info into json format for post processing. Additionaly, it could output a 'ciation network' which is helpful for deep digging of your citations. For my own task, I need to set the search depth to "0", so only first level of citations were including. To use etudier, I need to extract all my publications into a list, and use each element of that list as an input to etudier. My final code for this task is posted in my github(https://github.com/RaymondSHANG?tab=repositories).

#### Config selenium
Before using selenium, download [chromedriver](https://chromedriver.chromium.org/). You may need to download the same version to your chrome browser.

Then, make chromedriver executable(chmod +x chromedriver), put the executable file to your system_path, or you need to specify the path in webdriver.Chrome().
An example key code from [etudier.py](https://github.com/edsu/etudier/blob/master/etudier.py)
```python
from selenium import webdriver
from selenium.common.exceptions import NoSuchElementException
import requests_html

#driver = webdriver.Chrome(executable_path=r"C:\path\to\chromedriver.exe")
driver = webdriver.Chrome()

def get_html(url):
    """
    get_html uses selenium to drive a browser to fetch a URL, and return a
    requests_html.HTML object for it.
    
    If there is a captcha challenge it will alert the user and wait until 
    it has been completed.
    """
    global driver

    time.sleep(random.randint(1,5))
    driver.get(url)
    while True:
        try:
            recap = driver.find_element_by_css_selector('#gs_captcha_ccl,#recaptcha')
        except NoSuchElementException:

            try:
                html = driver.find_element_by_css_selector('#gs_top').get_attribute('innerHTML')
                return requests_html.HTML(html=html)
            except NoSuchElementException:
                print("google has blocked this browser, reopening")
                driver.close()
                driver = webdriver.Chrome()
                return get_html(url)

        print("... it's CAPTCHA time!\a ...")
        time.sleep(5)
driver.close()
```

#### Other useful references:
1. Dimitry Zub wrote an awesome blog about '[Scrape Google Scholar with Python](https://dev.to/dimitryzub/scrape-google-scholar-with-python-32oh)', which has dicussed almost all bugs/problems I met.
https://dev.to/dimitryzub/scrape-google-scholar-with-python-32oh

2. [**SerpApi**](https://serpapi.com/) is another cool tool to bypass the blocking problems. We could register an account to get an api_key for free. It's fast and coding is efficient. The only thing is probably you need to learn how to use SerApi.
https://serpapi.com/


---
