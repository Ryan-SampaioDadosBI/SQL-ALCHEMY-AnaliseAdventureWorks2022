# SQL-ALCHEMY-AnaliseAdventureWorks2022
Analise de dados da tabela "Production.Product" usando SQLAlchemy, Pandas, e PowerBI 

# Analise de Dados
Este projeto tem como objetivo analisar dados da tabela Production.Product do banco AdventureWorks 2022, aplicando técnicas de SQLAlchemy (ORM), pandas para manipulação e limpeza dos dados, e Power BI para visualização.

#Banco de Dados AdventureWorks2022
Este projeto utiliza como fonte de dados o **AdventureWorks 2022**, um banco de dados de exemplo amplamente utilizado para práticas de SQL e análise de dados.  

O banco contém diversas tabelas relacionadas a produtos, vendas, funcionários, clientes e produção. Neste projeto, o foco está na tabela **`Production.Product`**, onde analisamos informações como:

- Identificação do produto (`ProductID` e `ProductNumber`)  
- Nome do produto (`Name`)  
- Tempo de fabricação (`DaysToManufacture`)  
- Custo e preço de venda (`StandardCost` e `ListPrice`)  
- Peso (`Weight`)  
- Cor (`Color`)
- 
# Requisitos
```python

from sqlalchemy import create_engine, MetaData, Table, Column, asc, desc
from sqlalchemy import select
from sqlalchemy.ext.automap import automap_base

from sqlalchemy.orm import sessionmaker
from sqlalchemy.orm import Session

import pandas as pd
import pyodbc

```
# SQL Alchemy
SQLAlchemy é usado para fazer a ponte entre o banco de dados e python sem a necessidade de usar código SQL "Puro"
# 1  SQL Alchemy Engine
Construimos a engine que faz a ponte entre python e o banco de dados com o código

```python
server = "DESKTOP-USPUN2G\\SQLEXPRESS"
database = "AdventureWorks2022"
connection_url = f"mssql+pyodbc://@{server}/{database}?driver=ODBC+Driver+18+for+SQL+Server&trusted_connection=yes&Encrypt=no"
engine = create_engine(connection_url, echo=True)

```

# 1.1 SQL Alchemy Automapping
Neste Código utilizamos o SQLAlchemy Automap para mapear as tabelas do banco de dados como objetos Python. Dessa forma, cada tabela é representada como uma classe, e cada linha da tabela um objeto. (e.g, Product.ListPrice)
```python
Base= automap_base()
Base.prepare(engine, reflect= True, schema="Production")
print(Base.classes.keys())
Product = Base.classes.Product
ProductPhoto = Base.classes.ProductProductPhoto

```

# 1.2 SQL ALchemy Query
Após mapear, podemos usar o ORM e python para fazer queries
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
Este Query no SQL Alchemy usando o ORM seria equivalente a:
```sql
SELECT 
    Product.ProductID,
    [Name],
    DaysToManufacture,
    ProductNumber,
    StandardCost,
    [Weight],
    ListPrice,
    [Color]


FROM
	Production.Product

INNER JOIN Production.ProductProductPhoto as product_photo
	on Product.ProductID = product_photo.ProductID
```

E que tem como resultado (15 primeiros resultados) com order by(ListPrice) DESC


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



# SQL Alchemy Para CSV
Para transformar o query, em um dataframe (pd.df) usamos para que possa ser feita a limpeza dos dados
```python
product_df = pd.read_sql(querySql, engine)

```

# Limpeza de Dados com Pandas
Após a extração, números de algumas colunas que separam as casas decimais estão como vírgula ao invés de ponto (virgula é considerado separador em arquivos CSV)
```python
product_df["ListPrice"] = product_df["ListPrice"].astype(str).str.replace(",", ".").astype(float)
product_df["StandardCost"] = product_df["StandardCost"].astype(str).str.replace(",", ".").astype(float)
product_df["ProductProfit"] = product_df["ListPrice"] - product_df["StandardCost"]
product_df["Color"] = product_df["Color"].fillna("Unknown")

```


E podemos finalmente verificar se a limpeza dos dados foi feita de forma correta
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



# CSV Para PowerBI
Após realizar a query no banco com SQLAlchemy e carregar os dados em um DataFrame do pandas, podemos exportar o resultado para um arquivo Excel ou CSV. Esse arquivo pode ser importado diretamente no Power BI para criação de dashboards.
```python
product_df.to_excel("nome_arquivo.xlsx", index=False)
```


