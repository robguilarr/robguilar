# GenAI-Driven Pokémon Go Companion App: Augmenting onboarding experience with RAG


**Abstract**: This article introduces an E2E application aimed at enhancing the onboarding experience for new Pokémon Go players through a Langchain-framework-based app that integrates with the OpenAI API, leveraging GPT-4's capabilities. Central to the application is a GenAI-powered Pokémon index, designed as a companion to reduce new user friction and deepen understanding of the game's universe. By employing a Retrieval-Augmented Generation system with FAISS, Meta's vector database, the app ensures information retrieval. With a workflow controlled entirely by a LLM, it offers functionalities such as Pokémon information, squad creation, and defense strategies. The app is aiming to improve user retention, widen the game's demographic reach, and ease the learning curve. The project shows the potential of linking AI with mobile gaming to craft intuitive, engaging platforms to improve user retention and game understanding among newcomers.

---

**:(fas fa-exclamation-circle fa-fw): Important: Any views, material or statements expressed are mines and not those of my employer, read the disclaimer to learn more.[^1]**

---

{{< admonition info "Looking for an interactive experience?" true >}}

:rocket: Clone the source code from Github, available <a href="https://github.com/robguilarr/genai_pokemon_strategy_assistant/tree/master">here</a>, then follow the instructions on the README file to install the Application on your local environment

{{< /admonition >}}

{{< youtube 2sj2iQyBTQs >}}

## ⚠️ Introduction to problem

Introducing a GenAI companion app for guiding new users in Pokémon Go makes a lot of sense when we look at the challenges the community faces. Pokémon Go isn't just a game; it's a phenomenon that has captured the attention of millions, combining engaging gameplay with real-world exploration.

