# Video Games History explained with Pandas


The industry of video games revenues is reaching the $173.7 billion in value, with around 2.5 billion users enjoying them worldwide, with a forecasted value of $314.40 billion by 2026 according to Mordor Intelligence.

Impressive facts, right? Nowadays this market is no longer considered a simple hobby for kids, it has become a constantly growing giant which attracts more and more customers as it takes advantage of the growth of streaming platforms. But this industry, as we well know, is taking more fields, born from ambitious expectations such as the Nintendo World Championships in the 90's to what today many have adopted as a lifestyle also known as <i>Esports</i>.

<!--more-->

---

{{< admonition info "Looking for an interactive experience?" true >}}
:rocket: Download the Jupyter Notebook, available <a href="https://nbviewer.jupyter.org/github/robguilarr/videogames-eda/blob/main/videogame_analysis/videogame-analysis.ipynb">here</a>
{{< /admonition >}}

{{< youtube K-NBcP0YUQI >}}

We’ll take a tour through the history of videogames, starting from late 70s and early the 80s. However, as a way of clarification, if you are a member of the culture, it's important to mention that due to limitations of the scope of data available for analysis, <a href="https://gamicus.gamepedia.com/Tomohiro_Nishikado">Tomohiro Nishikado's</a> masterpiece, released as Space Invaders, will not be part of the analysis; and in case you’re not a member don’t worry this is for you as well.

From an optimistic point of view, we will analyze quite important historical data, because is difficult to even think about getting the 70s data like <i>Pong</i>; and another advantage is that we can start our journey from the market revolution in the early 80s.

---

Before starting our journey, like any exploratory data analysis we must import our libraries.

```python
# To manage Dataframes
import pandas as pd
# To manage number operators
import numpy as np
# To do interactive visualizations
import plotly.express as px
import plotly.graph_objects as go
import plotly.figure_factory as ff
# Format
from vizformatter.standards import layout_plotly
```

Now, let's import our data. We must consider that we already prepared it, as shown in this articles's footnote[^1].

```python
# Data frame of videogames
df = pd.read_csv("data/videogames.csv", na_values=["N/A","", " ", "nan"],
                 index_col=0)
```

In addition to facilitate the management of dates in the visualizations, two extra columns will be generated, one as a <i>Timestamp</i> and another as a <i>String</i>, which will be used only if required.

```python
# Transform Year column to a timestamp
df["Year_ts"] = pd.to_datetime(df["Year_of_Release"], format='%Y')

# Transform Year column to a string
df["Year_str"] = df["Year_of_Release"].apply(str) \
                                        .str.slice(stop=-2)
```

Also we can import the layout format as a variable from my <a href="https://github.com/robguilarr/vizformatter/blob/master/vizformatter/standards.py">repository</a>.

```python
sign, layout = layout_plotly(height= 720, width= 1000, font_size= 15)
```


---

## Data integrity validation

First, we check our current dataset using the method <code>.info()</code>

```python
df.info()
```

```Code
<class 'pandas.core.frame.DataFrame'>
Int64Index: 16716 entries, 0 to 16718
Data columns (total 20 columns):
 #   Column           Non-Null Count  Dtype
---  ------           --------------  -----
 0   Name             16716 non-null  object
 1   Year_of_Release  16447 non-null  float64
 2   Publisher        16662 non-null  object
 3   Country          9280 non-null   object
 4   City             9279 non-null   object
 5   Developer        10096 non-null  object
 6   Platform         16716 non-null  object
 7   Genre            16716 non-null  object
 8   NA_Sales         16716 non-null  float64
 9   EU_Sales         16716 non-null  float64
 10  JP_Sales         16716 non-null  float64
 11  Other_Sales      16716 non-null  float64
 12  Global_Sales     16716 non-null  float64
 13  Critic_Score     8137 non-null   float64
 14  Critic_Count     8137 non-null   float64
 15  User_Score       7590 non-null   float64
 16  User_Count       7590 non-null   float64
 17  Rating           9950 non-null   object
 18  Year_ts          16447 non-null  datetime64[ns]
 19  Year_str         16716 non-null  object
dtypes: datetime64[ns](1), float64(10), object(9)
memory usage: 2.7+ MB
```

To one side we find a great variety of data and attributes, to the other one we see that of the total of 16,716 records there are several attributes with a significant number of null values, which we are going to see next, in percentage terms.

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

```Code
NaN Values per column:

Country has 44% of Null Values
City has 44% of Null Values
Developer has 40% of Null Values
Critic_Score has 51% of Null Values
Critic_Count has 51% of Null Values
User_Score has 55% of Null Values
User_Count has 55% of Null Values
Rating has 40% of Null Values
```

These correspond to the attributes that hold more than 5% of the null values considering a confidence standard, which consists of having at least 95% of the data.

In a visual way, we can look at it in the following graphic.

```python
# Make a dataframe of the number of Missing Values per attribute
df_na = df.isna().sum().reset_index()

# Rename our dataframe columns
df_na.columns = ["Column","Missing_Values"]

# Plot barchart of Missing Values
barna = px.bar(df_na[df_na["Missing_Values"] > 0].sort_values
               ("Missing_Values", ascending = False),
               y="Missing_Values", x="Column", color="Missing_Values", opacity=0.7,
              title = "Total of Missing Values per attribute", color_continuous_scale=
               "teal",
              labels = {"Missing_Values":"Missing Values"})

# Update layout
barna.update_layout(layout)
barna.update_annotations(sign)
barna.show()
```
<p align="center">
{{< image src="https://raw.githubusercontent.com/robguilarr/videogames-eda/main/videogame_analysis/plots-images/barplot_1.png?format=webpage" width="75%" height= "75%" alt="Barplot of Missing Values">}}
</p>

