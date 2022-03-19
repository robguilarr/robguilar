# Tactical Analytics: A/B Testing with Cookie Cats Game


## What are Tactical Analytics?

A type of user-oriented game analytics, with the purpose to <i>"aim to inform game design at the short-term, for example, an A/B test of a new game feature"</i> (A. Dranchen, 2013).

The applicability of statistics in new fields can be considered as one of the greatest advances for humanity. Nowadays, human-machine interactions are being monitored, in a good way in most cases.

The main purpose is not just to increase the companies revenue, one of the main objectives is to give a benefit in terms of User Experience (UX) and Engagement, and this can be covered using Data Science.

{{< admonition info "Looking for an interactive experience?" true >}}
:rocket: Use or Download the Jupyter Notebook, available <a href="https://nbviewer.org/github/robguilarr/ab_testing_cookie_cats/blob/main/ab_notebook.ipynb">here</a>
{{< /admonition >}}


{{< youtube GaP5f0jVTWE >}}


## Telemetry systems introduction

All this data is being collected through telemetry systems, according to Anders Drachen et al. (one of the <b>pioneers</b> in the Game Analytics field), from an interview made to Georg Zoeller of Ubisoft Singapore, the Game industry manages two kinds of telemetry systems:

- <b>Developer-facing:</b>  <i>"The main goal of the system is to facilitate and improve the production process, gathering and presenting information about how developers and testers interact with the unfinished game"</i>. Like the one mentioned in Chapter 7 of the <a href="https://link.springer.com/book/10.1007/978-1-4471-4769-5">"Game Analytics Maximizing the Value of Player Data"</a> book, which was implemented in Bioware's production process of <a href="https://en.wikipedia.org/wiki/Dragon_Age:_Origins">Dragon Age: Origins</a> in 2009.   
- <b>User-facing:</b> This one is <i>"collected after a game is launched and mainly aimed at tracking, analyzing, and visualizing player behavior"</i> mentioned in Chapters 4, 5, and 6 of the same book.

## About the data collected

<p> <img src="https://raw.githubusercontent.com/robguilarr/ab_testing_cookie_cats/main/images_jup/berry.png" style="width:120px; float:left" hspace="30" vspace="20"> 

Given the introduction of the telemetry systems, in this case, we are going to see a <i>"User-facing"</i> that provided us data from <a href="https://tactilegames.com/cookie-cats/">Cookie Cats</a> game, a mobile puzzle game of <i>"connect-three"</i>-style developed by <a href="https://tactilegames.com/">Tactile Entertainment</a>.  

To be in context, this game's main objective is to align 3 cookies of the same kind to feed a cat, and by this way finish each level. Which also as collectible credit you can earn Keys to unlock gates located at certain levels.

</p>

## Problem context

According to Rasmus Baath, Data Science Lead at <a href="https://castle.io/">castle.io</a>, Tactile Entertainment is planning to move Cookie Cats' time gates from level 30 to 40, but they don't know how much the user retention can be impacted by this decision.

This sort of time gate is usually seen in <a href="https://en.wikipedia.org/wiki/Free-to-play">free-to-play</a> models, and normally contains ads that can be skipped in exchange for in-game purchases, or in this case the player requires to submit a specific number of 'Keys', which also can be skipped in exchange of <a href="https://pegi.info/page/game-purchases">in-game purchases</a>.

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/ab_testing_cookie_cats/main/images_jup/cc_gates.png" style="width: 700px"></p>

So seeing this viewpoint, a decision like this can impact not only user retention, the expected revenue as well.


## About the dataset

This dataset contains around <b>90,189</b> records of players that started the game while the telemetry system was running, according to Rasmus Baath. Among the variables collected are the next:

- <i>userid</i> : unique identification of the user.

- <i>version</i> : the group in which the players were measured, for a time gate at level 30 it contains a string called <i>gate_30</i>, or for a time gate at level 40 it contains a string called <i>gate_40</i>.

- <i>sum_gamerounds</i> : number of game rounds played within the first 14 days since the first session played.

- <i>retention_1</i> : Boolean that defines if the player came back 1 day after the installation.

- <i>retention_7</i> : Boolean that defines if the player came back 7 days after the installation.

<b>Note:</b> An important fact to keep in mind is that in the game industry one crucial metric is <i>retention_1</i>, since it defines if the game is fulfilling and entertaining the final consumer.


