# Google Doc Loader

This loader takes in IDs of Google Docs and parses their text into `Document`s. To use this loader, you will need to authenticate with a Google Account the first time you run it locally. You can extract a Google Doc's ID directly from its URL. For example, the ID of `https://docs.google.com/document/d/1wf-y2pd9C878Oh-FmLH7Q_BQkljdm6TQal-c1pUfrec/edit` is `1wf-y2pd9C878Oh-FmLH7Q_BQkljdm6TQal-c1pUfrec`.

## Usage

To use this loader, you simply need to pass in an array of Google Doc IDs.

```python
from loader_hub import GoogleDocsReader

gdoc_ids = ['1wf-y2pd9C878Oh-FmLH7Q_BQkljdm6TQal-c1pUfrec']
documents = GoogleDocsReader.load_data(document_ids=gdoc_ids)
```

## Examples

This loader is designed to be used as a way to load data into [GPT Index](https://github.com/jerryjliu/gpt_index/tree/main/gpt_index) and/or subsequently used as a Tool in a [LangChain](https://github.com/hwchase17/langchain) Agent.

### GPT Index

```python
from loader_hub import GoogleDocsReader
from gpt_index import GPTSimpleVectorIndex

gdoc_ids = ['1wf-y2pd9C878Oh-FmLH7Q_BQkljdm6TQal-c1pUfrec']
documents = GoogleDocsReader.load_data(document_ids=gdoc_ids)
index = GPTSimpleVectorIndex(documents)
index.query('Where did the author go to school?')
```

### LangChain

Note: Make sure you change the description of the `Tool` to match your use-case.

```python
from loader_hub import GoogleDocsReader
from gpt_index import GPTSimpleVectorIndex
from langchain.agents import initialize_agent, Tool
from langchain.llms import OpenAI
from langchain.chains.conversation.memory import ConversationBufferMemory

gdoc_ids = ['1wf-y2pd9C878Oh-FmLH7Q_BQkljdm6TQal-c1pUfrec']
documents = GoogleDocsReader.load_data(document_ids=gdoc_ids)
index = GPTSimpleVectorIndex(documents)

tools = [
    Tool(
        name="Google Doc Index",
        func=lambda q: index.query(q),
        description=f"Useful when you want answer questions about the Google Documents.",
    ),
]
llm = OpenAI(temperature=0)
memory = ConversationBufferMemory(memory_key="chat_history")
agent_chain = initialize_agent(
    tools, llm, agent="zero-shot-react-description", memory=memory
)

output = agent_chain.run(input="Where did the author go to school?")
```