We see that there is a significant quantity of null values, predominantly in columns related to critics and their respective value (<a href="https://www.metacritic.com/browse/games/score/metascore/all/all/filtered">Metacritic</a>); as well as its content Rating made by <i>ESRB</i> (<a href="https://www.esrb.org/search/?searchKeyword= platform=Nintendo%203DS%2CWii%20U%2CPlayStation%204%2CPlayStation%203%2CXbox%20One%2CXbox%20360%2CPC%2COther&rating=E%2CE10%2B%2CT%2CM%2CAO&descriptor=All%20Content&pg=1&searchType=MostViewed">Entertainment Software Rating Board</a>).

Still, since these are not categorical variables, they won’t have an identifier role, in which case our main interest will be <i>"Name"</i> and <i>"Year_of_Release"</i>, and subsequently their elimination or omission will be evaluated if necessary.

---

## Exploratory Video Game Analysis (EVGA)

Before starting with our expedition we should begin by understanding the behavior of the data with which our analysis will be built, for this we’ll use the <code>.describe()</code> method.

```python
# Modify decimal number attribute
pd.options.display.float_format = "{:.2f}".format

# Print description
df.describe()
```

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/videogames-eda/main/videogame_analysis/plots-images/describe.png" style="width: 900px"></p>


The numerical attributes show us that we have a total of 40 years of records (from 1980 to 2020) of sales in North America, Europe, Japan, and other parts of the world. Where the median indicates that 50% of the years recorded are less than or equal to 2007, and we did not find outliers.

Also, the average sales value is higher in North America despite the fact that the average sale of the titles is around 263,000 units, but its variation is quite high, so it should be compared more exhaustively.

From a historical point of view, it makes sense, that knowing that the focus of sales is North America, cause even during the 60s the head of <i>Nintendo of America</i>, <a href ="https://en.wikipedia.org/wiki/Minoru_Arakawa">Minoru Arakawa</a>, decided to expand their operations in the United States starting from the world of the arcade, so we can have the hypothesis to see this as a place of opportunities for this market.

### Golden Age of Videogames

<blockquote><b>1977</b> – Launch of Atari 2600</blockquote>

<p align= center>
    <img src="https://upload.wikimedia.org/wikipedia/commons/b/b9/Atari-2600-Wood-4Sw-Set.jpg?format=webpage" width="600" height="400" alt="Atari_2600">
</p>

We will begin with the global view, I mean, the superficial perspective of the sales during this period.

```python
# Mask to subset games from 1980 to 1990
games8090 = df["Year_of_Release"].isin(np.arange(1980,1991))


# Top publishers between 1980 and 1990
top_pub = df[df["Year_of_Release"]<=1990].groupby("Publisher") \
                                        .sum("Global_Sales") \
                                        .sort_values("Global_Sales", ascending = False)["Global_Sales"] \
                                        .head(10)

# Dataframe for Line Plot of most frequent companies
df_sales_ts = df[games8090][df["Publisher"].isin(top_pub.index)] \
                .pivot_table(values = "Global_Sales",
                             index = ["Year_of_Release", "Year_str", "Year_ts",
                                      "Publisher","Platform"], aggfunc= np.sum) \
                .reset_index() \
                .sort_values("Year_of_Release", ascending = True) \
                .groupby(["Publisher","Year_ts"]) \
                .sum() \
                .reset_index()


# Plot a lineplot
gline = px.line(df_sales_ts, x="Year_ts", y="Global_Sales", color='Publisher',
               labels={"Year_ts": "Years",  "Global_Sales": "Millions of Units Sold", "total_bill": "Receipts"},
               title = "Millions of units during Golden Age sold by Publisher")

# To plot markers
for i in np.arange(0,10):
    gline.data[i].update(mode='markers+lines')

# Update Layout
gline.update_layout(layout)
gline.update_annotations(sign)
gline.show()
```
<p align="center">
{{< image src="https://raw.githubusercontent.com/robguilarr/videogames-eda/main/videogame_analysis/plots-images/lineplot_2.png?format=webpage" width="75%" height= "75%" alt="Lineplot of Golden Age">}}
</p>

As we can see at the beginning of the decade and probably after 1977, the market was dominated by Atari Studios while Activision was its main competitor in terms of <a href= "https://twinfinite.net/2016/06/video-games-what-does-ip-mean/">IPs</a>, because these competitors eventually published their titles on the Atari 2600, example of this was Activision with <i>Kaboom!</i> or Parker Bros with <i>Frogger</i>.

Another important fact is that in 1982 we can remember that it was one of the best times for Atari where they published titles that had a great impact such as Tod Frye's <i>Pac-Man</i>.

```python
# Mask of 1982 games
games82 = df[df.Year_of_Release == 1982]

# Distribution column
games82['Distribution'] = (games82.Global_Sales/sum(games82.Global_Sales))*100

# Extracting top titles of 1982
games82 = games82.sort_values('Distribution', ascending=False).head(10)

# Fix Publisher Issue of Mario Bros., this game was originally published by
# Nintendo for arcades
games82.loc[games82.Name == 'Mario Bros.','Publisher'] = 'Nintendo'

# Distribution
bar82 = px.bar(games82, y='Distribution', text='Distribution', x='Name', color =
'Publisher', title='Distribution of total sales in 1982 by Publisher',
               labels={"Distribution":"Market Participation distribution",
                       "Name":"Videogame title"})

# Adding text of percentages
bar82.update_traces(texttemplate='%{text:.3s}%', textposition='outside')

# Update layout
bar82.update_layout(layout)
bar82.update_annotations(sign)
bar82.show()
```
<p align="center">
{{< image src="https://raw.githubusercontent.com/robguilarr/videogames-eda/main/videogame_analysis/plots-images/barplot_3.png?format=webpage" width="75%" height= "75%" alt="Barplot of 1982">}}
</p>

