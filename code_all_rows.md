
# What 300,000 Kickstarter Campaigns Tell Us About Crowdfunding

# Background:

* Kickstarter is a crowdfunding website that raises money for all kinds of campaigns. During a set period of time, backers contribute to the campaigns of their choice. The all-or-nothing rule adds to the uncertainty of the outcomes-- you either get all the money, or you get nothing if the goal is not reached before the deadline you have set.
* Why do some kickstarter campaigns succede whereas others don't? What factors play important roles in the funds that kickstarter campaigns raise? More than 300,000+ kickstarter campaigns were examined, for a closer look at the relationships between different parameters and the outcomes.
* If a kickstarter campaign is planned to launch, which month of the year/ day of the week/ time in the day would be a good choice? How long should the campaign last for? How many would be good for the limit of backers? Which category and sub-category are most popular, and which should it be listed under after the product is decided? What would be a reasonable goal for the fund-raising? These are all the questions that an careful analysis of the dataset can lead to better answers to.
* Although the pledged amount of many failed campaigns were not zero dollars in this dataset, however, starters of the campaigns did not receive any in the end if their campaigns were not successful. Thus only the pledged amount of successful campaigns were meaningful than that of other campaigns -- they were substantial amount of dollars that arrived in fundraisers' bank accounts, instead of just numbers.
* Since the all or nothing scenario is an important feature of kickstarter 
campaigns, more emphasis would be focused on relationships between other parameters and different outcomes (e.g., successful), although some analysis were also performed upon that between other parameters and the amount of raised funds.

# Resources:

* The data was downloaded as a csv file from the kaggle dataset: "Kickstarter projects".
* Every campaign had a unique ID to it.
* Campaigns ended in as early as May 2009, and as late as March, 2013.
* The csv file contains data of 378,661 campaigns, among which 292,627 campaigns were created in the U.S.
* Live campaigns were around Janurary 2017, implicating the data collection was around silimar time.

# Note:

* Only U.S. campaigns were researched in this project.
* Without specification, the units of pledged and goals are in US dollars.
* Pledged amount and goal amount were both turned into log numbers in the following plots.
* Unless otherwise specified, all the date related variables plotted were related to the end dates of campaigns, given that last-minutes contributions were often made by consumers. The time related to campaigns were their launch time.


```python
%matplotlib inline
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib
from matplotlib import style
import matplotlib.ticker as mtick
from pylab import rcParams
import seaborn as sns; sns.set(style="white", color_codes=True)
from datetime import date
import numpy as np
import pylab as pl
```


```python
colors = ["#adc2eb", "#3366cc", "#1f3d7a", "#000000", "#7094db"]
single_color = "#adc2eb"
figsize = (9, 6)
big_figsize = (15, 8)
fontsize = 12
markersize = 5
edgecolor = "black"
linewidth = 0.5
bbox_to_anchor = (0.45,-0.17)
ncol = 5
#style.use('fivethirtyeight')
matplotlib.rcParams.update({'font.size': fontsize})
```


```python
df = pd.read_csv("raw_data.csv")
#df = df.sample(100)
```


```python
df = df.sort_values(by=["deadline"], ascending=True).reset_index(drop=True)
del(df["usd pledged"], df["pledged"], df["goal"], df["currency"])
df = df[df["country"]=="US"]
```


```python
df = df.rename(columns={"category":"sub_category", "usd_pledged_real":"pledged", "usd_goal_real":"goal", "deadline":"end", "state":"outcome", "launched":"launch"})
df = df.fillna(value=0)
```


```python
df["launch_full_date"] = pd.to_datetime(df["launch"])
df["end_full_date"] = pd.to_datetime(df["end"])
df["duration"] = [int(i.days) for i in (df["end_full_date"] - df["launch_full_date"])]

df["launch_year"] = pd.DatetimeIndex(df["launch"]).year
df["launch_month"] = pd.DatetimeIndex(df["launch"]).month
df["launch_date"] = pd.DatetimeIndex(df["launch"]).day
df["launch_day"] = pd.DatetimeIndex(df["launch"]).dayofweek

df["launch_time"] = df["launch"].str.split(" ", expand=True)[1].str.split(":", expand=True)[0]

df["end_year"] = pd.DatetimeIndex(df["end"]).year
df["end_month"] = pd.DatetimeIndex(df["end"]).month
df["end_date"] = pd.DatetimeIndex(df["end"]).day
df["end_day"] = pd.DatetimeIndex(df["end"]).dayofweek

del(df["launch"], df["ID"], df["country"], df["launch_full_date"], df["end_full_date"])

columns = ["name", "sub_category", "main_category", "outcome", "backers",
       "country", "pledged", "goal", "launch_full_date", "end_full_date",
       "duration", "launch_year", "launch_month", "launch_date", "launch_day",
       "launch_time", "end_year", "end_month", "end_date", "end_day"]

df["pledged_to_goal"] = df["pledged"] / df["goal"] * 100
df["pledged_to_goal"] = df["pledged_to_goal"].astype(int)

for i in ["backers", "pledged", "goal"]:
    df[i] = pd.to_numeric(df[i])

for i in ["launch_year", "launch_month", "launch_date", "launch_day",
          "launch_time", "end_year", "end_month", "end_date", "end_day"]:
     df[i] = df[i].astype("int")
```


```python
quarter_list = []
for month in df["end_month"].tolist():
    if month in [1,2,3]:
        quarter_list.append("Q1")
    elif month in [4,5,6]:
        quarter_list.append("Q2")
    elif month in [7,8,9]:
        quarter_list.append("Q3")
    elif month in [10,11,12]:
        quarter_list.append("Q4")
df["end_quarter"] = quarter_list
```


