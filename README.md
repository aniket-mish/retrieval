# an mvp rag system

a simple mvp rag system which you can build in an hour. obviously an enterprise grade product would be way too complex but here i show you how you can start. system architecture doesn't change a lot tbh. rag is nothing but a fancy way of saying information retrieval.

<img width="1196" alt="Screenshot 2024-06-19 at 4 17 39 PM" src="https://github.com/aniket-mish/llm-headline-generator/assets/71699313/4d2fde4a-92ad-4e9d-bb61-96367c0cb7db">

## data

| name | description | industry | category | url | headline | subhead | cta | founded | location | tags |
| -------- | ------- | -------- | -------- | ------- | -------- | ------- | -------- | ------- | -------- | ------- |
| GitLab | GitLab is the first single application for the entire DevOps lifecycle. Only GitLab enables Concurrent DevOps, unlocking organizations from the constraints of today’s toolchain. GitLab provides unmatched visibility, radical new levels of efficiency and comprehensive governance to significantly compress the time between planning a change and monitoring its effect. This makes the software lifecycle 200% faster, radically improving the speed of business. GitLab and Concurrent DevOps collapses cycle times by driving higher efficiency across all stages of the software development lifecycle. For the first time, Product, Development, QA, Security, and Operations teams can work concurrently in a single application. There’s no need to integrate and synchronize tools, or waste time waiting for handoffs. Everyone contributes to a single conversation, instead of managing multiple threads across disparate tools. And only GitLab gives teams complete visibility across the lifecycle with a single, trusted source of data to simplify troubleshooting and drive accountability. All activity is governed by consistent controls, making security and compliance first-class citizens instead of an afterthought. Built on Open Source, GitLab leverages the community contributions of thousands of developers and millions of users to continuously deliver new DevOps innovations. More than 100,000 organizations from startups to global enterprise organizations, including Ticketmaster, Jaguar Land Rover, NASDAQ, Dish Network and Comcast trust GitLab to deliver great software at new speeds. | B2B | Engineering, Product and Design | http://gitlab.com/ | Software. Faster. | GitLab is the most comprehensive AI-powered DevSecOps Platform. | Get free trial | 2012 | San Francisco | W15, PUBLIC, DEVELOPER-TOOLS, DEVSECOPS, OPEN-SOURCE, SAN FRANCISCO |

## stack im using

im using [LanceDB](https://lancedb.github.io/lancedb/) to build a mvp fast.

```bash
pip install lancedb
```

## load a bi-encoder

```python
db = lancedb.connect("/tmp/db")
model = get_registry().get("sentence-transformers").create(name="BAAI/bge-small-en-v1.5", device="cpu")
```

## load metadata

```python
class Documents(LanceModel):
    text: str = model.SourceField()
    vector: Vector(model.ndims()) = model.VectorField()

table = db.create_table("pd_table", schema=Documents)
```

## read data

```python
df = pd.read_csv("data.csv")
```

## transform data into required format

```python
df["text"] = df[df.columns.to_list()].astype(str).agg(" ".join, axis=1)

docs = [{"text": x} for x in df["text"]]
```

## add docs to a table

```python
table.add(docs)
```

## create a index on the column

```python
table.create_fts_index("text")
```

## reranking

```python
from lancedb.rerankers import CohereReranker

reranker = CohereReranker(api_key=cohere_api_key)
```

## query

```python
query = "How many repositories does github have?"

result = table.search(query, query_type="hybrid").limit(5).rerank(reranker=reranker).to_list()
```

## chat

```python
from langchain_cohere import ChatCohere

prompt = f"""
Context is given below:
{context}

Answer the question using the context: {query}
"""

cohere_chat_model = ChatCohere(cohere_api_key=key)
response = cohere_chat_model.invoke(prompt)
print(f"Response: \n {response.content}")
```
