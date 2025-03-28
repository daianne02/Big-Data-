# Big-Data-
Trabalho
Código Python (ingestão de dados):
from pymongo import MongoClient
from faker import Faker
import random

# Conectar ao MongoDB
client = MongoClient("mongodb://localhost:27017/")
db = client["petshop_db"]
vendas = db["vendas"]

# Gerar dados fictícios
fake = Faker()
produtos = ["Ração Premium", "Coleira", "Shampoo Pet", "Brinquedo"]
for _ in range(100):
    venda = {
        "cliente": fake.name(),
        "produto": random.choice(produtos),
        "quantidade": random.randint(1, 5),
        "data": fake.date_time_this_year().isoformat(),
        "valor": round(random.uniform(10, 100), 2)
    }
    vendas.insert_one(venda)

print("Dados inseridos com sucesso!")

Código Python (ETL com Pandas):
import pandas as pd
from pymongo import MongoClient

# Conectar ao MongoDB
client = MongoClient("mongodb://localhost:27017/")
db = client["petshop_db"]
vendas = db["vendas"]

# Ler dados
data = list(vendas.find())
df = pd.DataFrame(data)

# Limpeza e transformação
df = df.drop_duplicates(subset=["_id"])
df["total"] = df["quantidade"] * df["valor"]

# Exportar para CSV
df.to_csv("vendas_processadas.csv", index=False)
print("Dados processados e salvos em CSV!")


Código Python (FastAPI):

from fastapi import FastAPI
from pymongo import MongoClient

app = FastAPI()

# Conectar ao MongoDB
client = MongoClient("mongodb://localhost:27017/")
db = client["petshop_db"]
vendas = db["vendas"]

@app.get("/vendas")
def get_vendas():
    data = list(vendas.find({}, {"_id": 0}))  # Excluir campo _id
    return data

@app.get("/estatisticas")
def get_estatisticas():
    total_vendas = vendas.count_documents({})
    valor_total = sum(v["valor"] * v["quantidade"] for v in vendas.find())
    return {"total_vendas": total_vendas, "valor_total": round(valor_total, 2)}

# Rodar com: uvicorn nome_do_arquivo:app --reload

Código Python (Streamlit):

Copiar
import streamlit as st
import requests
import pandas as pd

st.title("Dashboard Pet Shop")

# Consultar API
vendas = requests.get("http://localhost:8000/vendas").json()
estatisticas = requests.get("http://localhost:8000/estatisticas").json()

# Exibir estatísticas
st.write(f"Total de Vendas: {estatisticas['total_vendas']}")
st.write(f"Valor Total: R${estatisticas['valor_total']}")

# Gráfico
df = pd.DataFrame(vendas)
st.bar_chart(df.groupby("produto")["total"].sum())

# Rodar com: streamlit run nome_do_arquivo.py
