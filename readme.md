# RAG-Documents-Method-1

This repository contains an implementation of Retrieval-Augmented Generation (RAG) using LangChain and Pinecone for document retrieval and Google Generative AI for generating responses.

## Installation

Run the following command to install the required dependencies:

```sh
pip install -qU langchain-pinecone langchain-google-genai
```

## Code Implementation

```python
# -*- coding: utf-8 -*-
"""RAG-Documents-Method -1 .ipynb
Automatically generated by Colab.

Original file is located at
    https://colab.research.google.com/github/Tariqameenkhan/LangChain-RAG/blob/main/RAG-Documents-Method%20-1%20.ipynb
"""

from pinecone import Pinecone, ServerlessSpec
from google.colab import userdata
import time
import os
from langchain_google_genai import GoogleGenerativeAIEmbeddings
from langchain_pinecone import PineconeVectorStore
from uuid import uuid4
from langchain_core.documents import Document
from langchain_google_genai import ChatGoogleGenerativeAI

# Get API Key
pinecone_api_key = userdata.get("PINECONE_API_KEY")
pc = Pinecone(api_key=pinecone_api_key)

# Create Index
index_name = "langchain-rag-project2"
pc.create_index(
        name=index_name,
        dimension=768,
        metric="cosine",
        spec=ServerlessSpec(cloud="aws", region="us-east-1"),
)

index = pc.Index(index_name)

# Set up Google API Key
os.environ["GOOGLE_API_KEY"] = userdata.get("GOOGLE_API_KEY")
embeddings = GoogleGenerativeAIEmbeddings(model="models/embedding-001")

# Initialize Vector Store
vector_store = PineconeVectorStore(index=index, embedding=embeddings)

# Sample Documents
documents = [
    Document(page_content="I had chocolate chip pancakes and scrambled eggs for breakfast this morning.", metadata={"source": "tweet"}),
    Document(page_content="Building an exciting new project with LangChain - come check it out!", metadata={"source": "tweet"}),
    Document(page_content="Wow! That was chocolate chip pancakes. I can't wait to see it again.", metadata={"source": "tweet"}),
    Document(page_content="LangGraph is the best framework for building stateful, agentic applications!", metadata={"source": "tweet"}),
    Document(page_content="I had a bad test chocolate chip pancakes very bad :(", metadata={"source": "tweet"}),
]

# Generate UUIDs for documents
uuids = [str(uuid4()) for _ in range(len(documents))]
vector_store.add_documents(documents=documents, ids=uuids)

# Initialize Chat Model
llm = ChatGoogleGenerativeAI(
    model="gemini-1.5-flash",
    temperature=0,
    max_tokens=None,
    timeout=None,
    max_retries=2,
)

# Interactive Query Loop
while True:
    user_input = input("Ask something: ")  # Get user input inside the loop
    if user_input.lower() == "quit":
        break
    else:
        results = vector_store.similarity_search(user_input, k=1)
        final_answer = llm.invoke(f"Answer this user query: {user_input}, here are some references to answer: {results}")
        print(f"*Initial query answer: {final_answer}")
```

## Features
- Uses Pinecone as a vector store for storing and retrieving documents.
- Embeds documents using Google's Generative AI.
- Implements a chatbot that answers user queries based on retrieved documents.
- Provides interactive querying with a user loop.

## Usage
Run the script and enter queries. Type `quit` to exit the loop.
