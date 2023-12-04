+++
title = "Views Prediction - A Quora Challenge - Part II (Regression)"
description = ""
tags = [
    "challenge",
    "quora",
    "regression",
    "machine-learning"
]
date = "2017-08-30"
categories = [
    "Challenges",
    "Machine-Learning"
]
draft = true
+++

## Recap

In the last post, we imported the data provided from Quora. It was split 9:1 into training and test data (unlabled with `__ans__` missing). But we will also need to split our data even further. I am talking about using part of the labeled training data to check the accuracy of our model.

In this post, I will do regression models and evaluate their accuracy. I am new to regression; as a result, this post won't be focused on the Quora challenge, rather, it is an exposition to help me better understand regression models.

Let's start by importing the data:

<!-- ### Linear Regression

Before we do linear regression, we will tweak the `train` data. First, we will enumerate the `anonymous` column. This describes whether the question was anonymous. Second, we will create a column with the sum of followers of the topics in the json blobs of the `topics` column. Finally, we will split the `train` dataframe in 8:2 to `train_df` and `test_df`. --> 


```python
import pandas as pd
import json
json_data = open('../views/sample/input00.in') # change to where your `input00.in` file is

data = []
for line in json_data:
    data.append(json.loads(line))

data.remove(9000)
data.remove(1000)

df = pd.DataFrame(data)
data_df = df[:9000]
```

<div class="j">
<div class="j_pre">
    <p>data_df</p>
    </div>