```python
launch_time = df["launch_time"].values
am_pm = []
for i in range(len(launch_time)):
    if(launch_time[i]<12):
        am_pm.append("am")
    else:
        am_pm.append("pm")
df["am_pm"] = am_pm
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>sub_category</th>
      <th>main_category</th>
      <th>end</th>
      <th>outcome</th>
      <th>backers</th>
      <th>pledged</th>
      <th>goal</th>
      <th>duration</th>
      <th>launch_year</th>
      <th>...</th>
      <th>launch_date</th>
      <th>launch_day</th>
      <th>launch_time</th>
      <th>end_year</th>
      <th>end_month</th>
      <th>end_date</th>
      <th>end_day</th>
      <th>pledged_to_goal</th>
      <th>end_quarter</th>
      <th>am_pm</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>drawing for dollars</td>
      <td>Illustration</td>
      <td>Art</td>
      <td>2009-05-03</td>
      <td>successful</td>
      <td>3</td>
      <td>35.0</td>
      <td>20.0</td>
      <td>8</td>
      <td>2009</td>
      <td>...</td>
      <td>24</td>
      <td>4</td>
      <td>21</td>
      <td>2009</td>
      <td>5</td>
      <td>3</td>
      <td>6</td>
      <td>175</td>
      <td>Q2</td>
      <td>pm</td>
    </tr>
    <tr>
      <th>1</th>
      <td>New York Makes a Book!!</td>
      <td>Journalism</td>
      <td>Journalism</td>
      <td>2009-05-16</td>
      <td>successful</td>
      <td>110</td>
      <td>3329.0</td>
      <td>3000.0</td>
      <td>17</td>
      <td>2009</td>
      <td>...</td>
      <td>28</td>
      <td>1</td>
      <td>13</td>
      <td>2009</td>
      <td>5</td>
      <td>16</td>
      <td>5</td>
      <td>110</td>
      <td>Q2</td>
      <td>pm</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Sponsor Dereck Blackburn (Lostwars) Artist in ...</td>
      <td>Rock</td>
      <td>Music</td>
      <td>2009-05-16</td>
      <td>failed</td>
      <td>2</td>
      <td>15.0</td>
      <td>300.0</td>
      <td>16</td>
      <td>2009</td>
      <td>...</td>
      <td>29</td>
      <td>2</td>
      <td>5</td>
      <td>2009</td>
      <td>5</td>
      <td>16</td>
      <td>5</td>
      <td>5</td>
      <td>Q2</td>
      <td>am</td>
    </tr>
    <tr>
      <th>3</th>
      <td>"All We Had" Gets Into Cannes -- $10 or More G...</td>
      <td>Documentary</td>
      <td>Film &amp; Video</td>
      <td>2009-05-20</td>
      <td>failed</td>
      <td>4</td>
      <td>40.0</td>
      <td>300.0</td>
      <td>19</td>
      <td>2009</td>
      <td>...</td>
      <td>30</td>
      <td>3</td>
      <td>22</td>
      <td>2009</td>
      <td>5</td>
      <td>20</td>
      <td>2</td>
      <td>13</td>
      <td>Q2</td>
      <td>pm</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Mr. Squiggles</td>
      <td>Illustration</td>
      <td>Art</td>
      <td>2009-05-22</td>
      <td>failed</td>
      <td>0</td>
      <td>0.0</td>
      <td>30.0</td>
      <td>9</td>
      <td>2009</td>
      <td>...</td>
      <td>12</td>
      <td>1</td>
      <td>23</td>
      <td>2009</td>
      <td>5</td>
      <td>22</td>
      <td>4</td>
      <td>0</td>
      <td>Q2</td>
      <td>pm</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 21 columns</p>
</div>



# General Stats

* Funds raised of campaigns:
* A campaign named "Pebble Time - Awesome Smartwatch, No Compromises", which ended in March 2015, raised the most funds($20,338,986).
* Top 3 campaigns were all in the "Product Design" sub-category under the "Design" category.

* Backers for campaigns:
* A tabletop game named "Exploding Kittens" had the greatest number of backers(219,382). The difference of backer quantities was drastic among campaigns.


```python
describe_df = df.describe()
describe_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>backers</th>
      <th>pledged</th>
      <th>goal</th>
      <th>duration</th>
      <th>launch_year</th>
      <th>launch_month</th>
      <th>launch_date</th>
      <th>launch_day</th>
      <th>launch_time</th>
      <th>end_year</th>
      <th>end_month</th>
      <th>end_date</th>
      <th>end_day</th>
      <th>pledged_to_goal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>292627.000000</td>
      <td>2.926270e+05</td>
      <td>2.926270e+05</td>
      <td>292627.000000</td>
      <td>292627.000000</td>
      <td>292627.000000</td>
      <td>292627.000000</td>
      <td>292627.000000</td>
      <td>292627.000000</td>
      <td>292627.000000</td>
      <td>292627.000000</td>
      <td>292627.000000</td>
      <td>292627.000000</td>
      <td>2.926270e+05</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>113.078615</td>
      <td>9.670193e+03</td>
      <td>4.403497e+04</td>
      <td>33.740954</td>
      <td>2013.935372</td>
      <td>6.408284</td>
      <td>15.259665</td>
      <td>2.426745</td>
      <td>12.928612</td>
      <td>2014.006947</td>
      <td>6.693801</td>
      <td>15.178049</td>
      <td>3.176433</td>
      <td>3.769142e+02</td>
    </tr>
    <tr>
      <th>std</th>
      <td>985.723400</td>
      <td>9.932942e+04</td>
      <td>1.108372e+06</td>
      <td>68.025055</td>
      <td>1.983963</td>
      <td>3.311575</td>
      <td>8.795847</td>
      <td>1.759184</td>
      <td>7.921125</td>
      <td>1.971462</td>
      <td>3.313367</td>
      <td>9.029737</td>
      <td>1.953617</td>
      <td>3.013830e+04</td>
    </tr>
    <tr>
      <th>min</th>
      <td>0.000000</td>
      <td>0.000000e+00</td>
      <td>1.000000e-02</td>
      <td>0.000000</td>
      <td>1970.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>2009.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.000000e+00</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>2.000000</td>
      <td>4.100000e+01</td>
      <td>2.000000e+03</td>
      <td>29.000000</td>
      <td>2012.000000</td>
      <td>4.000000</td>
      <td>8.000000</td>
      <td>1.000000</td>
      <td>5.000000</td>
      <td>2012.000000</td>
      <td>4.000000</td>
      <td>7.000000</td>
      <td>2.000000</td>
      <td>0.000000e+00</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>14.000000</td>
      <td>7.250000e+02</td>
      <td>5.250000e+03</td>
      <td>29.000000</td>
      <td>2014.000000</td>
      <td>6.000000</td>
      <td>15.000000</td>
      <td>2.000000</td>
      <td>16.000000</td>
      <td>2014.000000</td>
      <td>7.000000</td>
      <td>15.000000</td>
      <td>3.000000</td>
      <td>1.500000e+01</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>60.000000</td>
      <td>4.370000e+03</td>
      <td>1.500000e+04</td>
      <td>37.000000</td>
      <td>2015.000000</td>
      <td>9.000000</td>
      <td>23.000000</td>
      <td>4.000000</td>
      <td>20.000000</td>
      <td>2016.000000</td>
      <td>9.000000</td>
      <td>23.000000</td>
      <td>5.000000</td>
      <td>1.070000e+02</td>
    </tr>
    <tr>
      <th>max</th>
      <td>219382.000000</td>
      <td>2.033899e+07</td>
      <td>1.000000e+08</td>
      <td>14866.000000</td>
      <td>2018.000000</td>
      <td>12.000000</td>
      <td>31.000000</td>
      <td>6.000000</td>
      <td>23.000000</td>
      <td>2018.000000</td>
      <td>12.000000</td>
      <td>31.000000</td>
      <td>6.000000</td>
      <td>1.042779e+07</td>
    </tr>
  </tbody>
</table>
</div>



* The differences were all above 10,000 times between the max and median of backers, pledged and goal amount, showing a huge gap between the top campaigns and the rest participants.
* Among 292,627 campaigns in the U.S., the greatest funds pledged for a campaign was 207 times of the mean, and the most backers for a campaign was 1941 times of the mean. Thus median was a better statistic parameter to look at than the mean in terms of understanding the dataset as a whole.
* The average pledged amount (9,670 dollars) was 13.3 times of the median of pledged amount(725 dollars), and the median of pledged to goal ratio was around 15.78%. 
* 75% of all campaigns had less than 2 backers and a goal of more than 2,000 dollars.
* The average duration is around 34 days.
* The average goal is more than 4 times of that of pledged amount.


```python
describe_successful = df[df["outcome"]=="successful"].describe()
describe_successful
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>backers</th>
      <th>pledged</th>
      <th>goal</th>
      <th>duration</th>
      <th>launch_year</th>
      <th>launch_month</th>
      <th>launch_date</th>
      <th>launch_day</th>
      <th>launch_time</th>
      <th>end_year</th>
      <th>end_month</th>
      <th>end_date</th>
      <th>end_day</th>
      <th>pledged_to_goal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>109299.000000</td>
      <td>1.092990e+05</td>
      <td>1.092990e+05</td>
      <td>109299.000000</td>
      <td>109299.000000</td>
      <td>109299.000000</td>
      <td>109299.000000</td>
      <td>109299.000000</td>
      <td>109299.000000</td>
      <td>109299.000000</td>
      <td>109299.000000</td>
      <td>109299.000000</td>
      <td>109299.000000</td>
      <td>1.092990e+05</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>270.177952</td>
      <td>2.321289e+04</td>
      <td>9.695671e+03</td>
      <td>31.392108</td>
      <td>2013.694563</td>
      <td>6.339555</td>
      <td>15.082782</td>
      <td>2.372593</td>
      <td>13.057832</td>
      <td>2013.752157</td>
      <td>6.717921</td>
      <td>14.953010</td>
      <td>3.127787</td>
      <td>9.622973e+02</td>
    </tr>
    <tr>
      <th>std</th>
      <td>1593.147678</td>
      <td>1.607100e+05</td>
      <td>2.879007e+04</td>
      <td>12.033229</td>
      <td>2.028745</td>
      <td>3.298402</td>
      <td>8.822346</td>
      <td>1.744180</td>
      <td>7.704682</td>
      <td>2.012625</td>
      <td>3.292765</td>
      <td>9.026611</td>
      <td>1.946169</td>
      <td>4.911284e+04</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.000000</td>
      <td>1.000000e+00</td>
      <td>1.000000e-02</td>
      <td>0.000000</td>
      <td>2009.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>2009.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>8.500000e+01</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>34.000000</td>
      <td>2.060065e+03</td>
      <td>1.500000e+03</td>
      <td>29.000000</td>
      <td>2012.000000</td>
      <td>3.000000</td>
      <td>7.000000</td>
      <td>1.000000</td>
      <td>5.000000</td>
      <td>2012.000000</td>
      <td>4.000000</td>
      <td>7.000000</td>
      <td>2.000000</td>
      <td>1.040000e+02</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>71.000000</td>
      <td>5.185000e+03</td>
      <td>4.000000e+03</td>
      <td>29.000000</td>
      <td>2014.000000</td>
      <td>6.000000</td>
      <td>15.000000</td>
      <td>2.000000</td>
      <td>16.000000</td>
      <td>2014.000000</td>
      <td>7.000000</td>
      <td>15.000000</td>
      <td>3.000000</td>
      <td>1.150000e+02</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>167.000000</td>
      <td>1.320201e+04</td>
      <td>1.000000e+04</td>
      <td>34.000000</td>
      <td>2015.000000</td>
      <td>9.000000</td>
      <td>22.000000</td>
      <td>4.000000</td>
      <td>20.000000</td>
      <td>2015.000000</td>
      <td>10.000000</td>
      <td>23.000000</td>
      <td>5.000000</td>
      <td>1.580000e+02</td>
    </tr>
    <tr>
      <th>max</th>
      <td>219382.000000</td>
      <td>2.033899e+07</td>
      <td>2.000000e+06</td>
      <td>91.000000</td>
      <td>2017.000000</td>
      <td>12.000000</td>
      <td>31.000000</td>
      <td>6.000000</td>
      <td>23.000000</td>
      <td>2018.000000</td>
      <td>12.000000</td>
      <td>31.000000</td>
      <td>6.000000</td>
      <td>1.042779e+07</td>
    </tr>
  </tbody>
</table>
</div>




```python
describe_failed = df[df["outcome"]=="failed"].describe()
describe_failed
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>backers</th>
      <th>pledged</th>
      <th>goal</th>
      <th>duration</th>
      <th>launch_year</th>
      <th>launch_month</th>
      <th>launch_date</th>
      <th>launch_day</th>
      <th>launch_time</th>
      <th>end_year</th>
      <th>end_month</th>
      <th>end_date</th>
      <th>end_day</th>
      <th>pledged_to_goal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>152061.000000</td>
      <td>152061.000000</td>
      <td>1.520610e+05</td>
      <td>152061.000000</td>
      <td>152061.000000</td>
      <td>152061.000000</td>
      <td>152061.000000</td>
      <td>152061.000000</td>
      <td>152061.000000</td>
      <td>152061.000000</td>
      <td>152061.000000</td>
      <td>152061.000000</td>
      <td>152061.00000</td>
      <td>152061.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>16.772118</td>
      <td>1331.173917</td>
      <td>6.066424e+04</td>
      <td>34.477164</td>
      <td>2014.011153</td>
      <td>6.397590</td>
      <td>15.362940</td>
      <td>2.465557</td>
      <td>12.832173</td>
      <td>2014.080908</td>
      <td>6.727662</td>
      <td>15.318563</td>
      <td>3.21761</td>
      <td>8.937630</td>
    </tr>
    <tr>
      <th>std</th>
      <td>73.425843</td>
      <td>6999.568474</td>
      <td>1.356864e+06</td>
      <td>13.513946</td>
      <td>1.909592</td>
      <td>3.291926</td>
      <td>8.778117</td>
      <td>1.765711</td>
      <td>8.075102</td>
      <td>1.898457</td>
      <td>3.295430</td>
      <td>9.030117</td>
      <td>1.95823</td>
      <td>15.090471</td>
    </tr>
    <tr>
      <th>min</th>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>1.500000e-01</td>
      <td>0.000000</td>
      <td>2009.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>2009.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>1.000000</td>
      <td>5.000000</td>
      <td>2.600000e+03</td>
      <td>29.000000</td>
      <td>2013.000000</td>
      <td>4.000000</td>
      <td>8.000000</td>
      <td>1.000000</td>
      <td>4.000000</td>
      <td>2013.000000</td>
      <td>4.000000</td>
      <td>7.000000</td>
      <td>2.00000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>3.000000</td>
      <td>106.000000</td>
      <td>7.500000e+03</td>
      <td>29.000000</td>
      <td>2014.000000</td>
      <td>6.000000</td>
      <td>15.000000</td>
      <td>2.000000</td>
      <td>16.000000</td>
      <td>2014.000000</td>
      <td>7.000000</td>
      <td>15.000000</td>
      <td>3.00000</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>13.000000</td>
      <td>700.000000</td>
      <td>2.000000e+04</td>
      <td>39.000000</td>
      <td>2015.000000</td>
      <td>9.000000</td>
      <td>23.000000</td>
      <td>4.000000</td>
      <td>20.000000</td>
      <td>2015.000000</td>
      <td>9.000000</td>
      <td>23.000000</td>
      <td>5.00000</td>
      <td>11.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>6550.000000</td>
      <td>757352.940000</td>
      <td>1.000000e+08</td>
      <td>91.000000</td>
      <td>2017.000000</td>
      <td>12.000000</td>
      <td>31.000000</td>
      <td>6.000000</td>
      <td>23.000000</td>
      <td>2018.000000</td>
      <td>12.000000</td>
      <td>31.000000</td>
      <td>6.00000</td>
      <td>107.000000</td>
    </tr>
  </tbody>