```python
# Importing pandas
import pandas as pd

# Reading in the data
df = pd.read_csv('datasets/cookie_cats.csv')

# Showing the first few rows
df.head()
```

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/ab_testing_cookie_cats/main/plot_images/head.png" style="width: 450px"></p>


## Methodology

The most accurate way to test changes is to perform <a href="https://www.analyticsvidhya.com/blog/2020/10/ab-testing-data-science/">A/B testing</a> by targetting a specific variable, in the case <i>retention</i> (for 1 and 7 days after installation).

As we mentioned before, we have two groups in the <i>version</i> variable:

- <b>Control group:</b> The time gate is located at level 30. We are going to consider this one as a no-treatment group.

- <b>Treatment group:</b> The company plans to move the time gate to level 40. We are going to use this as a subject of study, due to the change involved.

In an advanced stage, we are going to perform a <a href="https://www.analyticsvidhya.com/blog/2021/06/bootstrap-the-source-of-its-power/">bootstrapping</a> technique, to be confident about the result comparison for the retention probabilities between groups.


```python
# Counting the number of players in each AB group.
players_g30 = df[df['version'] == 'gate_30']
players_g40 = df[df['version'] == 'gate_40']

print('Number of players tested at Gate 30:', str(players_g30.shape[0]), '\n',
     'Number of players tested at Gate 40:', str(players_g40.shape[0]))
```

```Code
Number of players tested at Gate 30: 44700 
 Number of players tested at Gate 40: 45489
```


## Exploring distribution

As we see the proportion of players sampled for each group is balanced, so for now, only exploring the Game Rounds data is in the queue.

Let's see the distribution of Game Rounds (The plotly **layout** created is available in the Jupyter Notebook).

```python
import matplotlib.pyplot as plt
%matplotlib inline

import plotly.express as px

# Author template
author = "Published at <a href='https://www.robguilar.com'>robguilar&#8482;</a><br>Made by:  R. Aguilar, 2022"
heightimg = 720
widthimg = 1000
fontimg = 13
colorfont= "#636363"

# Distribution Boxplot with outliers
box1 = px.box(df, x="sum_gamerounds",
            title = "Game Rounds Overall Distribution by player", labels = {"sum_gamerounds":"Game Rounds registered"})

box1.update_layout(layout)
box1.show()
```

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/ab_testing_cookie_cats/main/plot_images/box1.png" style="width: 900px"></p>


<p><img src="https://raw.githubusercontent.com/robguilarr/ab_testing_cookie_cats/main/images_jup/smokey.png" style="width:150px; float:left" hspace="20">

For now, we see that exist clear outliers in the dataset since one user has recorded <b>49,854</b> Game rounds played in less than 14 days, meanwhile, the max recorded, excluding the outlier, is around <b>2,900</b>. The only response to this case situation is a "bot", a "bug" or a "glitch".

Nevertheless, it's preferable to clean it, since only affected one record. Let's prune it.

 </p>

```python
df = df[df['sum_gamerounds'] != 49854]
```

We can make an <i>Empirical Cumulative Distribution Function</i>, to see the real distribution of our data.

<b>Note:</b> In this case, we won't use histograms to avoid some bining bias.

```python
import plotly.graph_objects as go

# Import numpy library
import numpy as np

# ECDF Generator function
def ecdf(data):
    # Generate ECDF (Empirical Cumulative Distribution Function)
    # for on dimension arrays
    n = len(data)

    # X axis data
    x = np.sort(data)

     # Y axis data
    y = np.arange(1, n+1) / n

    return x, y

# Generate ECDF data
x_rounds, y_rounds = ecdf(df['sum_gamerounds'])

# Generate percentile makers 
percentiles = np.array([5,25,50,75,95])
ptiles = np.percentile(df['sum_gamerounds'], percentiles)

# ECDF plot
ecdf = go.Figure()

# Add traces
ecdf.add_trace(go.Scatter(x=x_rounds, y=y_rounds,
                    mode='markers',
                    name='Game Rounds'))
ecdf.add_trace(go.Scatter(x=ptiles, y=percentiles/100,
                    mode='markers+text',
                    name='Percentiles', marker_line_width=2, marker_size=10,
                    text=percentiles, textposition="bottom right"))
ecdf.update_layout(layout)
ecdf.update_layout(title='Game Rounds Cumulative Distribution Plot', yaxis_title="Cumulative Probability")
ecdf.show()
```

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/ab_testing_cookie_cats/main/plot_images/ecdf1.png" style="width: 900px"></p>

