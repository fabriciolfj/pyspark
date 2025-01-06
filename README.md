# PySpark
## Conceitos
```
O Apache Spark é uma plataforma de computação distribuída projetada para processar grandes volumes de dados. Suas principais finalidades são:

Processamento de Big Data:


Processamento distribuído em clusters
Análise de grandes volumes de dados
Execução de operações em paralelo
Capacidade de lidar com petabytes de dados


Casos de Uso Principais:


ETL (Extração, Transformação e Carga de dados)
Machine Learning em larga escala
Processamento em tempo real (streaming)
Análise de dados complexos
Data Science e análises avançadas


Vantagens:


Velocidade (100x mais rápido que Hadoop para processamento em memória)
Processamento em memória
Tolerância a falhas
Compatibilidade com várias fontes de dados
Suporte a múltiplas linguagens (Python, Scala, Java, R)


Componentes Principais:


Spark Core: Motor de processamento base
Spark SQL: Para trabalhar com dados estruturados
Spark Streaming: Processamento em tempo real
MLlib: Biblioteca de Machine Learning
GraphX: Processamento de grafos


Integrações:


Hadoop HDFS
Apache Cassandra
Apache Kafka
Amazon S3
Bancos de dados relacionais

O Spark é especialmente útil quando você precisa processar grandes volumes de dados que não caberiam na memória de uma única máquina ou quando precisa de processamento distribuído para melhor performance.
```

