1) Add these image files to the website, they are in the \files folder

Alex_B.png
Alex_S.jpg
Zeph_J.jpg
Sunder_S.png

2) Given the Python notebook below which is our current state of the project, edit the details described in the website.

```py
# STEP 0: Imports and Setup
# @title -- Installs

!pip install -qU langchain-google-genai langgraph pinecone langchain[openai]
!pip install -qU dspy xlsxwriter
!pip install -q gradio
# @title -- Imports

import sys, os, re, io, string, json, time, copy, math, random, gc
import pathlib, urllib.request, tarfile
from google.colab import userdata, files, drive
drive.mount('/content/drive')

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from tqdm.auto import tqdm

import uuid
import spacy
import heapq
from sklearn.preprocessing import normalize

from langchain import tools
from langchain_core.tools import tool
from langchain.chat_models import init_chat_model
from langchain_core.messages import BaseMessage, AIMessage, ToolMessage, SystemMessage

from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages
from langgraph.prebuilt import ToolNode, tools_condition
from langgraph.checkpoint.memory import InMemorySaver

from openai import OpenAI
from pinecone import Pinecone, ServerlessSpec

import dspy
from typing import List, Literal, TypedDict, Dict, Any, Annotated
from typing_extensions import TypedDict

import xlsxwriter, contextlib

import email, imaplib
from email.header import decode_header

# @title -- Environment (& API) setups

os.environ["OPENAI_API_KEY"] = userdata.get("OPENAI_NETWATCH")
OPENAI_NETWATCH = os.environ["OPENAI_API_KEY"]
os.environ["PINECONE_API_KEY"] = userdata.get("PINECONE_NETWATCH")
PINECONE_API_KEY = os.environ["PINECONE_API_KEY"]

pc = Pinecone(api_key=PINECONE_API_KEY)
client = OpenAI(api_key=OPENAI_NETWATCH)

index_name = "netwatch-claims"
nlp = spacy.load("en_core_web_sm")
EMBED_MODEL = "text-embedding-3-large"
EMBED_DIMS =  3072

# DSPy/OpenAI logins:

lm_gpt = dspy.LM('openai/gpt-5-mini', api_key = OPENAI_NETWATCH, temperature=1.0, max_tokens=16000)
dspy.configure(lm=lm_gpt, cache=True)
output = lm_gpt(messages=[{"role": "user", "content": "If you see this return 'Confirmation DSPy and OpenAI API working'."}])
print(output[0])


llm = init_chat_model("gpt-5-mini", model_provider="OpenAI")
output = llm.invoke("If you see this return 'Confirmation Langchain and OpenAI API working")
print(output.content)

print("\nFiles in Netwatch folder:")
netwatch_folder = pathlib.Path("/content/drive/MyDrive/NetWatch")
if netwatch_folder.exists():
    for entry in netwatch_folder.iterdir():
        kind = "DIR " if entry.is_dir() else "FILE"
        print(f"{kind:4}  {entry.name}")
else:
    print("Path not found:", netwatch_folder)

# Step 1: Import data (email and CSV)
# @title -- Import excel and CSV into DF

excel_datastore_path = netwatch_folder / "claim_examples.xlsx"

with pd.ExcelFile(excel_datastore_path) as xls:
    example_claims_df = pd.read_excel(xls, sheet_name=xls.sheet_names[0], header=1, usecols="A:P")
    examples_msg_df = pd.read_excel(xls, sheet_name=xls.sheet_names[1], header=1)
    enron_claims_df = pd.read_excel(xls, sheet_name=xls.sheet_names[2], header=0)
    enron_msgs_df = pd.read_excel(xls, sheet_name=xls.sheet_names[3], header=0)
    gmail_claims_df = pd.read_excel(xls, sheet_name=xls.sheet_names[4], header=0)
    gmail_msgs_df = pd.read_excel(xls, sheet_name=xls.sheet_names[5], header=0)

    example_claims_df["msg_id"] = pd.to_numeric(example_claims_df["msg_id"], errors="coerce").astype("Int64")
    enron_claims_df["msg_id"] = pd.to_numeric(enron_claims_df["msg_id"], errors="coerce").astype("Int64")
    enron_msgs_df["msg_id"] = pd.to_numeric(enron_msgs_df["msg_id"], errors="coerce").astype("Int64")
    gmail_claims_df["msg_id"] = pd.to_numeric(gmail_claims_df["msg_id"], errors="coerce").astype("Int64")
    gmail_msgs_df["msg_id"] = pd.to_numeric(gmail_msgs_df["msg_id"], errors="coerce").astype("Int64")
    example_claims_df = example_claims_df.drop(columns=["value_of_info", "risk_rating", "Context"])

display(example_claims_df.head(2))
display(enron_claims_df.head(2))
display(enron_msgs_df.head(2))
display(gmail_claims_df.head(2))
display(gmail_msgs_df.head(2))



# @title -- Import emails into DF


USERNAME = "netwatch5541@gmail.com"
PASSWORD = userdata.get('APP_PASSWORD')
IMAP_SERVER = "imap.gmail.com"

print("Connecting to Gmail...")
mail = imaplib.IMAP4_SSL(IMAP_SERVER)
mail.login(USERNAME, PASSWORD)
print("Login successful!")

mail.select("inbox")

#status, email_ids = mail.search(None, "UNSEEN")
status, email_ids = mail.search(None, "ALL")

id_list = email_ids[0].split()
print(email_ids)
print(id_list)
'''
if not id_list:
    print("No new unread emails found.")
else:
    print(f"Found {len(id_list)} unread emails...")
'''

for email_id in id_list:
    status, msg_data = mail.fetch(email_id, "(RFC822)")
    mail.store(email_id, "+FLAGS", "\\Seen")

    for response_part in msg_data:
        if isinstance(response_part, tuple):
            msg = email.message_from_bytes(response_part[1])

            email_subject, encoding = decode_header(msg["Subject"])[0]
            if isinstance(email_subject, bytes):
                email_subject = email_subject.decode(encoding if encoding else "utf-8")

            email_sender, encoding = decode_header(msg.get("From"))[0]
            if isinstance(email_sender, bytes):
                email_sender = email_sender.decode(encoding if encoding else "utf-8")

            email_date = msg.get("Date", "")

            if msg.is_multipart():
                email_body = ''
                for part in msg.walk():
                    if part.get_content_type() == "text/plain":
                        email_body += part.get_payload(decode=True).decode()
            else:
                email_body = msg.get_payload(decode=True).decode()

        # print(msg_data)
        print("-" * 30)
        print(f"Msg_id: {email_id}, Date: {email_date}")
        print(f"Subject: {email_subject}")
        print(f"From: {email_sender}")
        print(f"Body (2): {email_body[:100]}")

        gmail_msgs_df.loc[len(gmail_msgs_df)] = {
            "msg_id": email_id.decode() if isinstance(email_id, bytes) else email_id,
            "raw_text": "TO SORT",
            "email_from": email_sender,
            "subject": email_subject,
            "body": email_body,
            "context": "TO SORT",
            "date": email_date,
        }


#   add_to_db({'sender': from_sender, 'subject': subject, 'text': body, 'context':"FAKE", 'date':"NOW"})

########### SUNDA TO DO
# Check if emails already in gmail_msgs_df
# Insert any new emails into gmail_msgs_df
# Update 'excel_datastore_path' XLSX, sheet 5, with new DF


print("-" * 30)
mail.close()
mail.logout()
print("Connection closed.")
# STEP 2: Break email into Claims

# STEP 3: Filter Claims based on DSPy classifiers

# STEP 4: RAG (embedding, retrieval) pipeline.
# @title -- Embedding of Claims into Pinecone Vector DB

claim_test = {
    "claim": "This is fun typing fake emails - I dont know what you are talking about.",
    "context": "An important email",
    "sender": "Slinga@abc.com",
    "subject": "Test email",
    "date": "Today",
}

emails_test = [claim_test]

def get_embedding(data):
    resp = client.embeddings.create(model=EMBED_MODEL, input=data)
    return resp.data[0].embedding

def build_db(emails):
  for claim in emails:
    # this is where we CAN split email into statements
    # also where we need to unpack the email structure
    # for statment in email:
    entity = {}
    entity['text'] = claim['claim']
    entity['context'] = claim['context']
    entity['sender'] = claim['sender']
    entity['subject'] = claim['subject']
    entity['date'] = claim['date']
    add_to_db(entity)


def add_to_db(email):
    if index_name not in pc.list_indexes().names():
      pc.create_index(
          name=index_name,
          dimension=EMBED_DIMS,
          metric="cosine",
          spec=ServerlessSpec(cloud="aws", region="us-east-1")
      )

    index = pc.Index(index_name)

    # sarcasm_rating = sarcasm(email['text'])
    # confidentiality_rating = confidentiality(email['text'])
    # etc.
    # if sarcasm > XYZ or confidentailty...
    # return

    emb = get_embedding(email['text'])
    vector = {
        "id": str(uuid.uuid4()),
        "values": emb,
        "metadata": {
            "text": email['text'],
            "context": email['context'],
            "sender": email['sender'],
            "subject": email['subject'],
            "date": email['date'],
        },
    }
            # "sarcasm": sarcasm_rating
            # "confidentiality": confidentiality_rating

    index.upsert(vectors=[vector])
    return None

build_db(emails_test)
print("Done.")
# @title -- Retrieval tools for RAG system

def db_lookup(query: str, n: int = 5) -> list[dict]:
    """
    Function to search the database. Returns a list of dicts each containing the Pinecone match 'id' plus all metadata fields.
    """
    try:
        embedding_response = get_embedding(query)   # -> list[float]
    except Exception:
        return []

    index = pc.Index(index_name)
    response = index.query(
        vector=embedding_response,
        top_k=n,
        include_metadata=True
    )

    # keep id AND metadata (metadata alone has no id). Each neighbor is a dict, containg all the relevant info.
    matches = response.get("matches", []) or []
    return [{"id": m["id"], **(m.get("metadata") or {})} for m in matches]

@tool
def lookup_in_rag(query: str, n: int=5) -> list[str]:
    """
    All in one function to search the database, perform ranking, etc. Returns the top-n neighbor dicts (each has 'id' + metadata fields).
    """
    neighbors = db_lookup(query, n * 2)

    if not neighbors: return []

    # initialize id -> score map
    scores = {nb["id"]: 0.0 for nb in neighbors}

    for neighbor in neighbors:

        # REPLACE WITH REAL FUNCTION BELOW
        import random
        rng = random.Random(42)
        scores[nb["id"]] += rng.random()

        # This is where the rrf goes, for ranking DURING retrieval
        # pre-database culling for confidentiality or whatever happens in the add_to_db function
        # EXAMPLE 1
        # scores[neighbor] += 1 if sarcasm(neighbor['text']) == 0
        # or with pre-computed rankings:
        # scores[neighbor] += 1 if neighbor['sarcasm'] == 0
        #
        # EXAMPLE 2 (pseudo code)
        # get sarcasm from all neighbors. sort that list.
        # add 1/n to each score, where n is the neighbors index in the sorted list.
        ...

    # pick top-n ids by score
    top_ids = [k for k, _ in heapq.nlargest(n, scores.items(), key=lambda kv: kv[1])]

    # map back to full neighbor dicts
    id_to_nb = {nb["id"]: nb for nb in neighbors}
    return [id_to_nb[i] for i in top_ids]

# @title -- LangGraph structure - and using tools for retrieval

tools = [lookup_in_rag]
llm_with_tools = llm.bind_tools(tools)

class State(TypedDict):
  messages: Annotated[list, add_messages]

def chatbot(state: State):
  existing_messages = state["messages"]
  instructions = [
      SystemMessage(content="You are an administrative assistant, tasked with answering questions and sourcing data from a database of corporate emails."),
      SystemMessage(content="You have access to a 'lookup_in_rag' tool that will allow you to find relevant emails in the database."),
  ]
  messages = instructions + existing_messages
  return {'messages': existing_messages + [llm_with_tools.invoke(messages)]}

graph_builder = StateGraph(State)
graph_builder.add_node("chatbot", chatbot)
tool_node = ToolNode(tools=tools)
graph_builder.add_node("tools", tool_node)

graph_builder.add_conditional_edges("chatbot", tools_condition)
graph_builder.add_edge("tools", "chatbot")
graph_builder.add_edge(START, "chatbot")
graph_builder.add_edge("chatbot", END)

graph = graph_builder.compile(checkpointer=InMemorySaver())

def prompt_rag(query):
  return graph.invoke({"messages": [{"role": "user", "content": query}]}, config={'configurable': {'thread_id': '1'}})

from IPython.display import Image, display
# just for fun, to see what the rag looks like
try:
    display(Image(graph.get_graph().draw_mermaid_png()))
except Exception:
    pass

# Examples of RAG calls

out = prompt_rag("What tools do you have available?")
print(out["messages"][-1].content)


out = prompt_rag("What did Sunder say?")
print(out["messages"][-1].content)
# STEP 5: User interface with RAG via Gradio
import gradio as gr
import time

def my_chatbot_function(message, history):
    bot_response = prompt_rag(message)

    return bot_response['messages'][-1].content

# THIS IS THE USER NAME AND PASSWORD COMBINATION - MAY NEED TO BE RECOPIED IN...
auth = ('teamNetwatch', '5541FinalProject')
gr.ChatInterface(fn=my_chatbot_function, title="Netwatch Final Project").launch(
    share=True,
    auth=auth
)
## old code from sunder
# ==============================
# K-SIMILARITY CHECK BETWEEN PROMPT AND QUERY
# ==============================
def similarity(query, index, k=2):# 2 most similar values will be considered for context
    query_emb = get_embedding(query)
    query_vector = np.array(query_emb).astype("float32").tolist()
    response = index.query(vector=query_vector, top_k=k + 5, include_metadata=True)
    seen = set()
    unique_chunks = []
    for match in response["matches"]:
        text = match["metadata"]["text"]
        if text not in seen:
            seen.add(text)
            unique_chunks.append(text)
        if len(unique_chunks) == k:
            break
    return unique_chunks


# ==============================
# TEXT EXTRACTION
# ==============================
def extract_text_from_pdf(pdf_path):
    text = ""
    doc = fitz.open(pdf_path)
    for page in doc:
        text += page.get_text()
    return text

# ==============================
# TEXT CHUNKING
# ==============================
def chunk_text(text):
    doc = nlp(text)
    return [sent.text.strip() for sent in doc.sents if sent.text.strip()]


# ==============================
# API GEMINI PROMPTing
# ==============================
def ask_gemini(query,context_chunks):
    context = "\n".join(context_chunks)
    prompt = f"""Use the following information to answer the question:
    {context}
    Question: {query}
    Answer:"""
    response = client.models.generate_content(
    model="gemini-2.5-flash", contents=prompt)
    return response.text


# ==============================
# MAIN CODE
# ==============================
if __name__ == "__main__":
    lines, index = knowledge_base()
    while True:
        query = input("enter the question: ")
        context_chunks = similarity(query, index, k=2)
        print(context_chunks)
        response = ask_gemini(query, context_chunks)
        print("\n Gemini's Answer:")
        print(response.strip())

# ==============================
# KNOWLEDGE BASE CREATION/VECTOR DATABASE
# ==============================
'''
def knowledge_base(pdf_folder="pdfs"):
    if index_name not in pc.list_indexes().names():
        pc.create_index(
            name=index_name,
            dimension=768,
            metric="cosine",
            spec=ServerlessSpec(cloud="aws", region="us-east-1")
        )

    index = pc.Index(index_name)

    all_chunks = []

    for filename in os.listdir(pdf_folder):
        if filename.endswith(".pdf"):
            pdf_path = os.path.join(pdf_folder, filename)
            full_text = extract_text_from_pdf(pdf_path)
            chunks = chunk_text(full_text)
            all_chunks.extend(chunks)

    vectors = []
    for chunk in all_chunks:
        emb = get_embedding(chunk)
        vectors.append({
            "id": str(uuid.uuid4()),
            "values": emb,
            "metadata": {"text": chunk}
        })
    index.upsert(vectors=vectors)
    return all_chunks, index
    '''

def knowledge_base(pdf_file="ATM usable Card 3.pdf"):
    if index_name not in pc.list_indexes().names():
        pc.create_index(
            name=index_name,
            dimension=768,
            metric="cosine",
            spec=ServerlessSpec(cloud="aws", region="us-east-1")
        )

    index = pc.Index(index_name)

    all_chunks = []
    full_text = extract_text_from_pdf(pdf_file)
    chunks = chunk_text(full_text)
    all_chunks.extend(chunks)
    vectors = []
    for chunk in all_chunks:
        emb = get_embedding(chunk)
        vectors.append({
            "id": str(uuid.uuid4()),
            "values": emb,
            "metadata": {"text": chunk}
        })
    index.upsert(vectors=vectors)
    return all_chunks, index
```