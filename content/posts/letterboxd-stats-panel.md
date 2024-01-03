---
layout: post
date: 2024-01-02
author: "Yohannes"
title: "Panel for Letterboxd"
weight: 1
tags: [
    "python",
    "scraper",
    "data-scraping",
    "panel",
    "dashboard",
    "data-visualization"
]
---

I made a Python Panel dashboard for Letterboxd. This dashboard resembles the stats page included in the Pro subscription.

The visualizations are interactive. When you hover over the bars, a popup displays more information (such as the counts, etc.). If I find the time, I will host it on Heroku or somewhere else. But for now, if you would like to run it on your own (with your own scraped data), check out the app code [here](https://github.com/ydkahin/misc-scripts/blob/main/panel_dashboard_app_for_letterboxd.py). In the same repository, you can also find the Letterboxd scraper that I talked about in the last post. This script generates a CSV that you can feed into the dashboard app to generate something like this:

![](https://i.imgur.com/V1vTFdu.png)