It is evident that the adaptation of this arcade game released in 1980, had outstanding sales, once it was introduced to the world of the Atari 2600. According to the documentary <i>"Ounce Upon Atari"</i>, episode 4 to be exactly, this title managed to sell more than 7 million copies, due to optimizations in the display and in the intelligence of the <a href="https://www.businessinsider.com/npc-meaning">NPCs</a>, compared to the original version.

- <b>FYI:</b> The version of Mario Bros in the dataset corresponds to the Atari 2600 and Arcade version are different from the success that was later introduced to the NES.

<blockquote><b>1983</b> – Crisis of the Video Game industry</blockquote>

<p align= center>
    <img src="http://uploads.neatorama.com/images/posts/677/89/89677/1460595792-0.jpg?format=webpage" width="700" height="400" alt="ET">
</p>

Undoubtedly, the timeline above shows a clear drop in sales from 1983.

And yes, I'm sure they want to know what happened here.

For sure, if we had <a href="https://en.wikipedia.org/wiki/Howard_Scott_Warshaw">Howard Scott Warshaw</a> talking with us, we would surely understand one of the crudest realities in this industry’s history, since he lived this in his own flesh. But in this case, I will explain.

In summary, he was one of the greatest designers of that moment, who was hired to design a video game based on one of the biggest hits in cinema, <i>E.T. the Extra-Terrestrial</i>. At the time Steven Spielberg shares the vision of a game very similar to Pac-Man, something extremely strange, and by the way a release date is designated just a few months after this.

As you may have thought, it was a complete disaster. Like this case, many developers saw the accelerated growth of the industry as an opportunity to launch titles in large numbers and with a very low quality content, as evidenced by the second quartile of our initial analysis.

There were many other causes such as the massive appearance of consoles and the flexibility of the guidelines for <i>third party developers</i>, but if you want a quick perspective, I recommend <a href = "https://www.ign.com/articles/2011/09/21/ten-facts-about-the-great-video-game-crash-of-83">this</a> IGN article.

<blockquote><b>1984</b> – A new foe has appeared! Challenger approaching</blockquote>

<p align= center>
    <img src="https://cdn.gamer-network.net/2016/usgamer/donkey_kong_nes_classic_03.jpg" width="600" height="400" alt="DK">
</p>

After the drop because of the oversupply of titles, Nintendo Entertainment saw a chance to take over the American market with its local bestseller the
<i>Famicom</i>. This was transformed through a redesign adapted for the North American public, being renamed as <a href="https://en.wikipedia.org/wiki/Nintendo_Entertainment_System#North_American_release">NES</a> (Nintendo Entertainment System), previously named Nintendo Advanced Video System.

Also, thanks to the great success known as Donkey Kong, the mastermind <a href="https://en.wikipedia.org/wiki/Shigeru_Miyamoto">Shigeru Miyamoto</a>, takes advantage of the success of Jumpman and Pauline; in 1983 he released Mario Bros and the rest is history.

The Donkey Kong game despite being called referring to the antagonist of the video game, was not the most interesting character for consumers, instead it was Jumpman, also
known as Mario.

The importance of Nintendo for the North American market can be seen through the following graph, where the global sales of titles are generally seen in the four regions, in which North America covers the largest numbers by far.

```python
# Aggregation dictionary for each region
agg_region = {'NA_Sales': 'sum', 'JP_Sales': 'sum', 'EU_Sales': 'sum', 'Other_Sales':
    'sum', 'Global_Sales': 'sum'}

# Dataframe of regions 80-90s
reg8090 = df[games8090].groupby("Year_of_Release").agg(agg_region).reset_index()\
    .sort_values("Year_of_Release", ascending=True)

# To loop and place the traces
region_suffix = '_Sales'
regions = ['NA', 'JP', 'EU', 'Other']
region_names = ['North America', 'Japan', 'Europe', 'Other']
i=0

# Generate graph object
regplot = go.Figure()
for region in regions:
    regplot.add_trace(go.Scatter(x = reg8090['Year_of_Release'], y =
    reg8090[region+region_suffix], mode='markers+lines', name=region_names[i]))
    i += 1

# Update layout
regplot.update_layout(layout)
regplot.update_annotations(sign)
regplot.show()
```
<p align="center">
{{< image src="https://raw.githubusercontent.com/robguilarr/videogames-eda/main/videogame_analysis/plots-images/lineplot_4.png?format=webpage" width="75%" height= "75%" alt="Lineplot Regional">}}
</p>

A conclusion that is worth mentioning is that even Nintendo's success today is not only due to its innovation and sense of affection for its IPs, but also because of the exclusivity of its titles. As shown, both the NES and the GameBoy had great sales in the North American market despite being Japanese companies.

<blockquote><b>1989</b> – Gunpei Yokoi, father of the Game & Watch series, creates the GameBoy, the ultimate portable console</blockquote>

<p align= center>
    <img src="https://images.squarespace-cdn.com/content/v1/5387c6c8e4b058cfb90dabb1/1508174421121-DD3XGEBDKOQUN8MSTFMK/ke17ZwdGBToddI8pDm48kIPMjyAxYAbEcFTs8zCKKJJ7gQa3H78H3Y0txjaiv_0fDoOvxcdMmMKkDsyUqMSsMWxHk725yiiHCCLfrh8O1z4YTzHvnKhyp6Da-NYroOW3ZGjoBKy3azqku80C789l0qN_-Z3B7EvygvPOPmeOryX1pkXg-pKyyA7MDP6ZuEHmJuzC89-qxFpXVqq6ylzIAA/P1010414.jpg" width="600" height="400" alt="gandw">
</p>

### I World Console War (WCWI)