</table>
</div>



# Number Of Campaigns vs. Outcomes


```python
data = pd.DataFrame(df["outcome"].value_counts())
data["percentage"] = data["outcome"] / data["outcome"].sum() * 100
data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>outcome</th>
      <th>percentage</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>failed</th>
      <td>152061</td>
      <td>51.964104</td>
    </tr>
    <tr>
      <th>successful</th>
      <td>109299</td>
      <td>37.350962</td>
    </tr>
    <tr>
      <th>canceled</th>
      <td>28311</td>
      <td>9.674774</td>
    </tr>
    <tr>
      <th>live</th>
      <td>1740</td>
      <td>0.594614</td>
    </tr>
    <tr>
      <th>suspended</th>
      <td>1216</td>
      <td>0.415546</td>
    </tr>
  </tbody>
</table>
</div>




```python
data.plot(kind="barh", y="outcome", color=single_color, edgecolor=edgecolor, linewidth=linewidth, legend=None, figsize=figsize)
plt.title("Outcomes vs. Number Of Campaigns")
plt.xlabel("Number of Campaigns")
plt.ylabel("Outcomes")
plt.xlim(0, 170000)
plt.gca().invert_yaxis()
frameon = True
ax = plt.gca()
ax.get_xaxis().set_major_formatter(plt.FuncFormatter(lambda x, loc: "{:,}".format(int(x))))
for i, v in enumerate(data["percentage"]):
    ax.text(v+200, i-0.3, "%.2f" % v + "%", color="black", fontweight="bold")
plt.xlim(0, 160000)
```




    (0, 160000)




![png](output_24_1.png)


* Successful campaigns made up only 37.35% of all campaigns, much less than that of failed ones (51.96%).
* Live and suspended campaigns only constituted approximately 1% of all campaigns, at the time of data collection.
* 9.67% of all campaigns were canceled before the campaign ended.

# Median Of Pledged, goal, pledged over goal, and backers vs. Outcomes


```python
outcome_df = df.groupby("outcome").median()
outcome_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>backers</th>
      <th>pledged</th>
      <th>goal</th>
      <th>duration</th>
      <th>launch_year</th>
      <th>launch_month</th>
      <th>launch_date</th>
      <th>launch_day</th>
      <th>launch_time</th>
      <th>end_year</th>
      <th>end_month</th>
      <th>end_date</th>
      <th>end_day</th>
      <th>pledged_to_goal</th>
    </tr>
    <tr>
      <th>outcome</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>canceled</th>
      <td>3.0</td>
      <td>100.0</td>
      <td>10000.0</td>
      <td>29.0</td>
      <td>2015.0</td>
      <td>7.0</td>
      <td>15.0</td>
      <td>2.0</td>
      <td>16.0</td>
      <td>2015.0</td>
      <td>7.0</td>
      <td>15.0</td>
      <td>3.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>failed</th>
      <td>3.0</td>
      <td>106.0</td>
      <td>7500.0</td>
      <td>29.0</td>
      <td>2014.0</td>
      <td>6.0</td>
      <td>15.0</td>
      <td>2.0</td>
      <td>16.0</td>
      <td>2014.0</td>
      <td>7.0</td>
      <td>15.0</td>
      <td>3.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>live</th>
      <td>4.0</td>
      <td>220.0</td>
      <td>6500.0</td>
      <td>31.0</td>
      <td>2017.0</td>
      <td>12.0</td>
      <td>15.0</td>
      <td>2.0</td>
      <td>16.0</td>
      <td>2018.0</td>
      <td>1.0</td>
      <td>13.0</td>
      <td>3.0</td>
      <td>4.0</td>
    </tr>
    <tr>
      <th>successful</th>
      <td>71.0</td>
      <td>5185.0</td>
      <td>4000.0</td>
      <td>29.0</td>
      <td>2014.0</td>
      <td>6.0</td>
      <td>15.0</td>
      <td>2.0</td>
      <td>16.0</td>
      <td>2014.0</td>
      <td>7.0</td>
      <td>15.0</td>
      <td>3.0</td>
      <td>115.0</td>
    </tr>
    <tr>
      <th>suspended</th>
      <td>2.0</td>
      <td>30.5</td>
      <td>5000.0</td>
      <td>29.0</td>
      <td>2015.0</td>
      <td>6.0</td>
      <td>15.5</td>
      <td>2.0</td>
      <td>16.0</td>
      <td>2015.0</td>
      <td>6.0</td>
      <td>16.0</td>
      <td>3.0</td>
      <td>1.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
index_list = ["successful", "canceled", "live", "suspended", "failed"]
columns = ["pledged", "goal", "pledged_to_goal", "backers"]
data = pd.DataFrame(index=index_list, columns=columns)
for i in index_list:
    for j in columns:
        data.loc[i,j] = outcome_df.loc[i,j]
data
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>pledged</th>
      <th>goal</th>
      <th>pledged_to_goal</th>
      <th>backers</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>successful</th>
      <td>5185</td>
      <td>4000</td>
      <td>115</td>
      <td>71</td>
    </tr>
    <tr>
      <th>canceled</th>
      <td>100</td>
      <td>10000</td>
      <td>1</td>
      <td>3</td>
    </tr>
    <tr>
      <th>live</th>
      <td>220</td>
      <td>6500</td>
      <td>4</td>
      <td>4</td>
    </tr>
    <tr>
      <th>suspended</th>
      <td>30.5</td>
      <td>5000</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>failed</th>
      <td>106</td>
      <td>7500</td>
      <td>1</td>
      <td>3</td>
    </tr>
  </tbody>
</table>
</div>