<div class="j_table">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>__ans__</th>
      <th>anonymous</th>
      <th>context_topic</th>
      <th>num_answers</th>
      <th>promoted_to</th>
      <th>question_key</th>
      <th>question_text</th>
      <th>topics</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2.089474</td>
      <td>False</td>
      <td>{'followers': 500022, 'name': 'Movies'}</td>
      <td>4</td>
      <td>0</td>
      <td>AAEAAM9EY6LIJsEFvYiwKLfCe7d+hkbsXJ5qM7aSwTqemERp</td>
      <td>What are some movies with mind boggling twists...</td>
      <td>[{'followers': 500022, 'name': 'Movies'}]</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2.692308</td>
      <td>False</td>
      <td>{'followers': 179, 'name': 'International Math...</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAHM6f92B9jt43/y913/J7ce8vtE6Jn9LLcy3yK2RHFGD</td>
      <td>How do you prepare a 10 year old for Internati...</td>
      <td>[{'followers': 179, 'name': 'International Mat...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>4.612903</td>
      <td>False</td>
      <td>{'followers': 614223, 'name': 'Science'}</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAGzietPvCHLFvaKCjng43iiIeo9gAWnJlSrs+12uYtZ0</td>
      <td>Why do bats sleep upside down?</td>
      <td>[{'followers': 614223, 'name': 'Science'}]</td>
    </tr>
    <tr>
      <th>3</th>
      <td>8.051948</td>
      <td>False</td>
      <td>{'followers': 614223, 'name': 'Science'}</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAM2Sk4U3y4We5TELJXRQgIf6yit5DbbdBw6BCRvuFrcY</td>
      <td>Tell me everything about the Leidenfrost effec...</td>
      <td>[{'followers': 614223, 'name': 'Science'}, {'f...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.150943</td>
      <td>False</td>
      <td>{'followers': 1536, 'name': 'Android Tablets'}</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAALb43r/fn9KRKqJ0itd3NGbqZZSZalzi7vaulLxNGzeL</td>
      <td>Is the Nexus 10 any good despite the dual core...</td>
      <td>[{'followers': 1536, 'name': 'Android Tablets'}]</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0.084507</td>
      <td>False</td>
      <td>{'followers': 91, 'name': 'Smartphone Applicat...</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAABSJk8mrfAjuQuxEI6PV2rfGGcnuq/I5JyE0VJvYSoRp</td>
      <td>Is smartphone app download duplication account...</td>
      <td>[{'followers': 91, 'name': 'Smartphone Applica...</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1.073944</td>
      <td>True</td>
      <td>{'followers': 241809, 'name': 'Startups'}</td>
      <td>1</td>
      <td>200</td>
      <td>AAEAANLGb0hKlFULx6BPXvVvHQ1SJ2jJTqDCVighUGs/ZDya</td>
      <td>Are there any CEO's that go by a different nam...</td>
      <td>[{'followers': 526597, 'name': 'Business'}, {'...</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0.007299</td>
      <td>False</td>
      <td>None</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAJinkGKp/YRlvWhFbapFNzPkD7coBuf4QRwsmN/c3q7Z</td>
      <td>Guys who love being sissies?</td>
      <td>[{'followers': 289, 'name': 'Needs to Be Clear...</td>
    </tr>
    <tr>
      <th>8</th>
      <td>0.298893</td>
      <td>True</td>
      <td>{'followers': 3006, 'name': 'Warfare'}</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAB0zSrCTxmdSYxoC1KD0HYeDwTzOyK2lBiXzuZwhhmgn</td>
      <td>How does one successfully become a conscientio...</td>
      <td>[{'followers': 59, 'name': 'Pacifism'}, {'foll...</td>
    </tr>
    <tr>
      <th>9</th>
      <td>30.614035</td>
      <td>False</td>
      <td>{'followers': 224, 'name': 'Boston Marathon Te...</td>
      <td>3</td>
      <td>0</td>
      <td>AAEAAA7uRr9KzVD3rd+L3OptfwxaWFYfE6D8Wqskxx+ZkfGb</td>
      <td>How do people in countries with regular and la...</td>
      <td>[{'followers': 224, 'name': 'Boston Marathon T...</td>
    </tr>
    <tr>
      <th>10</th>
      <td>24.250000</td>
      <td>False</td>
      <td>{'followers': 4155, 'name': 'Organization'}</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAFHJ+5Ozj1TQKLUwzB/pIwiXMwtTJip8NQO1jCCyRS5a</td>
      <td>How do top entrepreneurs/CEOs stay organized a...</td>
      <td>[{'followers': 321001, 'name': 'Entrepreneursh...</td>
    </tr>
    <tr>
      <th>11</th>
      <td>1.666667</td>
      <td>False</td>
      <td>{'followers': 304, 'name': 'Web Development on...</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAM3WIWO/Ct/u5SvzPHfwwrHNWrPgHd8Ch7/QdM6CjfZL</td>
      <td>What companies/independent developers located ...</td>
      <td>[{'followers': 199, 'name': 'Technology Indust...</td>
    </tr>
    <tr>
      <th>12</th>
      <td>0.006231</td>
      <td>False</td>
      <td>{'followers': 3292, 'name': 'United States Gov...</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAI0XKapeO3+B75zZYu1Kjv5SGn+Mm/4qT3rBNJrDNKce</td>
      <td>How can I call and talk with President Obama o...</td>
      <td>[{'followers': 1240, 'name': 'U.S. Presidents'}]</td>
    </tr>
    <tr>
      <th>13</th>
      <td>0.575972</td>
      <td>True</td>
      <td>{'followers': 13867, 'name': 'Parenting'}</td>
      <td>1</td>
      <td>100</td>
      <td>AAEAAMRnMKIzVIfH55p+aKXak1M0vBd2m7xNKm5KttYyVspo</td>
      <td>What do parents wish they had known before hav...</td>
      <td>[{'followers': 26, 'name': 'Deciding Whether t...</td>
    </tr>
    <tr>
      <th>14</th>
      <td>5.220859</td>
      <td>False</td>
      <td>{'followers': 72820, 'name': 'Biology'}</td>
      <td>2</td>
      <td>0</td>
      <td>AAEAAFsd5v511xAQgtEaPDHb+fYF8YSJnsRuMd7715LVQQPf</td>
      <td>How much time does it take to go from DNA to p...</td>
      <td>[{'followers': 3693, 'name': 'Medical Research...</td>
    </tr>
    <tr>
      <th>15</th>
      <td>0.079755</td>
      <td>False</td>
      <td>{'followers': 2049, 'name': 'Tablet Devices an...</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAOnLQc1KPDw0pqCMpROkdovdRCzJlTsI5nDo9vxjcOm+</td>
      <td>What % of tablet users use a stylus?</td>
      <td>[{'followers': 2049, 'name': 'Tablet Devices a...</td>
    </tr>
    <tr>
      <th>16</th>
      <td>1.075758</td>
      <td>False</td>
      <td>{'followers': 999, 'name': 'Luxury'}</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAI0A/7ASkze8LKdTLRwCFtUcFZ2xIpNnS55Zk1xtCwXN</td>
      <td>Which company make awesome luxury coach?</td>
      <td>[{'followers': 999, 'name': 'Luxury'}, {'follo...</td>
    </tr>
    <tr>
      <th>17</th>
      <td>0.717557</td>
      <td>False</td>
      <td>{'followers': 39744, 'name': 'Web Development'}</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAKKuKEgb+lBKi1iEEST89zLKFr/88CUAwC9xeCdGYss1</td>
      <td>I Require  professional and creative web desig...</td>
      <td>[{'followers': 39744, 'name': 'Web Development'}]</td>
    </tr>
    <tr>
      <th>18</th>
      <td>1.622010</td>
      <td>False</td>
      <td>{'followers': 3566, 'name': 'Online Video'}</td>
      <td>2</td>
      <td>0</td>
      <td>AAEAAJloaH19a32hbe1Cs0wdW8WWptHbbaa5JG7dKbMwvtaJ</td>
      <td>What are the short/medium prospects of service...</td>
      <td>[{'followers': 11842, 'name': 'Internet Advert...</td>
    </tr>
    <tr>
      <th>19</th>
      <td>3.388889</td>
      <td>False</td>
      <td>{'followers': 268430, 'name': 'Television Seri...</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAFHbVuHDWmLslFWH8UZMnN7M5VHntFVDpY19+3ndhmy5</td>
      <td>Why can't we create something like game of thr...</td>
      <td>[{'followers': 268430, 'name': 'Television Ser...</td>
    </tr>
    <tr>
      <th>20</th>
      <td>11.457143</td>
      <td>False</td>
      <td>{'followers': 74, 'name': 'Noodle Education'}</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAKkkEpxZp4qBn45yEhzGTnJNhB5e/Mppn75kx1on5ajr</td>
      <td>Would Noodle make other education lead generat...</td>
      <td>[{'followers': 167176, 'name': 'Business Model...</td>
    </tr>
    <tr>
      <th>21</th>
      <td>1.013889</td>
      <td>False</td>
      <td>{'followers': 490, 'name': 'Online Reputation'}</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAABu6kZw48xJyS1xXW3RjQ0xANWgzwjp6LLVTryDWUM8I</td>
      <td>ORM is all too often seen simply as minimising...</td>
      <td>[{'followers': 490, 'name': 'Online Reputation...</td>
    </tr>
    <tr>
      <th>22</th>
      <td>0.389474</td>
      <td>True</td>
      <td>{'followers': 264984, 'name': 'Dating and Rela...</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAANJFMVE7CEr6gX3LEC9NWGz0a65TjFd1AjHMY9CWujBJ</td>
      <td>What are some of the funniest noises you have ...</td>
      <td>[{'followers': 67265, 'name': 'Sex'}, {'follow...</td>
    </tr>
    <tr>
      <th>23</th>
      <td>0.319231</td>
      <td>False</td>
      <td>{'followers': 6608, 'name': 'Science and Relig...</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAKTTk5EUSD+0CPGP0gLS0aBQR97xe0p2+DPRX+ddtoV2</td>
      <td>Was Moses talking about humans becoming fully ...</td>
      <td>[{'followers': 2525, 'name': 'Theology'}, {'fo...</td>
    </tr>
    <tr>
      <th>24</th>
      <td>4.000000</td>
      <td>False</td>
      <td>{'followers': 1707, 'name': 'Democracy'}</td>
      <td>4</td>
      <td>200</td>
      <td>AAEAAFJxWj/4n1NhOgwpS0d5X7tcBia4jBtjrfw6YxNcYXDr</td>
      <td>'Scarcely a human freedom has been obtained wi...</td>
      <td>[{'followers': 4332, 'name': 'Occupy Movement'...</td>
    </tr>
    <tr>
      <th>25</th>
      <td>0.174194</td>
      <td>False</td>
      <td>{'followers': 2450, 'name': 'Wanting and Makin...</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAH+k9PIQg6lOxHRIqXYE5u18aWO4QqJZTtdahJw9+grv</td>
      <td>Do actors get paid after they do the movie or ...</td>
      <td>[{'followers': 2450, 'name': 'Wanting and Maki...</td>
    </tr>
    <tr>
      <th>26</th>
      <td>0.678322</td>
      <td>True</td>
      <td>{'followers': 3567, 'name': 'Private Equity'}</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAG1dK7AL02j78ugkPaKf5TiFb6iBCaIICXxUd5OYpQYz</td>
      <td>Private Equity in relation to Movie Industry,c...</td>
      <td>[{'followers': 3567, 'name': 'Private Equity'}...</td>
    </tr>
    <tr>
      <th>27</th>
      <td>1.007246</td>
      <td>False</td>
      <td>{'followers': 238905, 'name': 'Facebook'}</td>
      <td>2</td>
      <td>120</td>
      <td>AAEAAA15xfSTbyLayedu+P4P6upyw1gZisruGcd5sBeOiO2w</td>
      <td>What language was used by Mark Zuckerberg to w...</td>
      <td>[{'followers': 238905, 'name': 'Facebook'}]</td>
    </tr>
    <tr>
      <th>28</th>
      <td>46.333333</td>
      <td>False</td>
      <td>{'followers': 238905, 'name': 'Facebook'}</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAALJZ+JSLst8wPq+EApQqSTqcCW9iaHg7OWcdwg+xJEVo</td>
      <td>How employees at Facebook avoid procrastinatio...</td>
      <td>[{'followers': 238905, 'name': 'Facebook'}]</td>
    </tr>
    <tr>
      <th>29</th>
      <td>0.615385</td>
      <td>True</td>
      <td>{'followers': 986, 'name': 'Glenn Gould'}</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAETPeE1zK+aIF5IeOLu2EzMwzSM22EM4i14ZnwDTU17t</td>
      <td>Was Glenn Gould a fast typist?</td>
      <td>[{'followers': 986, 'name': 'Glenn Gould'}]</td>
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
    </tr>
    <tr>
      <th>8970</th>
      <td>0.044610</td>
      <td>False</td>
      <td>{'followers': 3899, 'name': 'Quora (company)'}</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAKy7JJbuCln18hVXHhNCqaW/2t4BOJekq4EoJZeZlQAf</td>
      <td>When was the Quora application launched for An...</td>
      <td>[{'followers': 3899, 'name': 'Quora (company)'}]</td>
    </tr>
    <tr>
      <th>8971</th>
      <td>0.028571</td>
      <td>False</td>
      <td>{'followers': 53, 'name': 'The Flame'}</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAIPL+PIfMKP6q/d3vCLPeN6qaE5wVTv9jyLyP46riITO</td>
      <td>1A group element to show flame test why?</td>
      <td>[{'followers': 13, 'name': 'Flame Effects'}, {...</td>
    </tr>
    <tr>
      <th>8972</th>
      <td>0.356364</td>
      <td>False</td>
      <td>{'followers': 39744, 'name': 'Web Development'}</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAFbZen2AwK+cWhTuxYq89OYYGUdvPKxObTA49ZZY5EoF</td>
      <td>Hire best webdevelopers to design and maintain...</td>
      <td>[{'followers': 39744, 'name': 'Web Development'}]</td>
    </tr>
    <tr>
      <th>8973</th>
      <td>101.000000</td>
      <td>True</td>
      <td>{'followers': 264984, 'name': 'Dating and Rela...</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAEKenPCvJnNPSn3GoyxJGHOgO9Ft1sXtcQIRtzX36ls7</td>
      <td>Is it absolutely necessary to have a lot in co...</td>
      <td>[{'followers': 264984, 'name': 'Dating and Rel...</td>
    </tr>
    <tr>
      <th>8974</th>
      <td>0.424658</td>
      <td>False</td>
      <td>{'followers': 0, 'name': 'Numeration'}</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAJ6h5w42SdKWHM1ba0mDrLXfRNF35x5J1ImAXc5k7r9g</td>
      <td>What civilization had the earliest written num...</td>
      <td>[{'followers': 1460, 'name': 'Numbers'}, {'fol...</td>
    </tr>
    <tr>
      <th>8975</th>
      <td>1.787879</td>
      <td>True</td>
      <td>{'followers': 7928, 'name': 'Recruiting'}</td>
      <td>2</td>
      <td>0</td>
      <td>AAEAAI0sA6MSlU+uVCoRL9a3wpk5dt0Mnwy/8h0l2jkqoE9K</td>
      <td>I am never contacted by recruiters (I don't ha...</td>
      <td>[{'followers': 7928, 'name': 'Recruiting'}]</td>
    </tr>
    <tr>
      <th>8976</th>
      <td>0.183673</td>
      <td>False</td>
      <td>{'followers': 4729, 'name': 'Software'}</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAHQncZgikc5wl/qwturXJ89iVLVLNrUJNGpMfiSGkZLK</td>
      <td>What is finance software?</td>
      <td>[{'followers': 4729, 'name': 'Software'}]</td>
    </tr>
    <tr>
      <th>8977</th>
      <td>0.118902</td>
      <td>True</td>
      <td>{'followers': 93447, 'name': 'Amazon'}</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAIo1t0EvlXTB3vDMjt4DCsJeG6U43ZF/aD8SiMmlmof0</td>
      <td>What impact do foreign currency fluctuations h...</td>
      <td>[{'followers': 93447, 'name': 'Amazon'}]</td>
    </tr>
    <tr>
      <th>8978</th>
      <td>0.494382</td>
      <td>True</td>
      <td>{'followers': 4859, 'name': 'Sales'}</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAACkrgngYk0Gq2+2Jm/c+yfNG3tFyUrZAUpEsbvfAnuAo</td>
      <td>What are some tips to write a great sales copy?</td>
      <td>[{'followers': 4859, 'name': 'Sales'}, {'follo...</td>
    </tr>
    <tr>
      <th>8979</th>
      <td>2.323699</td>
      <td>True</td>
      <td>{'followers': 13108, 'name': 'Career Advice'}</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAKF/1Zvtgov2mB0Ji4jg1/0aE+lQkKxIEhQuLX8yuwzY</td>
      <td>Is it a good idea to use technical jargon when...</td>
      <td>[{'followers': 13108, 'name': 'Career Advice'}]</td>
    </tr>
    <tr>
      <th>8980</th>
      <td>0.044944</td>
      <td>True</td>
      <td>{'followers': 3, 'name': 'Done Genetics'}</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAN2Q0FsMgTWonNul3FPog92E1XkMDr+1QcPsIivJyv/z</td>
      <td>What's it like to work at Done Genetics?</td>
      <td>[{'followers': 3, 'name': 'Done Genetics'}]</td>
    </tr>
    <tr>
      <th>8981</th>
      <td>2.752941</td>
      <td>True</td>
      <td>{'followers': 20107, 'name': 'Mumbai, Maharash...</td>
      <td>3</td>
      <td>0</td>
      <td>AAEAAPJKiEWYD6YX59MhOtX8+DnzOqzul+SpdL2Xnl0KJoxx</td>
      <td>Where can I connect with people in general in ...</td>
      <td>[{'followers': 6467, 'name': 'Colleges and Uni...</td>
    </tr>
    <tr>
      <th>8982</th>
      <td>0.011662</td>
      <td>True</td>
      <td>{'followers': 3168, 'name': 'Dublin, Ireland'}</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAOZFkLJxzpcxaMgr6sM3pBQcOKWQq+ykds/w3GXXgZqo</td>
      <td>Have anyone any concern or recommendation for ...</td>
      <td>[{'followers': 289, 'name': 'Needs to Be Clear...</td>
    </tr>
    <tr>
      <th>8983</th>
      <td>0.133929</td>
      <td>False</td>
      <td>{'followers': 238905, 'name': 'Facebook'}</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAFxvsqu2YCjgWfq2u+anT+J0OXvtANXSu+elB7CuM1G2</td>
      <td>How can I tell facebook to stop emailing me?</td>
      <td>[{'followers': 238905, 'name': 'Facebook'}]</td>
    </tr>
    <tr>
      <th>8984</th>
      <td>0.207373</td>
      <td>True</td>
      <td>{'followers': 809, 'name': 'Auto Insurance'}</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAHaEN4jlW0hrNd+wj1Xx9bWV2XiE0+5kkBbpk0cS/XCD</td>
      <td>Who does cover hound use to provide auto rate ...</td>
      <td>[{'followers': 809, 'name': 'Auto Insurance'}]</td>
    </tr>
    <tr>
      <th>8985</th>
      <td>0.662722</td>
      <td>True</td>
      <td>None</td>
      <td>2</td>
      <td>0</td>
      <td>AAEAAO10zYF1GIDQenYcsr4KV8T5f53wX0d491daxFrxJfxE</td>
      <td>Is Wuthering Heights a feminist text?</td>
      <td>[{'followers': 590279, 'name': 'Books'}, {'fol...</td>
    </tr>
    <tr>
      <th>8986</th>
      <td>4.123288</td>
      <td>False</td>
      <td>{'followers': 500022, 'name': 'Movies'}</td>
      <td>1</td>
      <td>200</td>
      <td>AAEAAIO1CRxV7oxl//ejIL5mkNUJqShjAlSNiOaXEiQFChq+</td>
      <td>How will better audience data change the movie...</td>
      <td>[{'followers': 1, 'name': 'User Data'}, {'foll...</td>
    </tr>
    <tr>
      <th>8987</th>
      <td>0.352941</td>
      <td>False</td>
      <td>{'followers': 361, 'name': 'Royalty'}</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAOKeIMX2TbFS2CdHXtBWs++0egdI5p4FB2TZTlLyph7D</td>
      <td>In what condition to descendants of royalty live?</td>
      <td>[{'followers': 361, 'name': 'Royalty'}]</td>
    </tr>
    <tr>
      <th>8988</th>
      <td>0.488525</td>
      <td>False</td>
      <td>{'followers': 24985, 'name': 'Innovation'}</td>
      <td>0</td>
      <td>400</td>
      <td>AAEAALlzDzfzi6ULFlDFEbxjfkQTdJRmyDeB7Oz343GX3Zzz</td>
      <td>Will the developing world be more innovative d...</td>
      <td>[{'followers': 24985, 'name': 'Innovation'}, {...</td>
    </tr>
    <tr>
      <th>8989</th>
      <td>6.908078</td>
      <td>False</td>
      <td>{'followers': 782, 'name': 'Samurai'}</td>
      <td>5</td>
      <td>0</td>
      <td>AAEAAJj02/B0zzfVYoV/ujoCNopCXWyQ1l1joancxgjHTOvc</td>
      <td>Are there living samurai in this age?</td>
      <td>[{'followers': 12423, 'name': 'Japan'}, {'foll...</td>
    </tr>
    <tr>
      <th>8990</th>
      <td>2.077295</td>
      <td>False</td>
      <td>{'followers': 20706, 'name': 'Self-Improvement'}</td>
      <td>1</td>
      <td>200</td>
      <td>AAEAAF3/vH5TKSTV79CQ2wWCe7UPuPUQ7GqKZ+zy/WZpyWTS</td>
      <td>How is it like to hire a life coach or persona...</td>
      <td>[{'followers': 11233, 'name': 'Life Advice'}, ...</td>
    </tr>
    <tr>
      <th>8991</th>
      <td>0.490798</td>
      <td>False</td>
      <td>{'followers': 13867, 'name': 'Web Applications'}</td>
      <td>2</td>
      <td>400</td>
      <td>AAEAABeFQkBTvekTH2LwM9QRMeaYrb43rq4KQTB3ixkjIUW6</td>
      <td>What happens to my links if I disconnect my cu...</td>
      <td>[{'followers': 1483, 'name': 'Bitly'}, {'follo...</td>
    </tr>
    <tr>
      <th>8992</th>
      <td>0.197080</td>
      <td>False</td>
      <td>{'followers': 295, 'name': 'Curiosity (Mars Ro...</td>
      <td>2</td>
      <td>0</td>
      <td>AAEAAG1ehKZlxWfZ3u8Ty8xu5l2N+kghEcv62oITGGiqZj62</td>
      <td>Can Curiosity record sound?</td>
      <td>[{'followers': 295, 'name': 'Curiosity (Mars R...</td>
    </tr>
    <tr>
      <th>8993</th>
      <td>3.375000</td>
      <td>False</td>
      <td>{'followers': 6423, 'name': 'Germany'}</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAITna52dCVn2KCVY3+xhMh4dbliiirA/gdi74f+v2zyG</td>
      <td>Is it realistic to think you can get a decent ...</td>
      <td>[{'followers': 6423, 'name': 'Germany'}]</td>
    </tr>
    <tr>
      <th>8994</th>
      <td>0.482143</td>
      <td>False</td>
      <td>{'followers': 0, 'name': 'Syrian War'}</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAB8MgrMHBPrcEd+z8uWtFtE0jU1VhHBx4bSj3dLaMait</td>
      <td>Can someone explain to me how the Syrian war b...</td>
      <td>[{'followers': 854, 'name': 'Syria'}, {'follow...</td>
    </tr>
    <tr>
      <th>8995</th>
      <td>2.850000</td>
      <td>False</td>
      <td>{'followers': 10123, 'name': 'Animals'}</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAANESwEBxHZJt2IIhT1/YgqvPOOHySqU0TPMHvefVX6Rn</td>
      <td>How old are the wolves growly pants and truck ...</td>
      <td>[{'followers': 10123, 'name': 'Animals'}, {'fo...</td>
    </tr>
    <tr>
      <th>8996</th>
      <td>0.274648</td>
      <td>False</td>
      <td>{'followers': 7149, 'name': 'iPad Applications'}</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAABT6OZZisoRkeAFNxYWAssLUu45g4iNjeQiaAEVkPa2Z</td>
      <td>What is the best Agenda (notes, gant charts, C...</td>
      <td>[{'followers': 7149, 'name': 'iPad Application...</td>
    </tr>
    <tr>
      <th>8997</th>
      <td>3.146893</td>
      <td>True</td>
      <td>{'followers': 146, 'name': 'The Hobbit (movie ...</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAAUdDfIgSIYlFRrY4jOKvHsvSLmlKjyZ576oP8a+L694</td>
      <td>December 2012: Are all the Hobbit films alread...</td>
      <td>[{'followers': 18741, 'name': 'The Hobbit (193...</td>
    </tr>
    <tr>
      <th>8998</th>
      <td>0.131086</td>
      <td>False</td>
      <td>{'followers': 7260, 'name': 'Real Estate'}</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAADeVLG2awg6hG+dTiPKRiII64khN7fS/OeFLek4lg2QQ</td>
      <td>Rental Navigator asks...If you are a real esta...</td>
      <td>[{'followers': 7260, 'name': 'Real Estate'}]</td>
    </tr>
    <tr>
      <th>8999</th>
      <td>2.738739</td>
      <td>False</td>
      <td>{'followers': 199773, 'name': 'Physics'}</td>
      <td>2</td>
      <td>0</td>
      <td>AAEAAKeoz9V34X6kt6Iq+ywD6JKTBkJxointHniuxnF2ejHa</td>
      <td>Energy of motion turns into mass?</td>
      <td>[{'followers': 199773, 'name': 'Physics'}]</td>
    </tr>
  </tbody>
</table>
<p>9000 rows × 8 columns</p>
</div>
</div>


## Preparation

We will not deal with columns with text (in the next posts, we will explore these features). That is, `question_text` will be dropped. Moreover, as `context_topic` is [depreciated](https://productupdates.quora.com/Context-Topics-New-Name-and-Guidelines-for-Primary-Topics), this column has no use for now.

```python
data_df.drop(['context_topic', 'question_text'], axis=1, inplace=True)
```

<!-- insert head of data_df -->

The `topics` column has JSON object blobs. The total number of followers of the topics the question shows up under is a meaningful feature. Add it to the data frame as follows:

```python
# JSON blob cleanup for topics column
def funn(x): #where x will be a row when running `apply`
    return sum(x[i]['followers'] for i in range(len(x)))
data_df['topics_followers'] = data_df['topics'].apply(funn)
data_df.drop(['topics'], axis =1, inplace=True)
```

The `anonymous` column has boolean values. Since the machine learning algorithms only take numerical values, the column needs to be relabeled. 

```python
# Turn the boolean valued 'anonymous' column to 0/1
data_df['anonymous'] = data_df['anonymous'].map({False: 0, True:1}).astype(int)
```


<!-- <div class="j">
<div class="j_pre">
    <p>data_df.info()</p>
</div>
<div class="j_out">
<pre style="background-color:#ddd !important">
<code class="hljs scala" style="background-color:#ddd !important">
RangeIndex: 9000 entries, 0 to 8999
Data columns (total 9 columns):
__ans__                    9000 non-null float64
anonymous                  9000 non-null int64
num_answers                9000 non-null int64
promoted_to                9000 non-null int64
question_key               9000 non-null object
question_text              9000 non-null object
context_topic_followers    8731 non-null float64
context_topic_name         8731 non-null object
topics_followers           9000 non-null int64
dtypes: float64(2), int64(4), object(3)
memory usage: 632.9+ KB
</code>
</pre>
</div> -->

<!-- We have missing values in `context_topic_followers` and `context_topic_name`. This can be fixed either using sklearn preprocessing or manually fixed. I will do it manually.


```python
# Replace the number of the missing values by the mean of the column.
ct_fol_mean = data_df['context_topic_followers'].mean()
data_df['context_topic_followers'] = data_df['context_topic_followers'].fillna(ct_fol_mean)
```


```python
data_df_original = data_df.copy()
```
 -->

Finally, set`question_key` as the index and check that it's unique.

```python
data_df.set_index(data_df['question_key'], 
                  inplace=True,
                  verify_integrity=True
                 )
data_df.drop(['question_key'], axis=1, inplace=True)
data_df.index.name = None 
```
 
 Since the new index has a name ("question_key"), pandas will add an extra row with this index name in the first entry and 0 in the rest. Fix this by setting the index name to `None`.

 <div class= "j">
<div class="j_pre">
    <p> data_df.head()</p>
</div>
<div class="j_table">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>__ans__</th>
      <th>anonymous</th>
      <th>num_answers</th>
      <th>promoted_to</th>
      <th>topics_followers</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>AAEAAM9EY6LIJsEFvYiwKLfCe7d+hkbsXJ5qM7aSwTqemERp</th>
      <td>2.089474</td>
      <td>0</td>
      <td>4</td>
      <td>0</td>
      <td>500022</td>
    </tr>
    <tr>
      <th>AAEAAHM6f92B9jt43/y913/J7ce8vtE6Jn9LLcy3yK2RHFGD</th>
      <td>2.692308</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>179</td>
    </tr>
    <tr>
      <th>AAEAAGzietPvCHLFvaKCjng43iiIeo9gAWnJlSrs+12uYtZ0</th>
      <td>4.612903</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>614223</td>
    </tr>
    <tr>
      <th>AAEAAM2Sk4U3y4We5TELJXRQgIf6yit5DbbdBw6BCRvuFrcY</th>
      <td>8.051948</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1029363</td>
    </tr>
    <tr>
      <th>AAEAALb43r/fn9KRKqJ0itd3NGbqZZSZalzi7vaulLxNGzeL</th>
      <td>0.150943</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1536</td>
    </tr>
  </tbody>
</table>
</div>
</div>



## Preprocessing

Before choosing which preprocessing algorithm to use, the existence of outliers in our data must be checked. Normally, one would use `MaxMinScaler` or `StandardScaler` from `sklearn.preprocessing`. But in a data with outliers, a better option is `RobustScaler`. The rationale is discussed below.

### Do we have outliers?

To gain a perspective, run:

<div class="j">
<div class="j_pre">
    <p>data_df.describe()</p>
    </div>
<div class="j_table">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>__ans__</th>
      <th>anonymous</th>
      <th>num_answers</th>
      <th>promoted_to</th>
      <th>topics_followers</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>9000.000000</td>
      <td>9000.000000</td>
      <td>9000.000000</td>
      <td>9000.000000</td>
      <td>9.000000e+03</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>7.753822</td>
      <td>0.297556</td>
      <td>1.849222</td>
      <td>49.137778</td>
      <td>1.757266e+05</td>
    </tr>
    <tr>
      <th>std</th>
      <td>37.803602</td>
      <td>0.457208</td>
      <td>5.837850</td>
      <td>187.168118</td>
      <td>2.634619e+05</td>
    </tr>
    <tr>
      <th>min</th>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000e+00</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>0.457780</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>6.621000e+03</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>1.328110</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>4.784200e+04</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>4.019578</td>
      <td>1.000000</td>
      <td>2.000000</td>
      <td>0.000000</td>
      <td>2.649840e+05</td>
    </tr>
    <tr>
      <th>max</th>
      <td>1569.000000</td>
      <td>1.000000</td>
      <td>311.000000</td>
      <td>5600.000000</td>
      <td>2.838840e+06</td>
    </tr>
  </tbody>
</table>
</div>
</div>



This hints that there are outliers in almost all of our features, except `anonymous` (and leaving out `__ans__` as it's often not required to scale the target). For more insight, let's plot some boxplots. 


```python
# boxplots
import matplotlib.pyplot as plt
import seaborn as sns

sns.set_style("whitegrid")

plt.subplot(1, 3, 1)#i.e., this is the first (the last 1) in a 1 row plot of 2 columns.
plt.boxplot(data_df.num_answers, 1, 'gd')
plt.title('num_answers')
plt.ylabel('counts')

plt.subplot(1, 3, 2) 
plt.boxplot(data_df.promoted_to, 1, 'gd')
plt.title('promoted_to')
plt.ylabel('counts')

plt.subplot(1, 3, 3) 
plt.boxplot(data_df.topics_followers, 1, 'gd')
plt.title('topics_followers')
plt.ylabel('counts')


plt.tight_layout()
plt.show()
```


![png](https://res.cloudinary.com/dlqivdu1d/image/upload/v1504115335/output_18_0_uogacy.png)

There are a few data points outside the IQR in each of the boxplots (in the first two, the boxes are the dark lines at the bottom). Certainly, these columns have outliers.

Because we have outliers, the sklearn [`RobustScaler`](http://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.RobustScaler.html#sklearn.preprocessing.RobustScaler) will be used. 

>Standardization of a dataset is a common requirement for many machine learning estimators. Typically this is done by removing the mean and scaling to unit variance. However, outliers can often influence the sample mean / variance in a negative way. **In such cases, the median and the interquartile range often give better results.**

For a good measure, we will use a quartile range of `(5.0, 95.0)` which is 2 standard deviations (see [*three-sigma-rule*](https://en.wikipedia.org/wiki/68%E2%80%9395%E2%80%9399.7_rule)).


### Splitting

If preprocessing is done on the whole data before splitting the data to training and test sets, "information from the test set will 'leak' into your training data." (see [this](https:seetats.stackexchange.com/a/270299) stats.SE answer.)

Now, not to confuse with the 9:1 split that was discussed in the previous post, this split is being done on the first 9000 data points. The other 1000, of the 10,000, are unlabled; so, we can only use the 9000 to train and check the validity of our models.

We will split `data_df` 8:2, train to test. 

```python
# Split data_df to train and test 
from sklearn.model_selection import train_test_split
y = data_df.__ans__ # the target column
x = data_df.drop('__ans__', axis=1)
x_train, x_test, y_train, y_test = train_test_split(x, y, 
                                                   test_size = 0.2,
                                                   random_state = 123)
```


<div class="j">
<div class="j_pre">
    <p>pd.DataFrame(x_train).head()</p>
<div class="j_table">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>anonymous</th>
      <th>num_answers</th>
      <th>promoted_to</th>
      <th>topics_followers</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>AAEAALxMqeRHdD7LRQLR7RZfcSXRNvMoqdZQ1GAKFrFutwlU</th>
      <td>1</td>
      <td>6</td>
      <td>0</td>
      <td>65367</td>
    </tr>
    <tr>
      <th>AAEAAAM93+e/vn8MQXZUdE4ECSfPVr/TY1wGwYU33sRFZOLU</th>
      <td>0</td>
      <td>0</td>
      <td>200</td>
      <td>3210</td>
    </tr>
    <tr>
      <th>AAEAAHioJGo60T+xv1nrk+HnPJ68sHjUVAktaItioPwvRV1w</th>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>24161</td>
    </tr>
    <tr>
      <th>AAEAAM9qePqcSQmZK+lRCFDMJ5Zx7l/MkW4smb8WROdeqZmK</th>
      <td>0</td>
      <td>1</td>
      <td>70</td>
      <td>15157</td>
    </tr>
    <tr>
      <th>AAEAAI2b9z3fIB+4uTzBGGWEw3eDYIiH0ettb3kViJsI7RoA</th>
      <td>1</td>
      <td>3</td>
      <td>0</td>
      <td>19195</td>
    </tr>
  </tbody>
</table>
</div>
</div>

Now prepare the scaler. By the way, since we are not composing estimators, we don't need to use a pipeline.


```python
# RobustScaler
from sklearn.preprocessing import RobustScaler
scaler = RobustScaler(quantile_range=(5,95)).fit(x_train)
```

Then apply the scaler to both the train and test datasets, excluding the target feature; notice that the target feature is stored in dataframe `y`. 

```python
X_train = scaler.transform(x_train)
X_test = scaler.transform(x_test)
```

## Regression

At this point, we are ready to use regression algorithms.

### Decision Tree Regressor

Using the decision tree regressor, our model is created by predicting the value of the target feature by learning simple decision rules inferred from the data set we fit it on. In simpler terms, the model is created by performing a large scale pivoting.

First we will do it on our training set; that is, fitting the regressor with `X_train` and `y_train`:

```python
from sklearn.tree import DecisionTreeRegressor

tree_reg = DecisionTreeRegressor()
tree_reg.fit(X_train, y_train)
```

Then make a prediction of the target feature from `X_train`.


```python
y_pred_from_train = tree_reg.predict(X_train)
```

Finally, check the accuracy by calculating Root Mean Squared Error, that is, check how far off the predicted `y` is from the actual `y_train`.


```python
from sklearn.metrics import mean_squared_error
import numpy as np
tree_mse = mean_squared_error(y_pred_from_train, y_train)
tree_rmse = np.sqrt(tree_mse)
```

<div class="j">
<div class="j_out">
<div class="j_pre">
    <p>tree_rmse</p>
</div>
<pre style="background-color: #ddd !important; display: flex">
<code class="hljs python" style="background-color: #ddd !important; display: flex">
2.9395110047712159
</code>
</pre>
</div>
</div>

This sounds too good to be true! Now, do this on the test dataset. Calculate rmse after predicting `y` using `X_test`. Recall that the regressor is trained on `X_train` and has not seen `X_test`. So, this is the *real* test of our model.

```python
y_pred_from_test = tree_reg.predict(X_test) #make the prediction
tree_mse_test = mean_squared_error(y_pred_from_test, y_test) #calculate rmse
tree_rmse_test = np.sqrt(tree_mse_test)
```

<div class="j">
<div class="j_out">
<div class="j_pre">
    <p>tree_rmse_test</p>
</div>
<pre style="background-color: #ddd !important; display: flex">
<code class="hljs python" style="background-color: #ddd !important; display: flex">
40.278871508455829
</code>
</pre>
</div>
</div>

Okay, the error on the test set is 14 times that of the training set!! This doesn't look good. But it's not surprising as Decision Tree Regressor is prone to *overfitting*. So, it overfit the training set and thus performed badly on an unknown set, i.e., our test set.

Overfitting can be abated using sklearn's **Cross-Validation** feature. I will demonstrate how we can perform *K-fold cross-validation* with `K=10`.

```python
from sklearn.model_selection import cross_val_score
tree_scores = cross_val_score(tree_reg, X_train, y_train,
                        scoring = "neg_mean_squared_error", cv=10
                        )
tree_rmse_scores = np.sqrt(-tree_scores)
```

```python
# taken from Aurelien Geron's Hands on ML book
def display_scores(scores):
    print("scores: ", scores)
    print("mean:", scores.mean())
    print("standard deviation:", scores.std())
```

<div class="j">
<div class="j_out">
<div class="j_pre">
    <p>display_scores(tree_rmse_scores)</p>
</div>
<pre >
<code class="hljs python">
scores:  [ 29.75540989  56.09513377  44.12899068  73.75383811  39.68194397
      38.37672842  53.38223776  61.47702739  47.48127876  37.58361917]
    mean: 48.1716207909
    standard deviation: 12.4800354025
</code>
</pre>
</div>
</div>


### Linear Regressor

A linear regression model works by computing the target feature (`__ans__` here) as a weighted sum of the predictor features (columns of `X`) plus an intercept term (which the model determines looking at the relationship between the predictors and the target features). Decision Tree, on the other hand, finds complex non-linear relationships in our data that the linear regressor misses. 

Now let's see how well the Linear Regression model does. 


```python
from sklearn.linear_model import LinearRegression

lin_reg = LinearRegression()
lin_reg.fit(X_train, y_train)

y_pred_train_lin = lin_reg.predict(X_train)
```

```python
lin_rmse_train = np.sqrt(mean_squared_error(y_pred_train_lin, y_train))
```

<div class="j">
<div class="j_out">
<div class="j_pre">
    <p>lin_rmse_train</p>
</div>
<pre style="background-color: #ddd !important; display: flex">
<code class="hljs python" style="background-color: #ddd !important; display: flex">
26.527626721102603
</code>
</pre>
</div>
</div>

How well does it do on the test set?


```python
y_pred_test_lin = lin_reg.predict(X_test)
lin_rmse_test = np.sqrt(mean_squared_error(y_pred_test_lin, y_test))
```

<div class="j">
<div class="j_out">
<div class="j_pre">
    <p>lin_rmse_test</p>
</div>
<pre style="background-color: #ddd !important; display: flex">
<code class="hljs python" style="background-color: #ddd !important; display: flex">
36.563500562427755
</code>
</pre>
</div>
</div>

That's quite impressive. On the test set, it performed better than the tree regressor. This shows that it's realiable. Okay, but how badly is it overfitting?


```python
lin_scores = cross_val_score(lin_reg, X_train, y_train,
                        scoring = "neg_mean_squared_error", cv=10
                        )
lin_rmse_scores = np.sqrt(-lin_scores)
```

<div class="j">
<div class="j_out">
<div class="j_pre">
    <p>display_scores(lin_rmse_scores)</p>
</div>
<pre style="background-color: #ffffff">
<code class="hljs python" style="background-color: #ffffff">
scores:  [ 30.05448244  33.26047826  39.80200849  42.4842238   21.47302677
      31.36803597  18.7646719   63.04366609  44.5649017   30.41392853]
    mean: 35.5229423942
    standard deviation: 12.0935723986
</code>
</pre>
</div>
</div>

Indeed, the Linear Model is not overfitting as bad as the Decision Tree model, and in fact, it also has a lower generalization error.

### Random Forest Regressor

Random Forest model is prepared by performing decision tree regression on random subsets of the training data (`X_train`) and averaging out the predictions (essentially reducing variance). It is a type of [ensemble learning algorithm](http://scikit-learn.org/stable/modules/ensemble.html). 

```python
from sklearn.ensemble import RandomForestRegressor
forest_reg = RandomForestRegressor()
```

First fit the regressor with `X_train` and `y_train`:

<!-- why do we not wanna use transform_fit -->
<div class="j">
<div class="j_out">
<div class="j_pre">
    <p>forest_reg.fit(X_train, y_train)</p>
</div>
<pre style="background-color: #ddd">
<code class="hljs python" style="background-color: #ddd ">
RandomForestRegressor(bootstrap=True, criterion='mse', max_depth=None,
           max_features='auto', max_leaf_nodes=None,
           min_impurity_split=1e-07, min_samples_leaf=1,
           min_samples_split=2, min_weight_fraction_leaf=0.0,
           n_estimators=10, n_jobs=1, oob_score=False, random_state=None,
           verbose=0, warm_start=False)
</code>
</pre>
</div>
</div>

<!---
```python
RandomForestRegressor(bootstrap=True, criterion='mse', max_depth=None,
           max_features='auto', max_leaf_nodes=None,
           min_impurity_split=1e-07, min_samples_leaf=1,
           min_samples_split=2, min_weight_fraction_leaf=0.0,
           n_estimators=10, n_jobs=1, oob_score=False, random_state=None,
           verbose=0, warm_start=False)

```
--->

Then make a prediction of the target feature from `X_train`.

```python
y_pred_train_forest = forest_reg.predict(X_train)
```

Finally, check the accuracy by calculating Root Mean Squared Error, that is, check how far off the predicted `y` is from the actual `y_train`.

```python
# root mean square error
from sklearn.metrics import mean_squared_error
import numpy as np
forest_mse_train = mean_squared_error(y_pred_from_train, y_train)
forest_rmse_train = np.sqrt(forest_mse)
```

<div class="j">
<div class="j_out">
<div class="j_pre">
    <p>forest_rmse_train</p>
</div>
<pre style="background-color: #ddd !important; display: flex">
<code class="hljs python" style="background-color: #ddd !important; display: flex">
17.194583863664128
</code>
</pre>
</div>
</div>

Now, do this on the test dataset. Calculate rmse after predicting `y` using `X_test`. Recall that the regressor is trained on `X_train` and has not seen `X_test`. So, this is the **real** test of our model.



```python
y_pred_test_forest = forest_reg.predict(X_test) #make the prediction
forest_mse_test = mean_squared_error(y_pred_from_test, y_test) #calculate rmse
forest_rmse_test = np.sqrt(forest_mse_test)
```

<div class="j">
<div class="j_out">
<div class="j_pre">
    <p>forest_rmse_test</p>
</div>
<pre style="background-color: #ddd !important; display: flex">
<code class="hljs python" style="background-color: #ddd !important; display: flex">
35.263863915914143
</code>
</pre>
</div>
</div>

The generalization error is only twice the training error. As expected, since the Random Forest model averages out it's prediction on subsets of the training set, it is less prone to overfitting than the Decision Tree model which had a generalization error that was 14 times the training error.

## Looking Ahead

### Regularization

To abate overfitting of our models, constraints known as *hyperparameters* need to be introduced on the learning algorithms; thus, effectively simplifying our model. One such hyperparameter is *regularization*.

A hyperparameter can be too constraining, if it's set to too large a value for instance. In that case, overfitting will certainly be avoided, but the model will be less likely to find a good solution. So, fine tuning the hyperparameters of the Decision Tree Algorithm, for example, is a natural next step. 

### Richer Features

Another action to improve our models is to, for instance, use the Linear Regressor but with a richer set of features. That is engineering features from the predictor features, including the text features we dropped for this post. This also involves getting rid of irrelevant features (maybe the `anonymous` series?)

### Another Look at the Outliers?

Yet another option is to fix the quality of our data. That is, discarding the outliers or manually fixing them. 

One concern here is that the outliers in the data, and the ones we detected above are not actually a result of a measurement or data collection error. Indeed, at the time of collection, any quora question has a certain value of number of questions, number of followers, etc. So, just discarding the anomalies as noise does not feel right. Need to think about this more.


## Side Note

That the Linear Regression model performed as good (and even better) as the more complex decision tree model points the same direction as what Michele Banko and Eric Brill showed in their paper *The Unreasonable Effectiveness of Data*. They found out that fairly simple algorithms were able to perform identically well on a complex ml problem once both models were fed enough data. But don't be mistaken, our dataset is not really large enough for us to actually reach the same conclusions as they did!

---
<h3 style="text-align: center"> References</h3>

1. [Scikit-Learn Documentation](http://scikit-learn.org/).
2. *Hands-On Machine Learning with Scikit-Learn and TensorFlow* by  Aurélien Géron.

<!-- ## Evaluating the Models

### RMSE

```python
# root mean square error
from sklearn.metrics import mean_squared_error
import numpy as np
forest_mse = mean_squared_error(_y_pred, y_test)
forest_rmse = np.sqrt(forest_mse)
```

<div class="j">
<div class="j_out">
<div class="j_pre">
    <p>forest_rmse</p>
</div>
<pre style="background-color: #ddd !important; display: flex">
<code class="hljs python" style="background-color: #ddd !important; display: flex">
34.811285980256926
</code>
</pre>
</div>
</div> -->


<!-- 
### RMSLE

A popular Kaggle standard of measuring error is the Root Mean Squared Logarithmic Error. The function is available [here](https://www.kaggle.com/wiki/RootMeanSquaredLogarithmicError).


```python
# Root Mean Squared Logarithmic Error
def rmsle(predicted,real):
    sum=0.0
    for x in range(len(predicted)):
        p = np.log(predicted[x]+1)
        r = np.log(real[x]+1)
        sum = sum + (p - r)**2
    return (sum/len(predicted))**0.5
```


<div class="j">
<div class="j_out">
<div class="j_pre">
    <p>rmsle(_y_pred, y_test)</p>
</div>
<pre style="background-color: #ddd !important; display: flex">
<code class="hljs python" style="background-color: #ddd !important; display: flex">
1.1638916052747352
</code>
</pre>
</div>
</div> -->