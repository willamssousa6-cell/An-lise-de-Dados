---
title: 'Análise de Dados '
author: "Willams Sousa Brito"
date: "20/10/2025"
output:
  html_document: default
  word_document: default
  pdf_document: default
---

## **Estudo do Preço do combustível automotivo no Brasil em 2025**

### Uso da Ferramenta de análise de dados - Software R

### [Objetivos da Análise:]{.underline}

#### ➡️ **Demonstrar através da análise exploratoria de dados (EDA) o processo para contar uma história a partir fontes de dados brutos**

#### ➡️ **Explorar nos dados, descobertas, estruturação, limpeza, união, validação e apresentação**

#### ➡️ **Descobrir relações entre variáveis e verificar sua qualidade, entender o conjunto de dados para extrair insights**

#### ➡️ **Utilizar visualizações de dados geográficos e métodos estatísticos para explorar os dados**

### [Fonte de Dados:]{.underline}

#### 1- ANP: Dados do Preço do Combustível Automotivo - 1º semestre do ano de 2025.

#### Acesso através do Portal de Dados Abertos, dados da Agência Nacional do Petróleo, Gás Natural e Biocombustíveis - ANP

#### 2- IBGE: Dados de localização dos Municípios.

#### Acesso através da Pacote de Dados Geográficos Brasileiros (geobr), permite coletar dados dos shapefiles do Instituto Brasileiro de Geografia e Estatística (IBGE) e outros conjuntos oficiais de dados espaciais do Brasil

#### Acesso a BigQuery <https://basedosdados.org/dataset/49ace9c8-ae2d-454b-bed9-9b9492a3a642?table=b39609b4-ffb2-4b4f-a182-47b0d160037b>

### [Importações pacotes e dados para análise:]{.underline} 

### 🎯 Objetivos 

#### - Instalar e importar pacotes relevantes para análise.

#### - Carregar os dados em um DataFrame, converte arquivos em estruturas manipulaveis.

#### - Realizar análise prévia dos dados carregados.

#### - Identificar problemas como codificação de texto, separadores, tipos de variáveis.

#### 🎯Instalar pacotes utilizando a função install.packages().

``` r
#Instalar pacotes utilizando 

install.packages("readr")
install.packages("dplyr")
install.packages("tidyverse")
install.packages("naniar")
install.packages("readxl")
install.packages("leaflet")
#grafico de geolocalização
install.packages("geobr")
install.packages("sf")
```

#### 🎯 Importar pacotes utilizando a função library().

``` r
#Importar pacotes 

library(naniar)
library(readr)
library(readxl)
library(dplyr)
library(ggplot2)
library(plotly)
library(httr)
library(jsonlite)
library(tidyverse)
library(leaflet)
library(geobr)
library(sf)
library(stringi)
```

#### [Carregar dados em um DataFrame.]{.underline}

#### O 1º conjunto de dados escolhido está no formato de um arquivo xls (Excel), disponivel para Download no Portal de Portal de Dados Abertos.

#### 🎯 Carregar os dados em um DataFrame e salvar em uma variável chamada (dados.anp) e realizar análise prévia dos dados.

``` r
Preços_semestrais_AUTOMOTIVOS_2025_01 <- read_excel("Preços semestrais - AUTOMOTIVOS_2025.01.xlsx")

#salvar em uma variável chamada (dados.anp)
dados.anp<-Preços_semestrais_AUTOMOTIVOS_2025_01

#A função str(), exibe informações básica dos dados importados,como o tipo de objeto (ex: data frame, vetor, lista), número de elementos ou dimensões, nomes das variáveis e seus tipos de dados e uma amostra dos dados.
#revisão básica dos dados importados
str(dados.anp)
```

#### 💡 O banco de dados dos preços dos combustiveis é um data.frame, possui 42.0409 linhas e 16 Colunas(Variaveis), a maioria das variaveis são qualitativas e a variável "Valor de Venda" a única numérica, do tipo contínua".

#### O 2º conjunto de dados escolhido está disponivel na BigQuery plataforma de análise de dados do Google, os dados podem ser acessados pelo pacote(geobr).

#### 🎯 Carregar os dados em um DataFrame e salvar em uma variável chamada (municipio) e realizar análise prévia dos dados.

``` r
# Carregar os dados dos municípios 
municipio <- read_municipality(year = 2024) #a função permite escolha do ano em que o dado foi coletado.
str(municipio)
```

#### 💡 O banco de dados geografico é um data.frame, possui 5.571 linhas e 8 Colunas(Variaveis), são variaveis qualitativas e uma variável especial "geom", composta por uma lista de geometrias espaciais.

#### A variavel "geom", contem uma lista de dados, para a presente análise iremos utilizar a coordenadas de longitude e latitude dos municipios.

#### 🎯 Adicionar coordenadas de latitude e longitude no data.frame