<blockquote><b>1989</b> - Sega Enterprises Inc. launches worldwide Sega Megadrive Genesis</blockquote>
<blockquote><b>1991</b> - Nintendo launches worldwide Super Nintendo Entertainment system</blockquote>

<p align= center>
    <img src="https://chiscroller.files.wordpress.com/2016/07/mario-vs-sonic.png?format=webpage" width="90%" height="90%" alt="gamingwar">
</p>

At the beginning of the 90s, after the launch of the SEGA and Nintendo consoles, the First World War of Videogames began. Mainly in two of their biggest titles, Sonic The Hedgedog and Super Mario Bros.

During 1990, approximately 90% of the US market was controlled by Nintendo, until in 1992 SEGA began to push with strong marketing campaigns aimed at an older audience.

One of the most remarkable fact of this period was the launch of Mortal Kombat in 1992, where Nintendo censored part of its content (<a href="https://en.wikipedia.org/wiki/Controversies_surrounding_Mortal_Kombat">blood-content</a>) since they had an initiative to be a <i>family friendly</i> company, and of course this became very annoying the followers of this series.

```python
# Transform current dataframe as long format
df_long = df.melt(id_vars = ["Name","Platform","Year_of_Release","Genre",
                           "Publisher", "Developer", "Rating", "Year_str", "Year_ts",
                           "Country", "City","Critic_Score","User_Score"],
                  value_vars = ["NA_Sales", "EU_Sales","JP_Sales","Other_Sales"],
                  var_name = ["Location"],
                  value_name = "Sales")


# Giving a better format to the location's Name
df_long = df_long.replace({"Location": {"NA_Sales": "North America",
                                        "EU_Sales": "Europe",
                                        "JP_Sales": "Japan",
                                        "Other_Sales": "Rest of the World"} })

# To delete columns without sales registry
df_long =  df_long[df_long["Sales"] > 0].dropna(subset = ["Sales"])

# Dataframe
df_gen90 = df_long[(df_long["Year_of_Release"] > 1989) & (df_long["Year_of_Release"] < 2000)] \
                            .pivot_table(values = "Sales", index = "Genre",
                                        columns = "Location", aggfunc = np.sum)

# Image plot
ima = px.imshow(df_gen90.reset_index(drop=True).T,
                y= ["Europe","Japan","North America","Rest of the Worlds"],
                x= ['Action', 'Adventure', 'Fighting', 'Misc', 'Platform', 'Puzzle',
                    'Racing', 'Role-Playing', 'Shooter', 'Simulation', 'Sports','Strategy'],
               labels=dict(color="Total Sales in Millions"),
               color_continuous_scale='RdBu_r',
               title = "Heatmap of Location vs Genre during WCWI")

# Update layout
ima.update_layout(layout)
ima.update_annotations(sign)
ima.show()
```
<p align="center">
{{< image src="https://raw.githubusercontent.com/robguilarr/videogames-eda/main/videogame_analysis/plots-images/heatmap_5.png?format=webpage" width="75%" height= "75%" alt="Heatmap of WCWI">}}
</p>

Following the Mortal Kombat censorship, Nintendo was hit, noticing that fighting genres were among the most purchased during the 90s. However, the success of Nintendo IPs such as The Legend of Zelda and Super Mario, ended up destroying the SEGA console in 1998, in addition because of Nintendo grew stronger thanks to role-playing games during these years.

```python
# Dataframe with just SNES and GEN, Super Mario was removed to avoid outliers
df_sn = df_long[(df_long["Year_of_Release"] > 1989) &
                (df_long["Year_of_Release"] < 2000) &
               ((df_long["Platform"].isin(["GB","NES","SNES","GEN","PC","PS","N64","DC"]))
               )].sort_values("Year_of_Release", ascending=True).drop(18)

# Plot of sales during 90s
strip90 = px.strip(df_sn, x = "Year_of_Release", y = "Sales", color = "Platform",
                  hover_name="Name",
                  labels={"Name":"Title", "Year_of_Release":"Year"},
                 hover_data=["Country"])

# Update layout
strip90.update_layout(layout)
strip90.update_traces(jitter = 1)
strip90.update_annotations(sign)
strip90.show()
```
<p align="center">
{{< image src="https://raw.githubusercontent.com/robguilarr/videogames-eda/main/videogame_analysis/plots-images/scatterplot_6.png?format=webpage" width="75%" height= "75%" alt="Scatterplot of 90s">}}
</p>

The first impression, when looking at this graph is that we notice the great dominance of Nintendo since the sales of the Sega Genesis collapsed in 1995, until during the Sega Dreamcast campaign, where the leadership was taken by the GameBoy and the <i>Nintendo 64</i>, followed by the new competitor <i>Play Station</i>, a topic that we will cover later.

### Role-playing game revolution

<p align= center>
    <img src="https://pokejungle.net/wp-content/uploads/2019/11/rby-banner.jpg?format=webpage" width="600" height="400" alt="pokemon">
</p>

One of the most characteristic events of this time was the implementation of 16-bit graphic technologies, which at the time was double what was available. Along with this, the Japanese once again made another master move, which gave a decisive turn to a genre, after the expected fall of RPGs on the PC platform.

Before highlighting the Japanese originality, it is necessary to know after successes of role-playing games such as <a href="https://en.wikipedia.org/wiki/Ultima_VIII:_Pagan">Ultima VIII: Pagan (PC)</a>, this genre had a slow development, since the CD-ROMs generated great graphic expectations for the developers, by the way prolonging the releases, and for sure this caused a lack of interest from the community, and began to move towards action games or first person shooter such as <a href ="https://en.wikipedia.org/wiki/GoldenEye_007_(1997_video_game)">Golden Eye (1997)</a>. However, success stories continued to appear in this genre such as <a href="https://en.wikipedia.org/wiki/Diablo_(video_game)">Diablo (1996)</a>, developed by Blizzard Entertainment.

