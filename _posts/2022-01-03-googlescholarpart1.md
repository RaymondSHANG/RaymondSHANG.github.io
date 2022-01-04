---
layout: post
title: "”google scholar part1”"
subtitle: "”using selenium”"
date: 2022-01-03 21:40:28
header-style: text
catalog: true
author: "Yuan"
tags: [selenium,chromedriver,CAPTCHApython]
---

>A year's plan starts with spring

During the Christmas holidays, I rewrote my google scholar analytic piplines. Though the current version is still tough, it returns the basic information we want to analyze someone's google scholar profiles.
## Repository and Usage
The current version is released at [github](https://github.com/RaymondSHANG/googleScholar) at version 1.0. Anyone is welcome to star, fork, and download it.
Implementation is simple:
1. Download all files into onefolder, go to that folder
2. creat a "data" folder
3. creat a "parameters.txt" in the data folder. In parameters.txt, you need to specify 3 parameters: the googleProfileID, pubmed email, and pubmed api_key. An example of parameters.txt is:
    ```
    {"gid":"APooktAAAAAJ"
    "pubemail":"xxx@exxxx.edu"
    "api_key":"xxxxxxapikeyxxxxxx"}
    ```
4. run getcitations_1.py to getcitations_9.py. Mannual inspect all author matchings, make corrections in "selfcitations" value, and save as "xxxxx_modify.csv"
5. run getcitations_10.py to get a summary results for each paper.

## Packages used
This post records some of the techniques used during the coding. I used selenium to pass google CAPTCHA checks, used pymed and e-utilities to search results from pubmed databases.

1. overcome scrolldown and "show more.." clicks in google scholar profile
    ```python
    from selenium.webdriver.support import expected_conditions as EC

    driver = webdriver.Chrome("/usr/local/bin/chromedriver")
    # &cstart=100&pagesize=100
    driver.get(
        f'https://scholar.google.com/citations?hl=en&user={gid}&hl=en&cstart=0&pagesize=100')

    try:
        # Scroll down to bottom
        driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")

        # Wait to load page
        time.sleep(20)
        # Wait up to 10s until the element is loaded on the page
        element = WebDriverWait(driver, 10).until(
            # Locate element by id
            EC.presence_of_element_located((By.ID, 'gsc_bpf_more'))
        )
    finally:
        element.click()

    html_source = driver.page_source
    soup = BeautifulSoup(html_source, 'html.parser')
    ```

2. google CAPTCHA checks
    ```python
    driver.get(url)
    while True:
        try:
            recap = driver.find_element_by_css_selector(
                    '#gs_captcha_ccl,#recaptcha')
        except NoSuchElementException:
            try:
                htmlpage = driver.page_source #for beautifulsoup
                break
            except NoSuchElementException:
                print("google has blocked this browser, reopening")
                driver.close()
                driver = webdriver.Chrome()
                driver.get(url)

        print("... it's CAPTCHA time!\a ...")
        time.sleep(5)
    ```

---