As we see 95% of our data is below 500 Game Rounds.

```python
print("The 95 percentile of the data is at: ", ptiles[4], "Game Rounds","\n",
"This means ", df[df["sum_gamerounds"] <= ptiles[4]].shape[0], " players")
```
```Code
The 95 percentile of the data is at:  221.0 Game Rounds 
 This means  85706  players
 ```

 For us, this can be considered a valuable sample.

In the plot above, we saw some players that installed the game but, then never return (0 game rounds).

```python
print("Players inactive since installation: ", df[df["sum_gamerounds"] == 0].shape[0]) 
```
```Code
Players inactive since installation:  3994
```

And in most cases, players just play a couple of game rounds in their first two weeks. But, we are looking for players that like the game and to get hooked, that's one of our interests.

A common metric in the video gaming industry for how fun and engaging a game is 1-day retention as we mentioned before.

Before proceeding let's make it clear.

### What is retention and, why is soo important?

<p><img src="https://raw.githubusercontent.com/robguilarr/ab_testing_cookie_cats/main/images_jup/rita.png" style="width:150px; float:right" hspace="20"> 

Retention is the percentage of players that come back and plays the game one day after they have installed it. The higher 1-day retention is, the easier it is to retain players and build a large player base.

According to Anders Drachen et al. (2013), these customer kind metrics <i>"are notably interesting to professionals working with marketing and management of games and game development"</i>, also this metric is described simply as <i>"how sticky the game is"</i>, in other words, it's essential.

As a first step, let's look at what 1-day retention is overall.


</p>

```python
# The % of users that came back the day after they installed
prop = len(df[df['retention_1'] == True]) / len(df['retention_1']) * 100

print("The overall retention for 1 day is: ", str(round(prop,2)),"%")
```
```Code
The overall retention for 1 day is:  44.52 %
```

Less than half of the players come back one day after installing the game.

Now that we have a benchmark, let's look at how 1-day retention differs between the two AB groups.

## 1-day retention by A/B Group

Computing the retention individually, we have the next results.

```python
# Calculating 1-day retention for each AB-group

# CONTROL GROUP
prop_gate30 = len(players_g30[players_g30['retention_1'] == True])/len(players_g30['retention_1']) * 100

# TREATMENT GROUP
prop_gate40 = len(players_g40[players_g40['retention_1'] == True])/len(players_g40['retention_1']) * 100

print('Group 30 at 1 day retention: ',str(round(prop_gate30,2)),"%","\n",
     'Group 40 at 1 day retention: ',str(round(prop_gate40,2)),"%")
```
```Code
Group 30 at 1 day retention:  44.82 % 
 Group 40 at 1 day retention:  44.23 %
```

It appears that there was a slight decrease in 1-day retention when the gate was moved to level 40 (<b>44.23%</b>) compared to the control when it was at level 30 (<b>44.82%</b>).

It's a smallish change, but even small changes in retention can have a huge impact. While we are sure of the difference in the data, how confident should we be that a gate at level 40 will be more threatening in the future?

For this reason, it's important to consider bootstrapping techniques, this means <i>"a sampling with replacement from observed data to estimate the variability in a statistic of interest"</i>. In this case, retention, and we are going to do a function for that.

```python
# Bootstrapping Function
def draw_bs_reps(data,func,iterations=1):
    boot_Xd = []
    for i in range(iterations):
        boot_Xd.append(func(data = np.random.choice(data, len(data))))
    return boot_Xd
# Retention Function
def retention(data):
    ret = len(data[data == True])/len(data)
    return ret
```

### Control Group Bootstrapping

```python
# Bootstrapping for gate 30
btg30_1d = draw_bs_reps(players_g30['retention_1'], retention, iterations = 1000)
```

### Treatment Group Bootstrapping

```python
# Bootstrapping for gate 40
btg40_1d = draw_bs_reps(players_g40['retention_1'], retention, iterations = 1000)
```

Now, let's check the results.