```python
# Dataframe for Genre lineplot
df90G = df_long[(df_long["Year_of_Release"] > 1989) &
                (df_long["Year_of_Release"] < 2000) &
               ((df_long["Genre"] == "Role-Playing") | (df_long["Genre"] == "Action") |
                (df_long["Genre"] == "Platform") | (df_long["Genre"] == "Fighting")
               )] \
               .groupby(["Genre", "Year_ts"]).sum("Sales").reset_index()

# Plot an animated lineplot
linegen = px.line(df90G,
                 x="Year_ts", y="Sales", color="Genre",
                title = "Millions of units sold during 90s by Genre ",
                 labels={"Sales": "Millions of Units Sold", "Year_ts":"Years"})

# To plot markers
for i in np.arange(0,4):
    linegen.data[i].update(mode='markers+lines')

# Update layout
linegen.update_layout(layout)
linegen.update_annotations(sign)
linegen.show()
```
<p align="center">
{{< image src="https://raw.githubusercontent.com/robguilarr/videogames-eda/main/videogame_analysis/plots-images/lineplot_7.png?format=webpage" width="75%" height= "75%" alt="Lineplot of 90s">}}
</p>

Among all genres, the growth of RPGs over time must be underlined. The release of <a href = "https://en.wikipedia.org/wiki/Pok%C3%A9mon#:~:text=The%20third%20versi%C3%B3n%20(fourth%20en,1%20of%20october%20of%201999.">Pokémon</a> in 1996 for GameBoy by the developer Game Freak was a success for Nintendo, which swept everything in its path, with its first generation of Pokemon Blue, Red and Yellow that was released in 1999, the latter is the fourth Japanese version.

### A new Japanese Ruler takes the throne

<blockquote><b>1994</b> - Sony Computer Entertainment's PlayStation is born</blockquote>

<p align= center>
    <img src="https://upload.wikimedia.org/wikipedia/commons/3/39/PSX-Console-wController.jpg?format=webpage" width="60%" height="60%" alt="ps1">
</p>

RPGs not only boosted Nintendo, but multiplatform IPs like Final Fantasy VII gave companies such as Sony Computer Entertainment a boost during the introduction of their new 32-bit console and at the same time to publicize the <a href="https://mediawiki.middlebury.edu/FMMC0282/JRPG">JRPGs</a> within the western market.

In 1995, when Sony planned their introduction of the PlayStation to America, they chose not to focus their Video Game market on a single type of genre or audience, but instead diversified their video game portfolio and memorable titles such as Crash Bandicoot, Metal Gear Solid and Tekken emerged.

```python
# Subset only PS games
df_sony = df[(df["Year_of_Release"].isin([1995,1996])) & (df["Platform"] == "PS")]

# Subset the columns needed
df_sony = df_sony[["Name","Year_of_Release","Publisher","Platform","Genre",
                     "Global_Sales"]]

# Pie plot
piesony = px.pie(df_sony, values= "Global_Sales",
             names='Genre',
            color_discrete_sequence = px.colors.sequential.Blues_r,
                 labels={"Global_Sales":"Sales"})

# Update layout
piesony.update_layout(layout)
piesony.update_traces({"textinfo":"label+text+percent",
                       "hole":0.15})
piesony.update_annotations(sign)
piesony.show()
```
<p align="center">
{{< image src="https://raw.githubusercontent.com/robguilarr/videogames-eda/main/videogame_analysis/plots-images/piechart_8.png" width="75%" height= "75%" alt="PlayStation Piechart">}}
</p>

As we can see in the graph, Sony's video game distribution during its first two years in the North American market. Even if we pay attention, titles like Tekken and Mortal Kombat had a significant presence by showing the highest levels of sales by genre (referring to "Fighting" genre).

### Content control warnings

<blockquote><b>1994</b> - Foundation of Entertainment Software Rating Board</blockquote>

<p align= center>
    <img src="https://images-na.ssl-images-amazon.com/images/G/01/img17/video-games/esrb/esrb_ratings_categories.jpg?format=webpage" width="600" height="400" alt="ESRB">
</p>

After titles like Doom, Wolfenstein and Mortal Kombat, an American system arises to classify the content of video games, and assign it a category depending on its potential audience maturity. It was established in 1994 by the Entertainment Software Association, the formerly called the Interactive Digital Software Association.

```python
# ESRB Rating Dataframe
df_r = df[df["Rating"].isna() == False]

df_r = df_r.groupby(["Rating","Platform"]).count()["Name"] \
                                    .reset_index() \
                                    .pivot_table(values = "Name", index = "Rating",
                                                 columns = "Platform", aggfunc = [np.sum]) \
                                    .fillna(0)

# Drop empty classifications
df_r = df_r.drop(["AO","EC","K-A","RP"])

# Heatmap of ESRB Rating vs Consoles
gesrb = px.imshow(df_r.reset_index(drop=True),
                x= ["3DS","DC","DS","GBA","GC","PC","PS","PS2","PS3","PS4","PSP","PSV",
                    "Wii","WiiU","Xbox 360","Xbox","Xbox One"],
                y= ['E', 'E10+', 'M', 'T'],
               labels=dict(x="Console", y="ESRB Rating", color="Number of Titles"),
               color_continuous_scale='BuGn',
               title = "Heatmap of ESRB Rating vs Consoles updated to 2016")

# Update layout
gesrb.update_layout(layout)
gesrb.update_annotations(sign)
gesrb.show()
```
<p align="center">
{{< image src="https://raw.githubusercontent.com/robguilarr/videogames-eda/main/videogame_analysis/plots-images/heatmap9.png?format=webpage" width="75%" height= "75%" alt="Heatmap ESRB">}}
</p>

