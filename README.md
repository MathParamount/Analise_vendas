# 🛍️ Projeto de Análise de Vendas e Devoluções

👨‍💻 **Autor:** [Gabriel da cruz]
📅 **Data:** 22/12/2024

## 📃 Descrição
Este projeto foi desenvolvido em Python com o objetivo de praticar habilidades de programação e análise de dados. Ele analisa vendas e devoluções de produtos de uma loja, utilizando datasets armazenados em arquivos CSV dentro de uma pasta.
O processo inclui:

- Tratamento de dados: análise, limpeza, formatação e classificação para garantir precisão e consistência.
- Documentação do código: explicações ao longo do código para facilitar o entendimento e a organização de ideias.
- Criação de métricas: geração de visualizações que ajudam na interpretação do DataFrame, incluindo gráficos e mapas.
- Este projeto não apenas aprimorou a clareza do trabalho desenvolvido, mas também produziu insights visuais relevantes para a análise de dados.

## ⚒️ Instalação

Para ter acesso aos dados e filtrar precisa instalar no prompt as bibliotecas abaixo, que são responsáveis pela criação do dataframe e do tratamento dos dados

```
pip install pandas
```

A biblioteca folium e o matplotlib serviu para criar o gráfico de bolhas e o mapa com as cidades com maior ticket médio
```
pip install folium
pip install matplotlib
```

Na primeira parte do código deve importar as bibliotecas com suas respectivas alias para serem incorporadas ao código que está sendo trabalhado

```
import pandas as pd
import glob
import os

import matplotlib.pyplot as mapyp
from matplotlib.lines import Line2D

import folium as fol
from folium import plugins
```
## 🧮 Código

Primeiramente, os arquivos CSV da pasta especificada foram importados e organizados em dois grupos:

Arquivos de vendas, identificados pelo prefixo "Vendas".
Arquivos de devoluções, identificados pelo prefixo "Devolucoes".
Os arquivos de cada grupo foram lidos e concatenados para formar dois DataFrames principais

```
tabelas_vendas = []
tabelas_devolucao = []

caminhos = glob.glob(r"C:\Users\gabri\Desktop\Curso Básico de Python-20241204T190456Z-001\Vendas\*.csv")
caminhos_v = [k for k in caminhos if os.path.basename(k).startswith("Vendas")]

for caminho in caminhos_v:
    leitor = pd.read_csv(caminho)
    tabelas_vendas.append(leitor)

caminhos_d = [n for n in caminhos if os.path.basename(n).startswith("Devolucoes")]

for caminho1 in caminhos_d:
    leitor2 = pd.read_csv(caminho1)
    tabelas_devolucao.append(leitor2)

vendas = pd.concat(tabelas_vendas,ignore_index = False)
devolucoes = pd.concat(tabelas_devolucao,ignore_index = False)
```

1️⃣ DataFrame de Vendas

Ordenação pela coluna Data em ordem crescente:
```
#Tratamento dos dados
df_vendas = pd.DataFrame(vendas)
df_devolucao = pd.DataFrame(devolucoes)

# Classificação da coluna Data na tabela vendas em ordem ascendente
df_vendas.sort_values(by = ['Data'])

#remoção da última coluna 
df_vendas1 = df_vendas.drop(df_vendas.columns[9], axis=1)

#Alteração do nome da coluna
df_vendas1 = df_vendas1.rename({'Unnamed: 0':'Id'}, axis = 1)

print(df_vendas1)
```

Formatação da coluna Data para o tipo datetime

```
#formatando a coluna data para datetime
df_vendas1['Data'] = pd.to_datetime(df_vendas1['Data'],format = '%m/%d/%Y')
```

```
#Leitura dos tipos de dados no dataframe df_vendas1
df_vendas1.dtypes
```

2️⃣ DataFrame de Devoluções

O mesmo tratamento foi aplicado ao DataFrame de devoluções:

Foi feito o mesmo tratamento do dataset anterior
```
#Tratamento dos dados em df_devolucao

df_devolucao = df_devolucao.drop(df_devolucao.columns[7], axis = 1)

df_devolucao1 = df_devolucao.rename({"Unnamed: 0": "Id", "Preço Unitário" : "Preco Unitario"},axis = 1)

df_devolucao1.sort_values(by = 'Data', ascending = True)

print(df_devolucao1.head(40))
```
Visualização dos tipos de dados no DataFrame
```
#Formatando a coluna data para Datetime64

df_devolucao1['Data'] = pd.to_datetime(df_devolucao1['Data'], format = '%m/%d/%Y')
```