```python
import plotly.figure_factory as ff

mean_g40 = np.mean(btg40_1d)
mean_g30 = np.mean(btg30_1d)

# A Kernel Density Estimate plot of the bootstrap distributions
boot_1d = pd.DataFrame(data = {'gate_30':btg30_1d, 'gate_40':btg40_1d},
                       index = range(1000))

# Plotting histogram
hist_1d = [boot_1d.gate_30, boot_1d.gate_40]
dist_1d = ff.create_distplot(hist_1d, group_labels=["Gate 30 (Control)", "Gate 40 (Treatment)"], show_rug=False, colors = ['#3498DB','#28B463'])
dist_1d.add_vline(x=mean_g40, line_width=3, line_dash="dash", line_color="#28B463")
dist_1d.add_vline(x=mean_g30, line_width=3, line_dash="dash", line_color="#3498DB")
dist_1d.add_vrect(x0=mean_g30, x1=mean_g40, line_width=0, fillcolor="#F1C40F", opacity=0.2)

dist_1d.update_layout(layout)
dist_1d.update_layout(xaxis_range=[0.43,0.46])
dist_1d.update_layout(title='1-Day Retention Bootstrapping by A/B Group', xaxis_title="Retention")
dist_1d.show()
```

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/ab_testing_cookie_cats/main/plot_images/dist1.png" style="width: 900px"></p>

The difference still looking close, for this reason, is preferable to zoom it by plotting the difference as an individual measure.

```python
# Adding a column with the % difference between the two AB-groups
boot_1d['diff'] = (
                    ((boot_1d['gate_30'] - boot_1d['gate_40']) / boot_1d['gate_40']) * 100
                )

# Ploting the bootstrap % difference
hist_1d_diff = [boot_1d['diff']]
dist_1d_diff = ff.create_distplot(hist_1d_diff, show_rug=False, colors = ['#F1C40F'],
                                    group_labels = ["Gate 30 - Gate 40"], show_hist=False)
dist_1d_diff.add_vline(x= np.mean(boot_1d['diff']), line_width=3, line_dash="dash", line_color="black")
dist_1d_diff.update_layout(layout)
dist_1d_diff.update_layout(xaxis_range=[-3,6])
dist_1d_diff.update_layout(title='Percentage of "1 day retention" difference between A/B Groups', xaxis_title="% Difference")
dist_1d_diff.show()
```

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/ab_testing_cookie_cats/main/plot_images/dist2.png" style="width: 900px"></p>

From this chart, we can see that the percentual difference is around <i>1% - 2%</i>, and that most of the distribution is above <i>0%</i>, in favor of a gate at level 30. 

But, what is the probability that the difference is above <i>0%</i>? Let's calculate that as well.

```python
# Calculating the probability that 1-day retention is greater when the gate is at level 30
prob = (boot_1d['diff'] > 0.0).sum() / len(boot_1d['diff'])

# Pretty printing the probability
print('The probabilty of Group 30 (Control) having a higher \n retention than Group 40 (Treatment) is: ', prob*100, '%')
```
```Code
The probabilty of Group 30 (Control) having a higher 
 retention than Group 40 (Treatment) is:  96.39999999999999 %
 ```

## 7-day retention by A/B Group

The bootstrap analysis tells us that there is a high probability that 1-day retention is better when the time gate is at level 30. However, since players have only been playing the game for one day, likely, most players haven't reached level 30 yet. That is, many players won't have been affected by the gate, even if it's as early as level 30.

But after having played for a week, more players should have reached level 40, and therefore it makes sense to also look at 7-day retention. That is: <i>What percentage of the people that installed the game also showed up a week later to play the game again?</i>

Let's start by calculating 7-day retention for the two AB groups.

```python
# Calculating 7-day retention for both AB-groups
ret30_7d = len(players_g30[players_g30['retention_7'] == True])/len(players_g30['retention_7']) * 100
ret40_7d = len(players_g40[players_g40['retention_7'] == True])/len(players_g40['retention_7']) * 100

print('Group 30 at 7 day retention: ',str(round(ret30_7d,2)),"%","\n",
     'Group 40 at 7 day retention: ',str(round(ret40_7d,2)),"%")
```
```Code
Group 30 at 7 day retention:  19.02 % 
 Group 40 at 7 day retention:  18.2 %
```

Like with 1-day retention, we see that 7-day retention is barely lower (<b>18.20%</b>) when the gate is at level 40 than when the time gate is at level 30 (<b>19.02%</b>). This difference is also larger than for 1-day retention.

We also see that the overall 7-day retention is lower than the overall 1-day retention; fewer people play a game a week than a day after installing.