```python
fig = plt.figure(figsize=figsize)
st = plt.suptitle("Pledged, goal, pledged over goal, and backers vs. Outcomes")

plt.subplot(221)
data["pledged"].plot(kind="bar", color=single_color)
ax = plt.gca()
ax.get_yaxis().set_major_formatter(plt.FuncFormatter(lambda x, loc: "{:,}".format(int(x))))
ax.set_ylabel("Pledged ($)")
ax.set_yscale('symlog')

plt.subplot(222)
data["goal"].plot(kind="bar", color=single_color)
ax = plt.gca()
ax.get_yaxis().set_major_formatter(plt.FuncFormatter(lambda x, loc: "{:,}".format(int(x))))
ax.set_ylabel("Goal ($)")
ax.set_yscale('symlog')

plt.subplot(223)
data["pledged_to_goal"].plot(kind="bar", color=single_color)
ax = plt.gca()
ax.get_yaxis().set_major_formatter(plt.FuncFormatter(lambda x, loc: "{:,}".format(int(x))))
ax.set_ylabel("% Pledged/ Goal")
ax.set_yscale('symlog')

plt.subplot(224)
data["backers"].plot(kind="bar", color=single_color)
ax = plt.gca()
ax.get_yaxis().set_major_formatter(plt.FuncFormatter(lambda x, loc: "{:,}".format(int(x))))
ax.set_ylabel("Backers")
ax.set_yscale('symlog')

plt.tight_layout()

# Shift subplots down
st.set_y(0.92)
fig.subplots_adjust(top=0.85)

fig.savefig("test.png")
```


![png](output_29_0.png)


# Pledged Amount vs. Above X Percent Of Population


```python
df_successful = df[df["outcome"]=="successful"]
data_in_successful = df_successful["pledged"].tolist()
data_in_successful.sort()

percent_in_successful = []
for i in range(len(data_in_successful)):
    percent_in_successful.append(i / len(data_in_successful) * 100)
    
data_in_successful[0]
percent_in_successful[0]
```




    0.0




```python
percent_in_all = []
data_in_all = df["pledged"].tolist()
data_in_all.sort()
for i in range(len(data_in_all)):
    percent_in_all.append(i / len(data_in_all) * 100)

data_in_all[0]
percent_in_all[0]
```




    0.0




```python
plt.figure(figsize=figsize)
plt.scatter(y=data_in_all, x=percent_in_all, s=2)
plt.scatter(y=data_in_successful, x=percent_in_successful, s=2)

ax = plt.gca()
ax.set_yscale('symlog')

plt.title("Pledged Amount vs. Above X Percent Of Population")
plt.xlabel("Above X Percent Of Population Among All/ Successful Campaigns")
plt.ylabel("Pledged Amount ($)")
plt.legend(["All", "Successful"])
ax.set_xticks([0, 10, 20, 30, 40, 50, 60, 70, 80, 90, 100])

plt.xlim(0, 100)
plt.ylim(0, 100_000_000)

plt.show()
```


![png](output_33_0.png)


# Pledged Amount vs. Above X Percent Of Population, Under Each Category


```python
main_category_list = df["main_category"].unique()
main_category_list
```




    array(['Art', 'Journalism', 'Music', 'Film & Video', 'Fashion',
           'Publishing', 'Food', 'Theater', 'Technology', 'Photography',
           'Design', 'Comics', 'Games', 'Crafts', 'Dance'], dtype=object)




```python
plt.figure(figsize=(15, 8))
    
legend_list = []
for i in range(len(main_category_list)):
    main_category = main_category_list[i]
    df_i = df[df["main_category"]==main_category]

    df_successful = df_i[df_i["outcome"]=="successful"]
    data_in_successful = df_successful["pledged"].tolist()
    data_in_successful.sort()

    percent_in_successful = []
    for i in range(len(data_in_successful)):
        percent_in_successful.append(i / len(data_in_successful) * 100)

    percent_in_all = []
    data_in_all = df_i["pledged"].tolist()
    data_in_all.sort()
    for i in range(len(data_in_all)):
        percent_in_all.append(i / len(data_in_all) * 100)

    plt.scatter(y=data_in_all, x=percent_in_all, s=2)
    plt.scatter(y=data_in_successful, x=percent_in_successful, s=0.5)
    
    legend_list.append(main_category)
    plt.legend(legend_list)

ax = plt.gca()
ax.set_yscale('symlog')

plt.title("Pledged Amount vs. Above X Percent Of Population")
plt.xlabel("Above X Percent Of Population Among All/ Successful Campaigns")
plt.ylabel("Pledged Amount ($)")
ax.set_xticks([0, 10, 20, 30, 40, 50, 60, 70, 80, 90, 100])

plt.xlim(0, 100)
plt.ylim(0, 100_000_000)

plt.show()
```


![png](output_36_0.png)


# Number Of Campaigns vs. Pledged Amount Range


```python
plt.figure(figsize=figsize)

data_all = df["pledged"].tolist()
pl.hist(data_all, bins=np.logspace(np.log10(0.1),np.log10(100_000_000), 50),\
        color=single_color, edgecolor=edgecolor)

data_successful = df[df["outcome"]=="successful"]["pledged"].tolist()
pl.hist(data_successful, bins=np.logspace(np.log10(0.1),np.log10(100_000_000), 50),\
        color=colors[1], edgecolor=edgecolor)

data_failed = df[df["outcome"]=="failed"]["pledged"].tolist()
pl.hist(data_failed, bins=np.logspace(np.log10(0.1),np.log10(100_000_000), 50),\
        color=colors[2], alpha=0.5, edgecolor=edgecolor)

data_canceled = df[df["outcome"]=="canceled"]["pledged"].tolist()
pl.hist(data_canceled, bins=np.logspace(np.log10(0.1),np.log10(100_000_000), 50),\
        color=colors[3], alpha=0.5, edgecolor=edgecolor)

data_suspended = df[df["outcome"]=="suspended"]["pledged"].tolist()
pl.hist(data_suspended, bins=np.logspace(np.log10(0.1),np.log10(100_000_000), 50),\
        color=colors[3], edgecolor=edgecolor)

plt.legend(["All", "Successful", "Failed", "Canceled", "Suspended"])

pl.gca().set_xscale("log")

ax = plt.gca()
ax.get_yaxis().set_major_formatter(plt.FuncFormatter(lambda x, loc: "{:,}".format(int(x))))

plt.title("Number Of Campaigns vs. Pledged Amount Range")
plt.xlabel("Pledged Amount Range ($)")
plt.ylabel("Number Of Campaigns")

pl.show()
```


![png](output_38_0.png)


# Number Of Campaigns vs. Goal Amount Range


```python
plt.figure(figsize=figsize)

data_all = df["goal"].tolist()
pl.hist(data_all, bins=np.logspace(np.log10(0.1),np.log10(100_000_000), 50),\
        color=single_color, edgecolor=edgecolor)

data_successful = df[df["outcome"]=="successful"]["goal"].tolist()
pl.hist(data_successful, bins=np.logspace(np.log10(0.1),np.log10(100_000_000), 50),\
        color=colors[1], edgecolor=edgecolor)

data_failed = df[df["outcome"]=="failed"]["goal"].tolist()
pl.hist(data_failed, bins=np.logspace(np.log10(0.1),np.log10(100_000_000), 50),\
        color=colors[2], alpha=0.5, edgecolor=edgecolor)

data_canceled = df[df["outcome"]=="canceled"]["goal"].tolist()
pl.hist(data_canceled, bins=np.logspace(np.log10(0.1),np.log10(100_000_000), 50),\
        color=colors[3], alpha=0.5, edgecolor=edgecolor)

data_suspended = df[df["outcome"]=="suspended"]["goal"].tolist()
pl.hist(data_suspended, bins=np.logspace(np.log10(0.1),np.log10(100_000_000), 50),\
        color=colors[3], edgecolor=edgecolor)

plt.legend(["All", "Successful", "Failed", "Canceled", "Suspended"])

pl.gca().set_xscale("log")

ax = plt.gca()
ax.get_yaxis().set_major_formatter(plt.FuncFormatter(lambda x, loc: "{:,}".format(int(x))))

plt.title("Number Of Campaigns vs. Goal Amount Range")
plt.xlabel("Goal Amount Range ($)")
plt.ylabel("Number Of Campaigns")

pl.show()
```


![png](output_40_0.png)


# Number Of Campaigns vs. Backer Quantity Range


```python
plt.figure(figsize=figsize)

data_all = df["backers"].tolist()
pl.hist(data_all, bins=np.logspace(np.log10(0.1),np.log10(100_000_000), 50),\
        color=single_color, edgecolor=edgecolor)

data_successful = df[df["outcome"]=="successful"]["backers"].tolist()
pl.hist(data_successful, bins=np.logspace(np.log10(0.1),np.log10(100_000_000), 50),\
        color=colors[1], edgecolor=edgecolor)

data_failed = df[df["outcome"]=="failed"]["backers"].tolist()
pl.hist(data_failed, bins=np.logspace(np.log10(0.1),np.log10(100_000_000), 50),\
        color=colors[2], alpha=0.5, edgecolor=edgecolor)

data_canceled = df[df["outcome"]=="canceled"]["backers"].tolist()
pl.hist(data_canceled, bins=np.logspace(np.log10(0.1),np.log10(100_000_000), 50),\
        color=colors[3], alpha=0.5, edgecolor=edgecolor)

data_suspended = df[df["outcome"]=="suspended"]["backers"].tolist()
pl.hist(data_suspended, bins=np.logspace(np.log10(0.1),np.log10(100_000_000), 50),\
        color=colors[3], edgecolor=edgecolor)

plt.legend(["All", "Successful", "Failed", "Canceled", "Suspended"])

pl.gca().set_xscale("log")

ax = plt.gca()
ax.get_yaxis().set_major_formatter(plt.FuncFormatter(lambda x, loc: "{:,}".format(int(x))))

plt.title("Number Of Campaigns vs. Backers Quantity Range")
plt.xlabel("Backers Quantity Range")
plt.ylabel("Number Of Campaigns")

pl.show()
```


