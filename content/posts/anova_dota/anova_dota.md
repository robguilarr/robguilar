---
#weight: 2
title: "Interpreting changelogs with ANOVA tests on DOTA 2"
date: 2022-07-24T21:57:40+08:00
lastmod: 2022-07-24T21:57:40+08:00
draft: false

description: "Inferential analysis with Kruskal–Wallis ANOVA to justify the Balance of Power 6.86 patch logs"
images: ["/images/posts/anova_dota.jpg"]
thumbnail: "/images/posts/anova_dota.jpg"
tags: ["Kruskal–Wallis ANOVA", "Mann-Whitney t-test","Kolmogorov-Smirnov", "Bootstrapping", "Python"]
categories: ["Projects"]

author: "Roberto Aguilar"
authorLink: "https://www.linkedin.com/in/robguilarr/"
hiddenFromHomePage: false
hiddenFromSearch: false

lightgallery: true
toc: true
math: true
featured-image: "/images/posts/anova_dota.jpg"
featuredImagePreview: "/images/posts/anova_dota.jpg"
seo:
  images: ["/images/posts/anova_dota.jpg"]
  image: "/images/posts/anova_dota.jpg"
  thumbnailUrl: "/images/posts/anova_dota.jpg"
---

In the world of Multiplayer Online Battle Arena games, also known as MOBA, a [changelog](https://www.dota2.com/patches/7.31d) is a list of updates that developers make to their games. These modifications are usually minor, but they can be meaningful if they affect the gameplay.

Among the major big titles in the MOBA genre is DOTA 2, which according to Minotti (2015), "*this game together with League of Legends (LOL), is considered the state-of-the-art in the genre of MOBA, considering that this game emerged from a mod of the widely known MMORPG, World of Warcraft, which then transformed into a major team-based strategy game*" according to Drachen et. al (2021).

This mod started its development in 2005, by some solo-developers and [Icefrog](https://dota2.fandom.com/wiki/IceFrog). The core gameplay design consists of two teams of five players each; each player controls a [hero](https://www.dota2.com/heroes). The two teams fight each other on a common battlefield using heroes from different [classes](https://stratz.com/heroes) that have distinctive abilities and roles within the team structure.

The main goal of this game is to use a [5v5](https://en.wikipedia.org/wiki/Multiplayer_online_battle_arena) strategy to defend your Ancient and attack the one from the enemy’s faction. The Ancients are massive structures found inside each faction's base and are the main objective.

In order to win, the enemy team's Ancient must be destroyed, while the own one must be kept alive. Ancients are guarded by their two tier 4 towers and are invulnerable until both towers are destroyed. The map is distributed per lane (refer to the next map designed); Offlane, Safelane, and Midlane, and among the lanes each faction has a total of 11 [Towers](https://dota2.fandom.com/wiki/Buildings#Towers) to attack enemies; and also each lane has two [Barracks](https://dota2.fandom.com/wiki/Buildings#Barracks), one for producing melee creeps and the other for ranged creeps.

{{< admonition info "Looking for an interactive experience?" true >}}

:rocket: Download the Jupyter Notebook, available <a href="https://github.com/robguilarr/anova_dota/blob/master/notebook.ipynb">here</a>

{{< /admonition >}}

{{< youtube -cSFPIwMEq4 >}}

## ⚠️ Introduction to problem

### Hypothesis

For online multiplayers, sustainability mirrors the voice of customers. In the case of DOTA, the platform's exclusivity due to the ownership of its developer, [Valve Corporation](https://www.valvesoftware.com/en/about), make it more reachable to gather community reviews on time from the Steam store. This, in comparison to other games' developers, who might require to access an endpoint to connect to [Steamworks API](https://partner.steamgames.com/doc/store/getreviews) to perform the same task.

The Consumer Insights team at Valve gathered all the internal reviews and external’s from social networks of high traffic in this niche, like Reddit. After an exhaustive NLP analysis, the team has found that players tend to comment about [Nerfing and Buffing](https://en.wikipedia.org/wiki/Game_balance#Buff_and_nerf) heroes, which is very common in MOBA games when every patch is released. And, since patch 6.86 was released on December 16 of 2015, also known as [Balance of Power](https://www.dota2.com/balanceofpower), the team has found many comments about this topic, so they require to generate actionable insights to let the Game Designers take action on time.

After carefully collecting player logs since the released date of this patch, they need to test if effectively the heroes’ stats are unbalanced or not. As the main interaction between enemy players is measured by the damage dealt, this will be defined as the core KPI of this study and with this, we will define our hypothesis as:

- After the release date of patch 6.86, the heroes **have recorded** a statistically equal damage.

- After the release date of patch 6.86, the heroes **haven’t recorded** a statistically equal damage.

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/anova_dota/master/images/map.png" style="width: 1025px"></p>


### Potential Stakeholders

- Level Designers: if the heroes are balanced in terms of damage, the damage dealt by the creeps will require an investigation, which depends on the design of the map lanes. They must hit a balance between creating easy or hard experiences, and this will rely on the creeps per lane. Another mapping issue that would need to address is the jungle, which is typically filled with [creeps](https://dota2.fandom.com/wiki/Creeps) that players can kill for [rewards](https://dota2.fandom.com/wiki/Jungling#), which the designers require to define how challenging they should be to defeat.

- Gameplay Designer and Engineer: they evaluate the design of the heroes to be certain that each team is balanced, and confirm there is no side having an unfair benefit. In such cases will have to rebalance the hero stats with the help of the Gameplay Engineer.

- User Retention team: by working around the Data Analysts, they will support delivering the final report to the Lead Game Designer, about the potential effect on the players' retention in case of not taking quick actions, in the scenario of having a true Null Hypothesis.

- PR team and Community Manager: they must be aligned to deliver an accurate [Patch note](https://www.dota2.com/news/updates) with an accurate summary of the next patch to be released.

- Players community: as the main stakeholder, the core sustainability of this IP comes from the community engagement, so they expect to have a more balanced experience at least for patch 6.86b.

**Note:** To facilitate the understanding of the roles of the development team, I invite you to take a look at **[this](https://www.robguilar.com/posts/gamedev_structure/)** [](https://miro.com/app/board/uXjVOwQVsRk=/?share_link_id=215280076044)diagram that I designed.

## 📥 About the data

This dataset was collected on December 17th of 2015 from a sample of [Ranked Matchmaking](https://dota2.fandom.com/wiki/Matchmaking#Ranked_Match) mode, which holds information from 49,855 different contests, where 149,557 different players formed part of them.

As we mentioned in the [preprocessing notebook](https://github.com/robguilarr/anova_dota/blob/master/preprocess.ipynb), the raw extracted data can be found in the [YASP.CO](https://academictorrents.com/details/5c5deeb6cfe1c944044367d2e7465fd8bd2f4acf) collection from Academic Torrents. This dataset was uploaded and gathered by [Albert Cui](https://academictorrents.com/browse.php?search=Albert+Cui) et. al (2015). However, you can extract your sample from the [OpenDota API](https://www.opendota.com/explorer) query interface for the comfort of use.

### Collection process and structure

According to Albert Cui et. al (2014), OpenDota is a volunteer-developed, open-source service to request DOTA 2 semistructured data gathered from the Steam WebAPI, which has the advantage of retrieving a vast amount of replay files in comparison to the source API. The service also provides a web interface for casual users to browse through the collected data, as well as an API to allow developers to build their applications with it.

The present sample can be found in an archive of [YASP.CO](https://academictorrents.com/details/5c5deeb6cfe1c944044367d2e7465fd8bd2f4acf), which was uploaded for the same developers of the OpenDota project, [Albert Cui](https://github.com/albertcui), [Howard Chung](https://github.com/howardchung), and [Nicholas Hanson-Holtry](https://github.com/nicholashh). Due to the size of the data (99 GB), you won't find the raw data in the [Github repository](https://github.com/robguilarr/anova_dota/tree/master/DOTA_data) of this project, however, you will find a random sample with its respective preprocessing explanation in a Jupyter Notebook.

As a suggestion, to preprocess data of this size, the most accurate solution is to use parallel computing frameworks like Apache Spark to do the transformations. Web services like Databricks with its Lakehouse Architecture, let us process semi-structured data with the capabilities of a Data Warehouse and the flexibility of a Data Lake, and to do this you can use the Data Science and Engineering environment of the [Community Edition](https://community.cloud.databricks.com/login.html) for free.

Without diving deeper, let's import the initially needed packages and the preprocessed data.

```python
# Basic packages
import pandas as pd
import numpy as np
from math import sqrt, ceil, floor

# Plotly packages
import plotly.express as px
import plotly.graph_objects as go
from plotly.subplots import make_subplots
# Plotly static config
static = {'staticPlot': True}

# Own package for layout of plots
from vizformatter.standards import layout_plotly

# Load layout base objects
sign, layout = layout_plotly(height= 720, width= 1000, font_size= 15)

# Import preprocessed data
data = pd.read_csv("DOTA_data/data.csv", index_col = 0, na_values=["N/A","", " ", "nan"])
data.head()
```

<p align="left"><img src="https://raw.githubusercontent.com/robguilarr/anova_dota/master/images/table_1.jpg" style="width: 1050px"></p>

As we can notice, in our data we have a total of 28 attributes, however, not all of them will be used in this study but for future investigations of extra user metrics, we see that all the assembled data could be a useful asset. In such case you require the description of one of these attributes, please guide yourself with the **Appendix** at the end of the [preprocess notebook](https://github.com/robguilarr/anova_dota/blob/master/preprocess.ipynb), where you will find a punctual explanation.

## 🔧 Data Preprocessing

Almost every transformation was made on the preprocessing notebook, there you will encounter different blocs like the joins of hero descriptions, items used per session, and match outcomes to the main table of players' registries. In the end, also will be a distribution study, to exclude outliers based on irregularities recorded on sessions (impossible values). Knowing this, we can say that the data is ready to go, so we can skip the Data Cleaning section. Here we are going to do an overall inspection.

### Data Consistency

Considering conceivable errors in the telemetry logs, we require to validate the lack of missing values on each one of the features.

One of the attributes that required a substantial transformation to infer and extract descriptive values for each player was the “player_slot”, which is no more in the present dataset. This value was an 8-bit unsigned integer, where the first digit defined the faction (Radiant | Dire), the next four digits were empty and the last three defined the lane role of the player within the team. The encoding is part of the reason why there were present some missing data that wasn’t describing the record.

After the pruning was executed, the sample remain significant in size with 290,000 records, and by the next table, we can verify it.

```python
def nullCounter(data):
    # Create empty lists
    columns = []
    nulls = []
    perc = []
    total_rows = data.shape[0]

    for i in range(len(data.columns)):
        # Null values
        null_counter = data.loc[:,data.columns[i]].isnull().sum()

        # Append only columns with null values
        if null_counter > 0:
            # Append values to lists
            columns.append(data.columns[i])
            nulls.append(null_counter)

            # Generate percentage
            perc_value = null_counter/total_rows
            perc.append("{:.2%}".format(perc_value))

    # Create DF from lists
    nullsTable = pd.DataFrame({'Null Values':nulls, 'Fraction':perc}).T
    nullsTable.columns = columns

    return nullsTable

nullCounter(data= data)
```

<p align="left"><img src="https://raw.githubusercontent.com/robguilarr/anova_dota/master/images/table_2.jpg" style="width: 700px"></p>

The columns with more NA values are the ones relative to the player position. In case of any comparison needed in terms of lane position, we will take another sample ensuring that we have a significant size to run our tests.

### Data Distribution Validation

Every column is already transformed into its respective data type, and depending on the analysis needed we should require to retransform it or even apply a linear scaling method to be interpreted.

```python
data.describe(include= ["number"])
```

<p align="left"><img src="https://raw.githubusercontent.com/robguilarr/anova_dota/master/images/table_3.jpg" style="width: 1050px"></p>

We can see each value defined by the percentiles, however, is preferable to have a straightforward vision to conclude about each feature.

```python
# Load a new layout for grid axis
signGrid, layoutGrid = layout_plotly(height= 900, width= 1200, font_size= 12)

# Function to generate grid of boxplots
def gridDist(data, layout, sign):
    # Validate not using ids in the grid
    for i in data.columns:
        if (i.endswith('_id')) or (i == 'id'):
            data[i] = data[i].astype(str)

    # Extract columns with numeric values
    numerics = ['int16', 'int32', 'int64', 'float16', 'float32', 'float64']
    df = data.select_dtypes(include=numerics)

    # Calculate grid space, for columns and rows in the layout
    num_col = len(df.columns)
    cols = ceil(sqrt(num_col))
    rows = cols

    # Generate subplots grid
    grid = make_subplots(rows=rows, cols=cols)

    # Populating spaces with Graphic objects
    counter = 0
    for i in range(1,rows+1):
            for j in range(1,cols+1):
                column_name = df.columns[counter]
                trace = go.Box(x= df[column_name], name= column_name)
                grid.append_trace(trace, i, j)
                if counter == num_col-1:
                    break
                else:
                    counter +=1

    # Adding layout conf
    grid.update_layout(layout)
    grid.update_layout(showlegend=False)
    grid.add_annotation(sign)

    # Disable interactivity by transforming image to bytes
    static = {'staticPlot': True}

    return grid.show(config = static)

gridDist(data= data, layout= layoutGrid, sign= signGrid)
```

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/anova_dota/master/images/plot_1.png" style="width: 1050px"></p>

From the dispersal of each attribute, we can notice that most of the data is right-skewed. We don't have any information bonded to system anomalies during data collection, so from the variables that we are going to use in our analysis, we can notice descriptive behaviors like:

- Per session players usually report, on average, a total of 11 assists and 6 kills with a median hero damage score of 10,500.

- There is a big segment of outliers from players who healed allies, with a value of over 233 points. Making general assumptions over its distribution wouldn't be accurate, because that segment could be linked with the Hard Support role.

- The same situation happens with the gold collected and the tower damage, where there is a great number of scores above the 75% percentile, which can be due to lane role differences like Soft Support which is usually a role used for [Jungling](https://dota2.fandom.com/wiki/Jungling#).

In our exploratory analysis, we will have to study these metrics by lane, because each lane has different roles, and some aren't directly comparable. Nevertheless, that's also something that we'll need to demonstrate with a hypothesis test between lanes.

## 🔍 Exploratory Analysis & In-game interpretations

Before going through the hypothesis test, let’s explore the behavior within the categorical variables to check the consistency within the descriptive attributes for the players on each lane.

```python
data.describe(include= ["object"])
```

<p align="left"><img src="https://raw.githubusercontent.com/robguilarr/anova_dota/master/images/table_4.jpg" style="width: 900px"></p>

At first sight, the most common role in our data corresponds to the Offlaner and the most used hero is Windranger, and this makes sense since is one of the heroes with a higher Win Rate on the Offlaner role according to [DOTABUFF](https://www.dotabuff.com/heroes/windranger). Also, the two most used shop items are the [Town Portal Scroll](https://dota2.fandom.com/wiki/Town_Portal_Scroll) and the [Blink Dagger](https://dota2.fandom.com/wiki/Blink_Dagger).

But since we have numerous combinations between heroes and items, this might not be the most accurate way to make inferences. So let's look at the combinations for the Radiant and Dire factions.

```python
# Get combination of items and heroes
def combItemHero(df, side):
    # Subset Faction/Side
    data= df[df.side == side]
    # Subset categorical variables
    itemHero = data.select_dtypes(include="object")
    # Exclude unused columns
    itemHero = itemHero.loc[:,~itemHero.columns.isin(['lane','role','side'])]
    # Pivot Table
    itemHero = itemHero.set_index("hero_name").stack()\
        .reset_index()
    # Add counter
    itemHero['counter'] = np.ones(itemHero.shape[0], dtype=int)
    # Drop items column
    itemHero = itemHero.drop(labels= ["level_1"], axis= 1)
    # Rename columns
    itemHero.columns = ['hero_name','item', 'value']
    # Get combinations DF
    itemHero = itemHero.groupby(['hero_name','item'])['value'].sum().reset_index()\
        .sort_values('value', ascending= False, ignore_index= True)
    # Create combination column
    itemHero['stage'] = itemHero.hero_name + ' with ' + itemHero.item
    # Reorder columns
    itemHero = itemHero.loc[:,sorted(itemHero.columns)].drop(columns = ['hero_name','item'])
    # Add side column
    itemHero['side'] = side

    return itemHero

# Drop empty hero names
data = data[data.hero_name.isnull() == False]

# Set new dataframes
itemHeroRad= combItemHero(df= data, side= 'Radiant')
itemHeroDire= combItemHero(df= data, side= 'Dire')
# Concatenate dataframes
allItemHero = pd.concat([itemHeroRad, itemHeroDire], axis=0).sort_values('value', ascending= False, ignore_index= True)
# Generate Visuals
allItemHero = px.funnel(allItemHero.head(17), x='value', y='stage',
                        color='side', color_discrete_sequence= ['#E25822','#2C538F'])
allItemHero.update_layout(layout)
allItemHero.update_layout(title = {'text':'Combinations recorded of Hero and Items from 50K sessions'},
                            yaxis_title="Combinations")
allItemHero.add_annotation(sign)
allItemHero.show(config = static)
```

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/anova_dota/master/images/plot_2.png" style="width: 1000px"></p>

Now we can verify the initial statement that the Windranger is the most played hero, but at the same time, we can check that the Town Portal Scroll and the Blink Dagger are not the most used for this hero, meaning that their use is because of commonness and not because it's a good fit for that hero.

- **Windranger** it's a ranged hero that can buff her stats by 3.75% in [movement speed](https://dota2.fandom.com/wiki/Movement_speed). According to [Tan Guan](https://dotesports.com/news/how-to-play-windranger-like-liquid-w33), these boots supply the most damage out of any boots choice, and the six armor allows an Intelligence hero to have a mana-less option to chase the enemy down. Also, the Phase active brings you up to max movement speed with Windrun (another physical ability for movement speed).

- The same case is for **Shadow Fiend** using Power Treads, where some advanced strategies are possible for players who like to micromanage or wish to maximize their efficiency in a competitive setting. Like switching the Power Treads to Intelligence before a fight to allow this hero to cast one more spell.

While examining patches, it's always important to examine the performance of the character by sampling the player statistics. In such case of DOTA, it would let the designers decide which heroes need to be nerfed and which need to be buffed, by delivering changes in a patch log attached to the bottom side of the [documentation website](https://www.dota2.com/balanceofpower). To put you in context, **nerfed** means a negative effect on the hero's stats, while **buffed** means the opposite, given an update (patch) on the game.

### 🌍 Overview of Metrics by Lane Roles

We saw that generalizing about all the heroes and extracting insights, it’s not the best way to produce findings, because many times the perspective can bias our interpretations, especially while dealing with Big Data.

As we move on to the next analysis, the main intent is to narrow down our scope of analysis (data sample), so to start let’s compare the core metrics by role lane.

First, let’s see the scores for how much a player heals their allies' heroes, how much the player damages the enemys' heroes, and how many times each player dies on average, taking as a group the role assigned in the lane.

```python
# Function to scale variables using "Linear Scaling"
def linearScale(array):
    scaled = (array - array.min()) / (array.max() - array.min())
    scaled = round(scaled, 5)
    return scaled

# Subset required columns
dataHeal = data.loc[:,data.columns.isin(['role','deaths','hero_healing','hero_damage'])]

# Reorder columns
dataHeal = dataHeal.loc[:,sorted(dataHeal.columns, reverse= True)]

# Extract only the subset of players that healed during session
dataHeal = dataHeal.groupby('role').mean().reset_index()

# Load a new layout for parallel axis
signPar, layoutPar = layout_plotly(height= 700, width= 1500, font_size= 14)

# Generate Parallel Coordinates plot
parallel = go.Figure(data=
    go.Parcoords(
        line= dict(color= [1,2,3,4,5],
               colorscale= [[0,'#3BE8B0'],[0.25,'#1AAFD0'],[0.5,'#6A67CE'],[0.75,'#FFB900'],[1,'#FC636B']]),
        dimensions = list([
            dict(range = [1,len(dataHeal.role)],
                 tickvals = [1,2,3,4,5],
                 label = 'Role', values = [1,2,3,4,5],
                 ticktext = dataHeal.role),
            dict(range = [floor(min(dataHeal.hero_healing)), ceil(max(dataHeal.hero_healing))],
                 #tickvals = [0,0.25,0.5,0.75,1],
                 label = 'Healing', values = dataHeal.hero_healing),
            dict(range = [floor(min(dataHeal.hero_damage)), ceil(max(dataHeal.hero_damage))],
                 #tickvals = [0,0.25,0.5,0.75,1],
                 label = 'Damage', values = dataHeal.hero_damage),
            dict(range = [floor(min(linearScale(dataHeal.deaths))), ceil(max(linearScale(dataHeal.deaths)))],
                 #tickvals = [0,0.25,0.5,0.75,1],
                 label = 'Scaled Deaths', values = linearScale(dataHeal.deaths))
        ])
    )
)

# Assign layout for parallel axis
parallel.update_layout(layoutPar)
parallel.update_layout(margin={"t": 125,
                        "l": 100})
parallel.update_layout(title = {'text':"User Metrics comparison by Role Lane"})
parallel.add_annotation(signPar)
parallel.show(config = static)
```

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/anova_dota/master/images/plot_3.png" style="width: 1500px"></p>

From the last visual we can drag the next insights:

- **Soft support** is the role where they dedicate less effort to healing their teammates (due to their freedom is the role used for Jungling), while **Midlaners** dedicate more time to healing teammates. This is probably due to their high mobility during the session, and because they have the major responsibility to be the team's playmakers, according to Zach the DOTA [coach](https://www.patreon.com/ZQuixotix).

- In terms of Damage dealt, we can say that by far Soft Support and Midlaners are the positions that dealt more damage, while **Hard Carry** is the opposite because they require a clear vision of the momentum between staying in the Safelane with the Hard Support or moving through the Offlane to attack enemy buildings.

- As being a **Hard Support** is the less complex role to start, heal more, and damage less. According to Zach the DOTA [coach](https://www.patreon.com/ZQuixotix), they need to farm a little and be actively healing their teammates. At the same time, they need to know where to be positioned, cause if one core is farming a lot, you will need to be actively doing the same; if one core is always in action they need to be healing.

We saw that Soft Support and the Midlaners have a very similar average of Damage Dealt per session, so we can analyze by checking the Kills logs by hero.

### ☠️ Kills logs comparison grouped by Role and Hero

The core interaction between enemy players is measured by Kills. We'll compare the logs by groups; one being grouped by Role and another by Hero, keeping in mind that the main intent is to discover the more buffed heroes since the release of the last patch.

```python
# Function to aggregate variables by group
def groupGen(data, grouper, variable, top):
    # Aggregate new dataframe
    dataCoord = data[[grouper,variable]]
    dataCoord = dataCoord[dataCoord[grouper].isnull() == False]
    dataCoord = dataCoord.groupby(grouper).mean().reset_index()
    # Normalize the data with Linear Scaling
    dataCoord[variable] = linearScale(dataCoord[variable])
    # Resort DF randomly
    dataCoord = dataCoord.sort_values(by= variable, ascending=False).head(top)
    dataCoord = dataCoord.sample(frac=1).reset_index(drop=True)

    return dataCoord

# Generate aggregated data for heros and roles
heroGroup = groupGen(data= data, grouper= 'hero_name', variable= 'kills', top= 30)
roleGroup = groupGen(data= data, grouper= 'role', variable= 'kills', top= 5)

# Load a new layout for polar axis
signPolar, layoutPolar = layout_plotly(height= 700, width= 1750, font_size= 13)

# Create subplot layout, with 2 subplots of polar type
polars = make_subplots(rows=1, cols=2, specs=[[{'type': 'polar'}]*2])

# Add subplot graphic objects
polars.add_trace(go.Scatterpolar(name = "Kills by role", r = roleGroup.kills, theta = roleGroup.role, line_color = "#007fff", mode= 'lines'), 
                            1, 1).update_traces(fill='toself')
polars.add_trace(go.Barpolar(name = "Kills by hero", r = heroGroup.kills, theta = heroGroup.hero_name),
                           1, 2)

# Add layout format
polars.update_layout(template="plotly_dark")
polars.update_layout(layoutPolar)
polars.update_layout(title = {'text':"Linearly scaled kills log by role and hero"})
polars.add_annotation(signPolar)
polars.show(config= static)
```

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/anova_dota/master/images/plot_4.png" style="width: 1500px"></p>

In terms of kills, we can justify these stats by looking at the updates given in the changelogs of patch 6.86:

- [Riki](https://dota2.fandom.com/wiki/Riki/Changelogs): since this hero was introduced with the new 'Cloak and Dagger' ability, he can become invisible each time attacks an enemy from behind, which is a perfect combination with ‘Blink Strike’ which was transformed into basic agility. This second gives the user bonus damage for attacks from behind and also does a quick teleport to the enemy's back, giving more to use this combination.

- [Huskar](https://dota2.fandom.com/wiki/Huskar/Changelogs): This hero received a hard magic resistance and attack bonus speed mechanical change with bonus changes from 5% to 50%.
- [Ursa](https://dota2.fandom.com/wiki/Ursa/Changelogs): known as one of the heroes most used to Jungling, in this patch the 'Aghanim's Scepter' was added to this hero which also let him cast the ‘Enrage’ ability while being stunned, slept, taunted, hidden, or during forced movement. The main use of ‘Enrage’ is to reduce the income damage by ~80%.


Midlaner and Soft Support are the roles with more kills registered, which are the 2 positions in which Riki, Huskar, Ursa, and Queen of Pain are used because they are in the [Carry](https://dota2.fandom.com/wiki/Category:Carries#) heroes category.

From the last global perspective of heroes and lanes, the purpose of the analysis is to let it evolve as granular as possible to handle detailed deductions, so let's compare the Soft Support and Midlaner roles to decide on which to concentrate our study.

### 🧝🏻‍♂️ Two Sample Mann-Whitney Test for Kills logs by Role Lane for Carries

We still doubt which of the two roles is the one preferred to use Carry heroes, in other words, those that are used to rank up by logging as many kills as possible.

To do this we can run a two-sample t-test, so first let's check the normality since this is a must-have assumption.

It's important to mention that to respect the [independence](https://en.wikipedia.org/wiki/Independence_(probability_theory)) assumption, in the last code we extracted one record per player from a random **resampled without replacement** data frame. This is in case the player count with multiple sessions logged. For this test, we prefer to have a randomized sample where the groups are **mutually exclusive**, so a player can only belong to one role group.

```python
from scipy import stats

# Normality check function
def normality_two_sample(data1, data2):
    stat_1, p_1 = stats.normaltest(data1)
    stat_2, p_2 = stats.normaltest(data2)

    # Validate p_value
    if p_1 < 0.5:
        if p_2 < 0.5:
            print('At least one sample is not normal,\nwhere '
            'the p values are: {p_1:.6f} and {p_2:.6f}'.format(p_1= p_1,p_2= p_2))
        else:
            print('Only Soft_Support sample is normal,\nwhere '
            'the p values are: {p_1:.6f} and {p_2:.6f}'.format(p_1= p_1,p_2= p_2))
    else:
        if p_2 < 0.5:
            print('Only Mid sample is normal,\nwhere '
            'the p values are: {p_1:.6f} and {p_2:.6f}'.format(p_1= p_1,p_2= p_2))
        else:
            print('At least one sample is not normal,\nwhere '
            'the p values are: {p_1:.6f} and {p_2:.6f}'.format(p_1= p_1,p_2= p_2))

# List of heroes for Carry role
carries = ['Shadow_Fiend', 'Ember_Spirit', 'Alchemist', 'Meepo', 'Queen_of_Pain',
        'Windranger', 'Spectre', 'Terrorblade', 'Faceless_Void', 'Brewmaster',
        'Viper', 'Doom', 'Spirit_Breaker', 'Slardar', 'Juggernaut',
        'Phantom_Assassin', 'Slark', 'Templar_Assassin', 'Necrophos', 'Invoker',
        'Luna', 'Tidehunter', 'Gyrocopter', 'Clinkz', 'Silencer',
        'Sven', 'Lina', 'Drow_Ranger', 'Night_Stalker', 'Zeus',
       'Chaos_Knight', 'Weaver', 'Tinker', 'Huskar', 'Troll_Warlord',
       'Anti-Mage', 'Axe', 'Wraith_King', 'Kunkka', 'Lycan',
       'Bristleback', 'Outworld_Devourer', 'Riki', 'Sniper', 'Magnus',
       'Leshrac', 'Phantom_Lancer', 'Ursa', 'Bloodseeker', 'Tiny',
       'Medusa', 'Morphling', 'Storm_Spirit', 'Lone_Druid', 'Death_Prophet',
       'Razor', 'Naga_Siren']

# Subset data from players in carry roles
data_carry = data[data.hero_name.isin(carries)]

# To repect independence assumption
data_carry = data_carry.sample(frac= 1)
data_carry = data_carry.groupby('account_id').first().reset_index()

# Extract samples of Mid Laners and Soft Support
data_carry_Mid = data_carry[data_carry.role == 'Mid']         
data_carry_Soft = data_carry[data_carry.role == 'Soft_Support']

# Call function for normality test
normality_two_sample(data1= data_carry_Mid.kills,
                    data2= data_carry_Soft.kills)
```

```
At least one sample is not normal,
where the p values are: 0.000000 and 0.000000
```

It seems that both distributions are not normal, so we can't proceed with a Two Sample t-test. For this reason, we'll use a two-sample [Mann-Whitney U](https://www.statstutor.ac.uk/resources/uploaded/mannwhitney.pdf) test where our hypothesis will be defined by:

- $H_0: P(x_{kills_{mid}} > x_{kills_{soft}}) = 0.5$ (The probability that a randomly drawn player in Midlane will record more kills than a Soft Support player, is 50%)
- $H_1: P(x_{kills_{mid}} > x_{kills_{soft}}) > 0.5$ (The probability that a randomly drawn player in Midlane will record more kills a Soft Support player, is more than 50%)

This test makes the following assumptions:

- The two samples data groups are independent
- The data elements in respective groups are continuous and not-normal

Let's start by plotting the distributions.

```python
# Load layout base objects
sign, layout = layout_plotly(height= 720, width= 1000, font_size= 15)

# Generate plot for dual distributions
def dual_dist_plot(data1, data2, title, xaxis_title, group_names):

    # Subset group masks
    group1_name = group_names[0]
    group2_name = group_names[1]

    # Generate array of lists and append them
    data_grouped = [data1, data2]
    all_data = data1.append(data2).reset_index(drop= True)

    # Calculate mean
    mean_group1 = np.mean(data1)
    mean_group2 = np.mean(data2)
    
    # Aesthetics
    color_group1 = '#00A699'
    color_group2 = '#FC642D'

    # Create graphic object
    dist_groups = ff.create_distplot(data_grouped, group_labels=[group1_name, group2_name], show_rug= False, show_hist= False, colors= [color_group1, color_group2])
    dist_groups.add_vline(x= mean_group1, line_width= 3, line_dash= "dash", line_color= color_group1)
    dist_groups.add_vline(x= mean_group2, line_width= 3, line_dash= "dash", line_color= color_group2)
    dist_groups.add_vrect(x0= mean_group1, x1= mean_group2, line_width= 0, fillcolor= "#99AAB5", opacity= 0.2)

    # Define range of value
    dist_groups.update_layout(xaxis_range=[min(all_data),
                                            max(all_data)])
    dist_groups.update_layout(title= title, xaxis_title= xaxis_title)
    
    return dist_groups

dist_groups = dual_dist_plot(data1= data_carry_Mid.kills, data2= data_carry_Soft.kills,
                            title= 'Carry roles Kills log by Position ', xaxis_title= 'Kills log',
                            group_names= ['Mid Laners','Soft Support'])
dist_groups.update_layout(layout)
dist_groups.add_annotation(sign)
dist_groups.show(config= static)
```

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/anova_dota/master/images/plot_5.png" style="width: 900px"></p>

So now that we know that our sample is independent and not normal, let's make the hypothesis test.

Notice that each one of the samples under test has a size of 13,000 records approximately. Seeing that our sample is very large, the test is at risk of incurring in normality bias. To avoid this, we are going to rely on an **Asymptotic P-value** calculation, which will generate a standardized value given by:

$z = \frac{U - m_u}{\sigma_U}$ 

Which is defined by:

$m_U = \frac{n_{Mid} n_{Soft}}{2}$

And our Standard Deviation Value won't be affected by tied ranks, so this won't require any correction[1]. This is because the categories don't have ranks and all the values are distinct and continuous.

$\sigma_U = \sqrt{\frac{n_{Mid} n_{Soft} (n_{Mid} + n_{Soft} + 1)}{12}}$

```python
from scipy.stats import mannwhitneyu

def mannwhitneyu_est(data1, data2, alt, method):
    statistic, p = mannwhitneyu(data1, data2, alternative = alt, method= method)
    return p
    
print('Mann-Whitney Test P-value: '+ str(mannwhitneyu_est(data1= data_carry_Mid.kills, data2= data_carry_Soft.kills,
                                                            alt= 'greater', method= 'asymptotic')))
```

```
Mann-Whitney Test P-value: 0.8700862933924327
```

Since the distributions are statistically equal seeing the p-value above 0.05, we don't have enough statistical evidence to reject our Null Hypothesis. This indicates that Soft Support players and Midline players tend to make the same amount of kills per session on Ranked Matchmaking mode using heroes of Carry type.

Now that we know this it doesn't make sense to separate Mid Laners from the Soft Support group, let's regroup these two positions and compare them with the rest of the roles.

### 🧝🏻👹 Two Sample Mann-Whitney Test for Kills of Mid Lanes and Soft Support vs Rest of Roles

Let's start by defining the new hypothesis:

- $H_0: P(x_{kills_{mid \cup soft}} > x_{kills_{rest}}) = 0.5$ (The probability that a randomly drawn player from Midlane or Soft Support will record more kills than any player, is 50%)
- $H_1: P(x_{kills_{mid \cup soft}} > x_{kills_{rest}}) > 0.5$ (The probability that a randomly drawn player from Midlane or Soft Support will record more kills than any player, is more than 50%)

Using the same process as before, our first assumption to check is the non-normality of the same random sample for each group.

```python
# Subset and create kills ample for carry role in Soft Support and Midlane positions
data_soft_mid = data_carry[data_carry.role.isin(['Soft_Support', 'Mid'])]

# Subset and create kills sample for carry role in the rest positions
data_not_soft_mid = data_carry[data_carry.role.isin(['Hard_Carry', 'Hard_Support', 'Offlaner'])]

# Call function for normality test
normality_two_sample(data1= data_soft_mid.kills, data2= data_not_soft_mid.kills)
```

```
At least one sample is not normal,
where the p values are: 0.000000 and 0.000000
```

Now that we know the sample are not normal, we can proceed with the Mann-Whitney Test.

```python
dist_groups = dual_dist_plot(data1= data_soft_mid.kills, data2= data_not_soft_mid.kills,
                            title= 'Carry roles Kills log by Position ', xaxis_title= 'Kills log',
                            group_names= ['Mid & Soft Sppt Kills','Rest. Positions Kills'])
dist_groups.update_layout(layout)
dist_groups.add_annotation(sign)
dist_groups.show(config= static)
```

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/anova_dota/master/images/plot_6.png" style="width: 900px"></p>

And as well, to avoid incurring in normality bias due to the samples sizes, we are going to rely on an **Asymptotic P-value** calculation.

```python
print('Mann-Whitney Test P-value: '+ str(mannwhitneyu_est(data1= data_soft_mid.kills, data2= data_not_soft_mid.kills, alt= 'greater', method= 'asymptotic')))
```

```
Mann-Whitney Test P-value: 0.015672996714278714
```

We can see that our P-Value is very close to the rejection section. However, since this is taken each time from a random sample, we can't conclude yet. The most accurate way to test this is to bootstrap the value and make judgments over the percentage of p-values that accept or reject our hypothesis.

The next function will return one array of bootstrapped estimates for the P-value, which we will describe the next:

- **p_value_mannwhitney:** Proportion of P-values measured from the Mann-Whitney Test, showing the probability that a randomly drawn Midlaner or Soft Support will record more or equal quantity of kills than any player.

- **boot_mannwhitney_replicates:** Array of bootstrapped p-values of Mann-Whitney Test.

Also, let's plot the bootstrapped replicates.

```python
# Bootstrapping Mann-Whitney U Test, and generate array of replicates
def draw_bs_test(data1, data2, iterations):
    boot_arr = []
    for i in range(iterations):
        estimate = mannwhitneyu_est(np.random.choice(data1, len(data1)),
                                    np.random.choice(data2, len(data2)),
                                    alt= 'greater', method= 'asymptotic')
        boot_arr.append(estimate)

    # Compute and print p-value of estimate with different distributions/medians
    p_value_test = np.sum([1 if i <= 0.05 else 0 for i in boot_arr]) / len(boot_arr)

    return boot_arr, p_value_test

# Generates bootstrapped estimates
boot_mannwhitney_replicates, p_value_mannwhitney = draw_bs_test(data1= data_soft_mid.kills, data2= data_not_soft_mid.kills, iterations= 2000)

# ECDF Generator function
def ecdf(data):
    # Generate ECDF (Empirical Cumulative Distribution Function) for one dimension arrays
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
    percentiles = np.array([5,25,50,75, p_value_mannwhitney*100, 95]) #Plot the percentile where the bootstrapped p-value is located
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

ecdf = ecdf_plot(boot_mannwhitney_replicates, "P-value replicates")
ecdf.update_layout(title = {'text':'ECDF of Bootstrapped P-values for Mann-Whitney U Test'},
                        yaxis = {'title':"Cumulative Probability"})
ecdf.show(config= static)
```

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/anova_dota/master/images/plot_7.png" style="width: 1000px"></p>

From 2000 iterations of the estimate, we can see that around ~90% of the bootstrapped Mann-Whitney U tests will have a value below 0.05 (in the hypothesis rejection zone). This indicates that the probability that a randomly drawn player from Midlaners or Soft Support will record more kills than any player is more than 50%. This approximately 90% of the time due to the significant difference in the distribution parameter.

With this, our next step is to restrict the analysis for Carry Heroes used in Soft Support and Midlaner positions, since these are the two positions where more kills are registered on the sessions.

After examining kills, we should study a more specific behavior, like the damage given to opponents by these same player records, and see if there is an unusual value in the different groups of heroes.

### 🧙‍♂️ Kruskal–Wallis One-Way ANOVA

First, we are going to try to test the difference between Carry heroes groups for Soft Support and Midlaners using a common One-Way ANOVA. Which is a parametric test which is used to test the statistically significant variance difference between 3 or more groups. In this case, our categorical variable will be the hero name, which will be used to segment the groups, to compare the hero damage factor between levels.

To select the hero groups, we can simply add the hero damage metric to use as a decision parameter, for this reason, let's see the cumulative metric in a Pareto chart for the Carry heroes.

```python
# Load layout for Pareto chart
sign_pareto, layout_pareto = layout_plotly(height= 700, width= 1400, font_size= 12)

def pareto_builder(data, var_value, var_cat, title, title_xaxis, title_yaxis):

    # Aggregate data by variable
    data = data.groupby([var_cat])[var_value].sum().reset_index(drop=False)

    # Sort data
    data = data.sort_values(var_value, ascending= False).reset_index(drop=True)

    # Generate CumSum Percentage
    cumulative_per =  data[var_value] / data[var_value].sum() * 100
    cumulative_per = cumulative_per.cumsum()

    # Append cumulative percentages column
    data = pd.concat([data, cumulative_per], axis= 1)

    # Fix column names and remove last repeated value error
    data.columns = data.columns.insert(-1, 'Cum_Sum')[:-1]

    # Pareto Chart
    pareto = make_subplots(specs=[[{"secondary_y": True}]])

    # Bar plot graphic object
    pareto.add_trace(go.Bar(x = data[var_cat], y = data[var_value], name = "Hero", marker_color= "#007FFF"),
                            secondary_y = False)

    # Scatter+Lines plot graphic object
    pareto.add_trace(go.Scatter(x = data[var_cat], y = data['Cum_Sum']/100, mode='lines+markers', name = "Percentage", marker_color= "#FF5A5F"),
                            secondary_y = True)

    # Layout
    pareto.update_layout(title = {'text':title},
                        xaxis = {"title":title_xaxis},
                        yaxis = {"title":title_yaxis})
    pareto.update_yaxes(tickformat = "0%", title = "Cumulative Percentage", secondary_y = True)
    pareto.add_hline(y=0.8, line_dash = "dash", line_color="red", opacity=0.5, secondary_y = True)

    return pareto

pareto = pareto_builder(data= data_soft_mid, var_value= 'hero_damage', var_cat= 'hero_name',
                        title= 'Pareto Chart of Hero Damage Logs', title_xaxis= 'Heroes', title_yaxis= 'Total Damage recorded')
pareto.update_layout(layout_pareto)
pareto.update_layout(showlegend=False)
pareto.show()
```

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/anova_dota/master/images/plot_8.png" style="width: 1500px"></p>

From the 110 different heroes, we'll consider that 20% of them approximately allocated 80% of the total damage recorded. For this reason, we are going to subset those top 28 heroes that will be under study for our ANOVA test.

In the last visual, due to the size of the displayed area, we are not seeing the 110 heroes, but we can see the cumulative sum of damage recorded on the right Y axis of the Pareto Chart.

```python
# Aggregate data by group
list_sub_heroes = data_soft_mid.groupby(['hero_name'])['hero_damage'].sum().reset_index(drop=False)

# Sort data
list_sub_heroes = list_sub_heroes.sort_values('hero_damage', ascending= False).reset_index(drop=True)

# Extract heroes list
list_sub_heroes = list_sub_heroes.head(28).hero_name

# Subset data from players for analysis
data_heroes = data_soft_mid[data_soft_mid.hero_name.isin(list_sub_heroes)]
```

As we mentioned before, the specificity parameter is our focus point to make the right deductions, so we'll still narrow our analysis to Carry heroes. The situation is that an ANOVA will tell us only if there is a hero Mean Damage is different from the rest.

To clarify the context of why we are going to use an ANOVA test, we have two reasons; the first is to check for potential nerfed heroes in later patches, that's why our core KPI will be the damage, which is a continuous attribute; and the second one is that the comparison is made by a hero and fortunately we count with 28 heroes after the subset was made.

This way we can define the elements of the ANOVA as:

- Levels: Hero Names
- Factor: Hero Damage logs

And our Hypothesis will be defined by:

- $H_0: \overline{X}_{Windranger} = \overline{X}_{ShadowFiend} = \overline{X}_{Queen Of Pain} =$  ...  $= \overline{X}_{Riki}$
- $H_1:$ At least one of the hero damage mean differ from the rest

Let's establish the initial assumptions that we need to verify:

- **Independence validation**

```python
# Randomize order again
data_heroes = data_heroes.sample(frac= 1).reset_index(drop= True)

# Prepare Levels and Factor data only
data_heroes_anova = data_heroes[['account_id','hero_name','hero_damage']]
```

To respect the independence assumption, we will extract one record per player in case the player count with multiple sessions logged in our sample. This is because we would prefer to have a randomized sample where the groups are **mutually exclusive**, since this process was made on the definition of the data_carry dataset, it's not necessary now.

- **Kolmogorov-Smirnov test:** [check normality](https://en.wikipedia.org/wiki/Kolmogorov%E2%80%93Smirnov_test) on residuals

Based on an Ordinary Least Squares model where our independent variable will be the hero_name groups and our dependant variable the hero_damage, we can generate the residual of our model.

```python
from statsmodels.formula.api import ols

# Generate model to extract residuals
model = ols('hero_damage ~ C(hero_name)', data= data_heroes_anova).fit()

# Probabilty plot generator
def qq_plot(data, dist_type, title):
    '''
    Requirements:
    - scipy.stats
    - plotly.graph_objects
    '''
    # Generate qq plot
    qq = stats.probplot(data, dist= dist_type)
    # Take min and max value of OSM (ordered statistic medians) and generate array
    x = np.array([qq[0][0][0], qq[0][0][-1]])
    # Create graphic object
    qq_plot = go.Figure()
    # Plot OSM (Ordered Statistic Medians) in x-axis, and OSR in y-axis (Ordered Responses)
    qq_plot.add_scatter(x=qq[0][0], y=qq[0][1], mode='markers')
    # Generate linear reg line using the generated array and the OSR data as parameters
    qq_plot.add_scatter(x=x, y=qq[1][1] + qq[1][0]*x, mode='lines')
    # Add titles
    qq_plot.update_layout(title = {'text': title},
                        xaxis = {"title":'Theorical Quantiles'},
                        yaxis = {"title":'Ordered Values'})
    return qq_plot

qq_plot = qq_plot(data= model.resid, dist_type= 'norm', title= "Gaussian Probabilty Plot of ANOVA's residuals")
qq_plot.update_layout(layout)
qq_plot.update_layout(showlegend=False)
qq_plot.add_annotation(sign)
qq_plot.show(config= static)
```

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/anova_dota/master/images/plot_9.png" style="width: 900px"></p>

A visual check is helpful when the sample is large. As the sample size increases, the statistical test's ability to reject the null hypothesis rises, so it gains the power to detect smaller differences as the sample size n increases.

From the last image, we can see on the left side, that the distribution tends to spread, and considering that we rely on approximately 20,000 residuals, we can conclude non-normality. However, the best way to decide this is to test it.

Here we **shouldn't use a Shapiro Welch** since our sample is above 50 observations, the best way to test the Goodness to Fit a Gaussian distribution, is to use the **Kolmogorov-Smirnov test**. Notice that size of our data is a relatively large sample, the calculation of the p-value will perform better if we use an asymptotic method.

```python
stats.kstest(model.resid, cdf= 'norm', mode= 'asymp')
```

```
KstestResult(statistic=0.5365555738561936, pvalue=0.0)
```

We have sufficient evidence to say that the sample data does not come from a normal distribution, since our **Kolmogorov-Smirnov** p-value is below 0.05. Knowing this we can't proceed with a One-way ANOVA, so we have two options, we will proceed with a non-parametric version of ANOVA, the [Kruskal-Wallis](https://en.wikipedia.org/wiki/Kruskal%E2%80%93Wallis_one-way_analysis_of_variance) test.

```python
# Get Kruskal-Wallis P-value
def kruskal_est(data, group_var, value_var, sample_size):
    # Extract sample from population
    data = data.sample(sample_size)
    # Extract 
    group_names = data[group_var].unique()
    number_groups = len(group_names)

    # List to allocate arrays of groups
    input_list = []

    # Populate array
    for i in range(number_groups):
        input_list.append(data[data[group_var] == group_names[i]][value_var] )
    input_list = np.array(input_list, dtype= 'object')

    # Execute test over list of group arrays
    H, p = stats.kruskal(*input_list)

    return p , number_groups

p , number_groups = kruskal_est(data= data_heroes_anova, group_var= 'hero_name', value_var= 'hero_damage', sample_size= 500)
print(p)
```

```
8.112743961607323e-13
```

The Kruskal-Wallis P-value in the rejection zone indicates that we have enough statistical evidence to reject the Null Hypothesis, meaning that at least one of the hero damage means differs from the rest.

Still, let's generate a visual of the intervals, including the mean and standard deviation, to see which of the hero groups are generating that significant difference.

```python
def interval_plot(data, category_col, value_col, max_cat):
    # Generate mean value and standard deviation per category
    data_mean = data.groupby([category_col], as_index= False)\
                    .agg(value_col_mean = (value_col, "mean"),
                        value_col_std = (value_col, "std"))
    # Select top categories by value
    data_mean = data_mean.sort_values("value_col_mean", ascending= False)\
                            .reset_index(drop= True).head(max_cat)
    # Generate interval plot
    interval_plot = px.scatter(data_mean,
                                x = "value_col_mean", y = category_col,
                                color = category_col, error_x ="value_col_std")
    # Comparison line                    
    interval_plot.add_vline(x= data_mean['value_col_mean'][0], line_dash = "dash",
                            line_color="grey", opacity=0.5)

    return interval_plot

interval_plot = interval_plot(data= data_heroes_anova, category_col= "hero_name",
                                value_col= "hero_damage", max_cat= 15)
interval_plot.update_layout(layout).add_annotation(sign)
interval_plot.update_layout(title = {'text': "Interval plot for Hero Damage Logs"},
                            xaxis = {"title":'Hero Damage'},
                            yaxis = {"title":'Hero'},
                            showlegend=False)
interval_plot.show(config= static)
```

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/anova_dota/master/images/plot_10.png" style="width: 900px"></p>

Effectively among 28 hero groups, we can see that there is a hero whose damage log confidence interval differs from others, this one is [Zeus](https://www.dota2.com/hero/zeus), and let's explore his behavior overall.

```python
# Subset of attributes
data_coord = data_heroes[['hero_name', 'gold_per_min', 'hero_damage', 'tower_damage', 'victory']]

# Aggregations mapping
aggregations = {'gold_per_min':'median', 'hero_damage':'median', 'tower_damage':'median', 'victory':'sum'}

# Extract top 6 heroes sorted by hero_damage
data_coord = data_coord.groupby("hero_name").agg(aggregations).reset_index(drop=False)
data_coord = data_coord.sort_values("hero_damage", ascending= False).reset_index(drop= True).head(6)

# Linear Scaling for numeric attributes
for i in data_coord[data_coord.select_dtypes(include=np.number).columns]:
    data_coord[i] = linearScale(data_coord[i])

# Format for coordinates plot, vertical spread
data_coord = data_coord.melt(id_vars=["hero_name"], var_name= "attribute")

# Load layout base objects
sign, layout = layout_plotly(height= 770, width= 1100, font_size= 15)

# Generate coordplot
fig = px.line_polar(data_coord, r="value", theta="hero_name", color="attribute", line_close=True, start_angle = 60,
                    color_discrete_sequence=px.colors.diverging.Portland, 
                    template="plotly_dark"
                    ).update_traces(fill='toself')

# Assign layout
fig.update_layout(layout).add_annotation(sign)
fig.update_layout(title = {'text': "Interval plot for Hero Damage Logs"},
                            xaxis = {"title":'Hero Damage'},
                            yaxis = {"title":'Hero'})
fig.show(config= static)
```

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/anova_dota/master/images/plot_11.png" style="width: 900px"></p>

From the linearly scaled features we visited in the last graph, we can infer two interesting insights:

- Shadow Fiend is the hero that presents the higher metrics in terms of gold collection, tower damage, and victories. This can be due to the '[Requiem of Souls](https://dota2.fandom.com/wiki/Shadow_Fiend#Requiem_of_Souls)' ability update, which is a wave turnover attack that was upgraded with the "Aghanim's Scepter", one of the most used in patch [6.86](https://dota2.fandom.com/wiki/Shadow_Fiend/Changelogs). And this because brings the capacity to reach a higher damage range, which was particularly useful for killing creeps and farming.

- On the other side, Zeus has the higher damage registry, and the logs surpass by far the other heroes. This was because of several reasons, first in the patch, the developers released a remodel of the character, then this is a hero mostly used in Soft Support roles, so was probably used for Jungling, and then it is a ranged hero with one point of [usage complexity](https://www.dota2.com/hero/zeus), which made more straightforward his playability.

## 🗒️ Final thoughts & takeaways

**What can the stakeholders understand and take in consideration?**

At the beginning we were looking that the most buffed heroes, in terms of kills were different heroes like Riki, Huskar, and Ursa, however, unless we start using statistical tests applied to a behavioral metric like the damage dealt to enemies, we began noticing contrasts, where some heroes were registering higher victory values like Shadow Fiend and others high damage ranges without having superiority in term of sessions succeeded. For this reason, is preferable to base the conclusion upon statistical techniques to study subjective behaviors.

**What could the stakeholders do to take action?**

For now, the major concern of the Game Designer, should not be to take immediate action once they see this kind of insight. Instead, they can start collecting more granular data about the registries of damage logs changes, especially when abilities were introduced. Then can study two samples, one after the patch and another before, using the data archived, to run an A/B test to prove its veracity.

**What can stakeholders keep working on?**

MOBA games tend to rely their sustainability on their community engagement and this can be studied by the player’s user experience. Usually, when buffed heroes are used for Jungling, which among DOTA 2 community sometimes is a frowned upon activity, this will make the users feel uncomfortable because of unfair mechanics, so the main solution is to build a User Metrics dashboards segmented by characters to track closely its behavior and take prompter actions each time a patch is released.

---

## ℹ️ Additional Information

- **About the article**

This article was developed from the content explained in the Inferential statistics section of Chapter 3 of the ‘Game Data Science book' and part of the statistical foundation was based on the book ‘Probability & Statistics for Engineers & Scientists’ by Walpole et. al (2016).

All the assumptions and the whole case scenario were developed by the author of this article, for any suggestion I want to invite you to go to my about section and contact me. Thanks to you for reading as well.

- **Related Content**

— Dota 2 Protracker [Role Assistant](https://www.dota2protracker.com/meta) by Stratz

— Game Data Science book and additional info at the [Oxford University Press](https://global.oup.com/academic/product/game-data-science-9780192897879?cc=cr&lang=en&)

— Anders Drachen personal [website](https://andersdrachen.com/)

— DOTA 2 Balance of Power [Patch Notes](https://www.dota2.com/balanceofpower)

- **Datasets**

This project was developed with a dataset provided by  [Albert Cui](https://github.com/albertcui), [Howard Chung](https://github.com/howardchung), and [Nicholas Hanson-Holtry](https://github.com/nicholashh), which also can be found at [YASP](https://academictorrents.com/details/5c5deeb6cfe1c944044367d2e7465fd8bd2f4acf) Academic Torrents or you can experiment with this great [OpenDota API interface](https://www.opendota.com/explorer) to make queries.

**Note:** You won't find the raw data in the [Github repository](https://github.com/robguilarr/anova_dota/tree/master/DOTA_data) of this project due the storage capabilities, instead you will find a random sample with its respective preprocessing explanation in a Jupyter Notebook.
