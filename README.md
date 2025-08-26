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

### E temos essa tabela como resultado das primeiras 15 linhas usando Order by ListPrice DESC

| ProductID | Name                    | DaysToManufacture | ProductNumber | StandardCost | Weight  | ListPrice | Color  |
|-----------|------------------------|-----------------|---------------|--------------|--------|-----------|--------|
| 749       | Road-150 Red, 62       | 4               | BK-R93R-62    | 2171,2942    | 15.00  | 3578,27   | Red    |
| 750       | Road-150 Red, 44       | 4               | BK-R93R-44    | 2171,2942    | 13.77  | 3578,27   | Red    |
| 751       | Road-150 Red, 48       | 4               | BK-R93R-48    | 2171,2942    | 14.13  | 3578,27   | Red    |
| 752       | Road-150 Red, 52       | 4               | BK-R93R-52    | 2171,2942    | 14.42  | 3578,27   | Red    |
| 753       | Road-150 Red, 56       | 4               | BK-R93R-56    | 2171,2942    | 14.68  | 3578,27   | Red    |
| 771       | Mountain-100 Silver, 38| 4               | BK-M82S-38    | 1912,1544    | 20.35  | 3399,99   | Silver |
| 772       | Mountain-100 Silver, 42| 4               | BK-M82S-42    | 1912,1544    | 20.77  | 3399,99   | Silver |
| 773       | Mountain-100 Silver, 44| 4               | BK-M82S-44    | 1912,1544    | 21.13  | 3399,99   | Silver |
| 774       | Mountain-100 Silver, 48| 4               | BK-M82S-48    | 1912,1544    | 21.42  | 3399,99   | Silver |
| 775       | Mountain-100 Black, 38 | 4               | BK-M82B-38    | 1898,0944    | 20.35  | 3374,99   | Black  |
| 776       | Mountain-100 Black, 42 | 4               | BK-M82B-42    | 1898,0944    | 20.77  | 3374,99   | Black  |
| 777       | Mountain-100 Black, 44 | 4               | BK-M82B-44    | 1898,0944    | 21.13  | 3374,99   | Black  |
| 778       | Mountain-100 Black, 48 | 4               | BK-M82B-48    | 1898,0944    | 21.42  | 3374,99   | Black  |
| 789       | Road-250 Red, 44       | 4               | BK-R89R-44    | 1518,7864    | 14.77  | 2443,35   | Red    |
| 790       | Road-250 Red, 48       | 4               | BK-R89R-48    | 1518,7864    | 15.13  | 2443,35   | Red    |

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

# Apos a limpeza dos dados temos essa tabela, e finalmente podemos extrair os dados para CSV

| ProductID | Name                    | DaysToManufacture | ProductNumber | StandardCost | Weight  | ListPrice | ProductProfit | Color   |
|-----------|------------------------|-----------------|---------------|--------------|--------|-----------|---------------|---------|
| 749       | Road-150 Red, 62       | 4               | BK-R93R-62    | 2171.2942    | 15.00  | 3578.27   | 1407.00       | Red     |
| 750       | Road-150 Red, 44       | 4               | BK-R93R-44    | 2171.2942    | 13.77  | 3578.27   | 1407.00       | Red     |
| 751       | Road-150 Red, 48       | 4               | BK-R93R-48    | 2171.2942    | 14.13  | 3578.27   | 1407.00       | Red     |
| 752       | Road-150 Red, 52       | 4               | BK-R93R-52    | 2171.2942    | 14.42  | 3578.27   | 1407.00       | Red     |
| 753       | Road-150 Red, 56       | 4               | BK-R93R-56    | 2171.2942    | 14.68  | 3578.27   | 1407.00       | Red     |
| 771       | Mountain-100 Silver, 38| 4               | BK-M82S-38    | 1912.1544    | 20.35  | 3399.99   | 1487.84       | Silver  |
| 772       | Mountain-100 Silver, 42| 4               | BK-M82S-42    | 1912.1544    | 20.77  | 3399.99   | 1487.84       | Silver  |
| 773       | Mountain-100 Silver, 44| 4               | BK-M82S-44    | 1912.1544    | 21.13  | 3399.99   | 1487.84       | Silver  |
| 774       | Mountain-100 Silver, 48| 4               | BK-M82S-48    | 1912.1544    | 21.42  | 3399.99   | 1487.84       | Silver  |
| 775       | Mountain-100 Black, 38 | 4               | BK-M82B-38    | 1898.0944    | 20.35  | 3374.99   | 1476.90       | Black   |
| 776       | Mountain-100 Black, 42 | 4               | BK-M82B-42    | 1898.0944    | 20.77  | 3374.99   | 1476.90       | Black   |
| 777       | Mountain-100 Black, 44 | 4               | BK-M82B-44    | 1898.0944    | 21.13  | 3374.99   | 1476.90       | Black   |
| 778       | Mountain-100 Black, 48 | 4               | BK-M82B-48    | 1898.0944    | 21.42  | 3374.99   | 1476.90       | Black   |
| 789       | Road-250 Red, 44       | 4               | BK-R89R-44    | 1518.7864    | 14.77  | 2443.35   | 924.56        | Red     |
| 790       | Road-250 Red, 48       | 4               | BK-R89R-48    | 1518.7864    | 15.13  | 2443.35   | 924.56        | Red     |