Vamos ver as chaves primárias comuns existentes nos dois datasets
```
# Verificação dos dados
chaves_comuns = df_vendas1['SKU'].isin(df_devolucao1['SKU']) & df_vendas1['Id'].isin(df_devolucao1['Id'])
quant = chaves_comuns.sum()

quant_coluna = df_vendas1['SKU'].count() + df_devolucao1['SKU'].count() + df_vendas1['Id'].count() + df_devolucao1['Id'].count()
print(f"a quantidade de chaves parecidas: {quant} em relacao ao total de coluna: {quant_coluna}")
```

Mesclagem dos dois dataframes em um único
```
#Mesclagem de dois dataframes com todos os dados
geral = pd.merge(df_vendas1,df_devolucao1,on = ['SKU','Id','Preco Unitario'],how = 'outer')
display(geral)
```

Renomeação das colunas e tratamento do tipo
```
geral = geral.rename(columns = {'Produto_x': 'Produto_vendido','Produto_y': 'Produto_devolvido','Data_x':'Data_venda' ,'Data_y':'Data_devolucao','Loja_x':'Loja_venda','Loja_y': 'Loja_devolucao'})
print(geral.head(50))

#Tratamento dos tipos de dados
print('\n Os tipos dos dados:')
tipos = geral.dtypes
display(tipos)

geral['Quantidade Vendida'] = pd.to_numeric(geral['Quantidade Vendida']).fillna(0).astype(int)
```

Transformação da coluna Produto_vendido para string e remoção de tipo nan
```
#Tratamento de erros de tipo
print(geral['Produto_vendido'].dtype)
geral['Produto_vendido'] = geral['Produto_vendido'].astype(str)

#Substituir valores nan
geral_filtrado = geral.dropna(subset=['Produto_vendido', 'Quantidade Vendida', 'Faturamento'])
```
Criação da coluna Faturamento
```
#Modificações no dataframe para análise
geral['Faturamento'] = geral['Quantidade Vendida'] * geral['Preco Unitario']
```

3️⃣ Visualização:

```
#Grafico de produto_vendido por preço_unitario

#Ajuste de frame
mapyp.figure(figsize = (20,8))

mapyp.scatter(geral['Produto_vendido'],geral['Preco Unitario'], s = geral['Quantidade Vendida'] * geral['Preco Unitario'], c = geral['Preco Unitario']
               , cmap = 'viridis',alpha = 0.9)
mapyp.title("Relato de Preco dos produtos")
mapyp.ylabel("Preço")
mapyp.xlabel("Itens")
```

Criação de objetos de faturamento para cada item
```
#Valores de faturamento por produto
faturamento_iphone = geral.loc[geral['Produto_vendido'] == 'iPhone', 'Faturamento'].sum()
faturamento_televisao = geral.loc[geral['Produto_vendido'] == 'Televisão', 'Faturamento'].sum()
faturamento_câmera = geral.loc[geral['Produto_vendido'] == 'Câmera', 'Faturamento'].sum()
faturamento_android = geral.loc[geral['Produto_vendido'] == 'Android', 'Faturamento'].sum()
faturamento_notebook = geral.loc[geral['Produto_vendido'] == 'Notebook', 'Faturamento'].sum()
faturamento_smartwatch = geral.loc[geral['Produto_vendido'] == 'SmartWatch', 'Faturamento'].sum()
faturamento_tablet = geral.loc[geral['Produto_vendido'] == 'Tablet', 'Faturamento'].sum()
```

Dados de legenda para cada item
```
legenda = [ Line2D([0],[0], marker = 'o', color = 'w', label = f"Faturamento da câmera: R$ {faturamento_câmera:,.2f}", markerfacecolor = '#53008C', markersize = 12),
            Line2D([0],[0], marker = 'o', color = 'w', label = f"Faturamento do iPhone: R$ {faturamento_iphone:,.2f}", markerfacecolor = 'yellow', markersize = 12),
            Line2D([0],[0], marker = 'o', color = 'w', label = f"Faturamento do tablet: R$ {faturamento_tablet:,.2f}", markerfacecolor = '#6D008A', markersize = 12),
            Line2D([0],[0], marker = 'o', color = 'w', label = f"Faturamento da televisão: R$ {faturamento_televisao:,.2f}", markerfacecolor = 'blue', markersize = 12),
            Line2D([0],[0], marker = 'o', color = 'w', label = f"Faturamento do smartwatch: R$ {faturamento_smartwatch:,.2f}", markerfacecolor = '#6D008A', markersize = 12),
            Line2D([0],[0], marker = 'o', color = 'w', label = f"Faturamento do notebook: R$ {faturamento_notebook:,.2f}", markerfacecolor = '#127F7D', markersize = 12),
            Line2D([0],[0], marker = 'o', color = 'w', label = f"Faturamento do android: R$ {faturamento_android:,.2f}", markerfacecolor = '#127F7D', markersize = 12)]

mapyp.legend(handles = legenda, loc = 'upper right')
```