In 2023, the game boasted [145 million MAU](https://playercounter.com/pokemon-go-player-count/), showcasing its massive appeal. Despite experiencing a dip in revenue in 2022, dropping to $0.77 billion from its peak, Pokémon Go continues to draw in a vast number of players as mentioned by [Mansoor Iqbal](https://www.businessofapps.com/data/pokemon-go-statistics/).

One key aspect of Pokémon Go's success is its broad and diverse player base. The demographics are varied, with a significant number of young adults playing the game. Additionally, there's a pretty balanced gender distribution among players, with 45.1% male and 54.9% female according to [Player Counter](https://playercounter.com/pokemon-go-player-count/). This diversity speaks volumes about the game's ability to attract and engage a wide range of people.

<figure align="middle">
  <img src="https://raw.githubusercontent.com/robguilarr/robguilar-website/main/content/posts/genai_pokemon_strategy_assistant/images/asset_01.png" style="width: 60%">
  <figcaption style="font-size: 70%;">Source: Statista</figcaption>
</figure>

The latest graph from [Statista](https://www.statista.com/statistics/589197/pokemon-go-players-us-age/) reveals an intriguing trend: a significant drop in the application's usage among users aged between 18 and 54, who previously used the app but now do not. Conversely, users above 55 have shown the opposite behavior, demonstrating increased engagement. This information is particularly crucial given that most users are young adults; however, daily engagement is something that should be measured using at least D1 to D7 Retention for a clearer diagnosis.

Understanding DAU behavior is vital for a game generating $4 million in daily revenue, as reported by [SensorTower](https://sensortower.com/). However, the graph does not suggest that [Niantic](https://nianticlabs.com/support/pokemongo?hl=en) may need to do more to help new users learn the game and keep them engaged.

Drawing on these insights, it's clear that a GenAI companion app could fulfill several key roles:

- **Onboarding and Education**: This function would help newcomers grasp the fundamental aspects of the game, including basic rules and strategic approaches. By demystifying the initial learning curve, the app ensures that new players can quickly become competent and enjoy their gaming experience.
- **Adaptation Support**: As games evolve, introducing new features or temporary changes can sometimes confuse even the most experienced players. Here, the GenAI companion app steps in to offer timely updates and tutorials, helping players adapt and continue to enjoy the game without interruption.
- **Guidance for New Joiners in the Pokemon Saga**: Specifically tailored to those new to the Pokemon series, this feature of the app would act as a QA support system. It addresses common queries and challenges that newcomers face, ensuring they feel supported as they embark on their Pokemon journey.

### Hypothesis

Following the perspective provided by the previous data, and given the limited availability of real data, it will be assumed as a base hypothesis that the creation of a complementary application will serve to substantially improve the key roles mentioned in the following hypothesis.

- $H_0:$ The companion app will significantly support onboarding, and offer adaptive assistance, thereby facilitating a smoother introduction for new members to the Pokémon series.

Now, under the premise that there must be enough statistical evidence to reject the null hypothesis, the following technical article will be explained, which will show the architecture and development of this GenAI application, but is not intended to be used as professional advice or consultation.

<figure align="middle">
  <img src="https://raw.githubusercontent.com/robguilarr/robguilar-website/main/content/posts/genai_pokemon_strategy_assistant/images/asset_02.png" style="width: 60%">
  <figcaption style="font-size: 70%;">Source: Remco Braas</figcaption>
</figure>

### Potential Stakeholders

- Programmers: Tasked with developing the technical infrastructure for GenAI integration, augmented reality features, and ensuring smooth app functionality across devices.
- Testers: Essential for identifying bugs and ensuring the GenAI interactions, with augmented reality features work as intended across various devices and real-world scenarios.
- Publisher: Interested in the market potential of combining Pokémon Go's popularity with cutting-edge GenAI technology to attract new players and retain existing ones.

**Note:** To facilitate the understanding of the roles of the development team, I invite you to take a look at **[this](https://www.robguilar.com/posts/gamedev_structure/)** diagram that I designed.

---

⚠️ Audience message: If the theoretical and technical details of this article are not of your interest, feel free to jump directly to the **Application Workflow** section.

---

## 📥 About the model

This GenAI app uses [OpenAI's GPT-4](https://openai.com/gpt-4) by default for its advanced NLP capabilities. However, understanding the various needs of users, I incorporated a feature that allows easy replacement of the AI model through a simple parameter adjustment, during the [initialization](https://github.com/robguilarr/genai_pokemon_strategy_assistant) of the OpenAI Chatbot instance.

The customization option ensures that users can align application performance with their specific requirements, balancing factors such as computational efficiency, cost, and task specificity, this one can be modified in the [global_conf.yml](https://github.com/robguilarr/genai_pokemon_strategy_assistant/blob/1173db1f4137165db8115142812660ad365ba1bf/conf/global_conf.yml#L5) file of the repository.

## 🏗️ Architecture components

To grasp the application workflow, it's important to first familiarize with the components that make up the architecture and understand their functionalities. To aid in this explanation, let’s refer to the following architecture diagram, which provides a comprehensive overview.

<figure align="middle">
  <img src="https://raw.githubusercontent.com/robguilarr/robguilar-website/main/content/posts/genai_pokemon_strategy_assistant/images/asset_03.png" style="width: 100%">
</figure>

### 📦 Setup Loader

When initializing the application or invoking the LLM, the first step is crucial but might not be showed in the last diagram. This step ensures the consistent return of the same LLM model instance, along with essential components like the [environment configuration](https://github.com/robguilarr/genai_pokemon_strategy_assistant/blob/master/.env), the logger, a [callback handler](https://github.com/robguilarr/genai_pokemon_strategy_assistant/blob/master/agents/callbacks_agent.py) for agent’s monitoring, and the [prompt library](https://github.com/robguilarr/genai_pokemon_strategy_assistant/blob/master/conf/prompt_template_library.yml).

This consistency is achieved through the Singleton design pattern. Essentially, this pattern ensures that every call to `SetupLoader()` results in the return of the identical instance configuration. By doing so, it guarantees the model's settings, such as temperature and other configurations remain uniform and constant across the server.

```python
class SetupLoader:
    """ -- Singleton design pattern to instantiate the application --
    Validation: Ensures that subsequent calls to SetupLoader() will return the same
    instance configuration.
    """

    _instance = None
    _is_initialized = False

    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super(SetupLoader, cls).__new__(cls)
        return cls._instance

    def __init__(self, new_model=False):
        load_dotenv()
        if not self.__class__._is_initialized:  # Ensure __init__ is only executed once
            self.logger = self._setup_logging()
            self._setup_environment()
            self.prompt_template_library = self._setup_prompt_library()
            self.global_conf = self._setup_global_conf()
            self.chat_openai = self._setup_chat_openai(
                callbacks=self._setup_callbacks()
            )
            self.__class__._is_initialized = True
        elif new_model:
            # After the first initialization, a new model can be created
            self.chat_openai = self._setup_chat_openai(
                callbacks=self._setup_callbacks()
            )

    def _setup_logging(self):
        logging.basicConfig(level=logging.INFO)
        return logging.getLogger(__name__)

    def _setup_prompt_library(self):
        return prompt_template_library

    def _setup_global_conf(self):
        return global_conf

    def _setup_callbacks(self):
        return [AgentCallbackHandler()]

    def _setup_environment(self):
        """Set the OpenAI API key from global_conf.yml if the user defined it."""
        if global_conf.get("OPENAI_API_KEY", None):
            os.environ["OPENAI_API_KEY"] = global_conf["OPENAI_API_KEY"]

    def _setup_chat_openai(self, callbacks=None):
        openai.api_key = os.environ.get("OPENAI_API_KEY")
        return ChatOpenAI(
            temperature=global_conf["MODEL_CREATIVITY"],
            model_name=global_conf["MODEL_NAME"],
            callbacks=callbacks,
        )
```

It's worth noting there's a special parameter, `new_model`, designed for scenarios where a new instance creation is explicitly required. This option adds flexibility for use cases that demand refreshing the LLM settings.

### 📦 Tagging

**Components**: Detect Intent

When examining the diagram's structure, we can identify a step that plays a pivotal role: evaluating user input and deciding the subsequent path for redirecting requests or questions. All this will happen on the first step of the [IntentHandler](https://github.com/robguilarr/genai_pokemon_strategy_assistant/blob/master/src/intent_handler.py#L55).

This task is achieved through the implementation of a [Tagging strategy](https://python.langchain.com/docs/use_cases/tagging), which is built upon a base Pydantic model. Put simply, this process is similar to a Named Entity Recognition system. However, a difference is the limitation to a predefined set of categories for text classification.

```python
from langchain.output_parsers.openai_functions import PydanticOutputFunctionsParser
from langchain_core.pydantic_v1 import BaseModel, Field

class IntentTagger(BaseModel):
    """Tag the piece of text with particular intent type, and detect the text
    structure describing the intent"""

    intent_type: str = Field(
        description="Intent type expressed in the text, must take one of the "
        "functionalities as: 'defense_suggestion', 'information_request' or "
        "'squad_build'",
        default=None,
    )
    intent_structure: str = Field(
        description="The type of sentence structure used to detect the intent must "
        "take one of the values: 'pokemon_names', 'natural_language_question' or "
        "'natural_language_description'",
        default=None,
    )

intent_parser = PydanticOutputFunctionsParser(pydantic_schema=IntentTagger)
```

The `intent_type` can be classified include:

- `defense_suggestion`: Prompt mentions that an unknown Pokémon has appeared and the user needs a suggestion to use a Pokémon against it. The user must provide the name of the Pokémon as part of the input.
- `information_request`: The user wants to know more about a Pokémon. In the input, the user must provide a request for information on one or multiple entities.
- `squad_build`: The user wants to build a squad of Pokémon based on the opponent's Pokémon types. The user must provide a list of Pokémon names as part of the input.

And, the `intent_structure`, which describe the syntactic composition of the text, can be classified in:

- `pokemon_names`: Sentence with the name of one or more Pokémon explicitly mentioned in the text with an expressed request.
- `natural_language_description`: Sentence with a description of a Pokémon in natural language without explicitly mentioning the name of the Pokémon with the intention of guessing what the Pokémon is.
- `natural_language_question`: Sentence with a question about the Pokémon whose name is mentioned explicitly, asking for a specific attribute(s) belonging to the Pokémon entity.

Note that `intent_parser` helps structure the LLM response, meaning that the output will be consistently returned as a Pydantic object, and based on this attributes, the system will define where to redirect the request.

### 📦 Named Entity Recognition (NER)

**Components**: Gather Name Entity | Gather Desc. Entities

Also known as [Multiple Extraction](https://python.langchain.com/docs/use_cases/extraction/quickstart#multiple-entities) approach, this phase involves a general procedure that incorporates Pydantic models, which will be used for various intents presented in the architecture. In this specific instance, a Pydantic model functions as an entity object.

It is characterized by defined attributes that are essential for the [Named Entity Recognition](https://en.wikipedia.org/wiki/Named-entity_recognition) extraction process. Let’s refer to the following code example:

```python
from typing import List
from langchain.output_parsers.openai_functions import PydanticOutputFunctionsParser
from langchain.pydantic_v1 import BaseModel, Field

class PokemonEntity(BaseModel):
    """Extract the piece of text with the Pokémon name"""

    name: str = Field(description="Name of the Pokémon mentioned in text", default=None)

class PokemonEntityList(BaseModel):
    """Extract list of pieces of text with Pokémon names"""

    name_list: List[PokemonEntity] = Field(
        description="List of names of the Pokémon mentioned in text", default=[]
    )

pokemon_entity_parser = PydanticOutputFunctionsParser(pydantic_schema=PokemonEntityList)
```

Here, the `PokemonEntity` will represent a Pokemon named in the prompt, and `name` is the identifiable entity attribute with its respective data type. However, the real challenge arises when we introduce multiple names into the prompt. This is where the `PokemonEntityList` comes into play.

The advantage of the `PokemonEntityList` is quite significant. It allows for the addition of more attributes to the `PokemonEntity` class seamlessly. This feature ensures that the LLM can accurately determine which entity each attribute belongs to.

To put this into perspective, consider a scenario where the prompt given is *"Charizard and Treecko are fire and grass type Pokémon respectively"*. The output structure generated from this prompt is both efficient and logical, demonstrating the system's capability to assign entities, simply great:

```python
PokemonEntityList.name_list = [
			PokemonEntity(name="Charizard", pokemon_type="Fire"), PokemonEntity(name="Treecko", pokemon_type="Grass")
]
```

This way, a chain is created and invoked by converting the Pydantic class into an OpenAI function.

```python
def get_pokemon_entity_chain() -> RunnableSequence:
    """Create a chain that can be used to identify the Pokémon entities of a user input
    by using an Extraction approach.
    Returns:
        RunnableSequence: Language model chain structured as RunnableSequence.
    """
    pokemon_template = dedent(
        prompt_template_library["stage_1_pokemon_entity_template"]
    )

    pokemon_entity_template = ChatPromptTemplate.from_messages(
        [("system", pokemon_template), ("human", "{input}")]
    )

    model = base_llm.bind(
        functions=[convert_pydantic_to_openai_function(PokemonEntityList)],
        function_call={"name": "PokemonEntityList"},
    )

    return pokemon_entity_template | model | pokemon_entity_parser
```

### 📦 API Tooling

**Components**: Gather Information

Langchain provides two solutions to [augment the output of LLMs](https://python.langchain.com/docs/use_cases/tool_use/) through two primary methods: "Agents" and "Chains." The "Agents" method involves calling an agent that intelligently selects and utilizes the necessary tools as often as needed to fulfill a request. This process is automatic, leveraging the agent's ability to determine the most suitable tool for the task at hand.

On the other hand, "Chains" offers a more [tailored approach](https://python.langchain.com/docs/use_cases/tool_use/prompting#choosing-from-multiple-tools), which is particularly beneficial for our specific needs. By employing "Chains", we can make a customized Pokemon API wrapper that functions as an OpenAI tool.

<figure align="middle">
  <img src="https://raw.githubusercontent.com/robguilarr/robguilar-website/main/content/posts/genai_pokemon_strategy_assistant/images/asset_12.png" style="width: 60%">
  <figcaption style="font-size: 70%;">Source: Langchain Documentation</figcaption>
</figure>

In the previous illustration, we outlined a process flow. Here, we can go deeper into each step:

1. **Input**: The system begins by reading the natural language input provided by the user. 
2. **Parser**: Next in the interpretation, the system employs a parser to analyze the type of input received. This analysis is essential for identifying the most appropriate tool within the model that is suited for processing the given input. The parser acts as a bridge, connecting the input with its relevant processing tool.
3. **Create Arguments**: Once the appropriate tool is identified, the system then proceeds to formulate the necessary arguments required to utilize the tool effectively. These arguments, along with the names of the selected tools, are temporarily stored in an output.
4. **Tool Mapping**: In the final step, the prepared arguments are forwarded to the actual function designated for the task. The system then executes the function, extracting the outputs generated.

Following this structured approach, the API tool code is designed as shown below:

```python
@tool("pokemon_api_wrapper", args_schema=ToolingEntry, return_direct=True)
def pokemon_api_wrapper(name_list: List[str]) -> Dict:
    """Useful for when you need to request information from the Pokémon API,
    considering a single Pokémon Entity or several of them as input."""
    client = pokepy.V2Client()
    pokemon_info_collection = {}

    for pokemon_name in name_list:
        try:
            logger.info(" PokemonAPIWrapper: Information Search ")
            pokemon_data = client.get_pokemon(pokemon_name.lower())
            pokemon = pokemon_data[0]
            info = {
                "id": pokemon.id,
                "stats": {stat.stat.name: stat.base_stat for stat in pokemon.stats},
                "height": pokemon.height,
                "weight": pokemon.weight,
                "types": [type_slot.type.name for type_slot in pokemon.types],
                "abilities": [
                    ability_slot.ability.name for ability_slot in pokemon.abilities
                ],
                "sprites": pokemon.sprites.__dict__,
            }

            logger.info(" PokemonAPIWrapper: Types Search ")
            if info["types"]:
                try:
                    # Extract Damage Relations
                    info["damage_relations"] = [
                        client.get_type(type_slot) for type_slot in info["types"]
                    ]
                    # Overwrite Damage Relations
                    damage_relations = info["damage_relations"][0][0].damage_relations
                    info["damage_relations"] = {
                        damage_type: damage_relations.__dict__.get(damage_type)[0].name
                        for damage_type in damage_relations.__dict__.keys()
                        if damage_type != "_subresource_map"
                        and damage_relations.__dict__.get(damage_type) != []
                    }
                except Exception as e:
                    logger.info(f"No 'damage_relations' were extracted: {e}")
                    info["damage_relations"] = {}
                    pass

            pokemon_info_collection[pokemon_name] = info
            logger.info(
                f" PokemonAPIWrapper: Information of '{pokemon_name}' was extracted "
            )

        except Exception as e:
            raise ToolException(f"Tool Error handling '{pokemon_name}': {e}")

    return pokemon_info_collection
```

### 📦 Retrieval-Augmented Generation - Retrieval QA

**Components**: Gather Answer | Search Sim. Desc | Gather Description | Gather Defense Suggestion | Gather Squad Suggestion

For better understanding, first let’s break down this kind of component into two key parts, focusing on how we handle data and answer questions using Large Language Models:

1. **Retrieval-Augmented Generation (RAG)**: This method uses a two-step process. First, it finds relevant information files (e.g.: Pokemon Wiki as PDF) related to the input. Then, it combines this information with the original request to create a response. This approach is great for generating detailed and enhanced descriptions.
2. **RetrievalQA**: This strategy is all about answering questions. It searches for and retrieves information related to the question from a knowledge base (e.g.: Vector Store). Then, it uses this information to either directly provide an answer or to create one. In such a system, when a question is asked, the model retrieves relevant documents or passages from a knowledge base and then either extracts or generates an answer based on this chunk of information.

☑️ **Vector Store**

To implement these methods, we're using a tool Vector Store FAISS, created by [Meta AI Research](https://engineering.fb.com/2017/03/29/data-infrastructure/faiss-a-library-for-efficient-similarity-search/). Faiss is designed for quickly finding information that's similar to your search query, even in huge databases. We chose to use FAISS for its features, explained simply:

- It can search multiple vectors simultaneously, even without a GPU enabled, thanks to its great multithreading capabilities.
- It finds not just the closest match but also the second closest, third closest, and so on, using a Euclidean distance (L2) to search for its similarity.
- It trades precision for speed, e.g.: *give an incorrect result 10% of the time with a method that's 10x faster or uses 10x less memory*.
- It saves data on the disk instead of using RAM.
- It works with binary vectors, which are simpler than the usual floating-point vectors.

More details can be found at the [Meta’s official wiki](https://github.com/facebookresearch/faiss/wiki).

☑️ **Process**

After loading the files using Langchain [Document Loaders](https://python.langchain.com/docs/modules/data_connection/document_loaders/).

<figure align="middle">
  <img src="https://raw.githubusercontent.com/robguilarr/robguilar-website/main/content/posts/genai_pokemon_strategy_assistant/images/asset_13.png" style="width: 60%">
  <figcaption style="font-size: 70%;">Source: Langchain Documentation</figcaption>
</figure>

**Split**:

- The first steps is to define the splitting strategy using [Text Splitters](https://python.langchain.com/docs/modules/data_connection/document_transformers/). The decision between [recursive or character-based text splitting](https://python.langchain.com/docs/modules/data_connection/document_transformers/recursive_text_splitter) is a preprocessing step to divide the document into manageable chunks. This is important because FAISS operates on vectors, and the text must first be converted into a numerical format (vector representation) that FAISS can work with. The splitting helps in managing document size and ensuring each chunk is meaningful for the vectorization process.

**Embed**:

Here are two process relevant to this step:

- Vectorization: While FAISS itself doesn't handle the text-to-vector conversion, this step is essential for preparing data for similarity search. The choice of embedding method (like `OpenAIEmbeddings`) significantly affects the quality of the search results, as it determines how well semantic similarities are captured in the vector space. Embeddings are high-dimensional vectors that represent the semantic content of the text. Each word, sentence, or document is mapped to a point in a high-dimensional space, where the distance between points corresponds to semantic similarity.
- Indexing: involves creating a structured representation of the files that allows for similarity searches.

**Store**:

- Saving the vector store locally means storing the indexed representation of the document chunks. This allows for persistent access to the indexed data for future similarity searches without needing to re-process the original documents. We chose to keep this index in the [same repository](https://www.notion.so/GenAI-Driven-Pok-mon-Go-Companion-App-Augmenting-onboarding-experience-with-RAG-82532964edee48a2840355ccc1202585?pvs=21) (`.faiss` & `.pkl`) for simplicity, but it's also possible to store it in a artifact repository manager like [JFrog](https://jfrog.com/artifactory/).

```python
def _index_vector_store() -> None:
    """Void Function to create an RAG index inside a vector store from the source
    file."""
    logger.info("Loading & Splitting Source File")
    pdf_path = f"{global_conf['SOURCE_PDF_PATH']}/pokedex_tabletop_content.pdf"

    loader = PyPDFLoader(file_path=pdf_path)
    documents = loader.load()
    documents = [
        doc
        for doc in documents
        if isinstance(doc.page_content, str) and doc.page_content.split() != ""
    ]

    if global_conf["RECURSIVE_SPLITTER"]:
        text_splitter = RecursiveCharacterTextSplitter(
            chunk_size=1100, chunk_overlap=200
        )
    else:
        text_splitter = CharacterTextSplitter(
            chunk_size=1100,  # 60
            chunk_overlap=200,
            separator="^([0-9A-Z]+)\.\s([A-Z\s]+)$",
            is_separator_regex=True,
        )

    docs = text_splitter.split_documents(documents=documents)

    logger.info("Embedding Source File")
    vectorstore = FAISS.from_documents(documents=docs, embedding=OpenAIEmbeddings())
```

☑️ Retrieval Chain

We had the option to utilize a vector store with either a `map-reduce` approach or a `parallel similarity search` based on the prompt's context.

In our specific case, the PDF contains a wiki-style format with one Pokemon per page. This format enables the Recursive Text Splitter to efficiently separate and index each subject (Pokemon), each of which is unique. Therefore, we require only a single chunk per Pokemon || Question.

- A map-reduce approach is not suitable for our needs. Although it can identify and summarize groups of similar chunks, as described by [Perplexity.ai](https://www.perplexity.ai/search/0b752aa8-122c-44c8-a88a-267bde9da075), we are only interested in extracting a single, relevant chunk.
- Consequently, we have decided to base our entire architecture on a [RunnableParallel](https://python.langchain.com/docs/expression_language/how_to/map#parallelize-steps) mechanism. This will allow us to make context searches and assign questions to each document in parallel.

```python
def _get_retrieval_qa_chain(qa_prompt: str) -> RunnableParallel:
    """Create a chain that can be used to performa semantic queries over FAISS Vector
    Store.
    Args:
        qa_prompt (str): QA prompt to be used.
    Returns:
        RunnableParallel: Language model chain structured as RunnableParallel.
    """
    new_vectorstore = FAISS.load_local(
        folder_path=f"{global_conf['VECTOR_STORE_PATH']}/pokedex_index_react",
        embeddings=OpenAIEmbeddings(),
    )
    retriever = new_vectorstore.as_retriever()
    prompt = ChatPromptTemplate(
        input_variables=["context", "question"],
        messages=[
            HumanMessagePromptTemplate(
                prompt=PromptTemplate(
                    input_variables=["context", "question"],
                    template=dedent(prompt_template_library[qa_prompt]),
                )
            )
        ],
    )

    rag_chain = (
        RunnablePassthrough.assign(context=(lambda x: format_docs(x["context"])))
        | prompt
        | base_llm
        | StrOutputParser()
    )
    # Adding sources to return
    rag_chain_with_source = RunnableParallel(
        {"context": retriever, "question": RunnablePassthrough()}
    ).assign(answer=rag_chain)

    return rag_chain_with_source
```

### 📦 API endpoints

The entire conditional process is consolidated into a single API endpoint, distinct from the UI's URL, ensuring the UI's independence from the computational resources required for the IntentHandler process. The infrastructure relies on [FastAPI](https://fastapi.tiangolo.com/), with a [Pydantic](https://docs.pydantic.dev/latest/) schema handling the I/O object.

```python
class Query(BaseModel):
    """Query model to handle the user query"""

    user_query: str = Field(description="User query to process", default=None)

@app.post("/intent_query/")
async def process_query(query: Query):
    """Endpoint to handle the intent execution and return the response to the
    user.\n
    All the responses will be structured same as defined by the corresponding
    agent in charge or the intent execution by using the `IntentHandler` class.\n
    Finally the response will be returned with the format defined by the
    `ResponseTemplate` object.\n
    **Note**: The response will be in JSON format which will be used by the streamlit
    app to display the response.
    """
    try:
        intent_handler = IntentHandler()
        intent_handler.user_input = query.user_query
        raw_response = intent_handler.run()
        final_response = raw_response.template_structure
        return {"response": final_response}
    except Exception as e:
        return {"error": str(e)}
```

To separately test the API independently, start the server with the command `make api_server`, and then access the [Swagger UI](http://localhost:8000/docs).

### 📦 User Interface

The UI dynamically adapts based on the response format, utilizing a custom template generator. This generator feeds data into the Streamlit UI, tailoring the display to match both the intent type and the syntactic structure of the user's prompt. The core of this functionality is a large Python [@dataclass](https://docs.python.org/3/library/dataclasses.html), which serves as the response template generator and [is accessible for reference](https://github.com/robguilarr/genai_pokemon_strategy_assistant/blob/master/src/common/response_template.py).

## ⚡ Application Workflow

The application simplifies setting up and deploying APIs built with FastAPI. The [README](https://github.com/robguilarr/genai_pokemon_strategy_assistant/blob/master/README.md) file includes a series of `Make` targets with a range of functions, from environment setup to API configuration and UI deployment. This section focuses on explaining the workflow and functionalities, rather than the technical details behind them.

The E2E process is guided by the prompt's [intent (type) and syntax (structure)](https://github.com/robguilarr/genai_pokemon_strategy_assistant/blob/master/conf/prompt_template_library.yml#L6). Using these, the application with assistance from the LLM, directs the workflow to the appropriate tools. If the response is not recognized, it is redirected to GPT-4's general knowledge, with a banner citing the source on the final response.

### UI Features

The first step involves familiarizing the user with the response feature. After launching the UI on a local port, a side panel appears on the left (see image below, labeled as 1), featuring a toggle for activating or deactivating JSON mode, set to off by default. This mode presents responses in JSON format (labeled as 2), facilitating debugging or analysis of the response's target within the “Intent Handler” workflow or “Response Formatter” (core components). This by default is set to `False`.

<figure align="middle">
  <img src="https://raw.githubusercontent.com/robguilarr/robguilar-website/main/content/posts/genai_pokemon_strategy_assistant/images/asset_04.png" style="width: 100%">
</figure>

Once we've defined the output format, the next step involves sending a request to the API. However, the API's response can vary significantly based on the type of intent. As previously discussed in the "Tagging" component, the Intent Handler can detect multiple combinations of intents. We will detail each of these intents in the following section:

### Information Retrieval Intent

This intent primarily seeks information on one or more Pokémon, with the process adapting to the syntactic structure of the request.

<figure align="middle">
  <img src="https://raw.githubusercontent.com/robguilarr/robguilar-website/main/content/posts/genai_pokemon_strategy_assistant/images/asset_05.png" style="width: 100%">
</figure>

There are 3 types of syntactic structures:

1. **Natural Language Question**: A question about a Pokémon initiates a RetrievalQA process through the Vector Database to find an answer. The Pokémon's name is identified as an entity, triggering an API wrapper request to deliver the final response. `E.g.`: “*Do you know in what kind of habitats I can find a Psyduck?*”.
2. **Natural Language Description**: This involves inputting a description used at identifying a Pokémon. The process selects the most similar document chunk from the Vector Database, including the Pokémon's name. Next, a NER process identifies the Pokémon name within the text, using this entity to make an API request. This retrieves information and structures a description from the Vector Database to formulate the answer. `E.g.`: “*Can you guess which Pokémon is a dual-type Grass/Poison Pokémon known for the plant bulb on its back, which grows into a large plant as it evolves*”.
3. **Single Named Entity**: The simplest approach, it directly uses NER to detect the Pokémon's name. This entity then prompts an API request to obtain information, which is used to form a structured description from the Vector Database to provide the answer. `E.g.`: “*Alright, Pokédex! It's time to find out everything about Snorlax and Pikachu!*”.

The expected structured output look as follows.

<figure align="middle">
  <img src="https://raw.githubusercontent.com/robguilarr/robguilar-website/main/content/posts/genai_pokemon_strategy_assistant/images/asset_06.png" style="width: 100%">
</figure>

### Defense Suggestion Intent

This intent aims to offer players advice on defending against Pokémon, useful for capturing Pokémon or winning battles against other players.

<figure align="middle">
  <img src="https://raw.githubusercontent.com/robguilarr/robguilar-website/main/content/posts/genai_pokemon_strategy_assistant/images/asset_07.png" style="width: 100%">
</figure>

When a user encounters an unidentified Pokémon and requests a recommendation for a counter Pokémon, they must specify the name of the Pokémon. `E.g.`: “*I stumbled upon a wild Grovyle lounging in the park! Which Pokemon should I choose for an epic battle to defeat it?*”.

1. Initially, we identify the Pokémon's name as a Named Entity using a [Pydantic Class for extraction](https://github.com/robguilarr/genai_pokemon_strategy_assistant/blob/master/agents/pydantic_agent.py#L41). This entity is then processed by the system, which aligns the input schema with the corresponding API entry.
2. Subsequently, we create an embedding request that highlights the opposing [Pokémon's vulnerabilities](https://github.com/robguilarr/genai_pokemon_strategy_assistant/blob/master/agents/rag_qa_agent.py#L113) based on its type.
3. Leveraging this information, the [LLM augment the response](https://github.com/robguilarr/genai_pokemon_strategy_assistant/blob/master/agents/rag_qa_agent.py#L113) using Vector Database context to suggest the most effective Pokémon to use in battle.

The expected structured output look as follows.

<figure align="middle">
  <img src="https://raw.githubusercontent.com/robguilarr/robguilar-website/main/content/posts/genai_pokemon_strategy_assistant/images/asset_08.png" style="width: 100%">
</figure>

### Squad Builder Intent

This intent facilitates the creation of a Pokémon squad tailored to counter the types of an opponent's Pokémon, in simple words it builds a squad. Users must input a list of the opponent's Pokémon names as part of the prompt. `E.g.`: “*Time to challenge the Fire Gym Leader! He's got a tough team with a Ninetales and Combusken, but I need your help to build a squad*”.

<figure align="middle">
  <img src="https://raw.githubusercontent.com/robguilarr/robguilar-website/main/content/posts/genai_pokemon_strategy_assistant/images/asset_09.png" style="width: 100%">
</figure>

1. Initially, the language model identifies each Pokémon in the opponent's squad and creates a vulnerability template for each, similar to the approach in the "defensive" intent.
2. Subsequently, it identifies suggested Pokémon entities and submits them to an API, which returns a structured response for each entity.

The expected structured output look as follows.

<figure align="middle">
  <img src="https://raw.githubusercontent.com/robguilarr/robguilar-website/main/content/posts/genai_pokemon_strategy_assistant/images/asset_10.png" style="width: 100%">
</figure>

### Sprites Design

Last but not least, each of the answers has a structure with the same format, which provides as part of the API wrapper the sprites of the classic designs of the series.

<figure align="middle">
  <img src="https://raw.githubusercontent.com/robguilarr/robguilar-website/main/content/posts/genai_pokemon_strategy_assistant/images/asset_11.png" style="width: 70%">
</figure>

## 🗒️ Final thoughts & takeaways

**What can the stakeholders understand and take in consideration?**

Implementing a GenAI-driven chatbot for user onboarding is beneficial, but it's important to also focus on the quality of supplementary resources, such as wikis. These play a significant role in enhancing response augmentation process. Additionally, scalability demands may require evaluating [various Vector Database](https://www.datacamp.com/blog/the-top-5-vector-databases) services, including Pinecone, elasticsearch, Cassandra, among others.

To assess the viability of this approach, it's essential to first evaluate the token consumption for all LLM calls related to each intent.

**What could the stakeholders do to take action?**

Prior to adopting such an approach, a thorough analysis of user needs is crucial. Examining user retention from D1 to D7 across different cohorts (e.g.: demography-based), supported by a [robust segmentation strategy,](https://www.is.com/community/blog/4-ways-to-boost-ad-monetization-with-user-segmentation/) provides valuable insights into user behavior and needs.

The current integration serves as a preliminary demonstration. Developing a companion app will require addressing a distinct set of integration requirements.

---

## ℹ️ Additional Information

Here you have a list of preferred materials for you to explore if you’re interested in similar topics:

- **Related Content**

— About [Langchain](https://www.langchain.com/).

— OpenAI models [Pricing page](https://openai.com/pricing).

— Meta’s FAISS [tutorials](https://github.com/facebookresearch/faiss/wiki/Getting-started).

— User Segmentation Strategies by [IronSource from Unity](https://www.is.com/community/blog/4-ways-to-boost-ad-monetization-with-user-segmentation/).

— Pokémon Go [release notes](https://niantic.helpshift.com/hc/en/6-pokemon-go/section/180-release-notes-known-issues/) by Niantic.

- **Augmentation Assets**

The RetrievalQA system was augmented using an unofficial Tabletop [Pokémon Guide](https://www.scribd.com/document/462604648/Pokemon-Tabletop-Adventures-GM-Guide-2-1). An official resource is suggested to improve response quality and avoid any bias.

[^1]:**Article disclaimer**: The information presented in this article is solely intended for learning purposes and serves as a tool for the author's personal development. The content provided reflects the author's individual perspectives and does not rely on established or experienced methods commonly employed in the field. Please be aware that the practices and methodologies discussed in this article do not represent the opinions or views held by the author's employer. It is strongly advised not to utilize this article directly as a solution or consultation material.

