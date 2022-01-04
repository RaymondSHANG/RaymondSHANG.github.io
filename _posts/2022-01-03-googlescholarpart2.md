---
layout: post
title: "”google scholar part2”"
subtitle: "”using pymed”"
date: 2022-01-03 22:49:46
header-style: text
catalog: true
author: "Yuan"
tags: [pubmed,pymed,e-utilities,python,json]
---
> I have nothing to say

## Continue

Most author information from google scholar are not complete. To overcome this problem, and get a more precise author overlap analysis. I searched each paper in pubmed and pmc using the titles.

Pymed package is easy to use: it applies query through e-utilities. By default, it will do all analysis through database "pubmed", we need to change this parameter each time if you tried another database such as "pmc".

Also, based on my result, query results from e-utilities api are not as acurate as pubmed web search. So I further searched the papers if no good matches were found by e-utilities api.

If only one results were found by websearch, we have to extract pubmed_id in the returning webpage using format=abstract. Otherwise, we could simply get all pubmed_id results by adding a parameter (format=pmid) when request html.

3. pubmed search using pymed:
    ```python
    articleList = []
    # Firstly, seach pubmed database
    pubmed.parameters["db"] = "pubmed"
    results = pubmed.query(search_term, max_results=20)
    for article in results:
        articleDict = article.toDict()
        if articleDict not in articleList:
            articleList.append(articleDict)
    # Search PMC database for a second time. I found PMC and pubmed some times returns different results
    pubmed.parameters["db"] = "pmc"
    results2 = pubmed._getArticleIds(query=search_term, max_results=10)
    if len(list(results2)) > 0:
        pubmed.parameters["db"] = "pubmed"
        results3 = pubmed._getArticles(article_ids=results2)
        for result in results3:
            articleDict = result.toDict()
            print(articleDict['title'])  # title_pubmed = article['title']
            if articleDict not in articleList:
                articleList.append(articleDict)
    ```
4. Pubmed search using request:
   ```python
    articleList = []
    response2 = requests.get(
        f"https://pubmed.ncbi.nlm.nih.gov/?format=abstract&term={search_term}")
    if re.search('<span class="single-result-redirect-message">', response2.text, re.MULTILINE):
        pubid_search = re.search(
                '<meta name="citation_pmid" content="(.*)">', response2.text)
        pubid = pubid_search.groups()[0]
            # print(pubid)
        results = pubmed._getArticles(article_ids=pubid)
        for article in results:
            articleList.append(article.toDict())

    else:
        response1 = requests.get(
            f"https://pubmed.ncbi.nlm.nih.gov/?format=pmid&term={search_term}")
        soup = BeautifulSoup(response1.text, "html.parser")
        aa = soup.find_all('pre')
        if aa == []:
            #No matches found!
            continue
        pubid = aa[0].contents[0]
        results = pubmed._getArticles(article_ids=pubid)
        for article in results:
            articleList.append(article.toDict())
   ```
   5. pandas.read_json
   When using pd.read_json to read a json file, it will do some automatical converting large numbers to int. In this case, some mistakes may happen. To avoid this, add d_type={} parameter to pd.read_json() function

    ```python
    data = pd.read_json(jsonfile,orient='records', lines=True, dtype={})
    #pandas groupby and aggregate
    df2 = df2.groupby('id_target')['selfcitations'].agg(
    selfcitations='sum', total='count').reset_index()
    ```
---
