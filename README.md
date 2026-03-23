
# Template Guia — Docker, dbt e Airflow

Este material é um guia prático para entender **como estruturar código e configurações** das três ferramentas muito utilizadas em pipelines de dados modernos:

- Docker
- dbt
- Airflow

A ideia não é apenas copiar código, mas entender **a estrutura que cada ferramenta exige**, quais **parâmetros precisam ser definidos** e como **elas se conectam dentro de um pipeline de dados**.

---

# Visão Geral da Stack

Em muitos projetos de engenharia de dados essa combinação aparece:

Docker → cria o ambiente  
Airflow → orquestra o pipeline  
dbt → transforma os dados  

Fluxo típico:

1. Docker sobe os serviços  
2. Airflow executa o pipeline  
3. Uma task chama o dbt  
4. dbt executa transformações no banco  

---

# 1 — Docker

Docker permite empacotar aplicação, dependências e ambiente em um container reproduzível.

Arquivos principais:

Dockerfile  
docker-compose.yml  

---

# Dockerfile — Template Base

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["python", "app.py"]
```

Parâmetros importantes:

- FROM → imagem base  
- WORKDIR → diretório interno  
- COPY → envia arquivos para container  
- RUN → executa comandos  
- EXPOSE → porta exposta  
- CMD → comando inicial  

---

# docker-compose.yml — Template Base

```yaml
version: "3.9"

services:
  app:
    build: .
    container_name: meu_app
    ports:
      - "8000:8000"
    volumes:
      - .:/app
    environment:
      - ENV=dev
    depends_on:
      - db

  db:
    image: postgres:15
    container_name: meu_postgres
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: usuario
      POSTGRES_PASSWORD: senha
      POSTGRES_DB: banco
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

---

# Exemplo 1 — Docker Python simples

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY . .
CMD ["python", "app.py"]
```

```python
print("Olá Docker")
```

---

# Exemplo 2 — Docker com dependências

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

requirements.txt

```
pandas
requests
```

---

# Exemplo 3 — Docker Compose com PostgreSQL

```yaml
version: "3.9"

services:
  app:
    build: .
    container_name: app_python
    ports:
      - "8000:8000"
    volumes:
      - .:/app
    depends_on:
      - postgres
    environment:
      - DB_HOST=postgres
      - DB_PORT=5432
      - DB_NAME=meubanco
      - DB_USER=walter
      - DB_PASSWORD=123456

  postgres:
    image: postgres:15
    container_name: postgres_db
    environment:
      POSTGRES_DB: meubanco
      POSTGRES_USER: walter
      POSTGRES_PASSWORD: 123456
    ports:
      - "5432:5432"
```

---

# 2 — dbt

dbt organiza transformações SQL com modularização, testes e lineage.

Estrutura comum:

dbt_project.yml  
profiles.yml  
models/  
seeds/  
tests/  

---

# dbt_project.yml — Template

```yaml
name: 'meu_projeto_dbt'
version: '1.0.0'
config-version: 2

profile: 'meu_profile'

model-paths: ["models"]
seed-paths: ["seeds"]
test-paths: ["tests"]

models:
  meu_projeto_dbt:
    staging:
      +materialized: view
    marts:
      +materialized: table
```

---

# profiles.yml — Template

```yaml
meu_profile:
  target: dev
  outputs:
    dev:
      type: postgres
      host: localhost
      user: usuario
      password: senha
      port: 5432
      dbname: meu_banco
      schema: analytics
      threads: 4
```

---

# Modelo dbt — Template básico

```sql
select
    id,
    nome,
    data_criacao
from {{ source('origem', 'clientes') }}
```

---

# Modelo usando ref()

```sql
select
    cliente_id,
    count(*) as total_pedidos
from {{ ref('stg_pedidos') }}
group by cliente_id
```

---

# Exemplo 1 — Modelo staging

```sql
select
    id as cliente_id,
    nome,
    email
from clientes
```

---

# Exemplo 2 — Modelo intermediate

```sql
select
    cliente_id,
    count(*) as qtd_pedidos,
    sum(valor_total) as valor_total
from {{ ref('stg_pedidos') }}
group by cliente_id
```

---

# Exemplo 3 — Modelo mart

```sql
select
    p.pedido_id,
    p.cliente_id,
    c.nome as nome_cliente,
    p.valor_total
from {{ ref('stg_pedidos') }} p
left join {{ ref('stg_clientes') }} c
on p.cliente_id = c.cliente_id
```

---

# 3 — Airflow

Airflow orquestra pipelines definindo DAGs.

Componentes principais:

- DAG
- tarefas
- dependências
- agenda

---

# DAG — Template Base

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta

default_args = {
    "owner": "data_team",
    "retries": 1,
    "retry_delay": timedelta(minutes=5)
}

def minha_tarefa():
    print("Executando tarefa")

with DAG(
    dag_id="dag_exemplo",
    default_args=default_args,
    schedule="@daily",
    start_date=datetime(2026,3,1),
    catchup=False
) as dag:

    tarefa = PythonOperator(
        task_id="executar_funcao",
        python_callable=minha_tarefa
    )
```

---

# Exemplo 1 — DAG simples

```python
def ola():
    print("Olá Airflow")
```

---

# Exemplo 2 — Pipeline ETL

```python
def extrair():
    print("Extraindo")

def transformar():
    print("Transformando")

def carregar():
    print("Carregando")
```

Dependência:

```
extrair >> transformar >> carregar
```

---

# Exemplo 3 — BashOperator

```python
from airflow.operators.bash import BashOperator

listar = BashOperator(
    task_id="listar_arquivos",
    bash_command="ls -la /tmp"
)
```

---

# Chamando dbt dentro do Airflow

```python
rodar_dbt = BashOperator(
    task_id="rodar_dbt",
    bash_command="cd /opt/airflow/dbt && dbt run"
)
```

---

# Conectando as Ferramentas

Fluxo comum:

Docker → Airflow → dbt → Data Warehouse

Docker cria o ambiente  
Airflow executa pipelines  
dbt transforma os dados  

---

# Resumo

Docker → ambiente e containers  
dbt → transformação SQL  
Airflow → orquestração de pipelines  

Entender o **template mental dessas ferramentas** acelera muito o desenvolvimento de pipelines de dados.