![png](output_42_0.png)


# Number Of Campaigns vs. Pledged/ Goal Range


```python
x = "pledged_to_goal"
plt.figure(figsize=figsize)

data_all = df[x].tolist()
pl.hist(data_all, bins=np.logspace(np.log10(0.1),np.log10(100_000_000), 50),\
        color=single_color, edgecolor=edgecolor)

data_successful = df[df["outcome"]=="successful"][x].tolist()
pl.hist(data_successful, bins=np.logspace(np.log10(0.1),np.log10(100_000_000), 50),\
        color=colors[1], edgecolor=edgecolor)

data_failed = df[df["outcome"]=="failed"][x].tolist()
pl.hist(data_failed, bins=np.logspace(np.log10(0.1),np.log10(100_000_000), 50),\
        color=colors[2], alpha=0.5, edgecolor=edgecolor)

data_canceled = df[df["outcome"]=="canceled"][x].tolist()
pl.hist(data_canceled, bins=np.logspace(np.log10(0.1),np.log10(100_000_000), 50),\
        color=colors[3], alpha=0.5, edgecolor=edgecolor)

data_suspended = df[df["outcome"]=="suspended"][x].tolist()
pl.hist(data_suspended, bins=np.logspace(np.log10(0.1),np.log10(100_000_000), 50),\
        color=colors[3], edgecolor=edgecolor)

plt.legend(["All", "Successful", "Failed", "Canceled", "Suspended"])

pl.gca().set_xscale("log")

ax = plt.gca()
ax.get_yaxis().set_major_formatter(plt.FuncFormatter(lambda x, loc: "{:,}".format(int(x))))

plt.title("Number Of Campaigns vs. Pledged/ Goal Range")
plt.xlabel("Pledged/ Goal Range")
plt.ylabel("Number Of Campaigns")

pl.show()
```


![png](output_44_0.png)


# Number Of Campaigns vs. Duration Range


```python
x = "duration"
plt.figure(figsize=figsize)

data_all = df[x].tolist()
pl.hist(data_all, bins=np.logspace(np.log10(0.1),np.log10(100_000_000), 50),\
        color=single_color, edgecolor=edgecolor)

data_successful = df[df["outcome"]=="successful"][x].tolist()
pl.hist(data_successful, bins=np.logspace(np.log10(0.1),np.log10(100_000_000), 50),\
        color=colors[1], edgecolor=edgecolor)

data_failed = df[df["outcome"]=="failed"][x].tolist()
pl.hist(data_failed, bins=np.logspace(np.log10(0.1),np.log10(100_000_000), 50),\
        color=colors[2], alpha=0.5, edgecolor=edgecolor)

data_canceled = df[df["outcome"]=="canceled"][x].tolist()
pl.hist(data_canceled, bins=np.logspace(np.log10(0.1),np.log10(100_000_000), 50),\
        color=colors[3], alpha=0.5, edgecolor=edgecolor)

data_suspended = df[df["outcome"]=="suspended"][x].tolist()
pl.hist(data_suspended, bins=np.logspace(np.log10(0.1),np.log10(100_000_000), 50),\
        color=colors[3], edgecolor=edgecolor)

plt.legend(["All", "Successful", "Failed", "Canceled", "Suspended"])

pl.gca().set_xscale("log")

ax = plt.gca()
ax.get_yaxis().set_major_formatter(plt.FuncFormatter(lambda x, loc: "{:,}".format(int(x))))

plt.title("Number Of Campaigns vs. Pledged/ Goal Range")
plt.xlabel("Pledged/ Goal Range")
plt.ylabel("Number Of Campaigns")

plt.xlim(0, 10_000)

pl.show()
```

    /anaconda3/envs/PythonData/lib/python3.6/site-packages/matplotlib/axes/_base.py:3129: UserWarning: Attempted to set non-positive xlimits for log-scale axis; invalid limits will be ignored.
      'Attempted to set non-positive xlimits for log-scale axis; '



![png](output_46_1.png)


# Pledged Amount vs. Goal Amount


```python
label="Goal Amount ($)"
df[df["outcome"]=="successful"].plot(kind="scatter", x="goal", y="pledged", figsize=(8,8), s=0.5)

ax = plt.gca()
ax.set_xscale('symlog')
ax.set_yscale('symlog')

plt.title(f"Pledged Amount vs. {label}")
plt.xlabel(label)
plt.ylabel("Pledged Amount ($)")

plt.xlim(0, 100_000_000)
plt.ylim(0, 100_000_000)
```




    (0, 100000000)




![png](output_48_1.png)


# Pledged Amount vs. Backers


```python
label="Number Of Backers"
df[df["outcome"]=="successful"].plot(kind="scatter", x="backers", y="pledged", figsize=(8,8), s=0.1)

ax = plt.gca()
ax.set_xscale('symlog')
ax.set_yscale('symlog')

plt.title(f"Pledged Amount vs. {label}")
plt.xlabel(label)
plt.ylabel("Pledged Amount ($)")

plt.xlim(0, 100_000_000)
plt.ylim(0, 100_000_000)
```




    (0, 100000000)




![png](output_50_1.png)


# Pledged Amount vs. Duration


```python
label="Duration"

plt.figure(figsize=big_figsize)
sns.stripplot(df[df["outcome"]=="successful"]["duration"].tolist(),\
              df[df["outcome"]=="successful"]["pledged"].tolist(),\
              jitter=0.3, size=0.5)

ax = plt.gca()
ax.set_yscale('symlog')

plt.title(f"Pledged Amount vs. {label}")
plt.xlabel(label)
plt.ylabel("Pledged Amount ($)")

plt.xlim(0, 61)
plt.ylim(0, 100_000_000)
```




    (0, 100000000)




![png](output_52_1.png)


# Pledged Amount vs. Category


```python
label="Category"

plt.figure(figsize=big_figsize)
sns.stripplot(df[df["outcome"]=="successful"]["main_category"].tolist(),\
              df[df["outcome"]=="successful"]["pledged"].tolist(),\
              jitter=0.3, size=0.5)

ax = plt.gca()
ax.set_yscale('symlog')

plt.title(f"Pledged Amount vs. {label}")
plt.xlabel(label)
plt.ylabel("Pledged Amount ($)")

plt.ylim(0, 100_000_000)
```




    (0, 100000000)




![png](output_54_1.png)


# Pledged Amount vs. Year


```python
label="Year"

plt.figure(figsize=big_figsize)

sns.stripplot(df[df["outcome"]=="successful"]["end_year"].tolist(),\
              df[df["outcome"]=="successful"]["pledged"].tolist(),\
              jitter=0.3, size=0.5)

ax = plt.gca()
ax.set_yscale('symlog')

plt.title(f"Pledged Amount vs. {label}")
plt.xlabel(label)
plt.ylabel("Pledged Amount ($)")

plt.ylim(0, 100_000_000)
```




    (0, 100000000)




![png](output_56_1.png)


# Pledged Amount vs. Quarter


```python
label="Quarter"

plt.figure(figsize=big_figsize)

data = df[df["outcome"]=="successful"].sort_values("end_quarter")

sns.stripplot(data["end_quarter"].tolist(), data["pledged"].tolist(),\
              jitter=0.4, size=0.5)

ax = plt.gca()
ax.set_yscale('symlog')

plt.title(f"Pledged Amount vs. {label}")
plt.xlabel(label)
plt.ylabel("Pledged Amount ($)")

plt.ylim(0, 100_000_000)
```




    (0, 100000000)




![png](output_58_1.png)


# Pledged Amount vs. Month


```python
label="Month"

plt.figure(figsize=big_figsize)

sns.stripplot(df[df["outcome"]=="successful"]["end_month"].tolist(),\
              df[df["outcome"]=="successful"]["pledged"].tolist(),
              jitter=0.4, size=0.5)

ax = plt.gca()
ax.set_yscale('symlog')

plt.title(f"Pledged Amount vs. {label}")
plt.xlabel(label)
plt.ylabel("Pledged Amount ($)")

plt.ylim(0, 100_000_000)
```




    (0, 100000000)




![png](output_60_1.png)


# Pledged Amount vs. Date


```python
label="Date"

plt.figure(figsize=big_figsize)

sns.stripplot(df[df["outcome"]=="successful"]["end_date"].tolist(),\
              df[df["outcome"]=="successful"]["pledged"].tolist(),\
              jitter=0.4, size=0.5)

ax = plt.gca()
ax.set_yscale('symlog')

plt.title(f"Pledged Amount vs. {label}")
plt.xlabel(label)
plt.ylabel("Pledged Amount ($)")

plt.ylim(0, 100_000_000)
```




    (0, 100000000)




![png](output_62_1.png)


# Pledged Amount vs. Day Of Week


```python
label="Day Of Week"

plt.figure(figsize=big_figsize)

sns.stripplot(df[df["outcome"]=="successful"]["end_day"].tolist(),\
              df[df["outcome"]=="successful"]["pledged"].tolist(),\
              jitter=0.4, size=0.5)

ax = plt.gca()
ax.set_yscale('symlog')

plt.title(f"Pledged Amount vs. {label}")
plt.xlabel(label)
plt.ylabel("Pledged Amount ($)")

plt.ylim(0, 100_000_000)
```




    (0, 100000000)




