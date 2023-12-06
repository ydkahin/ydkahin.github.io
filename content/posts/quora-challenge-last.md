+++
title = "Views Prediction - A Quora Challenge - Final (EDA, Feature Engineering, and More)"
description = ""
tags = [
    "challenge",
    "quora",
    "eda"
]
date = "2017-09-06"
categories = [
    "Challenges"
]
+++

I have found out that it takes a lot of time porting big Jupyter notebooks to a markdown document; sure, Jupyter can do that for you, but tweaking the markdown document so that codes and tables match the style of your blog takes an unecessary amount of time. It takes even more time to explore the data and derive insights from visualizations. Now, this is a necessary task. That's why for the remaining journey of this Quora Challenge, I will be posting cleaned IPython notebooks along with a short summary of what I did in there. 

## Exploratory Data Analysis and Feature Engineering:


[`https://nbviewer.org/github/ydkahin/jupyter-notebooks/blob/master/notebooks/quora-views-challenge/quora_views_challenge-partiii-EDA_and_feature_engineering.ipynb`](https://nbviewer.org/github/ydkahin/jupyter-notebooks/blob/master/notebooks/quora-views-challenge/quora_views_challenge-partiii-EDA_and_feature_engineering.ipynb)

Summary:

* Started using [*plotnine*](http://plotnine.readthedocs.io/). Plotnine is an implementation of a grammar of graphics in Python, it is based on ggplot2.
* Made insightful plots and practiced data visualization with plotnine.
* Made a few functions to calculate correlation between custom columns and `__ans__` including one that calculates the correlation betweeen `__ans__` and a boolean column of question texts including a combination of words
* Created two new features.

## More Feature Engineering using Keywords:

[`https://nbviewer.org/github/ydkahin/jupyter-notebooks/blob/master/notebooks/quora-views-challenge/quora_views_challenge_partiii-more_text_analysis.ipynb`](https://nbviewer.org/github/ydkahin/jupyter-notebooks/blob/master/notebooks/quora-views-challenge/quora_views_challenge_partiii-more_text_analysis.ipynb)

Summary:

* Continuation of the previous notebook. A few more plots and analysis. 
* Explored combination of words using the function in the above notebook.

## Regression with the New Features:

[`https://nbviewer.org/github/ydkahin/jupyter-notebooks/blob/master/notebooks/quora-views-challenge/quora_views_challenge-partiii-Regression_with_the_new_features.ipynb=force_flush`](https://nbviewer.org/github/ydkahin/jupyter-notebooks/blob/master/notebooks/quora-views-challenge/quora_views_challenge-partiii-Regression_with_the_new_features.ipynb=force_flush)

Summary: 

* Added the progress from all the previous notebooks as features to the training datafram. 
* Applied Robust scaling to the training dataframe.
* Compared the correlation matrix (i.e., th emanheatmaps) of the features with target to the correlation matrix of the log of the features.
* With the new features in place, trained **linear, decision tree,** and **random forest tree regression models.**
* To improve the random forest model, applied GridSearchCV.
* Fine tuned the hyperparameters of the Random Forest Regressor.


# Conclusion:

This project more than anything has been a learning experience where I put the machine learning, data visualization, and cleaning lessons in practice. The results from the models weren't impressive. But the lack of challenge goals (set by Quora) also made it hard to track how well my efforts were to reaching an acceptable model. Therefore, I will put the project to halt here. 
