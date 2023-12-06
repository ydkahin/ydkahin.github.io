---
layout: post
date: 2023-12-06
author: "Yohannes"
title: "(How to write) A Letterboxd Scraper"
weight: 1
---

In this post, I will make a [Letterboxd](https://letterboxd.com) scrapper. In case you didn't know, Letterboxd is a film logging site with built-in social networking features. It is a great platform, but the company does not provide a public API for their data, so one has to use a scaper to get their hands on the film/user data. 

For this scraper, given a username, we want all the films the user has logged. Then, we will want to collect the
* runtime
* director(s)
* country
* languages
* genre(s)
* year
* user's rating

of each film.

We will also utilize Python's **concurrent processing** to significantly reduce the time it takes to scrap our data by optimizing how we send our requests. Without it, scraping is frustratingly slow even when scraping users with fewer than 200 entries in their logs. For the HTML parsing and scraping, we will use the Python package [BeautifulSoup](https://pypi.org/project/beautifulsoup4/). 

You can find the Jupyter notebook for this post [here](https://github.com/ydkahin/jupyter-notebooks/blob/d370fa7e8d7a0800eabe6691d41b197f9a9c9e75/letterboxd_scraper_to_blog.ipynb).


```python
import pandas as pd
import re
import requests
from bs4 import BeautifulSoup
import numpy as np

#from multiprocessing import Pool
import concurrent.futures
```


```python
def getNumPages(username):
    baseurl = 'https://letterboxd.com/{}/films'.format(username)
    r = requests.get(baseurl)
    sp = BeautifulSoup(r.text, 'html.parser')
    try:
        page = int(sp.select("li.paginate-page")[-1].text)
    except:
        page = int() # for those users whose logged films span just one page
    return page
```

Given a link that contains a wall of films (such as the paged user `/films/` pages), this function collects all the links of the films on that page.


```python
def the_filmlinks(url):#
    r = requests.get(url)
    sp = BeautifulSoup(r.text, 'html.parser')
    lbxbaseurl = "https://letterboxd.com/"
    return [
        lbxbaseurl + thing.get("data-target-link") for thing in sp.select(".really-lazy-load")
    ]
```

This function collects all the links of films from each page:


```python
def getAllLinks(username): #
    pages = getNumPages(username)
    baseurl = "https://letterboxd.com/{}/films/page/".format(username)
    #links = [] This is slower, so use ThreadPoolExecutor below
    #for page in range(pages+1):
    #    for item in get_film_links(baseurl+str(page)):
    #      links.append(item)
    
    pagelinks = [baseurl+str(i) for i in range(pages+1)]
    with concurrent.futures.ThreadPoolExecutor() as executor:
        futures = []
        for pagelink in pagelinks:
            futures.append(executor.submit(the_filmlinks, url=pagelink))
        links = [future.result() for future in concurrent.futures.as_completed(futures)]
    
    return [link for llink in links for link in llink]
```

The function below is pretty much independent from the rest as it is just regex and BeautifulSoup selector code.


```python
def the_details(url): #independent of the rest, lots of BeautifulSoup code went into this
    r = requests.get(url)
    sp = BeautifulSoup(r.text, 'html.parser')
    ratingblob = sp.select("head > meta:nth-child(20)")[0]
    
    if ratingblob.get("content").split()[0] == 'Letterboxd':
        rating_c = np.nan
    else:
        rating_c = float(ratingblob.get("content").split()[0])
        
    tmdbblob = sp.find('a', attrs={'data-track-action': 'TMDb'})
    directors = [name.text for name in sp.select("span.prettify")]
    
    res = re.search(r'\/movie\/(\d+)\/', tmdbblob.get("href")) # This grabs the TMDB
    #link; entries that aren't movies do not have a TMDB link, so we give them id 0
    if res:
        id = sp.find(class_="really-lazy-load").get("data-film-id")
    else:
        id = 0

    ### Stubs    
    genrestub = sp.select('a[href^="/films/genre/"]')

    try:
        countrystub = sp.select('a[href^="/films/country/"]')[0] #/films/country/usa/
        country = re.search(r"/country/(\w+)/", countrystub.get("href")).group(1)
    except:
        country = 0

    try:
        languagestub = sp.select('a[href^="/films/language/"]')
        langs = {languagestub[i].text for i in range(len(languagestub))} 
        #use set because original language, spoken languages repetition 
    except:
        langs = 0
        

    
    film ={
        'film_id': int(id), #will be used to exclude tv shows
        'film_title': sp.select_one("h1.headline-1").text,
        'film_year': int(sp.select_one("small.number").text),
        'director': [name.text for name in sp.select("span.prettify")],
        'average_rating': rating_c,
        'runtime': int(re.search(r'\d+', sp.select_one("p.text-link").text).group()),
        'country': country,
        'genres': [genrestub[i].text for i in range(len(genrestub))],
        'languages': langs #sp.select('a[href^="/films/language/"]')[0].text
        #'actors': []
    }
    return film
```

Let us test it out with this film:


```python
the_details('https://letterboxd.com/film/knives-out-2019/')
```




    {'film_id_tv': 475370,
     'film_title': 'Knives Out',
     'film_year': 2019,
     'director': ['Rian Johnson'],
     'average_rating': 4.01,
     'runtime': 131,
     'country': 'usa',
     'genres': ['Mystery', 'Comedy', 'Crime'],
     'languages': {'English', 'Spanish'}}


Great!

---

Now, we collect the film details from all of the films a given user has logged. This excludes the user's rating, which will be scraped later (see below). The function is also optimized by using `ThreadPoolExecutor`.  This is important since there is a lot of wasted downtime in between scraping and sending requests.


```python
def getLoggedFilmDetails(username): #non-user related details
    #film_details = []
    urls = getAllLinks(username)
    with concurrent.futures.ThreadPoolExecutor() as executor:
        futures = []
        for url in urls:
            futures.append(executor.submit(the_details,url))
        results = [future.result()
                   for future in concurrent.futures.as_completed(futures)]
    return results
```


```python
pd.DataFrame(getLoggedFilmDetails('indiewire'))
```





<div class="j_table">
<table border="0.5" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>film_id</th>
      <th>film_title</th>
      <th>film_year</th>
      <th>director</th>
      <th>average_rating</th>
      <th>runtime</th>
      <th>country</th>
      <th>genres</th>
      <th>languages</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>228594</td>
      <td>Blonde</td>
      <td>2022</td>
      <td>[Andrew Dominik]</td>
      <td>2.04</td>
      <td>167</td>
      <td>usa</td>
      <td>[Drama]</td>
      <td>{Italian, English}</td>
    </tr>
    <tr>
      <th>1</th>
      <td>801082</td>
      <td>A Man Named Scott</td>
      <td>2021</td>
      <td>[Robert Alexander]</td>
      <td>3.83</td>
      <td>95</td>
      <td>usa</td>
      <td>[Music, Documentary]</td>
      <td>{English}</td>
    </tr>
    <tr>
      <th>2</th>
      <td>560787</td>
      <td>Spider-Man: No Way Home</td>
      <td>2021</td>
      <td>[Jon Watts]</td>
      <td>3.86</td>
      <td>148</td>
      <td>usa</td>
      <td>[Action, Adventure, Science Fiction]</td>
      <td>{Tagalog, English}</td>
    </tr>
    <tr>
      <th>3</th>
      <td>519052</td>
      <td>The Tragedy of Macbeth</td>
      <td>2021</td>
      <td>[Joel Coen]</td>
      <td>3.77</td>
      <td>105</td>
      <td>usa</td>
      <td>[War, Drama]</td>
      <td>{English}</td>
    </tr>
    <tr>
      <th>4</th>
      <td>565654</td>
      <td>The Addams Family 2</td>
      <td>2021</td>
      <td>[Conrad Vernon, Greg Tiernan]</td>
      <td>2.28</td>
      <td>93</td>
      <td>canada</td>
      <td>[Animation, Fantasy, Horror, Family, Comedy, A...</td>
      <td>{Spanish, Latin, Ukrainian, English}</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>205</th>
      <td>457180</td>
      <td>Raya and the Last Dragon</td>
      <td>2021</td>
      <td>[Don Hall, Carlos López Estrada]</td>
      <td>3.30</td>
      <td>107</td>
      <td>usa</td>
      <td>[Family, Action, Animation, Fantasy, Adventure]</td>
      <td>{English}</td>
    </tr>
    <tr>
      <th>206</th>
      <td>713525</td>
      <td>I'm Your Man</td>
      <td>2021</td>
      <td>[Maria Schrader]</td>
      <td>3.55</td>
      <td>108</td>
      <td>germany</td>
      <td>[Comedy, Science Fiction, Romance]</td>
      <td>{German, Korean, Spanish, French, English}</td>
    </tr>
    <tr>
      <th>207</th>
      <td>508523</td>
      <td>Crisis</td>
      <td>2021</td>
      <td>[Nicholas Jarecki]</td>
      <td>2.73</td>
      <td>118</td>
      <td>belgium</td>
      <td>[Thriller, Drama, Crime]</td>
      <td>{German, English}</td>
    </tr>
    <tr>
      <th>208</th>
      <td>473304</td>
      <td>Cherry</td>
      <td>2021</td>
      <td>[Anthony Russo, Joe Russo]</td>
      <td>2.81</td>
      <td>140</td>
      <td>usa</td>
      <td>[Crime, Drama]</td>
      <td>{English}</td>
    </tr>
    <tr>
      <th>209</th>
      <td>579476</td>
      <td>Ninjababy</td>
      <td>2021</td>
      <td>[Yngvild Sve Flikke]</td>
      <td>3.81</td>
      <td>104</td>
      <td>norway</td>
      <td>[Comedy, Drama]</td>
      <td>{Norwegian}</td>
    </tr>
  </tbody>
</table>
<p>210 rows × 9 columns</p>
</div>



---

Now, we collect user ratings


```python
def getRatings(username):
    pages = getNumPages(username)
    baseurl = "https://letterboxd.com/{}/films/page/".format(username)
    rateid = []
    stars = {
        "★": 1, "★★": 2, "★★★": 3, "★★★★": 4, "★★★★★": 5, "½": 0.5, "★½": 1.5, "★★½": 2.5, 
        "★★★½": 3.5, "★★★★½": 4.5
      }

    for page in range(pages+1):
        film_p = baseurl+str(page)
        soup_p = BeautifulSoup(requests.get(film_p).text,'html.parser')
        for thing in soup_p.find_all('li', class_="poster-container"):
            try:
                userrating=stars[thing.find(class_="rating").get_text().strip()]
            except:
                userrating=np.nan
            
            filmp = {
                'film_id':int(thing.find(class_="really-lazy-load").get("data-film-id")),
                'user_rating': userrating
            }
            rateid.append(filmp)
  
    return rateid
```

Try it out with a user:
```python
pd.DataFrame(getRatings('indiewire'))
```




<div class="j_table">
<table border="0.5" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>film_id</th>
      <th>user_rating</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>228594</td>
      <td>2.5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>905069</td>
      <td>3.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>666269</td>
      <td>3.5</td>
    </tr>
    <tr>
      <th>3</th>
      <td>385511</td>
      <td>3.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>777185</td>
      <td>4.5</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>205</th>
      <td>399633</td>
      <td>1.5</td>
    </tr>
    <tr>
      <th>206</th>
      <td>468597</td>
      <td>1.5</td>
    </tr>
    <tr>
      <th>207</th>
      <td>448164</td>
      <td>3.0</td>
    </tr>
    <tr>
      <th>208</th>
      <td>381286</td>
      <td>3.0</td>
    </tr>
    <tr>
      <th>209</th>
      <td>11370</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>210 rows × 2 columns</p>
</div>



This will be useful for the recommendation engine part of my project, which is part of the reason I decided to write a scraper in the first place. Other than that I thought it would be a great way to hone my Python skills and revive this blog :) 

I hope you found this write-up/code useful!