Ajuste automático e rotação do eixo das abscissas
```
#Ajuste de proximidade dos pontos
mapyp.tight_layout() 

mapyp.xticks(rotation = 45)
mapyp.show()

```

Criação da métrica chamada ticket médio
```
# Filtrar valores inválidos
geral = geral[geral['Quantidade Vendida'] > 0]

# Calculando o ticket médio
tick_medio_df = (
    geral_filtrado.groupby('Loja_venda')
    .apply(lambda x: x['Faturamento'].sum() / x['Quantidade Vendida'].sum())
    .reset_index(name='tick_medio')
)
```

```
# Coordenadas para cidades
cidades_coordenada = {
    'Belo Horizonte': [-19.916681, -43.934493],
    'São Paulo': [-23.550520, -46.633308],
    'Recife': [-8.047562, -34.877000],
    'Rio de Janeiro': [-22.906847, -43.172897],
    'Fortaleza': [-3.717222, -38.543369],
    'Salvador': [-12.971399, -38.501392],
    'Porto Alegre': [-30.0346, -51.2177],
    'Goiás': [-16.6799, -50.1496],
    'Curitiba': [-25.4284, -49.2733]
}
```

Estrutura do mapa com a inicialização de sua coordenada 
```
# Inicializando o mapa
mapa = fol.Map(location=[-13.6707, -52.6349], zoom_start=5, control_scale=True)

# Adicionando círculos para cada cidade
for _, row in tick_medio_df.iterrows():
    cidade = row['Loja_venda']
    ticket = row['tick_medio']
    coordenadas = cidades_coordenada.get(cidade)
```
Marcadores e popups
```
if coordenadas:  # Verifica se as coordenadas existem
        # Adicionando marcador
        fol.Marker(
            coordenadas,
            popup=f"{cidade}: Ticket Médio R${ticket:.2f}"
        ).add_to(mapa)
```

Ajuste das coordenadas e do mapa
```
# Adicionando círculo proporcional ao ticket médio
        fol.Circle(
            location=coordenadas,
            radius=ticket * 30, 
            color='blue',
            fill=True,
            fill_color='blue',
            fill_opacity=0.4
        ).add_to(mapa)

# Exibindo o mapa
mapa
```

## 🚲 Testes

## 💡 Explicações

- Foi notado que criando dois arrays e separando os arquivos excel do tipo csv por nome e depois concatenando para transformar em dataframes se tornou muito mais simples e menos complexo o trabalho.

- Na parte da verificação dos dados foi feito uma contagem das chaves únicas de cada dataset. Isso é feito com a função isin(), ou seja, foi feito um join para ter uma razão entre a quantidade de chaves primárias comuns com o total. Assim, podendo servir para explicar a quantidade de nan no dataset geral.

- A mesclagem feita chamada de geral dos 2 dataframes foi feita para facilitar a visualização dos dados e os tratamentos ao longo do código.

- Na parte de visualização foi escolhido o gráfico do tipo scatter(espalhamento) porque foi o único que encontrei que se adaptou a demanda visual de aumento de tamanho do dado conforme o faturamento.

- O objeto chamado legenda foi criado com o intuito de pôr detalhes no gráfico, assim, tornando mais intuitivo e detalhado.

- A criação de coordenadas é essêncial para o uso da biblioteca folium para a criação do mapa interativo.

- Na inicialização do mapa a coordenada foi escolhida como sendo [-13.6707, -52.6349]. Isso é devido a centralização no Brasil, pois os dados são restringidos ao Brasil.

## 🚩 Conclusão

Este projeto é ideal para praticar técnicas de análise de dados, visualização e tratamento de datasets. Além disso, os insights gerados a partir das métricas e gráficos podem ser adaptados a outros contextos empresariais.

Contribuições e sugestões são bem-vindas!
