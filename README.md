# SQL-ALCHEMY-AnaliseAdventureWorks2022
Analise de dados da tabela "Production.Product" usando SQLAlchemy, Pandas, e PowerBI 




```python

from sqlalchemy import create_engine, MetaData, Table, Column, asc, desc
from sqlalchemy import select
from sqlalchemy.ext.automap import automap_base

from sqlalchemy.orm import sessionmaker
from sqlalchemy.orm import Session

import pandas as pd
import pyodbc

```


# Construindo a Engine Para connectar o Banco de Dados SQL (SSMS) Usando Python (SQLAlchemy)

```python
server = "DESKTOP-USPUN2G\\SQLEXPRESS"
database = "AdventureWorks2022"
connection_url = f"mssql+pyodbc://@{server}/{database}?driver=ODBC+Driver+18+for+SQL+Server&trusted_connection=yes&Encrypt=no"
engine = create_engine(connection_url, echo=True)

```
# Mapeando Cada Tabela do banco de dados como objetos classe: class.object

```python
Base= automap_base()
Base.prepare(engine, reflect= True, schema="Production")
print(Base.classes.keys())
Product = Base.classes.Product
ProductPhoto = Base.classes.ProductProductPhoto

```
# Fazendo a query usando SQL Alchemy

```python
querySql = (
    select(
        Product.ProductID,
        Product.Name,
        Product.DaysToManufacture,
        Product.ProductNumber,
        Product.StandardCost,
        Product.Weight,
        Product.ListPrice,
        Product.Color
    )
    .join(ProductPhoto, Product.ProductID == ProductPhoto.ProductID)

)


```
# Rodando a query com alchemy e exportando os resultados como objeto dataframe do pandas (df.pd)

```python
product_df = pd.read_sql(querySql, engine)

```

# Fazendo limpeza de dados usando pandas, substituindo trechos números que possuem vírgulas por pontos e os transformando de string para float

```python
product_df["ListPrice"] = product_df["ListPrice"].astype(str).str.replace(",", ".").astype(float)
product_df["StandardCost"] = product_df["StandardCost"].astype(str).str.replace(",", ".").astype(float)
product_df["ProductProfit"] = product_df["ListPrice"] - product_df["StandardCost"]
product_df["Color"] = product_df["Color"].fillna("Unknown")

```


# Lendo o dataframe e verificando se a limpeza foi feita de forma correta

```python
df = pd.read_csv("AdventureWorks.csv")
print(df.sort_values("ProductProfit", ascending=False))

```