Based on the classification established by the ESRB, from the data available it can be concluded that the video game console with more titles for universal use is the Nintendo DS, followed by the PS2 and then is the Wii, thus highlighting the impact they had on sales, which it will be shown later.

Meanwhile, the Xbox 360 and PS3 were geared towards a more mature audience with a significant presence of M-rated titles.

### Last years of 32 bit era

In the early 2000s, after the launch of the PS1, Sony continued leading the console market and diversifying its portfolio of games. On the other side of the coin SEGA, despite having launched the first console with an online system, in 2002 they retired their console from the market and dedicated itself exclusively to third-party development and Arcade, a situation that is outlined in the following graph.

```python
# Lineplot sales by platform before 2005
# Extract columns
games20 = df[["Year_of_Release","Platform","Global_Sales"]]

# Subset dates
games20 = games20[(games20.Year_of_Release > 1998) & (games20.Year_of_Release < 2005)]

# Omit WonderSwan by Bandai, SEGA Saturn due low sales profiles and NDS that is not
# relevant yet
games20 = games20[~games20.Platform.isin(["WS","DS","SAT","SNES","PSP"])]

# Group and summarize
games20 = games20.groupby(["Year_of_Release","Platform"]).agg(sum).reset_index()\
    .sort_values(["Year_of_Release","Platform"], ascending=True)

# Save an Array of Platforms
Platforms = games20.Platform.unique()

# Pivot to put in long format
games20 = games20.pivot_table(values="Global_Sales", index="Year_of_Release",
                              columns="Platform").reset_index()

# Assemble lineplot
line20 = go.Figure()
for platform in Platforms:
    line20.add_trace(go.Scatter(x = games20["Year_of_Release"], y = games20[platform],
                                name=platform, line_shape='linear'))

# Update layout
line20.update_layout(layout)
line20.update_annotations(sign)
line20.show()
```
<p align="center">
{{< image src="https://raw.githubusercontent.com/robguilarr/videogames-eda/main/videogame_analysis/plots-images/lineplot_10.png?format=webpage" width="75%" height= "75%" alt="Lineplot of 2000s">}}
</p>

The Japanese domain was becoming more and more determined, a situation that the software technology giant, Microsoft, takes as a challenge to enter a new market and start a contest with the PS2.

<blockquote><b>2000</b> - The beginning of the longest rivalry in the console market</blockquote>

<p align= center>
    <img src="https://i.redd.it/ahya0h6b08941.jpg?format=webpage" width="600" height="400" alt="billandrock">
</p>

This famous image of Bill Gates with Dwayne Johnson was part of a great marketing strategy carried out by Microsoft, they were willing to do whatever it took to strengthen the presence of Xbox in the market.

Microsoft's vision was to standardize the game <a href="https://en.wikipedia.org/wiki/Xbox_technical_specifications">hardware</a> so that it was as similar as possible to a PC, so they implemented Direct X, an Intel Pentium III, a 7.3GFLOPS Nvidia GPU and an 8GB hard drive, trying to secure a significant advantage over the competitors.

At this time, Nintendo announced the GameCube as a console contender, but it was not very successful, a situation that was neutralized with the sales of the Game Boy Advance within the portable market.

Nevertheless, the PS2 led the first part of the decade in terms of sales, while Xbox got the second place as we saw in the last graph. And of course, that was a very expensive silver medal, according to Vladimir Cole from <i>Joystiq</i>, Forbes estimated around $4 billion in total lost after that trip, but at the same time they proved that they could compete with the <i>Japanese Ruler</i> of that time.

```python
# Mask of 2000-2004 games
games2004 = (df.Year_of_Release > 1999) & (df.Year_of_Release < 2005)

# Array to Subset publishers with hightest sales from 2000 to 2004
toppub2004 = df[games2004].groupby\
    (["Publisher"])["Global_Sales"].agg(sum).reset_index()\
    .sort_values("Global_Sales",ascending=False).head(15)

# New DF with top  Titles per Publisher
toppub = df[games2004 & df.Publisher.isin(toppub2004.Publisher)]\
    .sort_values(["Publisher","Name"])

# Substitute empty scores with the mean
toppub.Critic_Score = toppub.Critic_Score.fillna(toppub.Critic_Score.mean())

# Top 3
toppub3 = toppub.sort_values(["Publisher","Global_Sales"], ascending = False)\
    .groupby("Publisher")["Name","Year_of_Release","Publisher","Global_Sales",
                          "Critic_Score", "Country","City"].head(3)\
    .sort_values("Global_Sales", ascending=True)

# Bubble plot
bubpub3 = px.scatter(toppub3, y="Publisher", x="Global_Sales", size="Critic_Score",
                 color="Critic_Score", hover_name="Name",
                 color_continuous_scale=px.colors.sequential.Greens,
                 labels={"Global_Sales":"Millions of units sold",
                         "Critic_Score":"Metacritic value"})

# Add reference line
bubpub3.add_vrect(x0 = 8.0, x1 = 8.98, y0= 0.32, y1=0.44, line_width=0,
                  fillcolor="#fff152", opacity=0.5)

# Master Chief image
bubpub3.add_layout_image(
    dict(
        source="https://media.fortniteapi.io/images/7bf522a34af664a172ce581441985e75/featured.png",
        xref="paper", yref="paper",
        x=1, y=0.021,
        sizex=0.4, sizey=0.4,
        xanchor="right", yanchor="bottom") )

# Update layout
bubpub3.update_layout(layout)
bubpub3.update_annotations(sign)
bubpub3.show()
```
<p align="center">
{{< image src="https://raw.githubusercontent.com/robguilarr/videogames-eda/main/videogame_analysis/plots-images/scatterplot_11.png" width="75%" height= "75%" alt="Bubblechart of 2000s">}}
</p>

