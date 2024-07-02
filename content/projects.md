---
title: "Projects"
layout: "projects"
url: "/projects"
summary: "Projects"
---
# Probability of Default Model

# NBA Playoffs Data Exploration and Monte-Carlo


# A Letterboxd Scraper (and Recommendation Model)
The popular social movie platform Letterboxd does not have a public API, so I decided to write a Python scraper for it using BeautifulSoup. The write-up and code can be found [here](https://ydkahin.github.io/posts/letterboxd-scraper-i/). This scraper is optimized to use Python's concurrent processing to speed up the webscraping. 

Using the scraper and the data it collects in real-time, I made a Panel (a Python library) dashboard for visualizing Letterboxd data. A user would input their username and the web-app would return the dashboard with charts and diagrams consisting of user’s most watched directors, genres, movies, etc. (The app is not hosted at the moment, but you can find the scripts for scraping and generating a dashboard [here](https://github.com/ydkahin/misc-scripts)).

<!---This is part of my ongoing personal project to make a recommendation model based on aggregated social data (from movies followers/following have logged) collected using the scraper. -->

The idea for the future is to make a recommendation model based on aggregated social data (made up from followers/following network) collected using the scraper. 


# Quora Views Prediction Challenge
A while back, I participated in a challenge set out by Quora. The task was to predict the number of views a question would get based on historical (uncleaned) data that Quora provided. 

This ended up being an exploratory data analysis project due to the lack of clear challenge goals. However, after setting up linear regression and forest-tree models, I did end up implementing Grid Search and hyperparameter tuning to improve the predictions of the models. The full write-up is presented in 3 parts on my blog. The notebooks are available [here](http://localhost:1313/posts/quora-challenge-last/).

# A Discord Bot
Using Discord.py (a Python library), I made a Discord chatbot for a friend and hosted it on an EC2 instance on AWS. The code is available [here](https://github.com/ydkahin/snapTen). See [this](https://ydkahin.github.io/posts/discord-bot-tut-ii/) for a tutorial on writing a Discord bot and hosting it.