## Arquitetura
```
A arquitetura do Apache Spark é baseada em um modelo mestre-escravo (ou driver-executor). Vou explicar cada componente:

Driver Program (Mestre):


Contém o programa principal
Executa o SparkContext
Cria o plano de execução (DAG)
Distribui as tarefas para os executores
Coordena a execução geral
Mantém as informações sobre o aplicativo


Cluster Manager:


Gerencia os recursos do cluster
Aloca recursos para aplicações
Pode ser:

Standalone Scheduler
YARN
Mesos
Kubernetes




Worker Nodes (Executores):


Executam as tarefas designadas
Armazenam dados em cache
Retornam resultados para o driver
Processam os dados localmente
Podem ter múltiplos executores por nó


Fluxo de Execução:

CopyDriver Program → Cluster Manager → Worker Nodes
     ↑                                  |
     |----------------------------------|
     (retorno dos resultados)

Comunicação:


Driver se comunica com Cluster Manager
Cluster Manager distribui recursos
Workers se comunicam com Driver
Executores processam dados em paralelo

Esta arquitetura permite processamento distribuído eficiente e tolerância a falhas, sendo altamente escalável.
```
## Diferença entre RDDS x DataFrame x DataSet
```
Vou explicar as principais diferenças entre RDDs, DataFrames e Datasets no Apache Spark:

RDD (Resilient Distributed Dataset):


Estrutura mais básica e antiga do Spark
Coleção distribuída de dados sem estrutura definida
Maior controle de baixo nível
Mais verboso e menos otimizado
Exemplo:

pythonCopyrdd = sc.parallelize([1, 2, 3, 4, 5])
rdd_mapped = rdd.map(lambda x: x * 2)

DataFrame:


Dados organizados em colunas nomeadas (como uma tabela)
Otimização automática de consultas
Similar ao pandas DataFrame
Mais fácil de usar com dados estruturados
Exemplo:

pythonCopyfrom pyspark.sql import SparkSession

spark = SparkSession.builder.getOrCreate()
df = spark.createDataFrame([
    (1, "João", 25),
    (2, "Maria", 30)
], ["id", "nome", "idade"])

Dataset:


Combina RDD e DataFrame
Tipagem forte (mais usado em Scala e Java)
Não disponível em Python (PySpark usa apenas DataFrame)
Verificação de tipos em tempo de compilação
Exemplo (Scala):

scalaCopycase class Person(name: String, age: Long)
val dataset = Seq(Person("João", 25)).toDS()
Principais diferenças em termos práticos:

Performance:


DataFrames/Datasets são mais otimizados
Catalyst Optimizer funciona melhor com DataFrames
RDDs não têm otimização automática


Facilidade de uso:


DataFrames são mais intuitivos
RDDs requerem mais código
Datasets oferecem segurança de tipos


Quando usar cada um:


RDDs: quando precisa de controle de baixo nível
DataFrames: para a maioria dos casos, especialmente com dados estruturados
Datasets: quando precisa de tipagem forte em Scala/Java

Em PySpark, o mais recomendado atualmente é usar DataFrames, pois oferecem melhor performance e uma API mais amigável.
```
## Pontos importantes
- a extensão do arquivo e ipynb
- aqui abaixo um exemplo de uso do pyspark
```
#%%
from pyspark.sql.functions import col
df.select([col(c).isNull().alias(c) for c in df.columns]).show()
#%%
# Exemplo de exclusão de linhas com valores nulos
df_clean = df.na.drop()
#%%
df.printSchema()
#%%
# Lista de nomes de colunas conforme o esquema com espaços
col_names = df_clean.schema.names

# Renomear colunas para remover espaços iniciais
for col_name in col_names:
    new_col_name = col_name.strip()  # strip() remove espaços do começo e do fim
    df_clean = df_clean.withColumnRenamed(col_name, new_col_name)

# Verificando o novo esquema
df_clean.printSchema()
#%%
# Exemplo de conversão de uma coluna de string para inteiro
df_clean = df_clean.withColumn("Merchant ID", df_clean["Merchant ID"].cast("integer"))
#%%
# NOTA: A conversão de tipos de dados é útil quando você precisa alterar o tipo de uma coluna. 
# No nosso DataFrame, a coluna 'Merchant ID' já é do tipo inteiro, conforme mostrado pelo esquema:
df_clean.printSchema()

# Portanto, a conversão para inteiro não é necessária neste caso. Se fosse uma coluna do tipo string
# que contém apenas números, a conversão seria realizada da seguinte maneira:
# df_clean = df_clean.withColumn("Merchant ID", df_clean["Merchant ID"].cast("integer"))
#%% md
## Análise Exploratória de Dados
#%%
#Calculando a Distribuição de Produtos por Categoria
from pyspark.sql.functions import count

# Agrupando por categoria e contando os produtos
categoria_distribuicao = df_clean.groupBy("Category Label").agg(count("Product ID").alias("Count")).orderBy("Count", ascending=False)

# Visualizando o resultado
categoria_distribuicao.show()
#%%
# Identificando os Comerciantes com Mais Ofertas
comerciantes_top = df_clean.groupBy("Merchant ID").agg(count("Product ID").alias("Total Products")).orderBy("Total Products", ascending=False)

comerciantes_top.show()
#%%
# Importando a função necessária
from pyspark.sql.functions import countDistinct

# Contando a quantidade de títulos de produtos únicos em cada categoria
diversidade_categoria = df_clean.groupBy("Category Label").agg(countDistinct("Product Title").alias("Unique Product Titles"))

# Exibindo o resultado
diversidade_categoria.show()
#%%

```

## DAG

````
O conceito de Directed Acyclic Graph (DAG) no contexto do Apache Spark de forma clara.
Um DAG (Grafo Acíclico Direcionado) no Spark é uma representação do fluxo de processamento dos dados, onde:

"Directed" (Direcionado) significa que o fluxo de dados tem uma direção específica, sempre movendo-se para frente
"Acyclic" (Acíclico) significa que não existem ciclos/loops no fluxo - os dados não voltam para um estágio anterior
"Graph" (Grafo) é a estrutura que representa as transformações como nós conectados por arestas

No Spark, quando você escreve seu código usando transformações (como map, filter, groupBy), o Spark não executa imediatamente. Em vez disso, ele:

Cria um plano de execução otimizado representado como um DAG
Divide o DAG em estágios (stages)
Organiza as transformações em uma sequência lógica
Executa apenas quando uma ação (como collect(), count(), save()) é chamada

Por exemplo, considere este código Spark:

# Criando um RDD
rdd = spark.sparkContext.parallelize([1, 2, 3, 4, 5])

# Aplicando transformações
filtered = rdd.filter(lambda x: x > 2)
doubled = filtered.map(lambda x: x * 2)
result = doubled.reduce(lambda x, y: x + y)

O DAG resultante seria algo como:
Input Data → Filter(>2) → Map(*2) → Reduce(+)
Cada seta (→) representa a direção do fluxo de dados, e cada operação é um nó no grafo. O Spark usa esta representação para:

Otimizar o processamento (combinando operações quando possível)
Determinar quais operações podem ser executadas em paralelo
Planejar a recuperação de falhas (sabendo quais etapas precisam ser refeitas)
Minimizar o shuffle de dados entre os nós do cluster

Você pode visualizar o DAG de suas operações usando a interface web do Spark (Spark UI), que mostra graficamente como suas transformações estão sendo organizadas e executadas.
Esta organização em DAG é uma das razões pela qual o Spark é tão eficiente em processar grandes volumes de dados, pois permite otimização automática do fluxo de 
````

## Shuffle
````
O conceito de shuffle no Apache Spark, que é um processo crucial para o desempenho das aplicações.
O shuffle é o processo de redistribuição dos dados entre as partições, geralmente ocorrendo entre os estágios (stages) de um job Spark. É um processo custoso pois envolve:

Escrita em disco
Transferência pela rede
Reorganização dos dados

Principais situações que causam shuffle:

Operações de Agregação

pythonCopy# Exemplo de groupBy que causa shuffle
df.groupBy("departamento").count()

Joins

pythonCopy# Join entre DataFrames causa shuffle
funcionarios.join(departamentos, "dept_id")

Reparticionamento

pythonCopy# Reparticionamento explícito
df.repartition(10)

Operações de Ordenação

pythonCopy# Sort causa shuffle
df.sort("data")
Para minimizar o impacto do shuffle, algumas estratégias importantes:

Broadcast Join

pythonCopy# Quando uma tabela é pequena, use broadcast
from pyspark.sql.functions import broadcast
grande.join(broadcast(pequena), "id")

Particionamento Inteligente

pythonCopy# Particionando por chave de join antes de operações
df1 = df1.repartition("join_key")
df2 = df2.repartition("join_key")
df1.join(df2, "join_key")

Redução de dados antes do shuffle

pythonCopy# Filtre e selecione apenas colunas necessárias
df.select("col1", "col2")
  .filter("col1 > 0")
  .groupBy("col1")
  .count()

Cache Estratégico

pythonCopy# Cache após operações de shuffle frequentemente usadas
df_agregado = df.groupBy("dept").count()
df_agregado.cache()
Impactos do Shuffle:

Memória:

Requer espaço em disco para shuffle write
Precisa de memória para buffer de leitura/escrita


Rede:

Transferência de dados entre executores
Pode causar congestionamento de rede


Computação:

Serialização/Deserialização dos dados
Ordenação e agregação



Para monitorar shuffles:

Spark UI:

Mostra quantidade de dados shuffled
Tempo gasto em operações de shuffle


Métricas importantes:

Shuffle Write Size
Shuffle Read Size
Spill Size (dados escritos em disco)



Dicas para otimização:

Reduza o volume de dados antes do shuffle
Use o número apropriado de partições
Aproveite broadcast quando possível
Monitore e ajuste conforme necessário
Considere particionamento físico dos dados
Use cache estrategicamente após shuffles

O shuffle é inevitável em muitos casos, mas entender quando ele ocorre e como otimizá-lo é fundamental para o desempenho das aplicações Spark.
````