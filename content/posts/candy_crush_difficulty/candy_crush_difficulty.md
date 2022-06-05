---
#weight: 2
title: "Leveling Difficulty in Candy Crush Saga"
date: 2022-04-19T21:57:40+08:00
lastmod: 2022-04-19T21:57:40+08:00
draft: false
author: "Roberto Aguilar"
authorLink: ""
description: "Leveling Difficulty in Candy Crush Saga"
images: ["images/posts/candy_crush_difficulty.jpg"]
thumbnail: "images/posts/candy_crush_difficulty.jpg"
tags: ["Bernoulli", "Python"]
categories: ["Projects"]
lightgallery: true
toc: true
math: true
resources:
- name: "featured-image"
  src: "images/posts/candy_crush_difficulty.jpg"
featuredImagePreview: "images/posts/candy_crush_difficulty.jpg"

---

Today's analysis will lead us to a world fulfilled with divine puzzle adventures at the side of Tiffi and Mr. Toffee, in which we'll glimpse the success rate of more than 6800 peers with eagerness for treats.

The game is [Candy Crush Saga](https://www.king.com/game/candycrush) a record-breaking mobile game developed by King, a subsidiary of Activision Blizzard since 2016.

In terms of its gameplay, this game used to have a total of 816 episodes, until one of the major add-ons was removed (Dreamworld). The point is that now the game has 771 episodes, in which the player has 5 episodes per world, and, each episode contains contain 15 levels approximately.

If you are one of the few that haven't played Candy Crush, here's a short intro:

{{< admonition info "Looking for an interactive experience?" true >}}
:rocket: Download the Jupyter Notebook, available <a href="https://nbviewer.org/github/robguilarr/candy_crush_difficulty/blob/master/notebook.ipynb">here</a>
{{< /admonition >}}

{{< youtube HGLGxnfs_t8 >}}

## ⚠️ Introduction to problem

### Hypothesis

We'll review a game that potentially can lead any developer to many unseen problems, considering the abundance of levels. From the perspective of a customer, there can be several points of view that can emerge and, at the same time, can become unnoticed. That's why our diagnosis will start from 2 potential hypothesis:

1. The game is too easy so it became boring over time
2. The game is too hard so the players leave it and become frustrated

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/candy_crush_difficulty/master/plots/map.png" style="width: 700px"></p>

### Potential Stakeholders

None of the past hypotheses are the main intentions of the developers. So they require a Data Analyst to help with this task since the developers are seeing only the backend factors affecting the game, but it's also critical to consider those external ones that affect the experience for the player and the sustainability of this game for the company. Among the main stakeholders could be:

- Level Designers: They work aligned with the rest of the Engineering Team because they still have a backend perspective and their next patch release needs to be aligned with the insights given by the analyst.
- Mobile Designer & User Retention Expert: This is a game whose main income input is in-game purchases because it’s a [F2P](https://en.wikipedia.org/wiki/Free-to-play), the main source of income is centered in retain the engagement in the game and keeping the consumers on the platform.
- Gameplay Engineer: They require to start working on the difficulty adjustment patch as soon as they receive the final statement.
- Executive Producer: Besides Candy Crush being an IP with internal producers since it's developed and published by King, the [parent company](https://www.activisionblizzard.com/) will expect to have an [ROI](https://www.forbes.com/advisor/investing/roi-return-on-investment/) aligned with their expectations.
- Players' community: They expect to have an endurable and great experience with a brief response in case of disconformities.


**Note:** To facilitate the understanding of the roles of the development team, I invite you to take a look at **[this](https://miro.com/app/board/uXjVOwQVsRk=/?share_link_id=215280076044)** diagram that I designed.

## 📥 About the data

### Collection process and structure

Before start let’s import the libraries we’re going to need

```python
import pandas as pd
import numpy as np

# For visualization
import matplotlib.pyplot as plt
%matplotlib inline
import plotly.express as px
import plotly.graph_objects as go

# Own layout design library
from vizformatter.standards import layout_plotly

# Load layout base objects
sign, layout = layout_plotly(height= 720, width= 1000, font_size= 15)
```

Due to the extensive size of possible episodes to analyze, we’ll limit the analysis to just one of them, which exactly will have data available for 15 levels. To do this, the analysts need to request a sample from the telemetry systems to get data related to the number of attempts per player in each episode level.

Also, it’s important to mention that in terms of privacy, this analysis requires importing the data with the *player_id* codified for privacy reasons. Fortunately, in this case, Rasmus Baath, Data Science Lead at [castle.io](https://castle.io/), provided us with a Dataset with a sample gathered in 2014.

```python
df = pd.read_csv("datasets/candy_crush.csv")
df.head()
```

<p align="left"><img src="https://raw.githubusercontent.com/robguilarr/candy_crush_difficulty/master/plots/table1.png" style="width: 600px"></p>

We can see that our data is **structured** and consists of 5 attributes:

- *player_id*: a unique player id
- *dt*: the date
- *level*: the level number within the episode, from 1 to 15
- *num_attempts*: number of level attempts for the player on that level and date
- *num_success*: number of level attempts that resulted in a success/win for the player on that level and date

## 🔧 Data Preprocessing

Before starting the analysis we need to do some validations on the dataset.

```python
# Count and display the number of unique players
print("Number of players: \n", df.player_id.nunique(), '\n',
        "Number of records: \n", len(df.player_id),'\n')

# Display the date range of the data
print("Period for which we have data: \nFrom: ",
        min(df.dt), ' To:', max(df.dt))
```

```
Number of players: 
 6814 
 Number of records: 
 16865 

Period for which we have data: 
From:  2014-01-01  To: 2014-01-07
```

### Data Cleaning

The data doesn’t require any kind of transformation and the data types are aligned with their purpose.

```python
print(df.dtypes)
```

```
player_id       object
dt              object
level            int64
num_attempts     int64
num_success      int64
dtype: object
```

### Data Consistency

The usability of the data it’s rather good, since we don’t count with “NAN” (Not A Number), “NA” (Not Available), or “NULL” (an empty set) values.

```python
# Function the plot the percentage of missing values
def na_counter(df):
    print("NaN Values per column:")
    print("")
    for i in df.columns:
        percentage = 100 - ((len(df[i]) - df[i].isna().sum())/len(df[i]))*100

        # Only return columns with more than 5% of NA values
        if percentage > 5:
            print(i+" has "+ str(round(percentage)) +"% of Null Values")
        else:
            continue

# Execute function
na_counter(df)
```

```
NaN Values per column:
 None
```

By this way, we can conclude that there were not errors in our telemetry logs during the data collection.

### Normalization

Next, we can conclude there were no impossible numbers, except for a player that tried to complete the level 11 in 258 attempts with just 1 success. This is the only registry we exclude since it can be an influential outlier and we don’t rely on more attributes about him to create conclusions.

Noticing the distribution of the quartiles and comprehending the purpose of our analysis, we can validate that the data is comparable and doesn’t need transformations.

```python
df = df[df.num_attempts != 258]
df.describe()
```

<p align="left"><img src="https://raw.githubusercontent.com/robguilarr/candy_crush_difficulty/master/plots/table2.png" style="width: 350px"></p>

## 🔍 Exploratory Analysis & In-game interpretations

### Summary statistics

Excluding the outliers we mentioned before, we got the next conclusions about their distribution and measurement:

- *player_id*
    - Interpretation: Not unique and counts with 6814 distinct values which make sense since there is a player with records of multiple levels
    - Data type: Nominal
    - Measurement type: Discrete/String
- *dt*
    - Interpretation: Only includes data from January 1st to January 7th of 2014. Also, the analysis won’t consider this as a lapse per player since the records per player are not continuous, so they will be limited as a timestamp
    - Data type: Ordinal
    - Measurement type: Temporal
- *level*
    - Interpretation: They're registered as numbers, but for further analysis will be transformed as factors. 50% of the records are equal to or less than level 9
    - Data type: Ordinal
    - Measurement type: Discrete/String
- *num_attempts*
    - Interpretation: The registries are consistent, the interquartile range mention that half of the players try between 1 and 7 time to complete each level. Furthermore, there are players with 0 attempts, so we need to evaluate if this is present at level 1, which can explain a problem in retention rate for that episode
    - Data type: Numerical
    - Measurement type: Integer
- *num_success*
    - Interpretation: Most of the players are casual gamers because 75% of them complete the level and don’t repeat it
    - Data type: Numerical
    - Measurement type: Integer

### Levels played in Episode

First, let’s examine the number of registries per player.

This will tell us, from the episode how many levels have each player recorded in the lapse of 7 days.

```python
from plotly.subplots import make_subplots

# Group data of amount of levels recorded by player id
countdf = df.groupby('player_id')['level'].nunique().reset_index()

# Count the number amount of players according to amount of levels recorded by player
countdf = countdf.groupby('level')['player_id'].nunique().reset_index()

# Arrange data according to amount of levels
countdf.level = [str(i)+'s' for i in countdf.level]
countdf = countdf.sort_values('player_id', ascending= False)

# Generate CumSum
cumulative_per =  countdf.player_id / countdf.player_id.sum() * 100
cumulative_per = cumulative_per.cumsum()

# Format new DF
countdf = pd.concat([countdf, cumulative_per], axis = 1)
countdf.columns = ["levels","players","Cum_per"]

# Pareto Chart
linec = make_subplots(specs=[[{"secondary_y": True}]])

# Bar plot graphic object
linec.add_trace(go.Bar(x = countdf.levels, y = countdf.players, name = "Players", marker_color= "#007FFF"),
                        secondary_y = False)

# Scatter plot graphic object
linec.add_trace(go.Scatter(x = countdf.levels, y = countdf.Cum_per/100, mode='lines+markers', name = "Percentage", marker_color= "#FF5A5F"),
                        secondary_y = True)

# Layout
linec.update_layout(title = {'text':'Pareto Chart of Number of Levels recorded by player'},
                    xaxis = {"title":"Number of Levels recorded"},
                    yaxis = {"title":"Unique players"})
linec.update_layout(layout)
linec.update_yaxes(tickformat = "0%", title = "Cumulative Percentage", secondary_y = True)
linec.update_layout(showlegend=False)
linec.add_hline(y=0.8, line_dash = "dash", line_color="red", opacity=0.5, secondary_y = True)
linec.add_annotation(sign)
linec.show()
```

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/candy_crush_difficulty/master/plots/pareto.png" style="width: 900px"></p>

From the last Pareto chart, we can deduce that 80% of the 6814 players just count with 3 levels recorded of 15. But, since this was extracted from a random sample, this won’t affect our study.

### Difficulty of completing a level in a single try

There is a combination of easier and challenging levels. Chance and skills make the number of attempts required to pass a level different from one player to another. The presumption is that difficult levels demand more tries on average than easier ones. That is, the harder a level is the lower the likelihood to pass that level in a single try.

In these circumstances, the [Bernoulli process](https://en.wikipedia.org/wiki/Bernoulli_process) might be useful. As a *Boolean* result, there are only two possibilities, win or lose. This can be measured by a single parameter:

$p_{win} = \frac{\Sigma wins }{\Sigma attempts }$:  the probability of completing the level in a single attempt

Let's calculate the difficulty $p_{win}$ individually for each of the 15 levels.

```python
difficulty = df.groupby('level').agg(attempts = ('num_attempts', 'sum'),
                                    wins =('num_success', 'sum')).reset_index()
difficulty['p_win'] = difficulty.wins / difficulty.attempts
difficulty
```

<p align="left"><img src="https://raw.githubusercontent.com/robguilarr/candy_crush_difficulty/master/plots/table3.png" style="width: 250px"></p>

We have levels where 50% of players finished on the first attempt and others that are the opposite. But let’s visualize it through the episode, to make it clear.

```python
# Lineplot of Success Probability per Level
line1 = px.line(difficulty, x='level', y="p_win",
                title = "Probability of Level Success at first attempt",
                labels = {"p_win":"Probability", "level":"Episode Level"})
line1.update_layout(layout)
line1.update_xaxes(range=[1,15], tick0 = 1, dtick = 1)
line1.update_yaxes(range=[0,0.7], tick0 = 0, dtick = 0.1)
line1.update_layout(yaxis_tickformat = "0%")
line1.add_annotation(sign)
line1.show()
```

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/candy_crush_difficulty/master/plots/line1.png" style="width: 900px"></p>


### Defining hard levels

It’s subjective what we can consider a hard level because not consistently depends on a single factor and for all player profile groups this can be different. So, for the outcomes of this study, we will arbitrarily assume that a difficult level is the one that has a probability to be completed in the first attempt of a 10% ($p_{win} < 10\%$).

```python
# Lineplot of Success Probability per Level
line2 = go.Figure(go.Scatter(
    x = difficulty.level,
    y = difficulty.p_win))
line2.update_layout(title = {'text':'Probability of Level Success at first attempt with Hard Levels'},
                    xaxis = {"title":"Episode Level"},
                    yaxis = {"title":"Probability"})
line2.update_layout(layout)
line2.update_xaxes(range=[1,15], tick0 = 1, dtick = 1)
line2.update_layout(yaxis_tickformat = "0%")
line2.update_layout(showlegend=False)
line2.add_hrect(y0=0.02, y1=0.1, line_width=0, fillcolor="red", opacity=0.2)
line2.add_annotation(sign)
line2.show()
```

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/candy_crush_difficulty/master/plots/line2.png" style="width: 900px"></p>

From our predefined threshold, we see that the level digit is not aligned with its difficulty. While we have hard levels as 5, 8, 9, and 15; others like 13 and 15 are unleveraged and need to be rebalanced by the level designers.

### Measuring the uncertainty of success

We should always report some calculation of the uncertainty of any provided numbers. Simply, because another sample will give us little different values for the difficulties measured by level.

Here we will simply use the [Standard error](https://en.wikipedia.org/wiki/Standard_error) as a measure of uncertainty:

$\sigma_{error} \approx \frac{\sigma_{sample}}{\sqrt{n}}$

Here n is the number of datapoints and $\sigma_{sample}$ is the sample standard deviation. For a Bernoulli process, the sample standard deviation is:

$\sigma_{sample} = \sqrt{p_{win}(1-p_{win})}$

Therefore, we can calculate the standard error like this:

$\sigma_{error} \approx \sqrt{\frac{p_{win}(1-p_{win})}{n} }$

Consider that every level has been played *n* number of times and we have their difficulty $p_{win}$. Now, let's calculate the standard error for each level of this episode.

```python
# Computing the standard error of p_win for each level
difficulty['error'] = np.sqrt(difficulty.p_win * (1 - difficulty.p_win) / difficulty.attempts)
difficulty
```

<p align="left"><img src="https://raw.githubusercontent.com/robguilarr/candy_crush_difficulty/master/plots/table4.png" style="width: 300px"></p>

We have a measure of uncertainty for each levels' difficulty. As always, this would be more appealing if we plot it.

Let's use *error bars* to show this uncertainty in the plot. We will set the height of the error bars to one standard error. The upper limit and the lower limit of each error bar should be defined by:

$p_{win}  \pm \sigma_{error}$

```python
# Lineplot of Success Probability per Level
line3 = px.line(difficulty, x='level', y="p_win",
                title = "Probability of Level Success at first attempt with Error Bars",
                labels = {"p_win":"Probability", "level":"Episode Level"},
                error_y="error")
line3.update_layout(layout)
line3.update_xaxes(range=[0,16], tick0 = 1, dtick = 1)
line3.update_yaxes(range=[0,0.65], tick0 = 0, dtick = 0.1)
line3.update_layout(yaxis_tickformat = "0%")
line3.add_hrect(y0=0.02, y1=0.1, line_width=0, fillcolor="red", opacity=0.2)
line3.add_annotation(sign)
line3.show()
```

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/candy_crush_difficulty/master/plots/line3.png" style="width: 900px"></p>

Looks like the difficulty estimates a very exact. Furthermore, for the hardest levels, the measure is even more precise, and that’s a good point because from this we can make valid conclusions based on that levels.

As a curious fact, also we can measure the probability of completing all the levels of that episode in a single attempt, just for fun.

```python
# The probability of completing the episode without losing a single time
prob = 1

for i in difficulty.p_win:
    prob = prob*i

# Printing it out
print("Probability of Success in one single attempt \nfor whole episode: ", prob*100, "%")
```

```
Probability of Success in one single attempt 
for whole episode:  9.889123140886191e-10 %
```

## 🗒️ Final thoughts & takeaways

**What can the stakeholders understand and take into consideration?**

From the sample extracted we conclude that just 33% of the levels are considered of high difficulty, which it’s acceptable since each episode counts with 15 levels, so by now the level designer should not worry about leveling the difficulty.

**What could the stakeholders do to take action ?**

As a suggestion, in the case that the Publisher decides to invest more in in-game mechanics, a solution for a long-time and reactive engagement could be the use of Machine Learning to generate a [DGDB](https://en.wikipedia.org/wiki/Dynamic_game_difficulty_balancing) as some competitors have adapted in IPs like EA Sports FIFA, Madden NFL or the “AI Director” of Left 4 Dead.

**What can stakeholders keep working on?** 

The way their level difficulty design work today is precise since our first hypothesis was that the game wasn’t too linear to unengaged the player and churn as consequence. Because as we saw, the game has drastic variations in the levels 5-6 and 8-10, which can help to avoid frustrations in players.

---

## ℹ️ Additional Information

- **About the article**

Based on the dataset provided, we will not proceed with a retention analysis as mentioned above. Because the data is from a random episode, if this were episode one, this type of analysis could be useful as it can explain the pool of players who log in, created an account, or install the game but never start playing, causing traction problems. Therefore, this will be left as a limitation to the scope of the analysis.

With acknowledgment to Rasmus Baraath for guiding this project. Which was developed for sharing knowledge while using cited sources of the material used.

Thanks to you for reading as well.

- **Related Content**

— Rasmus Baath personal [blog](https://www.sumsar.net/)

— Anders Drachen personal [website](https://andersdrachen.com/)

- **Datasets**

This project was developed with a dataset provided by Rasmus Baraath, which also can be downloaded at my <a href="https://github.com/robguilarr/candy_crush_difficulty/tree/master/datasets">Github</a> repository.