![png](output_64_1.png)


# Pledged Amount vs. Time


```python
label="Time"

plt.figure(figsize=big_figsize)

sns.stripplot(df[df["outcome"]=="successful"]["launch_time"].tolist(),\
              df[df["outcome"]=="successful"]["pledged"].tolist(),\
              jitter=0.4, size=0.5)

ax = plt.gca()
ax.set_yscale('symlog')

plt.title(f"Pledged Amount vs. {label}")
plt.xlabel(label)
plt.ylabel("Pledged Amount ($)")

plt.ylim(0, 100_000_000)
```




    (0, 100000000)




![png](output_66_1.png)


# Backers vs. Categories


```python
label="Category"

plt.figure(figsize=big_figsize)
sns.stripplot(df["main_category"].tolist(),\
              df["backers"].tolist(),\
              jitter=0.3, size=0.5)

ax = plt.gca()
ax.set_yscale('symlog')

plt.title(f"Pledged Amount vs. {label}")
plt.xlabel(label)
plt.ylabel("Pledged Amount ($)")

plt.ylim(0, 100_000_000)
```




    (0, 100000000)




![png](output_68_1.png)


# Correlations


```python
numeric_columns = ['backers', 'goal', 'duration', 'launch_year', 'launch_month', 'launch_date', 'end_date', 'end_day']
corr_index = []
corr_list = []

for i in list(df[numeric_columns]):
    corr = df[i].corr(df["pledged"])
    corr_index.append(i)
    corr_list.append(corr)

corr_df = pd.DataFrame({"corr":corr_list}, index=corr_index)
corr_df.sort_values("corr", ascending=False)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>corr</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>backers</th>
      <td>0.754823</td>
    </tr>
    <tr>
      <th>launch_year</th>
      <td>0.027897</td>
    </tr>
    <tr>
      <th>goal</th>
      <td>0.006060</td>
    </tr>
    <tr>
      <th>end_day</th>
      <td>0.002746</td>
    </tr>
    <tr>
      <th>end_date</th>
      <td>0.001530</td>
    </tr>
    <tr>
      <th>duration</th>
      <td>0.000806</td>
    </tr>
    <tr>
      <th>launch_date</th>
      <td>0.000345</td>
    </tr>
    <tr>
      <th>launch_month</th>
      <td>-0.001381</td>
    </tr>
  </tbody>
</table>
</div>




```python
df["goal"].corr(df["pledged"])
df["backers"].corr(df["pledged"])
df["duration"].corr(df["pledged"])
```




    0.0008062770053050585



The correlation between number of backers and pledged amount is the greatest (r=0.754), showing a strong positive linear relationship between the two.
There is little or not linear relationship between goal amount and pledged amount (r=0.006), nor between duration and pledged amount (r=-0.0008)


```python
ax = sns.jointplot("backers", "pledged", data=df, kind="reg")
import matplotlib.pyplot as plt
plt.xlabel("Backers")
plt.ylabel("Pledged ($)")
sns.set(rc={'figure.figsize':figsize})
framon = True
```

    /anaconda3/envs/PythonData/lib/python3.6/site-packages/matplotlib/axes/_axes.py:6462: UserWarning: The 'normed' kwarg is deprecated, and has been replaced by the 'density' kwarg.
      warnings.warn("The 'normed' kwarg is deprecated, and has been "



![png](output_73_1.png)


The p value = 0 here shows a significant relationship between the quantity of backers and the amount pledged.

# Number Of Campaigns vs. Launch Time


```python
def makeplot(x, label):
    columns_6 = ["total", "successful", "canceled", "live", "suspended", "failed"]
    colors_6 = ["green", "#adc2eb", "#3366cc", "#1f3d7a", "#000000", "#7094db"]
    labels_6 = columns_6
    
    data = pd.DataFrame(df.groupby([x, "outcome"])["name"].count())
    data = data.reset_index()
    data = data.pivot(index=x, columns="outcome", values="name")
    # Fill nan values in Nan columns with 0:
    data = data.fillna(value=0)
    data["total"] = data["successful"] + data["canceled"] + data["live"] + data["suspended"] + data["failed"]
    data = data[columns_6]
    
    print(data.head())
    
    data.plot(kind="line", marker="o", markersize=markersize*1.5,
              linewidth=linewidth, figsize=figsize, color=colors_6)

    ax = plt.gca()
    ax.get_yaxis().set_major_formatter(plt.FuncFormatter(lambda x, loc: "{:,}".format(int(x))))
    ax.set_title(f"Number Of Campaigns vs. {label}")
    ax.set_xlabel(label)
    ax.set_ylabel("Number Of Campaigns")

    plt.legend(labels=labels_6, loc='lower center', bbox_to_anchor=bbox_to_anchor, ncol=6, fancybox=True, shadow=True)
```


```python
x = "launch_time"
label = "Launch time Of Day (h)"

makeplot(x, label)

plt.xlim(0,23)
plt.ylim(0,25_000)
plt.xticks(range(24))

plt.show()
```

    outcome      total  successful  canceled  live  suspended  failed
    launch_time                                                      
    0            17733        6037      1764    92         73    9767
    1            16097        5607      1582    85         61    8762
    2            14456        5064      1332    89         68    7903
    3            12909        4484      1210    69         44    7102
    4            11462        4060      1064    53         49    6236



![png](output_77_1.png)


# Number Of Campaigns vs. End Year


```python
x = "end_year"
label = "End Year"

makeplot(x, label)

plt.xlim(2009, 2018)
plt.ylim(0, 60_000)

plt.show()
```

    outcome     total  successful  canceled  live  suspended   failed
    end_year                                                         
    2009        902.0       384.0     108.0   0.0        0.0    410.0
    2010       9098.0      4008.0     785.0   0.0        8.0   4297.0
    2011      25107.0     11768.0    2086.0   0.0       52.0  11201.0
    2012      41197.0     17956.0    2649.0   0.0       51.0  20541.0
    2013      38424.0     17133.0    2969.0   0.0       54.0  18268.0



![png](output_79_1.png)


# Number Of Campaigns vs. End Quarter


```python
x = "end_quarter"
label = "End Quarter"

makeplot(x, label)

plt.xlim(0, 3)
#plt.ylim(0, 80_000)
ax = plt.gca()
ax.set_xticks([0, 1, 2, 3])
ax.set_xticklabels(["Q1", "Q2", "Q3", "Q4"])

plt.show()
```

    outcome      total  successful  canceled  live  suspended  failed
    end_quarter                                                      
    Q1           62505       22688      5849  1737        302   31929
    Q2           77125       30058      7351     1        311   39404
    Q3           79961       28633      7896     1        306   43125
    Q4           73036       27920      7215     1        297   37603



![png](output_81_1.png)


# Number Of Campaigns vs. End Month


```python
x = "end_month"
label = "End Month"

makeplot(x, label)

plt.xlim(1,12)
plt.ylim(0, 30_000)

plt.xticks(range(1,13))

plt.show()
```

    outcome      total  successful  canceled    live  suspended   failed
    end_month                                                           
    1          18293.0      5856.0    1702.0  1451.0       84.0   9200.0
    2          18961.0      6887.0    1752.0   278.0      101.0   9943.0
    3          25251.0      9945.0    2395.0     8.0      117.0  12786.0
    4          25469.0     10111.0    2405.0     1.0      111.0  12841.0
    5          26504.0     10320.0    2534.0     0.0       92.0  13558.0



![png](output_83_1.png)


# Number Of Campaigns vs. Date


```python
x = "end_date"
label = "End Date"

makeplot(x, label)

plt.xlim(1,30)
plt.ylim(0, 18_000)

plt.xticks(range(1,32))

plt.show()
```

    outcome   total  successful  canceled  live  suspended  failed
    end_date                                                      
    1         17045        7028      1535    57         53    8372
    2         10780        4361      1045    52         34    5288
    3          9257        3486       876    75         40    4780
    4          9285        3371       901    87         24    4902
    5          9566        3601       939    84         37    4905



![png](output_85_1.png)


# Number Of Campaigns vs. End Day Of Week


```python
x = "end_day"
label = "End Day Of Week"

makeplot(x, label)

plt.xlim(0,6)
plt.ylim(0, 50_000)

ax = plt.gca()
ax.set_xticklabels(["Mon", "Tues", "Wed", "Th", "Fri", "Sat", "Sun"])

plt.show()
```

    outcome  total  successful  canceled  live  suspended  failed
    end_day                                                      
    0        38491       14755      3789   232        168   19547
    1        30421       11759      3027   168        141   15326
    2        40703       15259      3939   274        157   21074
    3        46197       17609      4470   288        187   23643
    4        47400       17563      4581   278        208   24770



![png](output_87_1.png)


# Number Of Campaigns vs. End Date(Full)


```python
x = "end"
label = "End Date(Full)"

makeplot(x, label)