``` r
# 1- Calcular os centroides (ponto central de cada município)

mun_centroids \<- st_centroid(municipio)

# 2- Extrair coordenadas de latitude e longitude

coords \<- st_coordinates(mun_centroids) coords

# Adicionar as coordenadas ao dataframe original

municipio \<- municipio %\>% mutate(longitude = coords[,1], latitude = coords[,2])

# Visualizar os dados

print(municipio)
```

### [Estruturação e Limpeza dos dados]{.underline}

### 🎯 Objetivos

#### - Unir os dois data.frame criados (dados.anp e municipios)

#### - Realizar análise prévia do novo data.frame

#### - Corrigir inconsistências, padronizar nomes de colunas, formatos de datas, categorias

#### - Uniformizar valores como “sim”, “Sim”, “SIM” → “Sim”

#### - Tratar valores ausentes, identificar NA, null ou campos vazios

#### - Decidir se deve imputar, remover ou marcar esses dados

#### - Remover duplicatas - Eliminar registros repetidos que podem distorcer análises

#### - Ajustar tipos de variáveis, converter variáveis para os tipos corretos: numeric, factor, character, etc.

#### - Filtrar e selecionar dados relevantes, remover colunas ou linhas desnecessárias

#### - Criar novas variáveis, derivar colunas com base em regras (categorizar) ou cálculos (métricas).

#### - Detectar e tratar outliers - Identificar valores extremos que podem afetar a análise

#### - Decidir se devem ser removidos, ajustados ou mantidos

#### [União do 1º e 2º Conjunto de dados]{.underline}

#### Análise inicial dos dados sugere que a variavel chave principal disponivel é o NOME DO MUNICIPIO, entretanto a ocorrência de nomes de municipios iguais em diferentes regioes é comum no Brasil, dessa maneira ao utilizar a segunda chave SIGLA DO ESTADO, ira garantir a correta uniao dos dados.

#### 🎯 Padronização dos nomes de Variaveis e uniformizar valores.

``` r
#### 
# Para realizar a união dos conjuntos de dados, inicialmente as variaveis "Nome do Municipio" e "Nome do Estado" deverão estar Padronizadas.

#Padronização dos nomes de Variaveis
#os nomes de variaveis estão com idiomas e padrao de escrita diferentes.
names(dados.anp)
names(municipio)

#Renomeando os nomes de variaveis do 2º conjunto de daods de acordo com o 1º.
municipio \<- setNames(municipio, c("codigo do Municipio", "Municipio", "Estado - Codigo", "Estado - Sigla","Nome do Estado","Regiao - Codigo","Nome da Regiao","geom","longitude","latitude" ))
names(municipio)


# Análise previa das variaveis Chaves.

#Chave:Nomes dos Municipios
print(dados.anp\$Municipio,) #A variavel:Municipio, contem nomes em letras maiúsculas e sem acentos.
print(municipio\$Municipio)  #A variavel:name_muni, contem nomes em letras minúsculas e com acentos.  

# 1ª- Padronizar tudo em letras MAIUSCULAS.
dados.anp \<- dados.anp %\>% mutate(Municipio = tolower(Municipio))
municipio \<- municipio %\>% mutate(Municipio = tolower(Municipio))

# 2ª-padronizar tudo em palavras sem acentuação.
dados.anp \<- dados.anp %\>% mutate(Municipio = stri_trans_general(Municipio, "Latin-ASCII"))
municipio \<- municipio %\>% mutate(Municipio = stri_trans_general(Municipio, "Latin-ASCII"))

#verificando resultado
print(dados.anp\$Municipio) #A variavel:Municipio, contem nomes em letras minúsculas e sem acentos.
print(municipio\$Municipio)  #A variavel:name_muni, contem nomes em letras minúsculas e sem acentos.  

#Chave:Nomes dos Estados já estão padronizados
head(dados.anp\$\`Estado - Sigla\`)    
head(municipio\$\`Estado - Sigla\`)    
```

#### 🎯 Unir os dados e avaliar o resultado

#### O novo conjunto de dados será salvo com o mesmo nome do 1º conjunto (dados.anp), o método utilizado (Left join) combina duas tabelas mantendo todas as linhas da tabela da esquerda e adicionando colunas da tabela da direita quando as chaves coincidem. Linhas da esquerda sem correspondência recebem valores ausentes nas colunas adicionadas

``` r


#verificar a dimensão dos conjuntos de dados antes da união
dim(dados.anp) #Linhas:420409 e Colunas:16
dim(municipio) #Linhas:5571 e  Coliunas:8

#verificar os nomes de veriaveis
names(dados.anp)
names(municipio)

#realizar a união dos conjuntos de dados, usando duas chaves
dados.anp <- left_join(dados.anp,municipio, by = c("Municipio","Estado - Sigla"))

#verificar anti_join, para listar chaves nao correspondentes e descobrir as possiveis causas
dados.anp1 <- anti_join(dados.anp,municipio, by = c("Municipio","Estado - Sigla"))
#View(dados.anp1) #existe um municipio (santana do livramento) do RS que nao possui dados do IBGE registrados em 2024. 
head(dados.anp1) #817 linhas 
```
