# Player Retention and Cohorts Creation Pipeline in Brawl Stars with Non-Supervised Models


Brawl Stars is one of the action-packed multiplayer gaming experiences offered by Supercell, the Finnish mobile game development company known for hits like Clash of Clans and Clash Royale. This company has consistently churned out hit after hit, racking up billions of downloads and earning a reputation as one of the most innovative and successful mobile game studios in the world. Part of their success is due to their solid data practices.

Retention metrics tracking is crucial for the long-term sustainability of mobile games like Brawl Stars, particularly during soft launches. These metrics are used to measure how many players continue to play the game after initial installation and engagement.

By analyzing these metrics, game developers can identify areas where players may be dropping off and make changes to improve the user experience and keep players engaged over time. This is essential for the success of a mobile game, as high retention rates lead to more in-app purchases and longer overall player lifetimes. Given the competitive landscape of mobile gaming, tracking retention metrics is a key component in keeping a game relevant and profitable over the long term.

---

<p align='center'> :(fas fa-exclamation-circle fa-fw): <b>Important: Any views, material or statements expressed are mines and not those of my employer</b> </p>

---

{{< admonition info "Looking for an interactive experience?" true >}}

:rocket: Access the Kedro Pipeline Visualization, available <a href="https://brawlstars-retention-pipeline-6u27jcczha-uw.a.run.app/">here</a>; or you can download the <a href="https://github.com/robguilarr/Brawlstars-retention-pipeline">source code</a> from GitHub

{{< /admonition >}}

{{< youtube CaryjOdYFa0 >}}

---

## ⚠️ Introduction to the problem

### Hypothesis

The analytics team of a mobile gaming developer is facing a problem while producing bounded retention metrics for player cohorts based on the game modes offered. They need a fully parameterized way to produce these metrics, instead of hardcoded methods per iteration.

Currently, the analytics team mines data batches of over 20,000 players in a single run to create unbounded retention metrics, and the job is processed every 56 days. However, the team need to track user retention within a given time frame and qualify player preferences based on the game modes offered and their game-experience.

Also, they require a model that can define cohorts based on the installation date or any other classification parameter and includes a feature store to keep track of the features used. Additionally, they would like to store the hyperparameters, evaluation scores, and estimators used for each experiment on a Cloud Service to reuse specific models later.

- $H_0:$ The pipeline is sufficient to produce retention metrics for player cohorts based on the game modes offered, and there is no need for a fully parameterized tool to create these metrics with unbounded approaches.
- $H_1:$ The current pipeline is not sufficient, and a fully parameterized tool is required to track user retention within a given time frame and qualify player preferences based on the game modes offered. Additionally, a Cloud Service-based feature store and model registry are needed to keep track of experiment versions.

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/robguilar-website/main/content/posts/brawlstars_retention_pipeline/images/asset_01.png" style="width: 1000px"></p>

### Potential Stakeholders

- Designers: Game Designers are interested in user retention metrics to understand how users are interacting with the game and to identify any areas that need to be improved to keep users engaged and coming back to the game.
- Product / Project Managers: Responsible for defining the goals and objectives for the game and ensuring that those goals are met, they need to follow user retention metrics to understand how well the game is performing in terms of retaining users and whether any changes need to be made to the game to improve retention.
- Marketer: Understand how effective their marketing campaigns are in “keeping users engaged” with the game, and whether any changes need to be made to the campaigns to improve retention.
- Programmers: Interested in identifying any technical issues that may be causing users to leave the game and make improvements to the game to improve retention.
- Publisher: Need to understand the financial performance of the game and whether it is generating sufficient revenue to justify the investment. They are also interested in understanding the potential for future growth and the impact of retention on the game's long-term sustainability.

