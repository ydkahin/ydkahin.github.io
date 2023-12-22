---
title: "Projects"
layout: "projects"
url: "/projects"
summary: "Projects"
---
# A Letterboxd Scraper (and Recommendation Model)
The popular social movie platform, Letterboxd, does not have a public API, so I decided to write a Python scraper for it using BeautifulSoup. The write-up is available [here](https://ydkahin.github.io/posts/letterboxd-scraper-i/). This scraper is optimized to use Python's concurrent processing (to speed up the webscraping). 

Using part of the data, I also used Panel (a Python library) to prepare a dashboard. The user would input their username and the web-app would return a dashboard consisting of visualizations of their Letterboxd data. 

This is part of my ongoing personal project to make a recommendation model based on aggregated social data (from movies followers/following have logged) collected using the scraper. 


# Quora Views Prediction Challenge
A while back, I participated in a challenge set out by Quora. The task was to predict the number of views a question would get based on historical (uncleaned) data that Quora provided. This ended up being an exploratory data analysis project due to the lack of clear challenge goals. However I did end up implementing Grid Search and hyperparameter tuning to improve the prediction of the model, after setting up linear regression and forest-tree models. The full write-up is presented in 3 parts on my blog. The notebooks are available [here](http://localhost:1313/posts/quora-challenge-last/) 

# A Discord Bot
Using Discord.py (a Python library), I made a Discord chatting bot for a friend and hosted it on an EC2 instance in AWS. The code is available [here](https://github.com/ydkahin/snapTen). You can find the write-up/tutorial for writing a Discord bot and hosting it on the [blog](https://ydkahin.github.io/posts/discord-bot-tut-ii/). 