In the graph we see that Microsoft, despite not becoming leaders in sales, were positioned by having very good reviews, specifically in Halo, with Metacritics above 95, including its title's sequels.

However, Microsoft's step did not go unnoticed, the launch of <i>Halo: Combat Evolved</i> marked a before and after in the world of online multiplayer FPS, not only because of its online gaming capabilities or because of its smooth joystick mechanism, which was crucial for the attraction of PC FPS players to consoles, but for his amazing character, <i>Master Chief</i> who became an emblem of the brand.

### The "Non-competitor" takes the lead

<blockquote><b>2005</b> - Microsoft launch the Xbox 360</blockquote>
<blockquote><b>2006</b> - PS3 and Nintendo Wii are released</blockquote>

<p align= center>
    <img src="https://images.nintendolife.com/a2c42676da2b5/1280x720.jpg" width="600" height="400" alt="wii">
</p>

As Microsoft and Sony continued competing for a market with high-definition titles, online connection services like Xbox Live and PSN, and high-capacity hard drives, Nintendo chose to follow a <a href="https://www.blueoceanstrategy.com/what-is-blue-ocean-strategy/">Blue Ocean Strategy</a> after the failure of the GameCube, who tried to compete in the big leagues.

Their strategy consisted of offering something new and innovative, instead of competing to be better in the characteristics offered by the competition, becoming the fastest selling console, reaching to sell 50 million units around the world according to D. Melanson from <a href="https://www.engadget.com 2009-06-12-nintendo-wii-sets-record-as-fastest-selling-console-in-the-us.html">Verizon Media</a>, so this is the best way to describe the Wii console.

```python
# Mask of 2005-2010 games
games2010 = (df.Year_of_Release > 2004) & (df.Year_of_Release < 2011)

# Dataframe of games
df2010 = df[games2010]

# Excluding data to focus on new consoles
df2010 = df2010[df2010.Platform.isin(['Wii','DS','X360','PS3'])]\
        .groupby(["Platform","Year_str"])["Global_Sales"].agg(sum).reset_index()\
        .sort_values(["Year_str","Global_Sales"])

# Plot of Sales by Platform
bar2010 = px.bar(df2010, color="Platform", y="Global_Sales", x="Year_str",
             barmode="group", labels={"Year_str":"Year",
                                      "Global_Sales":"Millions of Units"},
             pattern_shape="Platform", pattern_shape_sequence=["", "", "", "", "."],
             color_discrete_sequence=["#00DEB7","#0082C2","#1333A7","#5849B6"])

# Update layout
bar2010.update_layout(layout)
bar2010.update_annotations(sign)
bar2010.show()
```
<p align="center">
{{< image src="https://raw.githubusercontent.com/robguilarr/videogames-eda/main/videogame_analysis/plots-images/barplot_12.png?format=webpage" width="75%" height= "75%" alt="Barchart of 2000s">}}
</p>

As you can see, from the start of the Wii sales, the strategy of Nintendo began to flourish, surpassing the sales of its rivals by 4 years in a row.

An interesting aspect of Nintendo among the others, is that the success of their sales was due to exclusive titles involving their unique accessories with motion sensors.

Referring to sales, among the most successful titles are the following.

```python
# Dataframe for table with best-selling games
table_data = df[games2010]
table_data = table_data[table_data.Platform.isin(['Wii','DS','X360','PS3'])]\
                .sort_values(["Year_str","Platform","Global_Sales"], ascending=False)\
                .reset_index()\
                .groupby(["Platform","Year_str"]).head(1)\
                .sort_values("Platform", ascending = False)

table_data = table_data[["Year_str","Name","Publisher","Platform","Global_Sales"]]


# Plot of Table
table10 = go.Figure(data=[go.Table(
    header=dict(values=list(["Year","Game title", "Platform",
                             "Publisher","Units Sold"]),
                fill_color='#5849B6',
                align="center"),
    cells=dict(values=[table_data.Year_str, table_data.Name, table_data.Platform,
                       table_data.Publisher, np.round(table_data.Global_Sales *
                                                      1000000,0)],
               fill_color='lavender',
               align=list(['center', 'left', 'center', 'left', 'right'])))])

# Update layout
table10.update_layout(layout)
table10.update_traces({"header":{"font.color":"#fcfcfc",
                                 "font.size":fontimg+3}})
table10.update_annotations(sign)
table10.show()
```
<p align="center">
{{< image src="https://raw.githubusercontent.com/robguilarr/videogames-eda/main/videogame_analysis/plots-images/table_13.png?format=webpage" width="75%" height= "75%" alt="Table of 2000s">}}
</p>

Four of Wii's five most successful titles involve Nintendo Publishers, among its most famous IPs were Mario Kart and Wii Sports.

This innovation marked an era of hardware extensions and motion sensors, a situation that Activision was able to take advantage of, when acquiring <a href="https://www.gamesindustry.biz/articles/sec-filing-shows-activision-paid-100m-for-redoctane">Red Octane</a>, reaching around 13 titles of the IP known as Guitar Hero until 2009, being sold with its flagship item that imitated a Gibson SG.

### Prevalence of Social Gaming

After the success of some local-gaming titles, the decade from 2010 to 2020 took a more competitive or cooperative way in certain cases, guided by a new era of interconnectivity and mobility.

This reason motivated developers with extraordinary visions to create not only multiplayer, but also online content that maintains high audience rates.

