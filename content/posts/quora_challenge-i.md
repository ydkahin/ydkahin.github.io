+++
title = "Views Prediction - A Quora Challenge"
description = ""
tags = [
    "challenge",
    "quora",
]
date = "2017-08-29"
categories = [
    "Challenges"
]
+++

Hello!

I am doing the [Quora *Views*](https://www.quora.com/challenges#views) challenge. In upcoming posts, I will chronicle my progress. In this post, I discuss the data munging after introducing the challenge. 

# Prompt

The questions the challenge wants to address are the following:

 *  *Can you tell which questions can organically attract the most viewers?*
 *  *What about questiosn that eventually become viral?*
 *  *Which questions are timeless and can sustain traffic?*

"**Organically** attract." Hmm, what does that exactly mean?

The challenge is to predict _**the number of views per day in age of the question**_, given the following

1. quora question text, 
2. topic data, 
3. number of answers, and
4. number of people promoted to. 

# Training Data

The sample testcases [folder](http://hr-testcases.s3.amazonaws.com/691/sample.zip) contains two files `input00.in` and `output00.out`. 

The input file has 10,000 questions as valid json objects. They are divided into two groups. The first group has 9,000 questions with the following fields:

```json
question_key, question_text, context_topic, topics, anonymous, num_answers, promoted_to, __ans__
```

The last field is a (float) ratio of *viewers* to *age of the question* in days. In the second group, the questions have all but this field. 

The output file has 1,000 rows of valid json objects. There are two fields in each line. A `question_key` key containing the same unique identifier as in the test/input file. And the predicted value keyed by `__ans__`. 

So our data is split 9:1, training to test. 

### The Fields

*This is copied from the prompt page:*

* `question_key` (string): Unique identifier for the question.
* `question_text` (string): Text of the question.
* `context_topic` (object): The primary topic of a question, if present. Null otherwise. This object will contain `name`(string) and a count of `followers`(integer). The number of followers doesn't exceed 106. 
* `topics` (array of objects): All topics on a question, including the primary topic. Each topic object will contain a name and followers count.
* `anonymous` (boolean): Whether the question was anonymous.
* `num_answers` (integer): The number of visible non-collapsed answers the question has.
* `promoted_to` (integer): The number of people the question was promoted to.
* `__ans__` (float): The ratio of viewers to age of the question in days.

# Data Munging

After importing the `input00.in` file, each of its line is added to a list as a valid json object. Before converting the list `data`, the two integers marking the number of lines are removed. Finally, a dataframe `df` is generated from the list

```python
json_data = open('../views/sample/input00.in') # Edit this to where you have put the input00.in file
import pandas as pd
import json

data = []
for line in json_data:
    data.append(json.loads(line))

data.remove(9000)
data.remove(1000)

df = pd.DataFrame(data)

```


The tail of `df` has a column `__ans__` of `NaN`; as discussed earlier, they are missing because the last 1000 lines of our data are our test set and `__ans__` is the target variable.

<div class="j">
<div class="j_pre">
    <p>df</p>
</div>
<div class="j_table" style="height: 20em">
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
      <th>9970</th>
      <td>NaN</td>
      <td>False</td>
      <td>{'followers': 35, 'name': 'Public Restrooms'}</td>
      <td>7</td>
      <td>0</td>
      <td>AAEAAAHZ7eoo1x7ZduZlk826XPxN/yuX5x0SQHtoLpsxK2MA</td>
      <td>Is it OK to use the disabled restroom in rest ...</td>
      <td>[{'followers': 46198, 'name': 'Manners and Eti...</td>
    </tr>
    <tr>
      <th>9971</th>
      <td>NaN</td>
      <td>False</td>
      <td>{'followers': 7361, 'name': 'What Does It Feel...</td>
      <td>4</td>
      <td>0</td>
      <td>AAEAAKBzcyFahiqWIQ5HnBOr6JGdxyiXSuzDXaIP8LK/AVCW</td>
      <td>How does it feel like to actually be 999999th ...</td>
      <td>[{'followers': 7361, 'name': 'What Does It Fee...</td>
    </tr>
    <tr>
      <th>9972</th>
      <td>NaN</td>
      <td>False</td>
      <td>None</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAPg3XukDQ+HavVquEyHBWuoif46lRe787qBNZ0cy7c60</td>
      <td>Who are the biggest trolls of WWII?</td>
      <td>[{'followers': 47252, 'name': 'World War II'},...</td>
    </tr>
    <tr>
      <th>9973</th>
      <td>NaN</td>
      <td>False</td>
      <td>{'followers': 7735, 'name': 'Kolkata, West Ben...</td>
      <td>5</td>
      <td>0</td>
      <td>AAEAAIOmikV3Q8HsiQ3byAbxKOXi6BqpPNIcGGdTe/y4PCQs</td>
      <td>What are some good "vegeterian" restaurants in...</td>
      <td>[{'followers': 7735, 'name': 'Kolkata, West Be...</td>
    </tr>
    <tr>
      <th>9974</th>
      <td>NaN</td>
      <td>False</td>
      <td>{'followers': 2763, 'name': 'Acoustic Guitar'}</td>
      <td>3</td>
      <td>0</td>
      <td>AAEAAPL1yN77qhZM8ttw3/3DlTumpM9f7njgpVa8c+mxXPGW</td>
      <td>What is the best way to transport a guitar whe...</td>
      <td>[{'followers': 2763, 'name': 'Acoustic Guitar'}]</td>
    </tr>
    <tr>
      <th>9975</th>
      <td>NaN</td>
      <td>False</td>
      <td>{'followers': 240658, 'name': 'Fine Art'}</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAKMJys+KhCvsot3ulnAdamgxZUjM00y5KYW+O1ixN7LN</td>
      <td>How can we find why people buy vitrail dome?</td>
      <td>[{'followers': 240658, 'name': 'Fine Art'}]</td>
    </tr>
    <tr>
      <th>9976</th>
      <td>NaN</td>
      <td>False</td>
      <td>{'followers': 130510, 'name': 'India'}</td>
      <td>3</td>
      <td>0</td>
      <td>AAEAABirDGEz/r8CmOIGrPpe6UcwPtA7CS72NZoNGYcJv7VS</td>
      <td>What do you think that happens no where in the...</td>
      <td>[{'followers': 224, 'name': 'Indians On Quora'...</td>
    </tr>
    <tr>
      <th>9977</th>
      <td>NaN</td>
      <td>False</td>
      <td>{'followers': 614223, 'name': 'Science'}</td>
      <td>5</td>
      <td>200</td>
      <td>AAEAAJ6R9EimCjDPYlIpjOBYRQ7CPxobn9C/xCwurd+Zp67w</td>
      <td>I see a lot of bad science being reported on m...</td>
      <td>[{'followers': 28, 'name': 'Science Journalism...</td>
    </tr>
    <tr>
      <th>9978</th>
      <td>NaN</td>
      <td>True</td>
      <td>{'followers': 1115, 'name': 'Meeting New People'}</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAJo09/QYxyDglnKJuYngbSfA0py6jXkx5MiLJDEg61LK</td>
      <td>Who designed the Martini mobile app that conne...</td>
      <td>[{'followers': 1115, 'name': 'Meeting New Peop...</td>
    </tr>
    <tr>
      <th>9979</th>
      <td>NaN</td>
      <td>False</td>
      <td>{'followers': 307, 'name': 'Configuration Mana...</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAANkBVF9hcaNrU0AGLY4ynp8xGV+2ymgibGsTST8wFDIB</td>
      <td>Hosted or Privated Chef/Puppet?</td>
      <td>[{'followers': 307, 'name': 'Configuration Man...</td>
    </tr>
    <tr>
      <th>9980</th>
      <td>NaN</td>
      <td>False</td>
      <td>{'followers': 252, 'name': 'Holi'}</td>
      <td>4</td>
      <td>0</td>
      <td>AAEAAGKwWCFpoJsi/ZnroU3MhPB11LLDfYAcG+u3sHyNfDfv</td>
      <td>How would you justify the wastage of water dur...</td>
      <td>[{'followers': 252, 'name': 'Holi'}, {'followe...</td>
    </tr>
    <tr>
      <th>9981</th>
      <td>NaN</td>
      <td>False</td>
      <td>{'followers': 2360, 'name': 'Laptops'}</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAFrCnoffwcwVpDIsreCyT+XRzPuPt0wA5zpGaeKwh3AH</td>
      <td>Which processor is better for a graphic design...</td>
      <td>[{'followers': 773218, 'name': 'Technology'}, ...</td>
    </tr>
    <tr>
      <th>9982</th>
      <td>NaN</td>
      <td>False</td>
      <td>{'followers': 167230, 'name': 'Investing'}</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAHz2iHfkZ17beSDLdG36mqh76iJxbvy7ltVq/hI3dxKN</td>
      <td>Is Warren Buffett investing in companies becau...</td>
      <td>[{'followers': 167230, 'name': 'Investing'}]</td>
    </tr>
    <tr>
      <th>9983</th>
      <td>NaN</td>
      <td>True</td>
      <td>{'followers': 2080, 'name': 'Driving'}</td>
      <td>3</td>
      <td>0</td>
      <td>AAEAAGizmW9Cn964/fe563GXfoIj8jhk98sNBvvobTYxWcj0</td>
      <td>I've failed my driving test twice: how can I p...</td>
      <td>[{'followers': 2080, 'name': 'Driving'}]</td>
    </tr>
    <tr>
      <th>9984</th>
      <td>NaN</td>
      <td>False</td>
      <td>{'followers': 1892, 'name': 'Lacrosse'}</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAABuDHz22TdRoHnLiVYfQAyvh22kOL/zDvhoAJQoFUhRw</td>
      <td>Why are men's lacrosse and women's lacrosse so...</td>
      <td>[{'followers': 322541, 'name': 'Sports'}, {'fo...</td>
    </tr>
    <tr>
      <th>9985</th>
      <td>NaN</td>
      <td>False</td>
      <td>{'followers': 3551, 'name': 'Human Rights'}</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAJKYJgaXheTZGWPCunOIbCsKbGcjD2w1ENUikQPDeNpA</td>
      <td>Because men are Human beings too do they deser...</td>
      <td>[{'followers': 3551, 'name': 'Human Rights'}]</td>
    </tr>
    <tr>
      <th>9986</th>
      <td>NaN</td>
      <td>True</td>
      <td>{'followers': 130510, 'name': 'India'}</td>
      <td>10</td>
      <td>0</td>
      <td>AAEAAFZUD0HT9fdkyj3oTR6Ew5cXqntfzyUQrAmPr3LiFRWu</td>
      <td>Why do many Tamils refer to themselves as Tam ...</td>
      <td>[{'followers': 50, 'name': 'Tamil Culture'}, {...</td>
    </tr>
    <tr>
      <th>9987</th>
      <td>NaN</td>
      <td>False</td>
      <td>{'followers': 526597, 'name': 'Business'}</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAMXz0j0UAWLFLPK8SmQN5flyCv8HfYnXj3Gc6IOuDM5B</td>
      <td>How to start a website developing business in ...</td>
      <td>[{'followers': 526597, 'name': 'Business'}]</td>
    </tr>
    <tr>
      <th>9988</th>
      <td>NaN</td>
      <td>False</td>
      <td>None</td>
      <td>29</td>
      <td>200</td>
      <td>AAEAAN2HZuSZhaIt/UgzCSRzjztTUJQ2rDMOZmSC6enp4XDt</td>
      <td>What is the best rebuttal to "If it ain't brok...</td>
      <td>[{'followers': 364, 'name': 'Disruption'}, {'f...</td>
    </tr>
    <tr>
      <th>9989</th>
      <td>NaN</td>
      <td>False</td>
      <td>{'followers': 238905, 'name': 'Facebook'}</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAClprKo7msONt3rt+zrm/fP2ZkDg4AK6nBx6zMQkS0ky</td>
      <td>Facebook app center (versus) existing app stores?</td>
      <td>[{'followers': 6404, 'name': 'Google Play'}, {...</td>
    </tr>
    <tr>
      <th>9990</th>
      <td>NaN</td>
      <td>False</td>
      <td>{'followers': 3327, 'name': 'Google Docs'}</td>
      <td>0</td>
      <td>50</td>
      <td>AAEAAJCL484wkZIHiRw1MLBu33qqxkTmuJGDIrEgW4vMnyqu</td>
      <td>Why do Bold fonts and Italics disappear when c...</td>
      <td>[{'followers': 1915, 'name': 'Google Drive'}, ...</td>
    </tr>
    <tr>
      <th>9991</th>
      <td>NaN</td>
      <td>False</td>
      <td>{'followers': 382503, 'name': 'Health and Well...</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAC72X8TZvRNVSYZ6hTMnTfpBaHPuFpodk23H3xfrK6NH</td>
      <td>Why does being cold make you catch a cold?</td>
      <td>[{'followers': 382503, 'name': 'Health and Wel...</td>
    </tr>
    <tr>
      <th>9992</th>
      <td>NaN</td>
      <td>True</td>
      <td>{'followers': 241809, 'name': 'Startups'}</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAM10T++NbYU9JFMEb6QxUnhc5LJxtkOvG1HhfuxSpTg3</td>
      <td>What are the hottest under-10 person startups ...</td>
      <td>[{'followers': 164924, 'name': 'Lean Startups'...</td>
    </tr>
    <tr>
      <th>9993</th>
      <td>NaN</td>
      <td>True</td>
      <td>{'followers': 331, 'name': 'Indonesian (langua...</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAMPrlsjCEejyiB4x6Nmvi4yEMD5fUpJZo/WV/oc1L/Yk</td>
      <td>How do people from other countries think about...</td>
      <td>[{'followers': 331, 'name': 'Indonesian (langu...</td>
    </tr>
    <tr>
      <th>9994</th>
      <td>NaN</td>
      <td>False</td>
      <td>{'followers': 199, 'name': 'Technology Industr...</td>
      <td>1</td>
      <td>200</td>
      <td>AAEAAFF/WfbxS4V2roHkQ+WEgqXcVXxTCljXMuGkslg7XtOq</td>
      <td>What's the average deal size of an Israeli M&amp;A?</td>
      <td>[{'followers': 199, 'name': 'Technology Indust...</td>
    </tr>
    <tr>
      <th>9995</th>
      <td>NaN</td>
      <td>False</td>
      <td>{'followers': 5652, 'name': 'Social Entreprene...</td>
      <td>2</td>
      <td>0</td>
      <td>AAEAADHmhdj7ddMsxwLxIORZC+IfLoydhg+FXTwjBssCfemU</td>
      <td>What is the best career path to take to become...</td>
      <td>[{'followers': 256, 'name': 'Social Impact'}, ...</td>
    </tr>
    <tr>
      <th>9996</th>
      <td>NaN</td>
      <td>True</td>
      <td>{'followers': 10143, 'name': 'Jobs'}</td>
      <td>3</td>
      <td>0</td>
      <td>AAEAAF9ASgzjooq/SgiJg58it0eZ6PPYv8B/r4k6gH8VhdCG</td>
      <td>Is having a facial piercing considered unprofe...</td>
      <td>[{'followers': 4801, 'name': 'Engineering Recr...</td>
    </tr>
    <tr>
      <th>9997</th>
      <td>NaN</td>
      <td>True</td>
      <td>{'followers': 8017, 'name': 'Sarcasm'}</td>
      <td>3</td>
      <td>0</td>
      <td>AAEAAHaDxug0N4SrFLa65nyX0onGDf3C7DpBBF+u/4MTu9ye</td>
      <td>How does it feel to get sarcastically insulted...</td>
      <td>[{'followers': 8017, 'name': 'Sarcasm'}]</td>
    </tr>
    <tr>
      <th>9998</th>
      <td>NaN</td>
      <td>True</td>
      <td>{'followers': 2421, 'name': 'Kashmir'}</td>
      <td>3</td>
      <td>60</td>
      <td>AAEAABtiDQZgE2LOLxtgl33uaw4xfhMDFqlWV4wVsE9H/Dcq</td>
      <td>Why did Kashmiri Hindus have to leave their ho...</td>
      <td>[{'followers': 9393, 'name': 'Islam'}, {'follo...</td>
    </tr>
    <tr>
      <th>9999</th>
      <td>NaN</td>
      <td>False</td>
      <td>{'followers': 64684, 'name': 'Venture Capital'}</td>
      <td>3</td>
      <td>0</td>
      <td>AAEAAL8xXMY8ocS5aVwA+01/21LZ29oejC1zVXl2eS+a+9J6</td>
      <td>Is it possible to become a successful venture ...</td>
      <td>[{'followers': 64684, 'name': 'Venture Capital'}]</td>
    </tr>
  </tbody>
</table>
<p style="font-family: monospace;">10000 rows × 8 columns</p>
</div>
</div>


Notice that each row of `context_topic` and `topics` contains a valid json object and a list containing json objects, respectively. Recall that `context_topic` is the primary topic of the question and it's not always present. On the other hand, `topics` is all topics on a question.

For now, the `topics` column is left as it is. But the `context_topic` column will be split to `context_topic_followers` and `context_topic_name`.

```python
context_topic_df = df['context_topic'].apply(pd.Series).rename(columns='context_topic_{}'.format)
```


<div class="j">
<div class="j_pre">
    <p>context_topic_df</p>
</div>
<div class="j_table" style="height: 20em">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>context_topic_followers</th>
      <th>context_topic_name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>500022.0</td>
      <td>Movies</td>
    </tr>
    <tr>
      <th>1</th>
      <td>179.0</td>
      <td>International Mathematical Olympiad (IMO)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>614223.0</td>
      <td>Science</td>
    </tr>
    <tr>
      <th>3</th>
      <td>614223.0</td>
      <td>Science</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1536.0</td>
      <td>Android Tablets</td>
    </tr>
    <tr>
      <th>5</th>
      <td>91.0</td>
      <td>Smartphone Applications</td>
    </tr>
    <tr>
      <th>6</th>
      <td>241809.0</td>
      <td>Startups</td>
    </tr>
    <tr>
      <th>7</th>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>8</th>
      <td>3006.0</td>
      <td>Warfare</td>
    </tr>
    <tr>
      <th>9</th>
      <td>224.0</td>
      <td>Boston Marathon Terrorist Attacks (April 2013)</td>
    </tr>
    <tr>
      <th>10</th>
      <td>4155.0</td>
      <td>Organization</td>
    </tr>
    <tr>
      <th>11</th>
      <td>304.0</td>
      <td>Web Development on Mac OS X</td>
    </tr>
    <tr>
      <th>12</th>
      <td>3292.0</td>
      <td>United States Governments (Federal, State, Local)</td>
    </tr>
    <tr>
      <th>13</th>
      <td>13867.0</td>
      <td>Parenting</td>
    </tr>
    <tr>
      <th>14</th>
      <td>72820.0</td>
      <td>Biology</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2049.0</td>
      <td>Tablet Devices and Tablet Market</td>
    </tr>
    <tr>
      <th>16</th>
      <td>999.0</td>
      <td>Luxury</td>
    </tr>
    <tr>
      <th>17</th>
      <td>39744.0</td>
      <td>Web Development</td>
    </tr>
    <tr>
      <th>18</th>
      <td>3566.0</td>
      <td>Online Video</td>
    </tr>
    <tr>
      <th>19</th>
      <td>268430.0</td>
      <td>Television Series</td>
    </tr>
    <tr>
      <th>20</th>
      <td>74.0</td>
      <td>Noodle Education</td>
    </tr>
    <tr>
      <th>21</th>
      <td>490.0</td>
      <td>Online Reputation</td>
    </tr>
    <tr>
      <th>22</th>
      <td>264984.0</td>
      <td>Dating and Relationships</td>
    </tr>
    <tr>
      <th>23</th>
      <td>6608.0</td>
      <td>Science and Religion</td>
    </tr>
    <tr>
      <th>24</th>
      <td>1707.0</td>
      <td>Democracy</td>
    </tr>
    <tr>
      <th>25</th>
      <td>2450.0</td>
      <td>Wanting and Making Money</td>
    </tr>
    <tr>
      <th>26</th>
      <td>3567.0</td>
      <td>Private Equity</td>
    </tr>
    <tr>
      <th>27</th>
      <td>238905.0</td>
      <td>Facebook</td>
    </tr>
    <tr>
      <th>28</th>
      <td>238905.0</td>
      <td>Facebook</td>
    </tr>
    <tr>
      <th>29</th>
      <td>986.0</td>
      <td>Glenn Gould</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>9970</th>
      <td>35.0</td>
      <td>Public Restrooms</td>
    </tr>
    <tr>
      <th>9971</th>
      <td>7361.0</td>
      <td>What Does It Feel Like to X?</td>
    </tr>
    <tr>
      <th>9972</th>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>9973</th>
      <td>7735.0</td>
      <td>Kolkata, West Bengal, India</td>
    </tr>
    <tr>
      <th>9974</th>
      <td>2763.0</td>
      <td>Acoustic Guitar</td>
    </tr>
    <tr>
      <th>9975</th>
      <td>240658.0</td>
      <td>Fine Art</td>
    </tr>
    <tr>
      <th>9976</th>
      <td>130510.0</td>
      <td>India</td>
    </tr>
    <tr>
      <th>9977</th>
      <td>614223.0</td>
      <td>Science</td>
    </tr>
    <tr>
      <th>9978</th>
      <td>1115.0</td>
      <td>Meeting New People</td>
    </tr>
    <tr>
      <th>9979</th>
      <td>307.0</td>
      <td>Configuration Management</td>
    </tr>
    <tr>
      <th>9980</th>
      <td>252.0</td>
      <td>Holi</td>
    </tr>
    <tr>
      <th>9981</th>
      <td>2360.0</td>
      <td>Laptops</td>
    </tr>
    <tr>
      <th>9982</th>
      <td>167230.0</td>
      <td>Investing</td>
    </tr>
    <tr>
      <th>9983</th>
      <td>2080.0</td>
      <td>Driving</td>
    </tr>
    <tr>
      <th>9984</th>
      <td>1892.0</td>
      <td>Lacrosse</td>
    </tr>
    <tr>
      <th>9985</th>
      <td>3551.0</td>
      <td>Human Rights</td>
    </tr>
    <tr>
      <th>9986</th>
      <td>130510.0</td>
      <td>India</td>
    </tr>
    <tr>
      <th>9987</th>
      <td>526597.0</td>
      <td>Business</td>
    </tr>
    <tr>
      <th>9988</th>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>9989</th>
      <td>238905.0</td>
      <td>Facebook</td>
    </tr>
    <tr>
      <th>9990</th>
      <td>3327.0</td>
      <td>Google Docs</td>
    </tr>
    <tr>
      <th>9991</th>
      <td>382503.0</td>
      <td>Health and Wellness</td>
    </tr>
    <tr>
      <th>9992</th>
      <td>241809.0</td>
      <td>Startups</td>
    </tr>
    <tr>
      <th>9993</th>
      <td>331.0</td>
      <td>Indonesian (language)</td>
    </tr>
    <tr>
      <th>9994</th>
      <td>199.0</td>
      <td>Technology Industry in Israel</td>
    </tr>
    <tr>
      <th>9995</th>
      <td>5652.0</td>
      <td>Social Entrepreneurship</td>
    </tr>
    <tr>
      <th>9996</th>
      <td>10143.0</td>
      <td>Jobs</td>
    </tr>
    <tr>
      <th>9997</th>
      <td>8017.0</td>
      <td>Sarcasm</td>
    </tr>
    <tr>
      <th>9998</th>
      <td>2421.0</td>
      <td>Kashmir</td>
    </tr>
    <tr>
      <th>9999</th>
      <td>64684.0</td>
      <td>Venture Capital</td>
    </tr>
  </tbody>
</table>
<p style="font-family: monospace;">10000 rows × 2 columns</p>
</div>

Now we have a clean dataframe

```python
cleaned_df = pd.concat([df.drop(['context_topic'], axis=1), context_topic_df], axis = 1)
```

The data will have to be split 9:1, train to test. This is not my choice but this is the proportion it came with in `input00.in`.

```python
test = cleaned_df[9000:]
```
<div class="j">
<div class="j_pre">
    <p>test</p>
</div>

<div class="j_table" style="height: 20em">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>__ans__</th>
      <th>anonymous</th>
      <th>num_answers</th>
      <th>promoted_to</th>
      <th>question_key</th>
      <th>question_text</th>
      <th>topics_followers</th>
      <th>topics_name</th>
      <th>context_topic_followers</th>
      <th>context_topic_name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>9000</th>
      <td>NaN</td>
      <td>False</td>
      <td>0</td>
      <td>90</td>
      <td>AAEAACoD29W+HelUgB32vNEdu0NGxqJi7SH91iH1FLvNDOj2</td>
      <td>DCI, or should it be RIC(Role/Interaction/Cont...</td>
      <td>24761.0</td>
      <td>Software Engineering</td>
      <td>24761.0</td>
      <td>Software Engineering</td>
    </tr>
    <tr>
      <th>9001</th>
      <td>NaN</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAIEmRDzBkZ9M9+ff9T/chVUceCbKUjDRUevvvu5qU7ix</td>
      <td>What agreement does hotwire have with partner ...</td>
      <td>110108.0</td>
      <td>Hotels</td>
      <td>789.0</td>
      <td>Hotwire</td>
    </tr>
    <tr>
      <th>9002</th>
      <td>NaN</td>
      <td>False</td>
      <td>3</td>
      <td>0</td>
      <td>AAEAADPyyYuxYuSes2G7E9LHLkxu9ZNU6E237WmY1YOe/x0E</td>
      <td>Are all Autism or Asperger Syndrome people Int...</td>
      <td>715.0</td>
      <td>Asperger Syndrome</td>
      <td>828.0</td>
      <td>Autism</td>
    </tr>
    <tr>
      <th>9003</th>
      <td>NaN</td>
      <td>False</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAHZRcTcYkYH933/Cssn032ygfGEqeWYsmkQNDzoVGdA/</td>
      <td>What businesses hire 15 year olds?</td>
      <td>526597.0</td>
      <td>Business</td>
      <td>526597.0</td>
      <td>Business</td>
    </tr>
    <tr>
      <th>9004</th>
      <td>NaN</td>
      <td>False</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAD/FZeneJxjntn5ZNSVPrjE8H7zsPVePpjl6GjCgUVK2</td>
      <td>What are some great table wine magnums for aro...</td>
      <td>13119.0</td>
      <td>Wine</td>
      <td>13119.0</td>
      <td>Wine</td>
    </tr>
    <tr>
      <th>9005</th>
      <td>NaN</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAEaEgf32q+1yViJeHIJAuLQPZuAAv/Btejc3b/gSDgZc</td>
      <td>Why is there such a big power gap, between the...</td>
      <td>4649.0</td>
      <td>Politics of India</td>
      <td>4649.0</td>
      <td>Politics of India</td>
    </tr>
    <tr>
      <th>9006</th>
      <td>NaN</td>
      <td>False</td>
      <td>6</td>
      <td>200</td>
      <td>AAEAALsqS9OhREC42UojcxGk0w0xJKukJ4AFRvNYSGgD34Bw</td>
      <td>Who is your favourite couple in a comics unive...</td>
      <td>8760.0</td>
      <td>DC Comics</td>
      <td>7310.0</td>
      <td>Survey Questions</td>
    </tr>
    <tr>
      <th>9007</th>
      <td>NaN</td>
      <td>False</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAHHRJ0BlvQiYaY4jFjupBH/rT+SSVQR7RA9VaqhLHaut</td>
      <td>What are some good places to watch sports in M...</td>
      <td>3776.0</td>
      <td>Manhattan (NYC borough)</td>
      <td>3776.0</td>
      <td>Manhattan (NYC borough)</td>
    </tr>
    <tr>
      <th>9008</th>
      <td>NaN</td>
      <td>True</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAACWN2WzcacP0QeYb1R6i9/AwT14+CQxOYsGWuWSDyTbY</td>
      <td>What are some websites designers visit frequen...</td>
      <td>5782.0</td>
      <td>Dribbble</td>
      <td>5782.0</td>
      <td>Dribbble</td>
    </tr>
    <tr>
      <th>9009</th>
      <td>NaN</td>
      <td>False</td>
      <td>5</td>
      <td>0</td>
      <td>AAEAAL/9YN4leLj5G1q+OyqQ/vk+bm8gB29rsa4YC1uox4Vk</td>
      <td>Should we have a President of the Internet?</td>
      <td>773218.0</td>
      <td>Technology</td>
      <td>88471.0</td>
      <td>The Internet</td>
    </tr>
    <tr>
      <th>9010</th>
      <td>NaN</td>
      <td>False</td>
      <td>2</td>
      <td>200</td>
      <td>AAEAABqTNMM5ooou6dc6goW1jlnSTKePQFtnj5mvXWen/Ipd</td>
      <td>What is the evolutionary advantage of moths an...</td>
      <td>556.0</td>
      <td>Evolution (process)</td>
      <td>556.0</td>
      <td>Evolution (process)</td>
    </tr>
    <tr>
      <th>9011</th>
      <td>NaN</td>
      <td>False</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAFkVTXgpL1MeFPsD4PhGaftpGlEPcl5VWImjTM7bU3/W</td>
      <td>Why do so many stores in New York have long li...</td>
      <td>24781.0</td>
      <td>Retail</td>
      <td>282900.0</td>
      <td>Marketing</td>
    </tr>
    <tr>
      <th>9012</th>
      <td>NaN</td>
      <td>True</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAABiOwguGI1lILAGwbkPVhGuHQpFn4WiPfa1o9U8RjFbb</td>
      <td>How does the financial reward of creating and ...</td>
      <td>41458.0</td>
      <td>Television Business</td>
      <td>41458.0</td>
      <td>Television Business</td>
    </tr>
    <tr>
      <th>9013</th>
      <td>NaN</td>
      <td>True</td>
      <td>1</td>
      <td>120</td>
      <td>AAEAAJo0yQjaLPZysmGuk7bgctmnO6v+3NxoBEpNYjPMb0B4</td>
      <td>Would you be interested in a feed that display...</td>
      <td>7310.0</td>
      <td>Survey Questions</td>
      <td>4004.0</td>
      <td>Startup Ideas</td>
    </tr>
    <tr>
      <th>9014</th>
      <td>NaN</td>
      <td>False</td>
      <td>2</td>
      <td>0</td>
      <td>AAEAAB2v4935tdkYURjLB/iDeWvl45iTNYhFG1prQeFjzo4i</td>
      <td>What is the business model of gaana.com?</td>
      <td>321001.0</td>
      <td>Entrepreneurship</td>
      <td>2660.0</td>
      <td>Gaana.com</td>
    </tr>
    <tr>
      <th>9015</th>
      <td>NaN</td>
      <td>True</td>
      <td>2</td>
      <td>0</td>
      <td>AAEAALJaYVbBydD/6e/xXXO7mJvH0/JVd9d4yHKU0Y06MUoQ</td>
      <td>How can I split an email conversation in Gmail?</td>
      <td>38391.0</td>
      <td>Gmail</td>
      <td>38391.0</td>
      <td>Gmail</td>
    </tr>
    <tr>
      <th>9016</th>
      <td>NaN</td>
      <td>False</td>
      <td>3</td>
      <td>0</td>
      <td>AAEAAHYlmr8+DzSL7h9p8a4RE2pnFaPnb3TETVHvEoZ11TmX</td>
      <td>How easy is the German language?</td>
      <td>988.0</td>
      <td>German (language)</td>
      <td>988.0</td>
      <td>German (language)</td>
    </tr>
    <tr>
      <th>9017</th>
      <td>NaN</td>
      <td>True</td>
      <td>2</td>
      <td>30</td>
      <td>AAEAAPZSaoGHckv00smkoIAAUYCfden8K9ZudYK/eM9exU1T</td>
      <td>What would you if you could stop time?</td>
      <td>731.0</td>
      <td>What Would You Do If X?</td>
      <td>731.0</td>
      <td>What Would You Do If X?</td>
    </tr>
    <tr>
      <th>9018</th>
      <td>NaN</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAOrnDPcIl8r4dKNTds9OJELueKL6xZVOj/kCa7ampXyY</td>
      <td>What are less-known names of some atypical sha...</td>
      <td>16551.0</td>
      <td>Facts and Trivia</td>
      <td>151.0</td>
      <td>Shapes</td>
    </tr>
    <tr>
      <th>9019</th>
      <td>NaN</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAGHq58yiEm4rfQ6mFtZUE2iN08E4KoTdu1CTagYuZ33p</td>
      <td>What are the most innovative marketing uses of...</td>
      <td>29.0</td>
      <td>Location-based Marketing</td>
      <td>282900.0</td>
      <td>Marketing</td>
    </tr>
    <tr>
      <th>9020</th>
      <td>NaN</td>
      <td>False</td>
      <td>2</td>
      <td>400</td>
      <td>AAEAAGMs/2HJgXjZ9tK9zqOR7Gh+7DdcKjqjlzniHlHnwflm</td>
      <td>What are some alternative conceptualizations o...</td>
      <td>33423.0</td>
      <td>God</td>
      <td>33423.0</td>
      <td>God</td>
    </tr>
    <tr>
      <th>9021</th>
      <td>NaN</td>
      <td>False</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAJo8cflZCA3hcwCzV5yxXT9qe2viR3mZs7RwHpB40ncw</td>
      <td>Is there a Happy Bodnick meme?</td>
      <td>494.0</td>
      <td>Marc Bodnick</td>
      <td>494.0</td>
      <td>Marc Bodnick</td>
    </tr>
    <tr>
      <th>9022</th>
      <td>NaN</td>
      <td>False</td>
      <td>2</td>
      <td>0</td>
      <td>AAEAAIZV7jnw7LGFDGc/Ldse3CJl52+GCOkuiR8DM3nVG7jC</td>
      <td>Why there is an Age limit in Olympic Soccer, w...</td>
      <td>66768.0</td>
      <td>Football (Soccer)</td>
      <td>322541.0</td>
      <td>Sports</td>
    </tr>
    <tr>
      <th>9023</th>
      <td>NaN</td>
      <td>False</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAFdnuShyaJ85KUXcjsJfiDXKbEqvxo0FxAjrt32tvFFQ</td>
      <td>What would happen if China linked the Chinese ...</td>
      <td>333543.0</td>
      <td>Economics</td>
      <td>333543.0</td>
      <td>Economics</td>
    </tr>
    <tr>
      <th>9024</th>
      <td>NaN</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAGrWxL8SnVtVEG7tsu4JortfnuosAswBPKeymfrLYTiY</td>
      <td>Was Rehoboth Beach, Delaware seriously affecte...</td>
      <td>61.0</td>
      <td>Rehoboth Beach, Delaware</td>
      <td>61.0</td>
      <td>Rehoboth Beach, Delaware</td>
    </tr>
    <tr>
      <th>9025</th>
      <td>NaN</td>
      <td>False</td>
      <td>2</td>
      <td>9580</td>
      <td>AAEAAMZG9G0/WGABkSj04y6igmOTzdxOHZfwkTdcgUqjLijN</td>
      <td>If Shah Rukh Khan wrote a book, "How to be Sha...</td>
      <td>500022.0</td>
      <td>Movies</td>
      <td>35412.0</td>
      <td>Shah Rukh Khan</td>
    </tr>
    <tr>
      <th>9026</th>
      <td>NaN</td>
      <td>True</td>
      <td>13</td>
      <td>150</td>
      <td>AAEAAHH+Aquci2tRcdZGlJInuYdkHkMUz4RbChBnwGss1fhN</td>
      <td>What can we expect from Christopher Nolan's In...</td>
      <td>4.0</td>
      <td>Interstellar (movie in development)</td>
      <td>4.0</td>
      <td>Interstellar (movie in development)</td>
    </tr>
    <tr>
      <th>9027</th>
      <td>NaN</td>
      <td>True</td>
      <td>3</td>
      <td>190</td>
      <td>AAEAAAt9WYSIk+oxhUOG1hfxWq2RqBBi2JPUyRfGo0Am/g88</td>
      <td>What are some businesses, in which the real so...</td>
      <td>274839.0</td>
      <td>Business Strategy</td>
      <td>526597.0</td>
      <td>Business</td>
    </tr>
    <tr>
      <th>9028</th>
      <td>NaN</td>
      <td>False</td>
      <td>7</td>
      <td>600</td>
      <td>AAEAAJZa8tqrwo7AdudPrXTKAMF7eFcIzsA4zwVyTEA9qNf8</td>
      <td>What are the best ways to guarantee a return o...</td>
      <td>424.0</td>
      <td>Quora User Tips</td>
      <td>424.0</td>
      <td>Quora Credits</td>
    </tr>
    <tr>
      <th>9029</th>
      <td>NaN</td>
      <td>False</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAFOkBI5LhzxTJ+iOcefXdpIgo3WnnqgDJk8gD61ffBnG</td>
      <td>Could anyone suggest me a name for an event ma...</td>
      <td>278.0</td>
      <td>Event Management</td>
      <td>278.0</td>
      <td>Event Management</td>
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
      <td>...</td>
    </tr>
    <tr>
      <th>9970</th>
      <td>NaN</td>
      <td>False</td>
      <td>7</td>
      <td>0</td>
      <td>AAEAAAHZ7eoo1x7ZduZlk826XPxN/yuX5x0SQHtoLpsxK2MA</td>
      <td>Is it OK to use the disabled restroom in rest ...</td>
      <td>46198.0</td>
      <td>Manners and Etiquette</td>
      <td>35.0</td>
      <td>Public Restrooms</td>
    </tr>
    <tr>
      <th>9971</th>
      <td>NaN</td>
      <td>False</td>
      <td>4</td>
      <td>0</td>
      <td>AAEAAKBzcyFahiqWIQ5HnBOr6JGdxyiXSuzDXaIP8LK/AVCW</td>
      <td>How does it feel like to actually be 999999th ...</td>
      <td>7361.0</td>
      <td>What Does It Feel Like to X?</td>
      <td>7361.0</td>
      <td>What Does It Feel Like to X?</td>
    </tr>
    <tr>
      <th>9972</th>
      <td>NaN</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAPg3XukDQ+HavVquEyHBWuoif46lRe787qBNZ0cy7c60</td>
      <td>Who are the biggest trolls of WWII?</td>
      <td>47252.0</td>
      <td>World War II</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>9973</th>
      <td>NaN</td>
      <td>False</td>
      <td>5</td>
      <td>0</td>
      <td>AAEAAIOmikV3Q8HsiQ3byAbxKOXi6BqpPNIcGGdTe/y4PCQs</td>
      <td>What are some good "vegeterian" restaurants in...</td>
      <td>7735.0</td>
      <td>Kolkata, West Bengal, India</td>
      <td>7735.0</td>
      <td>Kolkata, West Bengal, India</td>
    </tr>
    <tr>
      <th>9974</th>
      <td>NaN</td>
      <td>False</td>
      <td>3</td>
      <td>0</td>
      <td>AAEAAPL1yN77qhZM8ttw3/3DlTumpM9f7njgpVa8c+mxXPGW</td>
      <td>What is the best way to transport a guitar whe...</td>
      <td>2763.0</td>
      <td>Acoustic Guitar</td>
      <td>2763.0</td>
      <td>Acoustic Guitar</td>
    </tr>
    <tr>
      <th>9975</th>
      <td>NaN</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAKMJys+KhCvsot3ulnAdamgxZUjM00y5KYW+O1ixN7LN</td>
      <td>How can we find why people buy vitrail dome?</td>
      <td>240658.0</td>
      <td>Fine Art</td>
      <td>240658.0</td>
      <td>Fine Art</td>
    </tr>
    <tr>
      <th>9976</th>
      <td>NaN</td>
      <td>False</td>
      <td>3</td>
      <td>0</td>
      <td>AAEAABirDGEz/r8CmOIGrPpe6UcwPtA7CS72NZoNGYcJv7VS</td>
      <td>What do you think that happens no where in the...</td>
      <td>224.0</td>
      <td>Indians On Quora</td>
      <td>130510.0</td>
      <td>India</td>
    </tr>
    <tr>
      <th>9977</th>
      <td>NaN</td>
      <td>False</td>
      <td>5</td>
      <td>200</td>
      <td>AAEAAJ6R9EimCjDPYlIpjOBYRQ7CPxobn9C/xCwurd+Zp67w</td>
      <td>I see a lot of bad science being reported on m...</td>
      <td>28.0</td>
      <td>Science Journalism</td>
      <td>614223.0</td>
      <td>Science</td>
    </tr>
    <tr>
      <th>9978</th>
      <td>NaN</td>
      <td>True</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAJo09/QYxyDglnKJuYngbSfA0py6jXkx5MiLJDEg61LK</td>
      <td>Who designed the Martini mobile app that conne...</td>
      <td>1115.0</td>
      <td>Meeting New People</td>
      <td>1115.0</td>
      <td>Meeting New People</td>
    </tr>
    <tr>
      <th>9979</th>
      <td>NaN</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAANkBVF9hcaNrU0AGLY4ynp8xGV+2ymgibGsTST8wFDIB</td>
      <td>Hosted or Privated Chef/Puppet?</td>
      <td>307.0</td>
      <td>Configuration Management</td>
      <td>307.0</td>
      <td>Configuration Management</td>
    </tr>
    <tr>
      <th>9980</th>
      <td>NaN</td>
      <td>False</td>
      <td>4</td>
      <td>0</td>
      <td>AAEAAGKwWCFpoJsi/ZnroU3MhPB11LLDfYAcG+u3sHyNfDfv</td>
      <td>How would you justify the wastage of water dur...</td>
      <td>252.0</td>
      <td>Holi</td>
      <td>252.0</td>
      <td>Holi</td>
    </tr>
    <tr>
      <th>9981</th>
      <td>NaN</td>
      <td>False</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAFrCnoffwcwVpDIsreCyT+XRzPuPt0wA5zpGaeKwh3AH</td>
      <td>Which processor is better for a graphic design...</td>
      <td>773218.0</td>
      <td>Technology</td>
      <td>2360.0</td>
      <td>Laptops</td>
    </tr>
    <tr>
      <th>9982</th>
      <td>NaN</td>
      <td>False</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAHz2iHfkZ17beSDLdG36mqh76iJxbvy7ltVq/hI3dxKN</td>
      <td>Is Warren Buffett investing in companies becau...</td>
      <td>167230.0</td>
      <td>Investing</td>
      <td>167230.0</td>
      <td>Investing</td>
    </tr>
    <tr>
      <th>9983</th>
      <td>NaN</td>
      <td>True</td>
      <td>3</td>
      <td>0</td>
      <td>AAEAAGizmW9Cn964/fe563GXfoIj8jhk98sNBvvobTYxWcj0</td>
      <td>I've failed my driving test twice: how can I p...</td>
      <td>2080.0</td>
      <td>Driving</td>
      <td>2080.0</td>
      <td>Driving</td>
    </tr>
    <tr>
      <th>9984</th>
      <td>NaN</td>
      <td>False</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAABuDHz22TdRoHnLiVYfQAyvh22kOL/zDvhoAJQoFUhRw</td>
      <td>Why are men's lacrosse and women's lacrosse so...</td>
      <td>322541.0</td>
      <td>Sports</td>
      <td>1892.0</td>
      <td>Lacrosse</td>
    </tr>
    <tr>
      <th>9985</th>
      <td>NaN</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAJKYJgaXheTZGWPCunOIbCsKbGcjD2w1ENUikQPDeNpA</td>
      <td>Because men are Human beings too do they deser...</td>
      <td>3551.0</td>
      <td>Human Rights</td>
      <td>3551.0</td>
      <td>Human Rights</td>
    </tr>
    <tr>
      <th>9986</th>
      <td>NaN</td>
      <td>True</td>
      <td>10</td>
      <td>0</td>
      <td>AAEAAFZUD0HT9fdkyj3oTR6Ew5cXqntfzyUQrAmPr3LiFRWu</td>
      <td>Why do many Tamils refer to themselves as Tam ...</td>
      <td>50.0</td>
      <td>Tamil Culture</td>
      <td>130510.0</td>
      <td>India</td>
    </tr>
    <tr>
      <th>9987</th>
      <td>NaN</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAMXz0j0UAWLFLPK8SmQN5flyCv8HfYnXj3Gc6IOuDM5B</td>
      <td>How to start a website developing business in ...</td>
      <td>526597.0</td>
      <td>Business</td>
      <td>526597.0</td>
      <td>Business</td>
    </tr>
    <tr>
      <th>9988</th>
      <td>NaN</td>
      <td>False</td>
      <td>29</td>
      <td>200</td>
      <td>AAEAAN2HZuSZhaIt/UgzCSRzjztTUJQ2rDMOZmSC6enp4XDt</td>
      <td>What is the best rebuttal to "If it ain't brok...</td>
      <td>364.0</td>
      <td>Disruption</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>9989</th>
      <td>NaN</td>
      <td>False</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAClprKo7msONt3rt+zrm/fP2ZkDg4AK6nBx6zMQkS0ky</td>
      <td>Facebook app center (versus) existing app stores?</td>
      <td>6404.0</td>
      <td>Google Play</td>
      <td>238905.0</td>
      <td>Facebook</td>
    </tr>
    <tr>
      <th>9990</th>
      <td>NaN</td>
      <td>False</td>
      <td>0</td>
      <td>50</td>
      <td>AAEAAJCL484wkZIHiRw1MLBu33qqxkTmuJGDIrEgW4vMnyqu</td>
      <td>Why do Bold fonts and Italics disappear when c...</td>
      <td>1915.0</td>
      <td>Google Drive</td>
      <td>3327.0</td>
      <td>Google Docs</td>
    </tr>
    <tr>
      <th>9991</th>
      <td>NaN</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAC72X8TZvRNVSYZ6hTMnTfpBaHPuFpodk23H3xfrK6NH</td>
      <td>Why does being cold make you catch a cold?</td>
      <td>382503.0</td>
      <td>Health and Wellness</td>
      <td>382503.0</td>
      <td>Health and Wellness</td>
    </tr>
    <tr>
      <th>9992</th>
      <td>NaN</td>
      <td>True</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAM10T++NbYU9JFMEb6QxUnhc5LJxtkOvG1HhfuxSpTg3</td>
      <td>What are the hottest under-10 person startups ...</td>
      <td>164924.0</td>
      <td>Lean Startups</td>
      <td>241809.0</td>
      <td>Startups</td>
    </tr>
    <tr>
      <th>9993</th>
      <td>NaN</td>
      <td>True</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAMPrlsjCEejyiB4x6Nmvi4yEMD5fUpJZo/WV/oc1L/Yk</td>
      <td>How do people from other countries think about...</td>
      <td>331.0</td>
      <td>Indonesian (language)</td>
      <td>331.0</td>
      <td>Indonesian (language)</td>
    </tr>
    <tr>
      <th>9994</th>
      <td>NaN</td>
      <td>False</td>
      <td>1</td>
      <td>200</td>
      <td>AAEAAFF/WfbxS4V2roHkQ+WEgqXcVXxTCljXMuGkslg7XtOq</td>
      <td>What's the average deal size of an Israeli M&amp;A?</td>
      <td>199.0</td>
      <td>Technology Industry in Israel</td>
      <td>199.0</td>
      <td>Technology Industry in Israel</td>
    </tr>
    <tr>
      <th>9995</th>
      <td>NaN</td>
      <td>False</td>
      <td>2</td>
      <td>0</td>
      <td>AAEAADHmhdj7ddMsxwLxIORZC+IfLoydhg+FXTwjBssCfemU</td>
      <td>What is the best career path to take to become...</td>
      <td>256.0</td>
      <td>Social Impact</td>
      <td>5652.0</td>
      <td>Social Entrepreneurship</td>
    </tr>
    <tr>
      <th>9996</th>
      <td>NaN</td>
      <td>True</td>
      <td>3</td>
      <td>0</td>
      <td>AAEAAF9ASgzjooq/SgiJg58it0eZ6PPYv8B/r4k6gH8VhdCG</td>
      <td>Is having a facial piercing considered unprofe...</td>
      <td>4801.0</td>
      <td>Engineering Recruiting</td>
      <td>10143.0</td>
      <td>Jobs</td>
    </tr>
    <tr>
      <th>9997</th>
      <td>NaN</td>
      <td>True</td>
      <td>3</td>
      <td>0</td>
      <td>AAEAAHaDxug0N4SrFLa65nyX0onGDf3C7DpBBF+u/4MTu9ye</td>
      <td>How does it feel to get sarcastically insulted...</td>
      <td>8017.0</td>
      <td>Sarcasm</td>
      <td>8017.0</td>
      <td>Sarcasm</td>
    </tr>
    <tr>
      <th>9998</th>
      <td>NaN</td>
      <td>True</td>
      <td>3</td>
      <td>60</td>
      <td>AAEAABtiDQZgE2LOLxtgl33uaw4xfhMDFqlWV4wVsE9H/Dcq</td>
      <td>Why did Kashmiri Hindus have to leave their ho...</td>
      <td>9393.0</td>
      <td>Islam</td>
      <td>2421.0</td>
      <td>Kashmir</td>
    </tr>
    <tr>
      <th>9999</th>
      <td>NaN</td>
      <td>False</td>
      <td>3</td>
      <td>0</td>
      <td>AAEAAL8xXMY8ocS5aVwA+01/21LZ29oejC1zVXl2eS+a+9J6</td>
      <td>Is it possible to become a successful venture ...</td>
      <td>64684.0</td>
      <td>Venture Capital</td>
      <td>64684.0</td>
      <td>Venture Capital</td>
    </tr>
  </tbody>
</table>
<p style="font-family: monospace;">1000 rows × 10 columns</p>
</div>
</div>

```python
train = cleaned_df[:9000]
```
<div class="j">
<div class="j_pre">
    <p>train </p>
</div>

<div class="j_table" style="height: 20em">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>__ans__</th>
      <th>anonymous</th>
      <th>num_answers</th>
      <th>promoted_to</th>
      <th>question_key</th>
      <th>question_text</th>
      <th>topics</th>
      <th>context_topic_followers</th>
      <th>context_topic_name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2.089474</td>
      <td>False</td>
      <td>4</td>
      <td>0</td>
      <td>AAEAAM9EY6LIJsEFvYiwKLfCe7d+hkbsXJ5qM7aSwTqemERp</td>
      <td>What are some movies with mind boggling twists...</td>
      <td>[{'followers': 500022, 'name': 'Movies'}]</td>
      <td>500022.0</td>
      <td>Movies</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2.692308</td>
      <td>False</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAHM6f92B9jt43/y913/J7ce8vtE6Jn9LLcy3yK2RHFGD</td>
      <td>How do you prepare a 10 year old for Internati...</td>
      <td>[{'followers': 179, 'name': 'International Mat...</td>
      <td>179.0</td>
      <td>International Mathematical Olympiad (IMO)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>4.612903</td>
      <td>False</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAGzietPvCHLFvaKCjng43iiIeo9gAWnJlSrs+12uYtZ0</td>
      <td>Why do bats sleep upside down?</td>
      <td>[{'followers': 614223, 'name': 'Science'}]</td>
      <td>614223.0</td>
      <td>Science</td>
    </tr>
    <tr>
      <th>3</th>
      <td>8.051948</td>
      <td>False</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAM2Sk4U3y4We5TELJXRQgIf6yit5DbbdBw6BCRvuFrcY</td>
      <td>Tell me everything about the Leidenfrost effec...</td>
      <td>[{'followers': 614223, 'name': 'Science'}, {'f...</td>
      <td>614223.0</td>
      <td>Science</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.150943</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAALb43r/fn9KRKqJ0itd3NGbqZZSZalzi7vaulLxNGzeL</td>
      <td>Is the Nexus 10 any good despite the dual core...</td>
      <td>[{'followers': 1536, 'name': 'Android Tablets'}]</td>
      <td>1536.0</td>
      <td>Android Tablets</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0.084507</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAABSJk8mrfAjuQuxEI6PV2rfGGcnuq/I5JyE0VJvYSoRp</td>
      <td>Is smartphone app download duplication account...</td>
      <td>[{'followers': 91, 'name': 'Smartphone Applica...</td>
      <td>91.0</td>
      <td>Smartphone Applications</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1.073944</td>
      <td>True</td>
      <td>1</td>
      <td>200</td>
      <td>AAEAANLGb0hKlFULx6BPXvVvHQ1SJ2jJTqDCVighUGs/ZDya</td>
      <td>Are there any CEO's that go by a different nam...</td>
      <td>[{'followers': 526597, 'name': 'Business'}, {'...</td>
      <td>241809.0</td>
      <td>Startups</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0.007299</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAJinkGKp/YRlvWhFbapFNzPkD7coBuf4QRwsmN/c3q7Z</td>
      <td>Guys who love being sissies?</td>
      <td>[{'followers': 289, 'name': 'Needs to Be Clear...</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>8</th>
      <td>0.298893</td>
      <td>True</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAB0zSrCTxmdSYxoC1KD0HYeDwTzOyK2lBiXzuZwhhmgn</td>
      <td>How does one successfully become a conscientio...</td>
      <td>[{'followers': 59, 'name': 'Pacifism'}, {'foll...</td>
      <td>3006.0</td>
      <td>Warfare</td>
    </tr>
    <tr>
      <th>9</th>
      <td>30.614035</td>
      <td>False</td>
      <td>3</td>
      <td>0</td>
      <td>AAEAAA7uRr9KzVD3rd+L3OptfwxaWFYfE6D8Wqskxx+ZkfGb</td>
      <td>How do people in countries with regular and la...</td>
      <td>[{'followers': 224, 'name': 'Boston Marathon T...</td>
      <td>224.0</td>
      <td>Boston Marathon Terrorist Attacks (April 2013)</td>
    </tr>
    <tr>
      <th>10</th>
      <td>24.250000</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAFHJ+5Ozj1TQKLUwzB/pIwiXMwtTJip8NQO1jCCyRS5a</td>
      <td>How do top entrepreneurs/CEOs stay organized a...</td>
      <td>[{'followers': 321001, 'name': 'Entrepreneursh...</td>
      <td>4155.0</td>
      <td>Organization</td>
    </tr>
    <tr>
      <th>11</th>
      <td>1.666667</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAM3WIWO/Ct/u5SvzPHfwwrHNWrPgHd8Ch7/QdM6CjfZL</td>
      <td>What companies/independent developers located ...</td>
      <td>[{'followers': 199, 'name': 'Technology Indust...</td>
      <td>304.0</td>
      <td>Web Development on Mac OS X</td>
    </tr>
    <tr>
      <th>12</th>
      <td>0.006231</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAI0XKapeO3+B75zZYu1Kjv5SGn+Mm/4qT3rBNJrDNKce</td>
      <td>How can I call and talk with President Obama o...</td>
      <td>[{'followers': 1240, 'name': 'U.S. Presidents'}]</td>
      <td>3292.0</td>
      <td>United States Governments (Federal, State, Local)</td>
    </tr>
    <tr>
      <th>13</th>
      <td>0.575972</td>
      <td>True</td>
      <td>1</td>
      <td>100</td>
      <td>AAEAAMRnMKIzVIfH55p+aKXak1M0vBd2m7xNKm5KttYyVspo</td>
      <td>What do parents wish they had known before hav...</td>
      <td>[{'followers': 26, 'name': 'Deciding Whether t...</td>
      <td>13867.0</td>
      <td>Parenting</td>
    </tr>
    <tr>
      <th>14</th>
      <td>5.220859</td>
      <td>False</td>
      <td>2</td>
      <td>0</td>
      <td>AAEAAFsd5v511xAQgtEaPDHb+fYF8YSJnsRuMd7715LVQQPf</td>
      <td>How much time does it take to go from DNA to p...</td>
      <td>[{'followers': 3693, 'name': 'Medical Research...</td>
      <td>72820.0</td>
      <td>Biology</td>
    </tr>
    <tr>
      <th>15</th>
      <td>0.079755</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAOnLQc1KPDw0pqCMpROkdovdRCzJlTsI5nDo9vxjcOm+</td>
      <td>What % of tablet users use a stylus?</td>
      <td>[{'followers': 2049, 'name': 'Tablet Devices a...</td>
      <td>2049.0</td>
      <td>Tablet Devices and Tablet Market</td>
    </tr>
    <tr>
      <th>16</th>
      <td>1.075758</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAI0A/7ASkze8LKdTLRwCFtUcFZ2xIpNnS55Zk1xtCwXN</td>
      <td>Which company make awesome luxury coach?</td>
      <td>[{'followers': 999, 'name': 'Luxury'}, {'follo...</td>
      <td>999.0</td>
      <td>Luxury</td>
    </tr>
    <tr>
      <th>17</th>
      <td>0.717557</td>
      <td>False</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAKKuKEgb+lBKi1iEEST89zLKFr/88CUAwC9xeCdGYss1</td>
      <td>I Require  professional and creative web desig...</td>
      <td>[{'followers': 39744, 'name': 'Web Development'}]</td>
      <td>39744.0</td>
      <td>Web Development</td>
    </tr>
    <tr>
      <th>18</th>
      <td>1.622010</td>
      <td>False</td>
      <td>2</td>
      <td>0</td>
      <td>AAEAAJloaH19a32hbe1Cs0wdW8WWptHbbaa5JG7dKbMwvtaJ</td>
      <td>What are the short/medium prospects of service...</td>
      <td>[{'followers': 11842, 'name': 'Internet Advert...</td>
      <td>3566.0</td>
      <td>Online Video</td>
    </tr>
    <tr>
      <th>19</th>
      <td>3.388889</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAFHbVuHDWmLslFWH8UZMnN7M5VHntFVDpY19+3ndhmy5</td>
      <td>Why can't we create something like game of thr...</td>
      <td>[{'followers': 268430, 'name': 'Television Ser...</td>
      <td>268430.0</td>
      <td>Television Series</td>
    </tr>
    <tr>
      <th>20</th>
      <td>11.457143</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAKkkEpxZp4qBn45yEhzGTnJNhB5e/Mppn75kx1on5ajr</td>
      <td>Would Noodle make other education lead generat...</td>
      <td>[{'followers': 167176, 'name': 'Business Model...</td>
      <td>74.0</td>
      <td>Noodle Education</td>
    </tr>
    <tr>
      <th>21</th>
      <td>1.013889</td>
      <td>False</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAABu6kZw48xJyS1xXW3RjQ0xANWgzwjp6LLVTryDWUM8I</td>
      <td>ORM is all too often seen simply as minimising...</td>
      <td>[{'followers': 490, 'name': 'Online Reputation...</td>
      <td>490.0</td>
      <td>Online Reputation</td>
    </tr>
    <tr>
      <th>22</th>
      <td>0.389474</td>
      <td>True</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAANJFMVE7CEr6gX3LEC9NWGz0a65TjFd1AjHMY9CWujBJ</td>
      <td>What are some of the funniest noises you have ...</td>
      <td>[{'followers': 67265, 'name': 'Sex'}, {'follow...</td>
      <td>264984.0</td>
      <td>Dating and Relationships</td>
    </tr>
    <tr>
      <th>23</th>
      <td>0.319231</td>
      <td>False</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAKTTk5EUSD+0CPGP0gLS0aBQR97xe0p2+DPRX+ddtoV2</td>
      <td>Was Moses talking about humans becoming fully ...</td>
      <td>[{'followers': 2525, 'name': 'Theology'}, {'fo...</td>
      <td>6608.0</td>
      <td>Science and Religion</td>
    </tr>
    <tr>
      <th>24</th>
      <td>4.000000</td>
      <td>False</td>
      <td>4</td>
      <td>200</td>
      <td>AAEAAFJxWj/4n1NhOgwpS0d5X7tcBia4jBtjrfw6YxNcYXDr</td>
      <td>'Scarcely a human freedom has been obtained wi...</td>
      <td>[{'followers': 4332, 'name': 'Occupy Movement'...</td>
      <td>1707.0</td>
      <td>Democracy</td>
    </tr>
    <tr>
      <th>25</th>
      <td>0.174194</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAH+k9PIQg6lOxHRIqXYE5u18aWO4QqJZTtdahJw9+grv</td>
      <td>Do actors get paid after they do the movie or ...</td>
      <td>[{'followers': 2450, 'name': 'Wanting and Maki...</td>
      <td>2450.0</td>
      <td>Wanting and Making Money</td>
    </tr>
    <tr>
      <th>26</th>
      <td>0.678322</td>
      <td>True</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAG1dK7AL02j78ugkPaKf5TiFb6iBCaIICXxUd5OYpQYz</td>
      <td>Private Equity in relation to Movie Industry,c...</td>
      <td>[{'followers': 3567, 'name': 'Private Equity'}...</td>
      <td>3567.0</td>
      <td>Private Equity</td>
    </tr>
    <tr>
      <th>27</th>
      <td>1.007246</td>
      <td>False</td>
      <td>2</td>
      <td>120</td>
      <td>AAEAAA15xfSTbyLayedu+P4P6upyw1gZisruGcd5sBeOiO2w</td>
      <td>What language was used by Mark Zuckerberg to w...</td>
      <td>[{'followers': 238905, 'name': 'Facebook'}]</td>
      <td>238905.0</td>
      <td>Facebook</td>
    </tr>
    <tr>
      <th>28</th>
      <td>46.333333</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAALJZ+JSLst8wPq+EApQqSTqcCW9iaHg7OWcdwg+xJEVo</td>
      <td>How employees at Facebook avoid procrastinatio...</td>
      <td>[{'followers': 238905, 'name': 'Facebook'}]</td>
      <td>238905.0</td>
      <td>Facebook</td>
    </tr>
    <tr>
      <th>29</th>
      <td>0.615385</td>
      <td>True</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAETPeE1zK+aIF5IeOLu2EzMwzSM22EM4i14ZnwDTU17t</td>
      <td>Was Glenn Gould a fast typist?</td>
      <td>[{'followers': 986, 'name': 'Glenn Gould'}]</td>
      <td>986.0</td>
      <td>Glenn Gould</td>
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
      <th>8970</th>
      <td>0.044610</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAKy7JJbuCln18hVXHhNCqaW/2t4BOJekq4EoJZeZlQAf</td>
      <td>When was the Quora application launched for An...</td>
      <td>[{'followers': 3899, 'name': 'Quora (company)'}]</td>
      <td>3899.0</td>
      <td>Quora (company)</td>
    </tr>
    <tr>
      <th>8971</th>
      <td>0.028571</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAIPL+PIfMKP6q/d3vCLPeN6qaE5wVTv9jyLyP46riITO</td>
      <td>1A group element to show flame test why?</td>
      <td>[{'followers': 13, 'name': 'Flame Effects'}, {...</td>
      <td>53.0</td>
      <td>The Flame</td>
    </tr>
    <tr>
      <th>8972</th>
      <td>0.356364</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAFbZen2AwK+cWhTuxYq89OYYGUdvPKxObTA49ZZY5EoF</td>
      <td>Hire best webdevelopers to design and maintain...</td>
      <td>[{'followers': 39744, 'name': 'Web Development'}]</td>
      <td>39744.0</td>
      <td>Web Development</td>
    </tr>
    <tr>
      <th>8973</th>
      <td>101.000000</td>
      <td>True</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAEKenPCvJnNPSn3GoyxJGHOgO9Ft1sXtcQIRtzX36ls7</td>
      <td>Is it absolutely necessary to have a lot in co...</td>
      <td>[{'followers': 264984, 'name': 'Dating and Rel...</td>
      <td>264984.0</td>
      <td>Dating and Relationships</td>
    </tr>
    <tr>
      <th>8974</th>
      <td>0.424658</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAJ6h5w42SdKWHM1ba0mDrLXfRNF35x5J1ImAXc5k7r9g</td>
      <td>What civilization had the earliest written num...</td>
      <td>[{'followers': 1460, 'name': 'Numbers'}, {'fol...</td>
      <td>0.0</td>
      <td>Numeration</td>
    </tr>
    <tr>
      <th>8975</th>
      <td>1.787879</td>
      <td>True</td>
      <td>2</td>
      <td>0</td>
      <td>AAEAAI0sA6MSlU+uVCoRL9a3wpk5dt0Mnwy/8h0l2jkqoE9K</td>
      <td>I am never contacted by recruiters (I don't ha...</td>
      <td>[{'followers': 7928, 'name': 'Recruiting'}]</td>
      <td>7928.0</td>
      <td>Recruiting</td>
    </tr>
    <tr>
      <th>8976</th>
      <td>0.183673</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAHQncZgikc5wl/qwturXJ89iVLVLNrUJNGpMfiSGkZLK</td>
      <td>What is finance software?</td>
      <td>[{'followers': 4729, 'name': 'Software'}]</td>
      <td>4729.0</td>
      <td>Software</td>
    </tr>
    <tr>
      <th>8977</th>
      <td>0.118902</td>
      <td>True</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAIo1t0EvlXTB3vDMjt4DCsJeG6U43ZF/aD8SiMmlmof0</td>
      <td>What impact do foreign currency fluctuations h...</td>
      <td>[{'followers': 93447, 'name': 'Amazon'}]</td>
      <td>93447.0</td>
      <td>Amazon</td>
    </tr>
    <tr>
      <th>8978</th>
      <td>0.494382</td>
      <td>True</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAACkrgngYk0Gq2+2Jm/c+yfNG3tFyUrZAUpEsbvfAnuAo</td>
      <td>What are some tips to write a great sales copy?</td>
      <td>[{'followers': 4859, 'name': 'Sales'}, {'follo...</td>
      <td>4859.0</td>
      <td>Sales</td>
    </tr>
    <tr>
      <th>8979</th>
      <td>2.323699</td>
      <td>True</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAKF/1Zvtgov2mB0Ji4jg1/0aE+lQkKxIEhQuLX8yuwzY</td>
      <td>Is it a good idea to use technical jargon when...</td>
      <td>[{'followers': 13108, 'name': 'Career Advice'}]</td>
      <td>13108.0</td>
      <td>Career Advice</td>
    </tr>
    <tr>
      <th>8980</th>
      <td>0.044944</td>
      <td>True</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAN2Q0FsMgTWonNul3FPog92E1XkMDr+1QcPsIivJyv/z</td>
      <td>What's it like to work at Done Genetics?</td>
      <td>[{'followers': 3, 'name': 'Done Genetics'}]</td>
      <td>3.0</td>
      <td>Done Genetics</td>
    </tr>
    <tr>
      <th>8981</th>
      <td>2.752941</td>
      <td>True</td>
      <td>3</td>
      <td>0</td>
      <td>AAEAAPJKiEWYD6YX59MhOtX8+DnzOqzul+SpdL2Xnl0KJoxx</td>
      <td>Where can I connect with people in general in ...</td>
      <td>[{'followers': 6467, 'name': 'Colleges and Uni...</td>
      <td>20107.0</td>
      <td>Mumbai, Maharashtra, India</td>
    </tr>
    <tr>
      <th>8982</th>
      <td>0.011662</td>
      <td>True</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAOZFkLJxzpcxaMgr6sM3pBQcOKWQq+ykds/w3GXXgZqo</td>
      <td>Have anyone any concern or recommendation for ...</td>
      <td>[{'followers': 289, 'name': 'Needs to Be Clear...</td>
      <td>3168.0</td>
      <td>Dublin, Ireland</td>
    </tr>
    <tr>
      <th>8983</th>
      <td>0.133929</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAFxvsqu2YCjgWfq2u+anT+J0OXvtANXSu+elB7CuM1G2</td>
      <td>How can I tell facebook to stop emailing me?</td>
      <td>[{'followers': 238905, 'name': 'Facebook'}]</td>
      <td>238905.0</td>
      <td>Facebook</td>
    </tr>
    <tr>
      <th>8984</th>
      <td>0.207373</td>
      <td>True</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAHaEN4jlW0hrNd+wj1Xx9bWV2XiE0+5kkBbpk0cS/XCD</td>
      <td>Who does cover hound use to provide auto rate ...</td>
      <td>[{'followers': 809, 'name': 'Auto Insurance'}]</td>
      <td>809.0</td>
      <td>Auto Insurance</td>
    </tr>
    <tr>
      <th>8985</th>
      <td>0.662722</td>
      <td>True</td>
      <td>2</td>
      <td>0</td>
      <td>AAEAAO10zYF1GIDQenYcsr4KV8T5f53wX0d491daxFrxJfxE</td>
      <td>Is Wuthering Heights a feminist text?</td>
      <td>[{'followers': 590279, 'name': 'Books'}, {'fol...</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>8986</th>
      <td>4.123288</td>
      <td>False</td>
      <td>1</td>
      <td>200</td>
      <td>AAEAAIO1CRxV7oxl//ejIL5mkNUJqShjAlSNiOaXEiQFChq+</td>
      <td>How will better audience data change the movie...</td>
      <td>[{'followers': 1, 'name': 'User Data'}, {'foll...</td>
      <td>500022.0</td>
      <td>Movies</td>
    </tr>
    <tr>
      <th>8987</th>
      <td>0.352941</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAAOKeIMX2TbFS2CdHXtBWs++0egdI5p4FB2TZTlLyph7D</td>
      <td>In what condition to descendants of royalty live?</td>
      <td>[{'followers': 361, 'name': 'Royalty'}]</td>
      <td>361.0</td>
      <td>Royalty</td>
    </tr>
    <tr>
      <th>8988</th>
      <td>0.488525</td>
      <td>False</td>
      <td>0</td>
      <td>400</td>
      <td>AAEAALlzDzfzi6ULFlDFEbxjfkQTdJRmyDeB7Oz343GX3Zzz</td>
      <td>Will the developing world be more innovative d...</td>
      <td>[{'followers': 24985, 'name': 'Innovation'}, {...</td>
      <td>24985.0</td>
      <td>Innovation</td>
    </tr>
    <tr>
      <th>8989</th>
      <td>6.908078</td>
      <td>False</td>
      <td>5</td>
      <td>0</td>
      <td>AAEAAJj02/B0zzfVYoV/ujoCNopCXWyQ1l1joancxgjHTOvc</td>
      <td>Are there living samurai in this age?</td>
      <td>[{'followers': 12423, 'name': 'Japan'}, {'foll...</td>
      <td>782.0</td>
      <td>Samurai</td>
    </tr>
    <tr>
      <th>8990</th>
      <td>2.077295</td>
      <td>False</td>
      <td>1</td>
      <td>200</td>
      <td>AAEAAF3/vH5TKSTV79CQ2wWCe7UPuPUQ7GqKZ+zy/WZpyWTS</td>
      <td>How is it like to hire a life coach or persona...</td>
      <td>[{'followers': 11233, 'name': 'Life Advice'}, ...</td>
      <td>20706.0</td>
      <td>Self-Improvement</td>
    </tr>
    <tr>
      <th>8991</th>
      <td>0.490798</td>
      <td>False</td>
      <td>2</td>
      <td>400</td>
      <td>AAEAABeFQkBTvekTH2LwM9QRMeaYrb43rq4KQTB3ixkjIUW6</td>
      <td>What happens to my links if I disconnect my cu...</td>
      <td>[{'followers': 1483, 'name': 'Bitly'}, {'follo...</td>
      <td>13867.0</td>
      <td>Web Applications</td>
    </tr>
    <tr>
      <th>8992</th>
      <td>0.197080</td>
      <td>False</td>
      <td>2</td>
      <td>0</td>
      <td>AAEAAG1ehKZlxWfZ3u8Ty8xu5l2N+kghEcv62oITGGiqZj62</td>
      <td>Can Curiosity record sound?</td>
      <td>[{'followers': 295, 'name': 'Curiosity (Mars R...</td>
      <td>295.0</td>
      <td>Curiosity (Mars Rover)</td>
    </tr>
    <tr>
      <th>8993</th>
      <td>3.375000</td>
      <td>False</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAITna52dCVn2KCVY3+xhMh4dbliiirA/gdi74f+v2zyG</td>
      <td>Is it realistic to think you can get a decent ...</td>
      <td>[{'followers': 6423, 'name': 'Germany'}]</td>
      <td>6423.0</td>
      <td>Germany</td>
    </tr>
    <tr>
      <th>8994</th>
      <td>0.482143</td>
      <td>False</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAB8MgrMHBPrcEd+z8uWtFtE0jU1VhHBx4bSj3dLaMait</td>
      <td>Can someone explain to me how the Syrian war b...</td>
      <td>[{'followers': 854, 'name': 'Syria'}, {'follow...</td>
      <td>0.0</td>
      <td>Syrian War</td>
    </tr>
    <tr>
      <th>8995</th>
      <td>2.850000</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAANESwEBxHZJt2IIhT1/YgqvPOOHySqU0TPMHvefVX6Rn</td>
      <td>How old are the wolves growly pants and truck ...</td>
      <td>[{'followers': 10123, 'name': 'Animals'}, {'fo...</td>
      <td>10123.0</td>
      <td>Animals</td>
    </tr>
    <tr>
      <th>8996</th>
      <td>0.274648</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAABT6OZZisoRkeAFNxYWAssLUu45g4iNjeQiaAEVkPa2Z</td>
      <td>What is the best Agenda (notes, gant charts, C...</td>
      <td>[{'followers': 7149, 'name': 'iPad Application...</td>
      <td>7149.0</td>
      <td>iPad Applications</td>
    </tr>
    <tr>
      <th>8997</th>
      <td>3.146893</td>
      <td>True</td>
      <td>1</td>
      <td>0</td>
      <td>AAEAAAUdDfIgSIYlFRrY4jOKvHsvSLmlKjyZ576oP8a+L694</td>
      <td>December 2012: Are all the Hobbit films alread...</td>
      <td>[{'followers': 18741, 'name': 'The Hobbit (193...</td>
      <td>146.0</td>
      <td>The Hobbit (movie series)</td>
    </tr>
    <tr>
      <th>8998</th>
      <td>0.131086</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>AAEAADeVLG2awg6hG+dTiPKRiII64khN7fS/OeFLek4lg2QQ</td>
      <td>Rental Navigator asks...If you are a real esta...</td>
      <td>[{'followers': 7260, 'name': 'Real Estate'}]</td>
      <td>7260.0</td>
      <td>Real Estate</td>
    </tr>
    <tr>
      <th>8999</th>
      <td>2.738739</td>
      <td>False</td>
      <td>2</td>
      <td>0</td>
      <td>AAEAAKeoz9V34X6kt6Iq+ywD6JKTBkJxointHniuxnF2ejHa</td>
      <td>Energy of motion turns into mass?</td>
      <td>[{'followers': 199773, 'name': 'Physics'}]</td>
      <td>199773.0</td>
      <td>Physics</td>
    </tr>
  </tbody>
</table>
<p style="font-family: monospace;">9000 rows × 10 columns</p>
</div>
</div>


The notebook can be found [here.](https://github.com/ydkahin/jupyter-notebooks/blob/master/notebooks/quora-views-challenge/quora-views_data_import.ipynb) (*Save Link As*)

In the next post, we will do an exploratory analysis. 