**Note:** To facilitate the understanding of the roles of the development team, I invite you to take a look at **[this](https://www.robguilar.com/posts/gamedev_structure/)** diagram that I designed.

---

## 📥 About the data

### Source data

The [Brawl Stars Public API](https://developer.brawlstars.com/#/) is the source of the data, which is accessible to anyone and completely free to use. By entering an identifier in the request function, the API gathers data on the player associated with that identifier, also known as the "Player Tag".

Each user account has a unique player tag, which I collected from a sample shared by [@SirWerto](https://www.kaggle.com/datasets/sirwerto/brawlstars-players-tags) on Kaggle. To supplement our tag list, also utilized the `.ranking()` function of [Brawlstats](https://brawlstats.readthedocs.io/en/latest/api.html), an open-source Python library. The dataset contains a total of 23841 Player Tags, which are saved in a `.txt` file and stored in a Google Cloud Bucket folder labeled as `/01_player_tags`.

### Data Catalog structure

Versioning plays a vital role in maintaining organized and traceable files. To accomplish this, we utilize a hierarchical storage approach, incorporating a timestamp in EPOCH format as a reference point. In order to leverage this functionality, we will employ the Kedro [versioning](https://docs.kedro.org/en/stable/data/data_catalog.html#version-datasets-and-ml-models) feature.

To ensure a comprehensive record of models, estimators, and features used in each run, the catalog offers eight distinct storage endpoints, including `/06_viz_data`, `/07_feature_store`, and `/08_model_registry`, all of which are versioned. By using versioning to manage and maintain these files, teams can easily track data and model changes, to ensure that each iteration is saved for future reference.

```yaml
gcp:
    # ----- Bucket Parent Locations -----
    player_tags: gs://animus-memory-bucket/brawlstars/01_player_tags
    raw_battlelogs: gs://animus-memory-bucket/brawlstars/02_raw_battlelogs
    raw_metadata: gs://animus-memory-bucket/brawlstars/03_raw_metadata
    enriched_data: gs://animus-memory-bucket/brawlstars/04_enriched_data
    curated_data: gs://animus-memory-bucket/brawlstars/05_curated_data
    viz_data: gs://animus-memory-bucket/brawlstars/06_viz_data
    feature_store: gs://animus-memory-bucket/brawlstars/07_feature_store
    model_registry: gs://animus-memory-bucket/brawlstars/08_model_registry

    # ----- Main GCP Project ID -----
    project_id: abstergo-corp-id
```

To easily locate the endpoint of each GCP Bucket output, simply refer to the [conf/gcp/catalog.yml](https://github.com/robguilarr/Brawlstars-retention-pipeline/blob/main/conf/gcp/catalog.yml) file. This file defines each destination using Dataset Groups, as shown in the following example:

```yaml
_pyspark: &pyspark
  type: spark.SparkDataSet
  file_format: parquet
  load_args:
    header: true
  save_args:
    mode: overwrite
    sep: ','
    header: True

_pandas: &pandas
  type: pandas.CSVDataSet
  load_args:
    sep: ","
  save_args:
    index: False
  fs_args:
    project: ${gcp.project_id}

cohort_activity_data@pyspark:
  <<: *pyspark
  filepath: ${gcp.curated_data}/cohort_activity_data.parquet
  layer: "primary"

player_metadata@pandas:
  <<: *pandas
  filepath: ${gcp.raw_metadata}/player_metadata.csv
  layer: "raw"
```

The outputs `cohort_activity_data@pyspark` and `player_metadata@pandas` are specifically mapped to the `_pyspark` and `_pandas` dataset groups, respectively. Moreover, we can assign Layers to these outputs, providing users with a clear understanding of their utility as intermediate data sources, for analysis, or as input for machine learning models.

Seven layers were defined in the catalog: raw, primary, model input, feature, models, model output, and reporting. To go deeper into the Layer definitions and best practices, I suggest reading the following [article](https://towardsdatascience.com/the-importance-of-layered-thinking-in-data-engineering-a09f685edc71) by Joel Schwarzmann (2021).

---

## ⚡Pipelines workflow

The pipeline architecture consists of five modular pipelines, with a total of 14 nodes. These pipelines will be constructed utilizing the open-source framework, [Kedro](https://kedro.org/).

- Two of these pipelines, `battlelogs_request_preprocess` and `metadata_request_preprocess`, will use asynchronous coroutines to request data from the API and Spark for preprocessing. The requested data will be processed in batches of over 400 MB using only Spark to avoid communication overhead while minimizing the number of actions taken to take advantage of the worker's process fluency.
- The `events_activity_segmentation` pipeline will use Spark SQL API to produce retention metrics. This pipeline will use the data gathered from the requesting nodes. It is important to consider the data dependency between functions, which is why multithreading won't be used.
- Another pipeline, the `player_cohorts_classifier`, will perform a multiprocessing step over a Grid Search and cross-validation across multiple cores to evaluate the best quantity of clusters in which we can classify the players based on the game modes’ behaviors. This depends on the distortion rate and the hyperparameters defined by the user.
- The last modular pipeline, `player_activity_clustering_merge`, will use the curated data to produce retention plots. It will use a single Python processing since it uses filtered data, which is a smaller subset.

This pipeline was designed to speed up processing by parallelizing. We can calculate the theoretical speedup by applying [Amdahl's Law](https://en.wikipedia.org/wiki/Amdahl%27s_law).

$$
S = \frac{1}{1-P+\frac{P}{N}} = \frac{1}{1-0.64286+\frac{0.64286}{10}} = 2.52
$$

The theoretical speedup will be approximately 2.52 for a single-machine cluster. Here, *S* represents theoretical speedup, *P* represents the fraction of the workload that can be parallelized, and *N* represents the number of processors.

If you want to learn more about the appropriate use cases for multithreading and multiprocessing, I highly recommend watching [this conference video](https://www.youtube.com/watch?v=w2eUdxPQQ78) by Chin Hwee Ong (Pycon Taiwan 2020). It's one of the best resources to get started.

---

### Battlelogs and Metadata Request Pipelines

**Pipeline `battlelogs_request_preprocess` and `metadata_request_preprocess`**

Both pipelines follow the same structure, the only difference is the request function used to gather the data from the API, one uses `.get_battle_logs()`, and the other uses the `.get_player()` function. And from these functions come two different data models [battle logs](https://brawlstats.readthedocs.io/en/latest/api.html#battle-logs) and [players](https://brawlstats.readthedocs.io/en/latest/api.html#player).

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/robguilar-website/main/content/posts/brawlstars_retention_pipeline/images/asset_02.png" style="width: 1000px"></p>

**Node: `battlelogs_request_node` and `player_metadata_request_node`**

The inconsistent batch sizes of our data make it unsuitable to rely on an `O(n)` frequency expectation, due factors such as:

- Product cycle (e.g., new feature or season launch).
- Player longevity can affect the data (e.g., newer players with less than 25 sessions).

To extract the logs, I opted to use low-level APIs, specifically Python's asynchronous coroutines. This approach allows executing one batch while waiting for a response from another, which is particularly helpful when dealing with variable batch sizes or waiting for processing, such as querying from a SQL database. However, the [API wrapper](https://brawlstats.readthedocs.io/en/latest/_modules/brawlstats/core.html#Client.get_battle_logs) used limits the multithreading pool size to 10, which means we had to separate the tags into batches of that size.

The main advantage of this approach is that it makes the code independent of the processor's clock speed, avoiding waiting times between requests. Although Python's [Global Interpreter Lock (GIL)](https://realpython.com/python-gil/) restricts running multiple threads simultaneously, we overcame this limitation by using coroutines to process future objects. If the pool size weren't limited, we could have used multithreading instead. Overall, this strategy allowed us to process our data more efficiently and achieve better results.

```python
def battlelogs_request(
    player_tags_txt: str, parameters: Dict[str, Any]
) -> pd.DataFrame:
    """
    Extracts battlelogs from Brawlstars API by asynchronously executing a list of
    futures objects, each of which is made up of an Async thread task due to blocking
    call limitations of the api_request submodule.
    Args:
        player_tags_txt: A list of player tags.
        parameters: Additional parameters to be included in the API request.
    Returns:
         A structured DataFrame that concatenates all the battlelogs of the players.
    """
    # Get key and validate it exists
    API_KEY = conf_credentials.get("brawlstars_api", None).get("API_KEY", None)
    try:
        assert API_KEY is not None
    except AssertionError:
        log.info(
            "No API key has been defined. Request one at https://developer.brawlstars.com/"
        )

    # Create a BrawlStats client object to interact with the API. Note: the
    # "prevent_ratelimit" function in the source code can be used this behavior
    client = brawlstats.Client(token=API_KEY)

    # Split the player tags text by commas to create a list of player tags.
    player_tags_txt = player_tags_txt.split(",")

    def api_request(tag: str) -> pd.DataFrame:
        """Requests and structures the 25 most recent battle logs from the Brawl
        Stars API for a given player."""
        try:
            # Retrieve raw battle logs data from API
            player_battle_logs = client.get_battle_logs(tag).raw_data
            # Normalize data into structured format
            player_battle_logs_structured = pd.json_normalize(player_battle_logs)
            # Add player ID to DataFrame
            player_battle_logs_structured["player_id"] = tag
        except:
            log.info(f"No battle logs extracted for player {tag}")
            player_battle_logs_structured = pd.DataFrame()
            pass
        return player_battle_logs_structured

    async def api_request_async(tag: str) -> pd.DataFrame:
        """
        Transform non-sync request function to async coroutine, which creates
        a future object by API request.
        The Coroutine contains a blocking call that won't return a log until it's
        complete. So, to run concurrently, await the thread and not the coroutine by
        using this method.
        """
        return await asyncio.to_thread(api_request, tag)

    async def spawn_request(player_tags_txt: list) -> pd.DataFrame:
        """
        This asynchronous function takes a list of player tags as input and creates a
        list of coroutines that request the player's battlelogs data. It then
        schedules their execution and waits for them to complete by using the
        asyncio.gather() method. Finally, it concatenates all the dataframes returned by
        the coroutines into one dataframe.
        """
        start = time.time()
        log.info(f"Battlelogs request process started")
        # Create a list of coroutines and schedule their execution
        requests_tasks = [
            asyncio.create_task(api_request_async(tag)) for tag in player_tags_txt
        ]
        # Wait for all tasks to complete and gather their results
        battlelogs_data_list = await asyncio.gather(*requests_tasks)
        # Concatenate all the dataframes returned by the coroutines into one
        raw_battlelogs = pd.concat(battlelogs_data_list, ignore_index=True)
        log.info(
            f"Battlelogs request process finished in {time.time() - start} seconds"
        )
        return raw_battlelogs

    def activate_request(n: int = None) -> pd.DataFrame:
        """
        This function checks if a sampling limit is provided, and if so, it calls the
        spawn_request() function to request the battlelogs data of the first n
        players. If n is not provided, it splits the player tags list into batches of 10
        and calls the spawn_request() function for each batch. It then concatenates
        the dataframes returned by each call into one.
        """
        raw_battlelogs = pd.DataFrame()
        if n:
            # Call spawn_request() for the first n players
            raw_battlelogs = asyncio.run(spawn_request(player_tags_txt[:n]))
        else:
            # Split the tags list into batches of 10 and call spawn_request() for each
            split_tags = np.array_split(player_tags_txt, len(player_tags_txt) / 10)
            for batch in split_tags:
                raw_battlelogs_tmp = asyncio.run(spawn_request(batch))
                # Concatenate the dataframes returned by each call into one
                try:
                    raw_battlelogs = pd.concat(
                        [raw_battlelogs, raw_battlelogs_tmp], axis=0, ignore_index=True
                    )
                except:
                    pass

        return raw_battlelogs

    # Call activate_request() with the specified battlelogs_limit parameter
    raw_battlelogs = activate_request(n=parameters["battlelogs_limit"])

    # Replace dots in column names with underscores
    raw_battlelogs.columns = [
        col_name.replace(".", "_") for col_name in raw_battlelogs.columns
    ]

    # Check if the data request was successful
    try:
        assert not raw_battlelogs.empty
    except AssertionError:
        # Log an error message if no Battlelogs were extracted
        log.info("No Battlelogs were extracted. Please check your Client Connection")

    return raw_battlelogs
```

Here was collected a substantial amount of battle logs and player metadata, amounting to 476 MB and 453 MB, respectively. To analyze the process efficiency, was compared a synchronous process using a loop with an asynchronous process. The results were benchmarked:

- The asynchronous process completed the task in just 287.93 seconds.
- The synchronous process took 1004.54 seconds.

This translates to a 71% improvement in speed for the logs request, demonstrating the effectiveness of the new approach. Upon further analysis, out of the 23841 Player Tags, only 9104 had a battle log registry. Nevertheless, we managed to retrieve player metadata for 22738 of them, which is a good number considering we were using a public non-official API.

**Node: `battlelogs_filtering_node` and `metadata_preparation_node`**

The node serves two main purposes:

- Transform retrieved data into a Spark data frame, ensuring accurate and consistent data with the expected data types.
- Enable users to specify time ranges for player cohorts. This feature empowers users to customize the quantity and number of cohorts needed for their specific study.

For instance, in the following example, the cohorts are divided into 8 time-ranges. Events with missing identifiers are automatically excluded. This function assumes that the user requires a model capable of defining cohorts based on factors such as installation date or any other classification parameter.

```python
# ----- Parameters for preprocessing and filtering ----
battlelogs_filter:
  exclude_missing_events: true
  cohort_time_range:
    - ("2019-10-11","2020-04-10")
    - ("2020-04-11","2020-10-10")
    - ("2020-10-11","2021-04-10")
    - ("2021-04-11","2021-10-10")
    - ("2021-10-11","2022-04-10")
    - ("2022-04-11","2022-10-10")
    - ("2022-10-11","2023-04-10")
    - ("2023-04-11","2023-10-10")
  raw_battlelogs_schema:
    - "battleTime STRING, 
       event_id BIGINT, 
       event_mode STRING,
       event_map STRING,
       battle_mode STRING,
       battle_type STRING,
       battle_result STRING,
       battle_duration BIGINT,
       battle_trophyChange BIGINT,
       battle_starPlayer_tag STRING,
       battle_starPlayer_name STRING,
       battle_starPlayer_brawler_id BIGINT,
       battle_starPlayer_brawler_name STRING,
       battle_starPlayer_brawler_power BIGINT,
       battle_starPlayer_brawler_trophies BIGINT,
       battle_teams STRING,
       battle_rank BIGINT,
       battle_players STRING,
       player_id STRING,
       battle_starPlayer STRING,
       battle_bigBrawler_tag STRING,
       battle_bigBrawler_name STRING,
       battle_bigBrawler_brawler_id BIGINT,
       battle_bigBrawler_brawler_name STRING,
       battle_bigBrawler_brawler_power BIGINT,
       battle_bigBrawler_brawler_trophies BIGINT,
       battle_level_name STRING,
       battle_level_id BIGINT"
```

---

### Events and Activity Segmentation Pipeline

**Pipeline `events_activity_segmentation`**

Our goal is to transform raw data into valuable insights. To achieve this, we will focus on processing and filtering battle logs to produce “activity” data. This includes segmentations per event type, retention metrics, and ratios based on retention. With these metrics, we can gain a deeper understanding of user behavior and make informed decisions that drive growth.

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/robguilar-website/main/content/posts/brawlstars_retention_pipeline/images/asset_03.png" style="width: 1000px"></p>

**Node: `battlelogs_deconstructor_node`**

To understand the basic game modes of this mobile game, an extensive research was required. All assumptions made in the event deconstruction process are based on the information provided by the [Brawl Stars Fandom Wiki](https://brawlstars.fandom.com/wiki/Category:Events).

The deconstruction process will classify the `battle_teams` column and divide all players in each session into groups based on the number of team members defined by the event type. These event types are categorized according to specific parameters, enabling users to easily classify events based on the distribution of players' teams per session.

```yaml
# ----- Parameters to deconstruct the data by event ----
battlelogs_deconstructor:
  event_solo: ['soloShowdown']
  event_duo: ['duoShowdown']
  event_3v3: ['gemGrab','brawlBall','bounty','heist','hotZone','knockout']
  event_special: ['roboRumble','bossFight','lastStand', 'bigGame']
  standard_columns:
    - 'battlelog_id'
    - 'cohort'
    - 'battleTime'
    - 'player_id'
    - 'event_id'
    - 'event_mode'
    - 'event_map'
    - 'is_starPlayer'
    - 'battle_type'
    - 'battle_result'
    - 'battle_rank'
    - 'battle_duration'
    - 'battle_trophyChange'
    - 'battle_level_name'
```

The use of parameters to segment the raw battle logs data is interpreted as follows:

- The `battle_teams` column in the JSON files contains a massive collection of data. When the battle type is in the list `['gemGrab','brawlBall','bounty','heist','hotZone','knockout']`, the code recognizes this as 3 vs 3 events, meaning that the battle teams column contains information on all six players.

To create the `players_collection` and `brawlers_collection` data, the six players are divided into two groups. This division results in two collections that provide valuable insights into player behavior and [brawler](https://brawlstars.fandom.com/wiki/Category:Brawlers) selection.

```
players_collection >> [[#2882QV9J0, #28Y9GJC0L, #2UPL0QCYG], [#U08LRGYJ, #2QQ9GY2YL, #22UJ88YQ9]]

brawlers_collection >> [[CROW, MORTIS, BUZZ], [MORTIS, SURGE, DYNAMIKE]]
```

By conducting an interaction study between players and the brawlers or characters they use, a data scientist can extract valuable insights into gameplay. For instance, if you're curious about how the exploder function works, you'll find this code particularly useful. Its high density is evidence of Spark high performance, which has enabled the minimization of Spark actions performed. However, you can satisfy your curiosity [here](https://github.com/robguilarr/Brawlstars-retention-pipeline/blob/main/src/brawlstars_etl/pipelines/events_activity_segmentation/nodes.py).

**Node: `activity_transformer_node`**

Imagine having the capacity to transform filtered battle logs activity into a wrapped format data frame that delivers retention metrics and n-sessions at the player level of granularity. That's exactly what this node does. Using a set of customizable parameters, it extracts the retention and number of sessions based on a `cohort frequency` parameter:

- When you select `daily` granularity, the cohort period is based on the day of the user's first event, showing you the number of sessions logged per user on the day they installed the game, on the day they returned, and so on.
- If you choose `weekly` granularity, the cohort period is based on the first consecutive Monday on or after the day of the user's first event. This allows you to track user behavior on a weekly basis, providing valuable insights into how engagement fluctuates throughout the week.
- And if you opt for `monthly` granularity, the cohort period is based on the first day of the month in which the user's first event occurred. This gives you an overview of user behavior on a monthly basis, perfect for understanding trends over time.

By default, this node is set to `daily`, making it easy to get started and quickly visualize your data. With its intuitive and powerful features, you'll gain deeper insights into your users' behavior, and be able to optimize your game for even greater success.

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/robguilar-website/main/content/posts/brawlstars_retention_pipeline/images/asset_04.png" style="width: 1000px"></p>

Each one of the days to measure can be defined as parameters as well, to let the user define the criteria of the retention study:

```yaml
# ----- Parameters to extract activity day per user ----
activity_transformer:
  cohort_frequency: daily
  retention_days:
    - 0
    - 1
    - 3
    - 7
    - 14
    - 28
    - 56
```

**Node: `ratio_register_node`**

In the final stage of this pipeline, we'll generate bounded retention metrics as ratios. These metrics will be used to demonstrate the effectiveness of our approach. While I may not have experience in gaming, I understand the importance of accurately measuring retention ratios. To ensure our metrics are properly measured, we'll ask the [Andreessen Horowitz](https://a16z.com/about/) Partner, [Andrew Chen](https://andrewchen.com/), a leading expert in the field.

{{< tweet 1184170127345893376 >}}

Well, the consultation went smoothly. So, we're ready to define our parameters. Specifically, we'll be establishing ratios for individual retention metrics, as well as ratios for combined retention metrics (analytical ratios):

```yaml
# ----- Parameters to aggregate retention metrics ----
ratio_register:
  ratios:
    - (1,0)
    - (3,0)
    - (7,0)
    - (14,0)
    - (28,0)
    - (56,0)
  analytical_ratios:
    - (3,1) # D3/D1
    - (7,3) # D7/D3
    - (28,7) # D28/D7
```

To gain a deeper understanding of retention metrics, I highly recommend delving into Timothy Daniel's (2022) [insightful article](https://medium.com/permutable-analytics/how-to-calculate-benchmark-day-1-day-7-and-day-30-retention-in-amplitude-analytics-d500a77eae04).

In essence, there are two types of retention metrics: bounded and unbounded. While the latter includes unique users returning on the same day or later, the first one is considered more reliable because it's not influenced by new data coming in. So if we're aiming for accuracy and consistency in the retention measurements, bounded retention should be the go-to.

This is how the output should look like:

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/robguilar-website/main/content/posts/brawlstars_retention_pipeline/images/asset_05.png" style="width: 1000px"></p>

---

### Player Classification and Cohorts Creation Pipeline

**Pipeline `player_cohorts_classifier`**

This pipeline is a parametrized method for classifying players. It saves the model in the registry and the artifacts for each experiment. The main objective of this pipeline is to make it easy for data scientists to set hyperparameters and save their models. This will help to prevent the spread of models across a repository into random Jupyter notebooks, without versioning. We know this is a common scenario that many data scientists are familiar with.

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/robguilar-website/main/content/posts/brawlstars_retention_pipeline/images/asset_06.png" style="width: 1000px"></p>

It is worth mentioning that each node in this pipeline was planned using an experimental Jupyter Notebook. If you want to check the raw version, you can download it from [here](https://github.com/robguilarr/Brawlstars-retention-pipeline/blob/main/notebooks/player_segmentation_node.ipynb).

**Node: `feature_scaler_node`**

This node will use as input the player metadata, which includes descriptive data about the lifetime statistics for each player per battle-type score, to get more detail on the data specs you can refer to the ["Player"](https://brawlstats.readthedocs.io/en/latest/api.html#player) data model.

To train a `KMeans` model effectively, it's crucial to prepare the data by scaling it. Let's first review one of my recent [experiments](https://github.com/robguilarr/Machine-Learning-Sandbox/blob/master/source/experiments/unsupervised_machine_learning/KMeans.ipynb) before delving into the scaling methods. The scaling method we choose depends on the nature of the data and our analysis requirements. To choose the best scaling method for your project, here's a brief overview of each option:

- `StandardScaler`: This scaling method scales each feature to have a mean of 0 and a standard deviation of 1. It's a good choice if our data follow a normal distribution and if we want to emphasize the differences between data points.
- `Normalizer`: This scaling method scales each sample to have a Euclidean length of 1. It's a good choice if we want to emphasize the direction of the data points and we're not interested in their magnitude.
- `MaxAbsScaler`: This scaling method scales each feature to have a maximum absolute value of 1. It's a good choice if we have sparse data or features with very different scales and we want to preserve the sparsity and relative magnitudes of the data, like when we have justified outliers.

So, as we have sparse data and features with very different scales, and we want to preserve the sparsity and relative magnitudes of the data I used `MaxAbsScaler`.

**Node: `feature_selector_node`**

To ensure a correct classification process, it is important to select appropriate features to classify players, but it is equally important to determine the appropriate way to do so. In this case, I did not use a scientific approach to define it, but rather based it on game criteria.

I considered four attributes based on the next metrics:

- Trophies: The number of trophies earned by a player in Brawlstars is a significant indicator of their performance level and skill.
- 3 vs. 3 Victories: This attribute shows how many 3 vs. 3 matches the player has won, which could be an indication of their teamwork ability.
- Solo and Duo Victories: These attributes represent the number of Solo or Duo matches the player has won, indicating their ability to play effectively without the support of teammates.
- Best RoboRumbleTime: This attribute represents the player's best time in defeating robots in the Robo Rumble event, indicating their skill in handling PvE (player versus environment) situations.

Note: Since `highestPowerPlayPoints` refers to the highest score in a competitive mode that is no longer active, we will remove this feature. After all, according to [Brawlstars Wiki](https://brawlstars.fandom.com/wiki/Power_Play#:~:text=Power%20Play%20Points&text=The%20total%20number%20of%20Power,could%20be%20earned%20is%201386), this competitive mode could be unlocked after earning a Star Power for any Brawler, meaning that won't be a descriptive attribute for every player.

Also, these attributes can be divided into four descriptive **inter-player interaction buckets**: 3 vs. 3 Victories sessions, Duo and Solo sessions, and Experience. However, we will exclude PvE from this project since we should use different game design methods to ensure better analysis of the player's behavior versus the environment.

To determine the most appropriate method to use, we need to select the variables that can give us better performance given a model construction. We have three options to consider:

- `VarianceThreshold`: This method removes all features whose variance doesn't meet a certain threshold. This technique is useful when our dataset has many features with low variance and can be removed without affecting the model's performance.
- `SelectKBest`: This technique selects the K most significant features based on statistical tests such as chi-squared or ANOVA.
- `Recursive Feature Elimination`: This method recursively removes features and evaluates the model's performance until the optimal number of features is reached. This method uses a model to assess the feature's importance and removes the least important ones iteratively.

In conclusion, we also need to consider classifying players based on their preferred type of event and their historical score achieved, which is measured by [trophies](https://brawlstars.fandom.com/wiki/Trophies). We can group highly skilled players with good teamwork into a cluster, while players with high solo and duo victories can be classified as excellent solo and duo players.

By defining these clusters, we can gain insights into the behavior and tendencies of different types of players and develop more targeted game design strategies. Therefore, we will set the parameter k of the `SelectKBest` method as 4 to select the top 4 features that can give the best model performance on the scaled data, which are going to be saved and versioned in the `/07_feature_store`.

```yaml
# ----- Parameters for feature selector ----
feature_selector:
  top_features: 4
```

**Node: `kmeans_estimator_grid_search_node`**

We begin by obtaining the scaled data along with the selected features. Once the preparation is complete, we can proceed to train a model and create the estimator object. The resulting artifacts will be stored in our model registry, which can be found at the path `/08_model_registry` within our GCP bucket.

To determine the most suitable algorithm for our task, two key factors were considered: the dataset size and the desired number of clusters. For larger datasets, we favor the computationally efficient and faster **K-Means** algorithm. On the other hand, when working with smaller datasets, **Hierarchical clustering** emerges as a more appropriate choice.

K-Means is a highly effective **partition-based** clustering algorithm that groups data points into a predetermined number of clusters. It begins by randomly selecting initial centroids (cluster centers) and proceeds to iteratively assign each data point to the nearest centroid based on a chosen distance metric. After the assignment, the centroids are updated by calculating the mean distance of all the points within each cluster. This process repeats until the centroids no longer change, ensuring the clusters reach a stable state.

In summary, this particular node performs a grid search combined with cross-validation to identify the optimal hyperparameters for the KMeans clustering algorithm.

```yaml
# ----- Parameters for player clustering ----
kmeans_estimator_grid_search:
  random_state: 42
  max_n_cluster: 13
  starting_point_method: ["k-means++"]
  max_iter: [200]
  distortion_tol: [0.0001]
  cross_validations: 5
  cores: -1
  # Inertia plot dimensions in px
  plot_dimensions:
    width: 500
    height: 500
```

To enhance the evaluation process, we utilize the [GridSearch](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.GridSearchCV.html) method, allowing users to define and evaluate multiple parameters. Let's explore the key parameters:

- `starting_point_method`: This parameter determines the method for selecting initial centroids in the K-means algorithm. We employ the **k-means++** initialization method, which enhances convergence by intelligently choosing initial centroids.
- `max_iter`: The maximum number of iterations that the K-means algorithm will perform is specified by this parameter. The algorithm will halt either when it reaches the specified **max_iter** iterations or when convergence is achieved before that.
- `distortion_tol`: This parameter controls the convergence criteria based on distortion (or inertia). The algorithm terminates if the change in distortion between two consecutive iterations falls below this value.
- `cores`: Used to define the number of cores used by the machine to evaluate and select the best model.

By leveraging GridSearch, we can identify the model with the highest negative inertia. The selected model will be stored in the model registry as a pickle file. Additionally, a versioned inertia plot will be generated, enabling easy tracking of the number of clusters selected and evaluated.

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/robguilar-website/main/content/posts/brawlstars_retention_pipeline/images/asset_07.png" style="width: 1000px"></p>

The node will capture four outputs, each one versioned with the execution timestamp from the pipeline:

- `kmeans_estimator`: This output stores the trained model as a pickle file.
- `best_params_KMeans`: This output represents the optimal estimator chosen through the GridSearch method.
- `eval_params_KMeans`: This output contains the hyperparameters used for evaluating the GridSearch.
- `inertia_plot`: This output encompasses the inertia plot displayed earlier in the form of a Plotly JSON dataset.

**Node: `kmeans_inference_node`**

The next step is to evaluate the model. This node will import the model from the model registry, produce the inferences, and generate labels that will be saved in the `player_metadata_clustered` dataset. It will also produce four metrics to evaluate the model:

- Inertia: This metric measures the sum of distances between each point and its assigned cluster center. A lower inertia value indicates that the data points are closer to their respective centroids.
- Silhouette score: This metric indicates how well each data point is assigned to its cluster. A higher silhouette score indicates that data points within a cluster are more similar to each other and dissimilar to points in other clusters.
- Davies-Bouldin index: This metric evaluates the average similarity between each cluster and its most similar cluster, taking into account both the scatter within the cluster and the separation between different clusters. A lower Davies-Bouldin index indicates a better separation between clusters.
- Calinski-Harabasz index: This metric measures the ratio of between-cluster variance to within-cluster variance. A higher Calinski-Harabasz index indicates a better separation between clusters.

All of these metrics will be saved in the `metrics_KMeans` output, which will be saved as a model artifact and also versioned in our GCP model registry. This way, you will be able to visualize the experiment tracking in the [Kedro Experiments](https://docs.kedro.org/en/stable/visualisation/experiment_tracking.html) section, as shown below.

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/robguilar-website/main/content/posts/brawlstars_retention_pipeline/images/asset_08.png" style="width: 1000px"></p>

This will allow you to track the performance of the model over time and identify any areas that need improvement.

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/robguilar-website/main/content/posts/brawlstars_retention_pipeline/images/asset_09.png" style="width: 1000px"></p>

**Node: `centroid_plot_generator_node`**

Finally, this node will generate visualizations of the centroids for each combination of X and Y variables. The plot below provides an example, where the centroids are represented by black-bordered squares.

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/robguilar-website/main/content/posts/brawlstars_retention_pipeline/images/asset_10.png" style="width: 1000px"></p>

---

### Player activity data and Cluster labels merge Pipeline

**Pipeline `player_activity_clustering_merge`**

The purpose of the upcoming pipeline is to streamline the process of augmenting the retention data generated by the `activity_transformer_node`. Specifically, this involves incorporating the tags and cluster labels obtained from the output produced by the `kmeans_inference_node`. The ultimate objective is to utilize this enriched dataset to generate compelling visualizations through the `user_retention_plot_gen_node`.

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/robguilar-website/main/content/posts/brawlstars_retention_pipeline/images/asset_11.png" style="width: 1000px"></p>

To illustrate the capabilities of this pipeline, let's examine an example of the plots it can generate. 

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/robguilar-website/main/content/posts/brawlstars_retention_pipeline/images/asset_12.png" style="width: 1000px"></p>

The cohort window displays the bounded retention data for players belonging to "Cluster 0". Among users who installed the game by January 1st, 2023, the analysis reveals that 87% returned after the first day, 78% returned after the third day, and 56% of them continued engagement after the seventh day. It is important to note that these results are intended solely for **demonstrative** purposes, as they may be subject to bias due to the data collection method employed.

## 🌐 Deployment Demo

The deployment will be structured in a simple way to define an architecture that can run the job without complex services. In this case, I used a stateless service to run the job. The tech stack used is composed of the following elements:

- **GitHub repository:** The code repository containing the code assembled as a Kedro project framework. This repository has two branches: one for development (`feature/develop`) and another for deployments (`main`).
- **Docker container:** A container created with a [Dockerfile](https://github.com/robguilarr/Brawlstars-retention-pipeline/blob/main/Dockerfile) that installs Java v8, Hadoop 3.1.1, and Spark 3.3.1 on top of a base image in Python 3.9. The container then executes the main `CMD` command to run the job.
- **Google Cloud Bucket:** A place to store the raw data, staging data, and model artifacts. The bucket is structured to provide more traceability to the experiments.
- **Google Cloud Build:** A service used for continuous development. When a user makes a pull request from the `feature/develop` branch to the `main` branch, an action is triggered in Google Cloud Build to rebuild the image based on the new code version.
- **Google Cloud Run:** A computing platform with scalable infrastructure where the execution command is pointing, using the code image provided by the Cloud Build service.

<p align="middle"><img src="https://raw.githubusercontent.com/robguilarr/robguilar-website/main/content/posts/brawlstars_retention_pipeline/images/asset_13.png" style="width: 1000px"></p>

Note that the execution of the code is pointing to the visualization command, that’s why the outputs are not loaded. This is because the main purpose of this deployment is to showcase this project. However, for a deployment on GCP, it is simple to change the `CMD` command at the end of the Dockerfile.

Additionally, the [deployment](https://brawlstars-retention-pipeline-6u27jcczha-uw.a.run.app/) was made public for this showcase using [Google’s Free Program](https://cloud.google.com/free/docs/free-cloud-features#cloud-run), and a VPC was not set up to let you access the [pipeline view](https://brawlstars-retention-pipeline-6u27jcczha-uw.a.run.app/).

## 🗒️ Final thoughts & takeaways

**What can the stakeholders understand and take into consideration?**

Implementing an automated and parametrized pipeline for bounded retention metrics allows for focused analysis of player engagement and behavior, providing actionable insights for game improvement.

Real-time and reliable data enables designers to identify trends, patterns, and potential issues affecting customer retention, facilitating the implementation of targeted strategies and interventions.

**What could the stakeholders do to take action?**

Planning the data processing method is crucial to assess scalability and long-term parallelization technologies, minimizing risks and optimizing resource allocation.

Establishing a robust monitoring system helps track key metrics and performance indicators before and after new season releases, aiding in the identification of potential data drift sources.

Implementing good practices in Continuous Development allows for seamless productionalization of new changes and facilitates testing and deployment.

To compare different modeling strategies, A/B testing can be used by assembling new nodes to alternative versions of the same pipeline.

**What can stakeholders keep working on?**

Centralizing the solution into a single cloud provider, like Google Cloud Services, minimizes tech debt, simplifies management, and optimizes resource allocation and cost management.

Leveraging tools like [Kedro](https://docs.kedro.org/en/stable/visualisation/experiment_tracking.html#when-should-i-use-experiment-tracking-in-kedro), MLflow, and Neptune.ai for experiment tracking helps keep artifacts versioned and trackable over time, enhancing reproducibility and collaboration

---

## ℹ️ Additional Information

- **About the article**

The following article is provided strictly for learning purposes and is intended solely as a learning tool for the author. The content of this article represents the author's personal perspectives and does not rely on established or experienced methods commonly employed in the field. The practices and methodologies discussed herein are not reflective of the opinions or views held by the author's employer. It is strongly advised against utilizing this article directly as a solution, as the process of data collection and utilization of public APIs may introduce biases and inaccuracies. Users are cautioned to exercise caution and discretion when considering its use.

- **Related Content**

Here you can find a list of preferred materials if you are interested in similar topics

— [Elite Game Developers Podcast](https://podcasts.apple.com/us/podcast/elite-game-developers-podcast/id1463752909) by Joakim Achren.

— Joakim Achren newsletter and [blog](https://www.elitegamedevelopers.com/blog/?ref=elitegamedevelopers.com).

— Game Data Science book and additional info at the [Oxford University Press](https://global.oup.com/academic/product/game-data-science-9780192897879?cc=cr&lang=en&).

— Getting started with [Kedro](https://docs.kedro.org/en/stable/get_started/install.html).

— Brawstats [homepage](https://brawlstats.com/), by overwolf.

- **Datasets**

The output dataset will be uploaded to Kaggle as part of its public source intention.