plt.show()
data["name"].max()
date_with_most_campaigns = data[data["name"]==data["name"].max()]["end"]
print("The date with most campaigns was: " + date_with_most_campaigns)
```

    outcome     total  successful  canceled  live  suspended  failed
    end                                                             
    2009-05-03    1.0         1.0       0.0   0.0        0.0     0.0
    2009-05-16    2.0         1.0       0.0   0.0        0.0     1.0
    2009-05-20    1.0         0.0       0.0   0.0        0.0     1.0
    2009-05-22    1.0         0.0       0.0   0.0        0.0     1.0
    2009-05-26    1.0         0.0       0.0   0.0        0.0     1.0



![png](output_89_1.png)


    9310    The date with most campaigns was: 2010-12-13
    Name: end, dtype: object


# Outcomes Of Each Main Category


```python
main_category_list = df["main_category"].value_counts()
main_category_list
len(main_category_list)
```




    15




```python
data = df.groupby(["main_category", "outcome"]).count().reset_index()
data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>main_category</th>
      <th>outcome</th>
      <th>name</th>
      <th>sub_category</th>
      <th>end</th>
      <th>backers</th>
      <th>pledged</th>
      <th>goal</th>
      <th>duration</th>
      <th>launch_year</th>
      <th>...</th>
      <th>launch_date</th>
      <th>launch_day</th>
      <th>launch_time</th>
      <th>end_year</th>
      <th>end_month</th>
      <th>end_date</th>
      <th>end_day</th>
      <th>pledged_to_goal</th>
      <th>end_quarter</th>
      <th>am_pm</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Art</td>
      <td>canceled</td>
      <td>1667</td>
      <td>1667</td>
      <td>1667</td>
      <td>1667</td>
      <td>1667</td>
      <td>1667</td>
      <td>1667</td>
      <td>1667</td>
      <td>...</td>
      <td>1667</td>
      <td>1667</td>
      <td>1667</td>
      <td>1667</td>
      <td>1667</td>
      <td>1667</td>
      <td>1667</td>
      <td>1667</td>
      <td>1667</td>
      <td>1667</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Art</td>
      <td>failed</td>
      <td>10953</td>
      <td>10953</td>
      <td>10953</td>
      <td>10953</td>
      <td>10953</td>
      <td>10953</td>
      <td>10953</td>
      <td>10953</td>
      <td>...</td>
      <td>10953</td>
      <td>10953</td>
      <td>10953</td>
      <td>10953</td>
      <td>10953</td>
      <td>10953</td>
      <td>10953</td>
      <td>10953</td>
      <td>10953</td>
      <td>10953</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Art</td>
      <td>live</td>
      <td>124</td>
      <td>124</td>
      <td>124</td>
      <td>124</td>
      <td>124</td>
      <td>124</td>
      <td>124</td>
      <td>124</td>
      <td>...</td>
      <td>124</td>
      <td>124</td>
      <td>124</td>
      <td>124</td>
      <td>124</td>
      <td>124</td>
      <td>124</td>
      <td>124</td>
      <td>124</td>
      <td>124</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Art</td>
      <td>successful</td>
      <td>9496</td>
      <td>9496</td>
      <td>9496</td>
      <td>9496</td>
      <td>9496</td>
      <td>9496</td>
      <td>9496</td>
      <td>9496</td>
      <td>...</td>
      <td>9496</td>
      <td>9496</td>
      <td>9496</td>
      <td>9496</td>
      <td>9496</td>
      <td>9496</td>
      <td>9496</td>
      <td>9496</td>
      <td>9496</td>
      <td>9496</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Art</td>
      <td>suspended</td>
      <td>71</td>
      <td>71</td>
      <td>71</td>
      <td>71</td>
      <td>71</td>
      <td>71</td>
      <td>71</td>
      <td>71</td>
      <td>...</td>
      <td>71</td>
      <td>71</td>
      <td>71</td>
      <td>71</td>
      <td>71</td>
      <td>71</td>
      <td>71</td>
      <td>71</td>
      <td>71</td>
      <td>71</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 21 columns</p>
</div>




```python
data = data[["main_category", "outcome", "name"]]
data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>main_category</th>
      <th>outcome</th>
      <th>name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Art</td>
      <td>canceled</td>
      <td>1667</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Art</td>
      <td>failed</td>
      <td>10953</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Art</td>
      <td>live</td>
      <td>124</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Art</td>
      <td>successful</td>
      <td>9496</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Art</td>
      <td>suspended</td>
      <td>71</td>
    </tr>
  </tbody>
</table>
</div>




```python
columns = ["successful", "canceled", "live", "suspended", "failed"]
pivot_df = data.pivot(index="main_category", columns="outcome", values="name")
pivot_df = pivot_df.fillna(value=0)
pivot_df = pivot_df[columns]
pivot_df["total"] = pivot_df["successful"] + pivot_df["live"] + pivot_df["canceled"] + pivot_df["suspended"] + pivot_df["failed"]
pivot_df = pivot_df.sort_values("total", ascending=False)
pivot_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>outcome</th>
      <th>successful</th>
      <th>canceled</th>
      <th>live</th>
      <th>suspended</th>
      <th>failed</th>
      <th>total</th>
    </tr>
    <tr>
      <th>main_category</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Film &amp; Video</th>
      <td>19782</td>
      <td>4693</td>
      <td>212</td>
      <td>84</td>
      <td>27151</td>
      <td>51922</td>
    </tr>
    <tr>
      <th>Music</th>
      <td>21718</td>
      <td>2706</td>
      <td>194</td>
      <td>113</td>
      <td>18507</td>
      <td>43238</td>
    </tr>
    <tr>
      <th>Publishing</th>
      <td>9965</td>
      <td>2805</td>
      <td>198</td>
      <td>46</td>
      <td>18712</td>
      <td>31726</td>
    </tr>
    <tr>
      <th>Games</th>
      <td>9356</td>
      <td>4099</td>
      <td>165</td>
      <td>143</td>
      <td>10873</td>
      <td>24636</td>
    </tr>
    <tr>
      <th>Art</th>
      <td>9496</td>
      <td>1667</td>
      <td>124</td>
      <td>71</td>
      <td>10953</td>
      <td>22311</td>
    </tr>
  </tbody>
</table>
</div>




```python
pivot_df.loc[:,columns].plot.barh(stacked=True, color=colors, alpha=1, edgecolor=edgecolor, linewidth=linewidth, figsize=figsize)
plt.title("Categories vs. Number Of Campaigns, Colored By Outcomes")
plt.xlabel("Number of Campaigns")
plt.ylabel("Categories")
plt.legend(loc='lower center', bbox_to_anchor=bbox_to_anchor, ncol=ncol, fancybox=True, shadow=True)
plt.gca().invert_yaxis()
plt.xlim(0, 55000)
frameon = True
ax = plt.gca()
ax.get_xaxis().set_major_formatter(plt.FuncFormatter(lambda x, loc: "{:,}".format(int(x))))
```


![png](output_95_0.png)


* The top 5 categories with the most campaigns were: film & video, music, publishing, games, and art.
* Although film & video had more campaigns than music, music turned out to yield a few more successful campaigns than film & video.
* Both film & video and music doubled in successful campaigns than that of publishing, games and art.


