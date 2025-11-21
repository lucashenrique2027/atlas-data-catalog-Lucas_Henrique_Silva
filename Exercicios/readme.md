## 📝 README.md para `meu-catalogo-atlas`

Aqui está um modelo completo de `README.md` que cobre todos os requisitos do exercício (Documentação: 20%) e o estrutura para a entrega.

-----

# 📚 Catálogo de Dados PostgreSQL com Apache Atlas

## Visão Geral do Projeto

Este projeto implementa um **Data Catalogger** automático para extrair metadados de um banco de dados **PostgreSQL** (Northwind) e catalogá-los no **Apache Atlas** utilizando sua API REST. O objetivo principal é automatizar a descoberta de metadados, a criação de entidades (Databases, Tabelas, Colunas) e a implementação de **Data Lineage** (Linhagem de Dados) baseada em chaves estrangeiras (Foreign Keys).

Este trabalho cumpre as Tarefas 1 a 4 do exercício prático de DataOps e catalogação de metadados.

## 🛠️ Componentes e Tecnologias

| Componente | Função |
| :--- | :--- |
| **Apache Atlas** | Catálogo de Dados e Repositório de Metadados (via API REST) |
| **PostgreSQL** | Fonte de dados (Banco Northwind) |
| **Python** | Linguagem de script principal |
| `requests` | Comunicação com a API REST do Atlas |
| `psycopg2` | Conexão e extração de metadados do PostgreSQL |
| `pandas` | Estruturação e manipulação de metadados para relatórios |

-----

## 🚀 Como Executar o Projeto

### Pré-requisitos

1.  **Docker** e **Docker Compose** instalados.
2.  Acesso ao repositório base do laboratório para inicialização do ambiente Docker.

### 1\. Inicialização do Ambiente

O ambiente é iniciado via `docker-compose.yml` contido no repositório base.

```bash
# Navegue até o diretório que contém o docker-compose.yml
docker-compose up -d

# Aguarde 5-10 minutos pela inicialização completa do Apache Atlas
docker-compose logs -f atlas

# Verifique os serviços:
# Atlas: http://localhost:21000 (admin/admin)
# PostgreSQL: localhost:2001
```

### 2\. Instalação das Dependências

Instale as bibliotecas Python necessárias (listadas em `requirements.txt`).

```bash
pip install -r requirements.txt
```

### 3\. Execução do Pipeline

O script `main.py` executa o pipeline completo: inicializa clientes, extrai metadados, cataloga as entidades e gera a linhagem.

```bash
python main.py
```

O script irá imprimir logs de cada etapa de catalogação e, ao final, gerar o relatório de descoberta.

-----

## 📂 Estrutura do Projeto

```
meu-catalogo-atlas/
├── README.md                 # Este arquivo
├── requirements.txt          # Dependências Python
├── config.py                 # Configurações de acesso (ATLAS, POSTGRES)
├── atlas_client.py           # Tarefa 1: Implementação da API REST do Atlas
├── postgres_extractor.py     # Tarefa 2: Extração de metadados do PostgreSQL
├── data_catalogger.py        # Tarefa 3: Lógica de mapeamento e catalogação no Atlas
├── discovery_report.py       # Tarefa 4: Geração de relatórios
└── main.py                   # Script principal de execução do pipeline
```

-----

## 📋 Detalhamento das Tarefas Implementadas

### Tarefa 1: `AtlasClient` (`atlas_client.py`)

A classe `AtlasClient` gerencia a comunicação com a API Atlas, implementando autenticação **HTTP Basic** e métodos de baixo nível:

  * `_make_request()`: Lida com requisições e tratamento de erros HTTP (`4xx`/`5xx`).
  * `search_entities(query)`: Busca entidades por nome qualificado (`qualifiedName`).
  * `create_entity(entity_data)`: Cria entidades em **bulk** (lote) usando a API `/entity/bulk`.
  * `get_entity(guid)` e `get_lineage(guid)`: Implementados para futuras extensões ou relatórios.

### Tarefa 2: `PostgreSQLExtractor` (`postgres_extractor.py`)

A classe se conecta ao PostgreSQL (`psycopg2`) e utiliza o `information_schema` para uma extração completa e estruturada:

  * `extract_table_and_column_metadata()`: Retorna o nome da tabela, nome da coluna, tipo de dados, nulidade e status de **Primary Key (PK)**.
  * `extract_relationships()`: Busca todas as chaves estrangeiras (Foreign Keys) para uso na linhagem.

### Tarefa 3: `DataCatalogger` (`data_catalogger.py`)

Esta é a orquestradora que traduz os metadados do Postgres para o formato de entidade do Atlas (Tipos `hive_db`, `hive_table`, `hive_column` e `Process`):

  * `_get_qualified_name()`: Cria o identificador único e hierárquico exigido pelo Atlas (ex: `northwind.customers@cluster1`).
  * `_create_database_entity()`: Garante que a entidade `hive_db` existe no Atlas.
  * `_create_table_and_column_entities()`: Cria a entidade `hive_table` e todas as suas `hive_column`s filhas em uma única chamada **bulk** (lote).
  * `_create_lineage_entities()`: Cria uma entidade **Process** do Atlas para cada relacionamento de Foreign Key encontrado, modelando o fluxo de dados (input -\> process -\> output).

### Tarefa 4: `DiscoveryReport` (`discovery_report.py`)

A classe utiliza o `AtlasClient` para buscar as entidades catalogadas e gerar estatísticas.

  * `generate_report(filename_base)`: Obtém o resumo de todas as entidades e exporta os dados de estatísticas em **JSON** e **CSV**.

## 📊 Relatório de Descoberta

Após a execução, os seguintes arquivos serão gerados no diretório raiz:

  * `discovery_report.json`
  * `discovery_report.csv`