```python
# Subset games from 2010 to 2015
games2010 = df[(df.Year_of_Release > 2009) & (df.Year_of_Release < 2016)]\
                .sort_values(["Year_str","Platform","Global_Sales"])

# Subset games with more sales from 2010 to 2015
topgames2010 = games2010.sort_values(["Genre","Global_Sales"], ascending = False)\
      .groupby("Genre").head(1).sort_values("Year_of_Release", ascending = True)\
        .sort_values("Global_Sales", ascending=False)

topgames2010 = topgames2010[["Year_of_Release","Name","Platform","Publisher","Genre",
                             "Global_Sales"]]

# Barplot Base
bargen10 = px.bar(topgames2010, y="Genre", x="Global_Sales", orientation="h",
                  text="Name", labels={"Name":"Title",
                                       "Global_Sales":"Millions of units sold"},
                  color="Genre",
                  color_discrete_sequence=["#8B58B0","#58B081","#B0B058","#535353",
                                           "#B05858","#58B09E","#B05890","#587FB0",
                                           "#B05858","#58B0AA","#686868","#C3A149"])
bargen10.update_traces(textposition='inside',
                       marker_line_color='#404040',
                       textfont = {"color":"#FFFFFF","family": "segoe ui"},
                        marker_line_width=1, opacity=0.7)

# Update layout
bargen10.update_layout(layout)
bargen10.update_annotations(sign)
bargen10.show()
```
<p align="center">
{{< image src="https://raw.githubusercontent.com/robguilarr/videogames-eda/main/videogame_analysis/plots-images/barplot_14.png?format=webpage" width="75%" height= "75%" alt="Barchart of 2010s">}}
</p>

Between 2010 and 2015, the best-selling title was Kinect Adventures for Xbox 360, which had a focus on enhancing multiplayer gameplay and taking advantage of the latest technological innovation of the moment, the Microsoft’s Kinect.

The second best-selling title at that time was Grand Theft Auto V for PS3, which to this day continues to prevail as one of the online systems with the largest number of users in the industry. Their vision went beyond creating an Open-World game, they had the intention of creating a dynamic online content structure, which provides seasonal content.

This type of model also motivated Publishers such as Epic Games and Activision, to innovate but in this case not selling games but focusing on aesthetics, where game content is offered as an extra to the online service without having to be paid as a <a href="https://en.wikipedia.org/wiki/Downloadable_content">DLC</a>.

```python
#pubgen
# Publishers with more sales in history
toppubarray = df.groupby("Publisher")["Global_Sales"].agg(sum).reset_index()\
                .sort_values("Global_Sales", ascending= False)\
                .head(len(df.Genre.unique()))["Publisher"]

# Extract publisher from raw df
puball = df[df.Publisher.isin(toppubarray)].groupby(["Publisher","Name","Genre"]).agg(sum)

# Add a column of 1s
puball["counter"] = np.ones(puball.shape[0])

# Create the pivot table
puball = puball.pivot_table("counter", index = "Publisher", columns="Genre",
                            aggfunc="sum")
# Display rounded values
pd.options.display.float_format = '{:,.0f}'.format


pubmatrix = ff.create_annotated_heatmap(puball.values, x=puball.columns.tolist(),
                                  y=puball.index.tolist(),
                                  annotation_text= np.around(puball.values, decimals=0),
                                  colorscale='sunset')

# Update layout
pubmatrix.update_layout(layout)

# Extra annotation to avoid overlapping of layers
pubmatrix.add_annotation(text=author,
                        align= "right", visible = True, xref="paper", yref="paper",
                         x= 1, y= -0.11, showarrow=False, font={"size":fontimg-1})
pubmatrix.update_annotations(sign)
pubmatrix.show()
```
<p align="center">
{{< image src="https://raw.githubusercontent.com/robguilarr/videogames-eda/main/videogame_analysis/plots-images/matrix_15.png?format=webpage" width="75%" height= "75%" alt="Historical Matrix">}}
</p>

The fact that every day more <i>Free to Play</i> games are announced, does not indicate that this is the exclusive focus companies will have on the industry now on. Beyond this, as we see in the previous graph, each of the most recognized Publishers in history has its own style in exclusive series, despite having titles in many genres.

Even today, large companies like Microsoft offer services such as Xbox GamePass, with subscriptions that offer big catalogs of games, which even include titles from Independent Developers, supporting their growth through advertising systems, helping to increasingly expand a growing industry.

---

## Summary

The video game industry, beyond having started as a simple experiment at MIT, is a lifestyle for many. Like any market, it has had its moments of glory and its difficulties, but if we can rescue something, it is that the secret of its success lies in the emotional bond it generates with its customers.

Through this article, my goal is to use data tools to inform the reader about curious events, which perhaps they did not know. I want to thank you for taking the time to read carefully. As a curious detail, there are no **easter eggs** :wink: Good luck!

---

# Additional Information

- <b>About the article</b>

  This infographic article was adapted to a general public, I hope you enjoyed it. In case you are interested in learning more about the development of the script, I invite you to the contact section in my <b>About Page</b>, and with pleasure we can connect.


- <b>Related Content</b>

  As a recommendation I suggest a look at <a href="https://www.visualcapitalist.com/50-years-gaming-history-revenue-stream/">this</a> great article, published by
  Omri Wallach on the Visual Capitalist infographic page, where many interesting
  details about the industry's history are covered.

- <b>Datasets</b>

  - <a href="https://www.kaggle.com/sidtwr/videogames-sales-dataset?select=Video_Games_Sales_as_at_22_Dec_2016.csv">Videogames Dataset</a>
  - <a href="https://www.kaggle.com/andreshg/videogamescompaniesregions?select=video-games-developers.csv">Videogames Developers Regions</a>
  - <a href="https://www.kaggle.com/andreshg/videogamescompaniesregions?select=indie-games-developers.csv">Indie Developers Regions</a>

  [^1]:
      <b>Footnote:</b> Specific datasets contain information from Publishers, which they were
      named in the source attribute as Developers, but not in all cases. For more details on the
      data transformation, please visit my Github <a href="https://github.com/robguilarr/videogames-eda/blob/main/videogame_analysis/ETL_script.py">repository</a>.

