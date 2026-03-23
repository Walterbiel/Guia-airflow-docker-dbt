# 📦 Guia Prático — Docker, dbt & Airflow

[![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-Live-brightgreen?style=flat-square&logo=github)](https://walterbiel.github.io/Guia-airflow-docker-dbt)
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)](https://docs.docker.com)
[![dbt](https://img.shields.io/badge/dbt-FF6B35?style=flat-square&logo=dbt&logoColor=white)](https://docs.getdbt.com)
[![Airflow](https://img.shields.io/badge/Airflow-017CEE?style=flat-square&logo=apache-airflow&logoColor=white)](https://airflow.apache.org/docs)

> Guia visual e interativo para entender a estrutura, configurações e integração das três ferramentas mais usadas em pipelines de dados modernos.

🔗 **[Acessar o guia online](https://walterbiel.github.io/Guia-airflow-docker-dbt)**

---

## 🗂️ O que você vai encontrar

### 🐋 Docker
- Template base de `Dockerfile` com cada linha explicada
- Template de `docker-compose.yml` com múltiplos serviços
- Exemplo real com app Python + PostgreSQL
- Boas práticas: cache de layers, volumes nomeados, variáveis de ambiente

### 🔧 dbt
- Estrutura de projeto e `dbt_project.yml`
- `profiles.yml` com múltiplos ambientes (dev/prod)
- Modelos SQL com `source()` e `ref()`
- Arquitetura em camadas: staging → intermediate → marts
- Tabela comparativa de materializações (view, table, incremental, ephemeral)

### 🌀 Airflow
- Template base de DAG com `default_args`
- Pipeline ETL com dependências entre tarefas
- `BashOperator` e `PythonOperator` na prática
- Integração Airflow + dbt com `dbt run` e `dbt test`

### 🔗 Stack completa
- `docker-compose.yml` com Airflow + Scheduler + Postgres + dbt rodando juntos

---

## 🚀 Fluxo da stack

```
Docker → Airflow → dbt → Data Warehouse
  ↓          ↓        ↓
ambiente  orquestra  transforma
```

---

## 📁 Estrutura do repositório

```
.
├── index.html      # Guia completo (HTML/CSS — sem dependências externas locais)
└── README.md
```

---

## 🛠️ Como usar localmente

Basta abrir o `index.html` no navegador — sem build, sem instalação.

```bash
git clone https://github.com/Walterbiel/Guia-airflow-docker-dbt.git
cd Guia-airflow-docker-dbt
open index.html   # macOS
# ou xdg-open index.html no Linux
# ou clique duplo no Windows
```

---

## 📚 Documentações oficiais

| Ferramenta | Documentação |
|---|---|
| Docker | [docs.docker.com](https://docs.docker.com) |
| dbt | [docs.getdbt.com](https://docs.getdbt.com) |
| Airflow | [airflow.apache.org/docs](https://airflow.apache.org/docs) |

---

## 👤 Autor

**Walter Biel**  
[![GitHub](https://img.shields.io/badge/GitHub-Walterbiel-181717?style=flat-square&logo=github)](https://github.com/Walterbiel)