But as before, let's use bootstrap analysis to figure out how sure we can be of the difference between the AB-groups.

### Control & Treatment Group Bootstrapping

```python
# Creating a list with bootstrapped means for each AB-group

# Bootstrapping for CONTROL group
btg30_7d = draw_bs_reps(players_g30['retention_7'], retention, iterations = 500)

# Bootstrapping for TREATMENT group
btg40_7d = draw_bs_reps(players_g40['retention_7'], retention, iterations = 500)

boot_7d = pd.DataFrame(data = {'gate_30':btg30_7d, 'gate_40':btg40_7d},
                       index = range(500))

# Adding a column with the % difference between the two AB-groups
boot_7d['diff'] = (boot_7d['gate_30'] - boot_7d['gate_40']) /  boot_7d['gate_30'] * 100

# Ploting the bootstrap % difference
hist_7d_diff = [boot_7d['diff']]
dist_7d_diff = ff.create_distplot(hist_7d_diff, show_rug=False, colors = ['#FF5733'],
                                    group_labels = ["Gate 30 - Gate 40"], show_hist=False)
dist_7d_diff.add_vline(x= np.mean(boot_7d['diff']), line_width=3, line_dash="dash", line_color="black")
dist_7d_diff.update_layout(layout)
dist_7d_diff.update_layout(xaxis_range=[-4,12])
dist_7d_diff.update_layout(title='Percentage of "7 day retention" difference between A/B Groups', xaxis_title="% Difference")
dist_7d_diff.show()

# Calculating the probability that 7-day retention is greater when the gate is at level 30
prob = (boot_7d['diff'] > 0).sum() / len(boot_7d)

# Pretty printing the probability
print('The probabilty of Group 30 (Control) having a higher \n retention than Group 40 (Treatment) is: ~', prob*100, '%')
```

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/ab_testing_cookie_cats/main/plot_images/dist3.png" style="width: 900px"></p>

```Code
The probabilty of Group 30 (Control) having a higher 
 retention than Group 40 (Treatment) is: ~ 100.0 %
```

## Final thoughts

<p><img src="https://raw.githubusercontent.com/robguilarr/ab_testing_cookie_cats/main/images_jup/ziggy.png" style="width:150px; float:right" hspace="20"> 

Now we have enough statistical evidence to say that 7-day retention is higher when the gate is at level 30 than when it is at level 40, the same as we concluded for 1-day retention.

If we want to keep retention high, we should not move the gate from level 30 to level 40, it means we keep our Control method in the current gate system. 

Far beyond, there are other metrics we could look at in the future, like:

- The number of game rounds played

- Daily active users

- In-game purchases estimates

- The cost of the customer acquisition

</p>


As we underlined retention is crucial, because if we don't retain our player base, it doesn't matter how much money they spend in-game purchases.

So, why is retention higher when the gate is positioned earlier? Normally, we could expect the opposite: The later the obstacle, the longer people get engaged with the game. But this is not what the data tells us.

According to Rasmus Baath, there is an explanation for this phenomenon and it's called, the <b>theory of hedonic adaptation</b>, which is <i>"the tendency for people to get less and less enjoyment out of a fun activity over time if that activity is undertaken continuously"</i>. 

So, by pushing players to take a break when they reach a gate, the fun of the game is postponed. But, when the gate is moved to level 40, they are more likely to quit the game because they simply got bored of it.

---

## Aditional Information

- <b>About the article</b> {{< version 0.1.1 >}}

With acknowledgment to Rasmus Baraath for guiding this project. Which was developed for sharing knowledge while using cited sources of the material used.

Thanks to you for reading as well. <i class="far fa-grin"></i>

- <b>Related content</b>

For more content related to the authors mentioned, I invite you to visit the next sources:

-- Anders Drachen personal <a href="https://andersdrachen.com/">website</a>.

-- Rasmus Baath personal <a href="https://www.sumsar.net/">blog</a>.

-- Georg Zoeller personal <a href="https://keybase.io/georgzoeller">keybase</a>.

Also in case you want to share some ideas, please visit the <a href="https://www.robguilar.com/about/">About</a> section and contact me. 

- <b>Datasets</b>

This project was developed with a dataset provided by Rasmus Baath, which also can be downloaded at my <a href="https://github.com/robguilarr/ab_testing_cookie_cats/tree/main/datasets">Github</a> repository. 