```python
columns = ["successful", "canceled", "live", "suspended", "failed"]
new_columns = ["successful_percent", "canceled_percent", "live_percent", "suspended_percent", "failed_percent"]
for i in range(len(columns)):
    pivot_df[new_columns[i]] = pivot_df[columns[i]] / pivot_df["total"] * 100
    pivot_df[new_columns[i]] = pd.to_numeric(pivot_df[new_columns[i]])
pivot_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>outcome</th>
      <th>successful</th>
      <th>canceled</th>
      <th>live</th>
      <th>suspended</th>
      <th>failed</th>
      <th>total</th>
      <th>successful_percent</th>
      <th>canceled_percent</th>
      <th>live_percent</th>
      <th>suspended_percent</th>
      <th>failed_percent</th>
    </tr>
    <tr>
      <th>main_category</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Film &amp; Video</th>
      <td>19782</td>
      <td>4693</td>
      <td>212</td>
      <td>84</td>
      <td>27151</td>
      <td>51922</td>
      <td>38.099457</td>
      <td>9.038558</td>
      <td>0.408305</td>
      <td>0.161781</td>
      <td>52.291899</td>
    </tr>
    <tr>
      <th>Music</th>
      <td>21718</td>
      <td>2706</td>
      <td>194</td>
      <td>113</td>
      <td>18507</td>
      <td>43238</td>
      <td>50.228965</td>
      <td>6.258384</td>
      <td>0.448679</td>
      <td>0.261344</td>
      <td>42.802627</td>
    </tr>
    <tr>
      <th>Publishing</th>
      <td>9965</td>
      <td>2805</td>
      <td>198</td>
      <td>46</td>
      <td>18712</td>
      <td>31726</td>
      <td>31.409569</td>
      <td>8.841329</td>
      <td>0.624094</td>
      <td>0.144991</td>
      <td>58.980016</td>
    </tr>
    <tr>
      <th>Games</th>
      <td>9356</td>
      <td>4099</td>
      <td>165</td>
      <td>143</td>
      <td>10873</td>
      <td>24636</td>
      <td>37.976944</td>
      <td>16.638253</td>
      <td>0.669752</td>
      <td>0.580451</td>
      <td>44.134600</td>
    </tr>
    <tr>
      <th>Art</th>
      <td>9496</td>
      <td>1667</td>
      <td>124</td>
      <td>71</td>
      <td>10953</td>
      <td>22311</td>
      <td>42.561965</td>
      <td>7.471651</td>
      <td>0.555780</td>
      <td>0.318229</td>
      <td>49.092376</td>
    </tr>
  </tbody>
</table>
</div>




```python
labels = columns
pivot_df.loc[:,new_columns].plot.barh(stacked=True, color=colors, alpha=1, edgecolor=edgecolor, linewidth=linewidth, figsize=figsize)
plt.title("Categories vs. Percentage Of Campaigns, Colored By Outcomes")
plt.xlabel("Percentage of Campaigns")
plt.ylabel("Categories")
plt.legend(labels=labels, loc='lower center', bbox_to_anchor=bbox_to_anchor, ncol=ncol, fancybox=True, shadow=True)
plt.gca().invert_yaxis()
plt.xlim(0,100)
frameon = True
ax = plt.gca()
ax.get_xaxis().set_major_formatter(plt.FuncFormatter(lambda x, loc: "{:,}".format(int(x))))
```


![png](output_98_0.png)


* Music had the most successful rate among the top 5 categories with most campaigns.
* All 15 categories had more successful rate than failed rate, except for games, comics, theater, and dance.


```python
pivot_df["live+suspended_percent"] = pivot_df["live_percent"] + pivot_df["suspended_percent"]
pivot_df.max()
```




    outcome
    successful                21718.000000
    canceled                   4693.000000
    live                        212.000000
    suspended                   247.000000
    failed                    27151.000000
    total                     51922.000000
    successful_percent           64.684015
    canceled_percent             16.638253
    live_percent                  0.937094
    suspended_percent             1.145853
    failed_percent               65.282486
    live+suspended_percent        2.082947
    dtype: float64



Among all categories, live and suspended campaigns only accounted for at most 2.08% for a each category.

# Outcomes Of Each Sub-category


```python
sub_category_list = df["sub_category"].value_counts()
sub_category_list
len(sub_category_list)
```




    159




```python
data = df.groupby(["sub_category", "outcome"]).count().reset_index()
data = data[["sub_category", "outcome", "name"]]
data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>sub_category</th>
      <th>outcome</th>
      <th>name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>3D Printing</td>
      <td>canceled</td>
      <td>46</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3D Printing</td>
      <td>failed</td>
      <td>195</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3D Printing</td>
      <td>live</td>
      <td>3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3D Printing</td>
      <td>successful</td>
      <td>154</td>
    </tr>
    <tr>
      <th>4</th>
      <td>3D Printing</td>
      <td>suspended</td>
      <td>9</td>
    </tr>
  </tbody>
</table>
</div>




```python
columns = ["successful", "canceled", "live", "suspended", "failed"]
pivot_df = data.pivot(index="sub_category", columns="outcome", values="name")
pivot_df = pivot_df.fillna(value=0)
pivot_df = pivot_df[columns]
pivot_df["total"] = pivot_df["successful"] + pivot_df["live"] + pivot_df["canceled"]\
+ pivot_df["suspended"] + pivot_df["failed"]
pivot_df = pivot_df.sort_values("total", ascending=False)
pivot_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>outcome</th>
      <th>successful</th>
      <th>canceled</th>
      <th>live</th>
      <th>suspended</th>
      <th>failed</th>
      <th>total</th>
    </tr>
    <tr>
      <th>sub_category</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Product Design</th>
      <td>5838.0</td>
      <td>2224.0</td>
      <td>132.0</td>
      <td>134.0</td>
      <td>7894.0</td>
      <td>16222.0</td>
    </tr>
    <tr>
      <th>Documentary</th>
      <td>5164.0</td>
      <td>1278.0</td>
      <td>32.0</td>
      <td>12.0</td>
      <td>7185.0</td>
      <td>13671.0</td>
    </tr>
    <tr>
      <th>Music</th>
      <td>5749.0</td>
      <td>573.0</td>
      <td>55.0</td>
      <td>41.0</td>
      <td>5177.0</td>
      <td>11595.0</td>
    </tr>
    <tr>
      <th>Tabletop Games</th>
      <td>5981.0</td>
      <td>1668.0</td>
      <td>66.0</td>
      <td>18.0</td>
      <td>2958.0</td>
      <td>10691.0</td>
    </tr>
    <tr>
      <th>Shorts</th>
      <td>5559.0</td>
      <td>787.0</td>
      <td>21.0</td>
      <td>6.0</td>
      <td>3975.0</td>
      <td>10348.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
pivot_df_top10 = pivot_df.head(10)
pivot_df_top10.loc[:,columns].plot.barh(stacked=True, color=colors, alpha=1, edgecolor=edgecolor, linewidth=linewidth, figsize=figsize)
plt.title("Sub-Categories vs. Number Of Campaigns, Colored By Outcomes")
plt.xlabel("Number Of Campaigns")
plt.ylabel("Sub-Categories")
plt.legend(loc='lower center', bbox_to_anchor=bbox_to_anchor, ncol=ncol, fancybox=True, shadow=True)
plt.gca().invert_yaxis()
plt.xlim(0, 17500)
frameon = True
ax = plt.gca()
ax.get_xaxis().set_major_formatter(plt.FuncFormatter(lambda x, loc: "{:,}".format(int(x))))
```


![png](output_106_0.png)


* The top 5 sub-categories with the most campaigns were: product design, documentary, music, tabletop games, and shorts.
* 3 of the top 10 sub-categories with the most campaigns, documentary, shorts and film & video, were under the main category of film & video.
* Number of successes among all the top 5 sub-categories with the most successes were actually quite similar, despite the differences in the number of total campaigns for each.


```python
columns = ["successful", "canceled", "live", "suspended", "failed"]
new_columns = ["successful_percent", "canceled_percent", "live_percent", "suspended_percent", "failed_percent"]
for i in range(len(columns)):
    pivot_df[new_columns[i]] = pivot_df[columns[i]] / pivot_df["total"] * 100
pivot_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>outcome</th>
      <th>successful</th>
      <th>canceled</th>
      <th>live</th>
      <th>suspended</th>
      <th>failed</th>
      <th>total</th>
      <th>successful_percent</th>
      <th>canceled_percent</th>
      <th>live_percent</th>
      <th>suspended_percent</th>
      <th>failed_percent</th>
    </tr>
    <tr>
      <th>sub_category</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Product Design</th>
      <td>5838.0</td>
      <td>2224.0</td>
      <td>132.0</td>
      <td>134.0</td>
      <td>7894.0</td>
      <td>16222.0</td>
      <td>35.988164</td>
      <td>13.709777</td>
      <td>0.813710</td>
      <td>0.826039</td>
      <td>48.662310</td>
    </tr>
    <tr>
      <th>Documentary</th>
      <td>5164.0</td>
      <td>1278.0</td>
      <td>32.0</td>
      <td>12.0</td>
      <td>7185.0</td>
      <td>13671.0</td>
      <td>37.773389</td>
      <td>9.348255</td>
      <td>0.234072</td>
      <td>0.087777</td>
      <td>52.556506</td>
    </tr>
    <tr>
      <th>Music</th>
      <td>5749.0</td>
      <td>573.0</td>
      <td>55.0</td>
      <td>41.0</td>
      <td>5177.0</td>
      <td>11595.0</td>
      <td>49.581716</td>
      <td>4.941785</td>
      <td>0.474342</td>
      <td>0.353601</td>
      <td>44.648555</td>
    </tr>
    <tr>
      <th>Tabletop Games</th>
      <td>5981.0</td>
      <td>1668.0</td>
      <td>66.0</td>
      <td>18.0</td>
      <td>2958.0</td>
      <td>10691.0</td>
      <td>55.944252</td>
      <td>15.601908</td>
      <td>0.617342</td>
      <td>0.168366</td>
      <td>27.668132</td>
    </tr>
    <tr>
      <th>Shorts</th>
      <td>5559.0</td>
      <td>787.0</td>
      <td>21.0</td>
      <td>6.0</td>
      <td>3975.0</td>
      <td>10348.0</td>
      <td>53.720526</td>
      <td>7.605334</td>
      <td>0.202938</td>
      <td>0.057982</td>
      <td>38.413220</td>
    </tr>
  </tbody>
</table>
</div>




```python
pivot_df_top10 = pivot_df.head(10)
labels = columns
pivot_df_top10.loc[:,new_columns].plot.barh(stacked=True, color=colors, alpha=1, edgecolor=edgecolor, linewidth=linewidth, figsize=figsize)
plt.title("Sub-Categories vs. Percentage Of Campaigns, Colored By Outcomes")
plt.xlabel("Percentage of Campaigns")
plt.ylabel("Sub-Categories")
plt.legend(labels=labels, loc='lower center', bbox_to_anchor=bbox_to_anchor, ncol=ncol, fancybox=True, shadow=True)
plt.gca().invert_yaxis()
plt.xlim(0,100)
frameon = True
ax = plt.gca()
ax.get_xaxis().set_major_formatter(plt.FuncFormatter(lambda x, loc: "{:,}".format(int(x))))
```


![png](output_109_0.png)


* Among the top 5 subcategories with the most campaigns, tabletop games had the greates successful rate and the smallest failed rate.
* Music, tabletop games and shorts had a greater successful rate than failed rate, whereas all others in the top 10 subcategories with the most campaigns had a smaller successful rate than failed rate.
