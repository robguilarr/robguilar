# Inspecting Control Mechanics using T-Tests on Fallout New Vegas


The plot of the sequel of Fallout 3 takes context in the year 2281, 4 years after the first escape from Vault 101, within a post-apocalyptic world after the Great War.

This RPG has an open world inspired by cities such as Nevada, California, Arizona, and Utah, allowing users to have a large number of places to explore and a world with NPCs to interact with, ideal for developing behavioral and exploratory analysis.

After its launch in 2010, like many titles of this IP, they have been exposed to mods developed by their community, some of them for Serious-Game purposes and others as non-Serious games. According to Anders Drachen et al. (2021, p. 5), it's "a term used to describe games developed for purposes other than entertainment, such as training, promoting health, citizen science, or psychological experiments".

In this case, the data was obtained from a mod developed at the PLAIT (Playable Interactive Technologies) by Northeastern University. The mod is called VPAL: Virtual Personality Assessment Lab, and also can be accessed to the raw telemetry data in the Game Data Science [book](https://global.oup.com/academic/product/game-data-science-9780192897879?cc=cr&lang=en&).

It’s important to keep in mind that the game has been already released, so we'll require to develop a post-production analysis with a help of a mod, so the main conclusions will be focused on a fictional patch improvement made by the Programming team.

{{< admonition info "Looking for an interactive experience?" true >}}

:rocket: Download the Jupyter Notebook, available <a href="https://nbviewer.org/github/robguilarr/control_mechanics_fallout/blob/master/notebook.ipynb">here</a>

{{< /admonition >}}

{{< youtube cIzOttk6Dv4 >}}

## ⚠️ Introduction to problem

### Hypothesis

The Producers are concerned for some comments about a change made in the **control mechanics** since the last patch was released.

The UX research team has communicated to them that the QA testers think that some of them found the quest's difficulty too easy and others very leveraged, especially for veteran players of the saga, reason why we can start inferring about changes made in the gameplay experience, which the major changes made were on the mechanics.

The problem is that the QA team is very diversified between experienced and non-experienced testers, so it won’t be easy to make conclusions. For this reason, we need to look for statistical evidence to validate a hypothesis based on the profile of those testers. Now let’s establish the main hypothesis for the analysis:

1. Experienced players/testers will have completed **more** quests and more kills (the usual), the mechanical changes were not significant.
2. Inexperienced players/testers will have completed **more or an equal** quantity of quests and kills (not usual), the mechanical changes were significant.

From all the KPIs that you will see next, it’s important to mention that we’ll bring kills as a secondary validation measure since Fallout it’s a survival-RPG-like where you have to kill radioactive NPCs in almost every quest. We won’t count with a difficulty category for each player, so the kills attribute will be one of the few not affected by the difficulty categorization, also because it’s an end measure, not a progressive one, like shots for example.

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/control_mechanics_fallout/master/images/game_capture.jpg" style="width: 750px"></p>

### Potential Stakeholders

There is an unconformity in the changes made in the control mechanics, the team requires the assistance of a Data Analyst to perform a test and validate if the QA tester's statements make sense. On the other side of the table we have some stakeholders on the development and testing side:

- Gameplay Engineers: They need to know if the control mechanics are affecting the interactions of the NPCs with the progress of the players since the release of the last patch.
- Level Designer: As the whole RPG is designed as quests on an isolated map, they need to know if these are balanced in the sequence or chapter located, or if there is a significant change in the patch that can explain the unconformity.
- QA Testers: They are a key source of information while communicating the QA statement to the UX Designer.
- UX Designer: They are working side by side with the consumer insights team and the testers to find a clear current situation and take immediate actions if required, by giving the final report to the level designers and the programmers.

**Note:** To facilitate the understanding of the roles of the development team, I invite you to take a look at **[this](https://www.robguilar.com/posts/gamedev_structure/)** diagram that I designed.

## 📥 About the data & preprocessing reference

In a summary, the data is a complete chaos allocated in a repository of text files, so this will require a pipeline to be transformed, cleaned, and get ready to use.

The purpose of data cleaning is to make sure the data is correct. It is rarely the case that once data is collected through the game and transferred to the server, it is automatically ready for analysis.

### Collection process and structure

Before start let’s import the libraries we’re going to need for the preprocess.

```python
import numpy as np
import pandas as pd

# Visuals
import plotly.express as px
import plotly.figure_factory as ff

# Own package
from vizformatter.standards import layout_plotly

# Load layout base objects
sign, layout = layout_plotly(height= 720, width= 1000, font_size= 15)
```

The instrumentation of the data and storage of all game actions within the game were done through TXT files that are stored on the client side per player, with an unstructured format. For each one, we created a file named [participantNumber].txt, which includes all session data for that participant. If the participant has more than one session, a folder is created, and multiple files are created for that participant.

The transfer process can be made by using a product like Databricks, to run an entire pipeline over a PySpark Notebook. As a demonstration we made an example in a Jupyter notebook, where we got several methods to target, extract, process singles labeled activities, and merge into activity-designated CSVs, which were originally parsed from TXT files by player record. In reality the best practices show that the data can be in a JSON file, stored in a MongoDB, or an SQL database.

The collection was an extensive process so you can access the simulation right **[here](https://github.com/robguilarr/parse_fallout_data/blob/master/parse_actions_notebook.ipynb)** with a complete explanation of the bad practices made and how we tackled them. Also, it’s important to mention that the data was parsed into a single data frame with counted activities per player. After this, we applied a Likert scale to an experience metric according to Anders Drachen et al. (2021), which helped us to separate the data into Experienced and Inexperienced players.

First let’s check our data.

```python
# Inexperienced group of players dataframe
InExperienced = pd.read_csv("data/ExpNeg_GameData.csv")

# Experienced group of players dataframe
Experienced = pd.read_csv("data/ExpPos_GameData.csv")

Experienced.head()
```

<p align="left"><img src="https://raw.githubusercontent.com/robguilarr/control_mechanics_fallout/master/images/head1.jpg" style="width: 1000px"></p>

Both data frames have 17 attributes and, the Experienced data have 28 rows, while the Inexperienced data has 42 rows. In our case the attributes in use will be “Quest Completed” and “Kills” will be the variables to test.

- *Quest Completed*: Integer number counting the number of quests the tester completed during the session
- *Kills*: Number of kills registered by the player during the session

## 🔧 Data Preprocessing

Before moving to our analysis it’s important to validate that our data is usable in order to make correct conclusions for the analysis.

### Data Cleaning & Consistency

First let’s validate the data types and the ranges for both groups.

```python
InExperienced.loc[:,["User ID","Kills", "Quest Completed"]].describe()
```

<p align="left"><img src="https://raw.githubusercontent.com/robguilarr/control_mechanics_fallout/master/images/describe1.jpg" style="width: 320px"></p>

```python
Experienced.loc[:,["User ID","Kills", "Quest Completed"]].describe()
```

<p align="left"><img src="https://raw.githubusercontent.com/robguilarr/control_mechanics_fallout/master/images/describe2.jpg" style="width: 320px"></p>

As we saw there is no problems linked to the data types, because in the data parsing noted above we took care of it. And from the ranges we infer that the maximum number of Kills registered is from an Inexperienced player with 44, but at the same time 75% of the Experienced players register a higher number in comparison to the Inexperienced ones, the same case apply to the number of Quests Completed.

## 🔍 Inferential Analysis & In-game interpretations

### Descriptive statistics

Now, we got the next conclusions about their distribution and measurement:

- User ID
    - Interpretation: Unique and counts with 70 distinct values which make sense since there is a player associated to each one of the data-row parsed
    - Data type: Nominal transformed to numerical
    - Measurement type: Discrete/String transformed to integer
- Quest Completed
    - Interpretation: Not unique and counts the quests registries per player, and at first sight make sense that the Experienced players show higher numbers than the Inexperienced ones
    - Data type: Numerical
    - Measurement type: Integer
- Kills
    - Interpretation: Not unique and counts the quests registries per player, and at first sight make sense that the Experienced players show higher numbers than the Inexperienced ones
    - Data type: Numerical
    - Measurement type: Integer

For the next test plots, we will plot a simulated graph showing how fitted is the actual distribution to an ideal normal curve from the same data, that’s why we are going to create two functions, one for the new distribution calculation from the median and another to plot it over the original data.

This will be our general approach because this plot will also let us see more clearly the existence of skewing elements.

```python
import math

# Generate simulated-std based on the median
def median_std(data):
     # Number of observations
     n = len(data)
     # Median of the data
     median = data.median()
     # Square deviations
     deviations = [(x - median) ** 2 for x in data]
     # Variance
     variance = sum(deviations) / n
     return math.sqrt(variance)

# Function to generate a plot of a vector to compare it vs a fitted normal distribution
def normalplot(data, var_name):
    # Generate subset of dataframe
    variable = data[var_name]

    # Max and min from variable
    max_var = max(variable)
    min_var = min(variable)

    # Create a random array of normally distributed number with the same parameter as the original data
    y_fit = np.random.normal(loc= variable.median(), scale= median_std(variable), size= len(variable))

    # Max and min from simulated curve
    max_fit = max(y_fit)
    min_fit = min(y_fit)

    # Group data together
    hist_data = [variable, y_fit]
    group_labels = [var_name, 'Median-Simulated <br> Normal Curve']
    colors = ["#007FFF","#FF5A5F"]

    # Create distplot with custom bin_size
    plot = ff.create_distplot(hist_data, group_labels, bin_size=[0.75,0.75],
                                show_rug=False, show_hist=True, colors= colors)

    # Update layout
    plot.update_layout(layout)
    plot.update_layout(title = {'text':'Density Plot of '+var_name+' vs a Normal Distribution'},
                        xaxis = {"title":var_name},
                        yaxis = {"title":"Density"})
    plot.update_xaxes(range=[min_var, max(max_var,max_fit)])
    plot.add_annotation(sign)
    return plot
```

### ⚔️ Two Sample T-Test for Quests Completed by Experience Group

A two sample t-test is used to test whether there is a significant difference between **two groups** means, where the hypothesis will be:

- $H_0: \mu_{ex} = \mu_{inex}$ (population mean of "Experienced Players Completed Quests" is equal to "Inexperienced Players Completed Quests")
- $H_1: \mu_{ex} \ne \mu_{inex}$ (population mean of "Experienced Players Completed Quests" is different from "Inexperienced Players Completed Quests")

This test makes the following assumptions:

- The two samples data groups are independent
- The data elements in respective groups follow any normal distribution
- The given two samples have similar variances (homogeneity assumption)

In this case, both groups are independent since none of them provide information about the other and vice versa.

**Normality Check**

First let's visualize the distribution for the Experienced players.

```python
expQuestnorm = normalplot(data= Experienced, var_name= 'Quest Completed')
expQuestnorm.update_layout(title = {'text':'Density Plot of Quest Completed by Experienced Players'})
expQuestnorm.show()
```

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/control_mechanics_fallout/master/images/dens1.jpg" style="width: 900px"></p>

And the distribution for Inexperienced players.

```python
inQuestnorm = normalplot(data= InExperienced, var_name= 'Quest Completed')
inQuestnorm.update_layout(title = {'text':'Density Plot of Quest Completed by Inexperienced Players'})
inQuestnorm.show()
```

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/control_mechanics_fallout/master/images/dens2.jpg" style="width: 900px"></p>

Notice that the data is normal for both cases and in terms of the quantity of "quests completed" by the experienced player's data is slightly higher than the inexperienced ones.

**Variance Homogeneity**

First let's see the variance for each group of testers.

```python
# Print the variance of both data groups
print("Quest Completed variance for Experienced players: " + str(np.var(Experienced["Quest Completed"])) + "\n",
    "Quest Completed variance for Inexperienced players: " + str(np.var(InExperienced["Quest Completed"])) + "\n")
```

```
Quest Completed variance for Experienced players: 3.25
 Quest Completed variance for Inexperienced players: 2.283446712018141
```

In the good practice the correct way to do this validation is by a Homogeneity Test, so let's make a **Barlett** test since our data is normal (for non-normal is preferred **Levene's**). So let’s start from the next hypothesis.

- $H_0: \sigma^2_{ex} = \sigma^2_{inex}$ (The variances are equal across in both groups)
- $H_1: \sigma^2_{ex} \ne \sigma^2_{inex}$ (The variances are not equal across in both groups)

```python
# Import the bartlett method
from scipy.stats import bartlett

# Bartlett's test in Python with SciPy:
bartlett(Experienced["Quest Completed"], InExperienced["Quest Completed"])
```

```
BartlettResult(statistic=1.0905002879637673, pvalue=0.29636038782360125)
```

Our P-value is above 0.05, so we have enough statistical evidence to accept the hypothesis that the variances are equal between the Experienced testers and the Inexperienced players.

Now that we checked that we have normal data and homogeneity in our variances, let's continue with our **two sample t-test**.

```python
# Import the stats module
import scipy.stats as stats

# Perform the two sample t-test with equal variances
stats.ttest_ind(a=Experienced["Quest Completed"], b=InExperienced["Quest Completed"], equal_var=True)
```

```
Ttest_indResult(statistic=3.8261587063084392, pvalue=0.0002855175095244425)
```

We have enough statistical evidence to reject the null hypothesis, the population mean of Quests Completed by "Experienced Players" is significantly different from the "Inexperienced Players".

As a supposition, we can say that the difficulty selected by the player can be a factor affecting the Completion of the Quest. Considering that the difficulty levels in Fallout New Vegas can vary by two variables which are "Damage taken from Enemy" and "Damage dealt with Enemy", without mentioning the Hardcore mode which was excluded from the samples.

The difficulty is an important factor to consider since an experienced player can know what is the level of preference, while for an inexperienced player is a "try and fail situation", so for a future study it’s important to reconsider how this variable would affect the results.

### 💀 Two Sample Mann-Whitney Test for Kills by Experience Group

A two sample [Mann-Whitney U](https://www.statstutor.ac.uk/resources/uploaded/mannwhitney.pdf) test is used to test whether there is a significant difference between **two groups** distributions by comparing medians, where the hypothesis will be:

- $H_0: P(x_{ex} > x_{inex}) = 0.5$ (population median of "Experienced Players Kills" is equal to "Inexperienced Players Kills", same distribution)
- $H_1: P(x_{ex} > x_{inex}) \ne 0.5$ (population median of "Experienced Players Kills" is different from "Inexperienced Players Kills", different distribution)

In other words, the null hypothesis is that, “the probability that a **randomly drawn member** of the first population will exceed a member of the second population, is 50%”.

This test makes the following assumptions:

- The two samples data groups are independent
- The data elements in respective groups are continuous and not-normal

It's the equivalent of a two sample t-test without the normality assumption. Also, both groups are independent since none of them provide information about the other and vice-versa.

**Normality Check**

First let's visualize the distribution for the Experienced players.

```python
expKillsnorm = normalplot(data= Experienced, var_name= 'Kills')
expKillsnorm.update_layout(title = {'text':'Density Plot of Kills by Experienced Players'})
expKillsnorm.show()
```
<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/control_mechanics_fallout/master/images/dens3.jpg" style="width: 900px"></p>

And for Inexperienced players.

```python
inKillsnorm = normalplot(data= InExperienced, var_name= 'Kills')
inKillsnorm.update_layout(title = {'text':'Density Plot of Kills by Inexperienced Players'})
inKillsnorm.show()
```
<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/control_mechanics_fallout/master/images/dens4.jpg" style="width: 900px"></p>

Both groups show a clear violation of the normality principle, where the data is highly skewed to the right.

For the Inexperienced players, despite considering excluding the players with 35 or more kills registered, it wouldn't be significant, it will still remain skewed to the right.

In cases where our data don't have a normal distribution, we need to consider **non-parametric** alternatives to make assumptions. That is the main reason why we are going to use a **Mann Whitney U Test**.

```python
from scipy.stats import mannwhitneyu

def mannwhitneyu_est(data1, data2):
    statistic, p = mannwhitneyu(data1, data2, alternative = 'two-sided')
    return p
    
print(mannwhitneyu_est(Experienced.Kills, InExperienced.Kills))
```

```
0.0020247429887724775
```

Finally we reject the null hypothesis, since our p-value is less than 0.05 with a 95% of confidence. This means that population median of "Experienced Player Kills" is different from "Inexperienced Player Kills", the same we can conclude from the distribution.

### 🔃 Two-sample bootstrap hypothesis test for Kills metric

After some meetings with the Level Designers, they aren't convinced to take action upon the recommendations given from a sample size of 28 and 42 for the Experienced and Inexperienced groups respectively. So they require us to take another type of validation to ensure the confidence of our conclusion.

Unfortunately, the Programmers require to get that insight as soon as possible to start fixing the patch. So, the only alternative to satisfy both parts is to perform a bootstrapping with the means and the p-value of the past test.

Bootstrapping is one of the best choices to perform better results in sample statistics when we count with few logs. This Statistical Inferential Procedure works by resampling the data. We will take the same values sampled by group, from a resampling with replacement, to an extensive dataset to compare the means between the Kills metric of Experienced players and Inexperienced players.

On the other side, we **can't** use a permutation of the samples, since we can't conclude that both groups' distributions are the same. Before this let's create our bootstrapping replicates generator function.

```python
# Bootstrapping Means Function
def draw_bs_reps(data,func,iterations):
    boot_arr = []
    for i in range(iterations):
        boot_arr.append( func(data = np.random.choice(data, len(data))) )
    return boot_arr

# Bootstrapping Mann-Whitney Test
def draw_bs_test(data1, data2, iterations):
    boot_arr = []
    for i in range(iterations):
        estimate = mannwhitneyu_est(np.random.choice(data1, len(data1)),np.random.choice(data2, len(data2)))
        boot_arr.append(estimate)
    return boot_arr
```

And with the next function we will create our new estimates for the empirical means and the values of the Mann-Whitney U Test.

This function will return two p-values and two arrays of bootstrapped estimates, which we will describe next:

- **p_emp:** P-value measured from the estimated difference in means between both groups, showing the probabilities of randomly getting a "Kills" value from the Experienced that will be greater than one of the Inexperienced group
- **bs_emp_replicates:** Array of bootstrapped difference in means
- **p_test:** P-value measured from the estimated probability of getting a statically significant p-value from the Mann-Whitney Test (p-value < 0.05), in other words how many tests present different distribution for both groups
- **bs_test_replicates:** Array of bootstrapped p-values of Mann-Whitney Test

```python
from statistics import mean

# Bootrapping probability estimates (p-value)
def twosampleboot(data1, data2, operation, iterations):
    # Compute n bootstrap replicates from the groups arrays
    bs_replicates_1 = draw_bs_reps(data1, operation, iterations= iterations)
    bs_replicates_2 = draw_bs_reps(data2, operation, iterations= iterations)

    # Get replicates of empirical difference of means
    bs_emp = pd.Series(bs_replicates_1) - pd.Series(bs_replicates_2)

    # Compute empircal difference in means for p-value, where of Experienced-Kills > InExperienced-Kills
    p_emp = np.sum(bs_emp >= 0) / len(bs_emp)

    # Get replicates of Mann-Whitney U Test
    bs_test = draw_bs_test(data1= Experienced.Kills, data2= InExperienced.Kills, iterations= iterations)

    # Compute and print p-value of estimate with different distributions/medians
    p_test = np.sum([1 if i <= 0.05 else 0 for i in bs_test]) / len(bs_test)

    return p_emp, bs_emp, p_test, bs_test
    
# Return p-values and bs_replicates
p_emp, bs_emp_replicates, p_test, bs_test_replicates = twosampleboot(Experienced.Kills, InExperienced.Kills, operation= mean, iterations= 10000)

print('Diff. Means p-value =', p_emp, '\n',
       'Mann-Whitney Test p-value =',p_test)
```

```
Diff. Means p-value = 0.9852 
 Mann-Whitney Test p-value = 0.8991
```

After making a sample of 10,000 replicates, we can conclude that 98% of the time, the disparity in the Kills average will be higher in Experienced Players than in Inexperienced players, and from that replicates 90% of them will present a significant difference between distributions.

Now let's plot the replicates of the Average Kills difference between groups.

```python
bar_annotation = str(round((1-p_emp)*100,2)) + "% " + "of replicates <br>present values below 0"

histboot = px.histogram(bs_emp_replicates)
histboot.add_vrect(x0=min(bs_emp_replicates), x1=0, 
              annotation_text= bar_annotation, annotation_position="left",
              fillcolor="red", opacity=0.25, line_width=0)
histboot.update_layout(layout)
histboot.update_layout(title = {'text':'Bootstrapped Average Kills difference of <br>Experienced vs Inexperienced Players'},
                        yaxis = {'title':"Count of replicates"}, xaxis = {'title':"Difference Estimate"}, showlegend=False)
histboot.add_annotation(sign)
histboot.show()
```
<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/control_mechanics_fallout/master/images/hist1.jpg" style="width: 900px"></p>

Just 1,48% of the Inexperienced players present more Kills registered than the Experienced ones, by taking a conclusion from 10,000 replicates of a resampled dataset.

In the case of the bootstrapped P-value for the Mann-Whitney U Test estimate, the best way to avoid the binning bias from a huge number of decimal values is to represent the probabilities with an Empirical Cumulative Distribution Function, as we are going to see next.

```python
import plotly.graph_objects as go

# Import numpy library
import numpy as np

# ECDF Generator function
def ecdf(data):
    # Generate ECDF (Empirical Cumulative Distribution Function)
    # for one dimension arrays
    n = len(data)

    # X axis data
    x = np.sort(data)

     # Y axis data
    y = np.arange(1, n+1) / n

    return x, y

# ECDF plot generator
def ecdf_plot(data, var_label):
    # Generate ECDF data
    x, y = ecdf(data)

    # Generate percentile makers 
    percentiles = np.array([5,25,50,75, p_test*100, 95]) #Plot the percentile where the bootstrapped p-value is located
    ptiles = np.percentile(data, percentiles)

    # ECDF plot
    plot = go.Figure()

    # Add traces
    plot.add_trace(go.Scatter(x=x, y=y,
                        mode='markers',
                        name=var_label))
    plot.add_trace(go.Scatter(x=ptiles, y=percentiles/100,
                        mode='markers+text',
                        name='Percentiles', marker_line_width=2, marker_size=10,
                        text=percentiles, textposition="bottom right"))
    plot.update_layout(layout)
    plot.add_annotation(sign)

    return plot

ecdf = ecdf_plot(bs_test_replicates, "P-value replicates")
ecdf.update_layout(title = {'text':'ECDF of Bootstrapped P-values for Mann-Whitney U Test'},
                        yaxis = {'title':"Cumulative Probability"})
ecdf.show()
```

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/control_mechanics_fallout/master/images/ecdf1.jpg" style="width: 900px"></p>

From this function with 89.91% of the 10,000 replicates of the tests, we can conclude that the distribution or the median between experienced and inexperienced players is different because its p-value is still below the 0.05 accepted.

## 🗒️ Final thoughts & takeaways

**What can the stakeholders understand and take in consideration?**

There is a significant advantage of veteran players over the new ones, giving us the conclusion that the changes in mechanics made from the last Fallout New Vegas patch still very similar and don’t present meaningful changes, reason why veteran players are finding it easy and have a clear advantage like it’s usual, so for now the programmers should not be concerned about fixing the patch.

**What could the stakeholders do to take action?**

The Level Designers can consider working side by side with the UX team, to make recurrent revisions before launching a new patch, because always the community will expect an experience-centric work of the whole development team in terms of the quality delivered.

Also for a future study we could consider to gather the difficulty category and merge it into the dataset and see if this variable is producing a significant variability in our final output.

**What can stakeholders keep working on?**

For significant changes in game mechanics, is recommended to do it just under a critical situation, like when is a significant bug affecting the player experience, otherwise it’s preferable to leave it until the launch of the next title reveal and test the audience reaction before the final implementation, if it’s possible.

---

## ℹ️ Additional Information

- **About the article**

This article was developed from the content explained in the Inferential statistics section of Chapter 3 of the Game Data Science book. All the conclusions made were inspired by a player-profiling from in-game metrics by using deductive reasoning where we assumed, and then we proved it using Significance Confidence and Variance, using Inferential analysis.

All the assumptions and the whole case scenario were developed by the author of this article, for any suggestion I want to invite you to go to my about section and contact me. Thanks to you for reading as well.

- **Related Content**

— Game Data Science book and additional info at the [Oxford University Press](https://global.oup.com/academic/product/game-data-science-9780192897879?cc=cr&lang=en&)

— Anders Drachen personal [website](https://andersdrachen.com/)

- **Datasets**

This project was developed with a dataset provided by Anders Drachen et. al (2021), respecting the author rights of this book the entire raw data won’t be published, however, you can access the transformed data in my [Github](https://github.com/robguilarr/parse_fallout_data/tree/master/parsed_data) repository.
