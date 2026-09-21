✈️ Engenharia de Dados com IA — Análise de Voos ANAC

Projeto desenvolvido durante a Imersão de Engenharia de Dados com IA, utilizando dados públicos da ANAC (Agência Nacional de Aviação Civil) para construir um pipeline de dados capaz de analisar voos, atrasos, cancelamentos e informações de companhias e aeroportos.

O projeto utiliza Databricks, Python, PySpark e SQL, aplicando uma arquitetura de dados em camadas (Bronze, Silver e Gold) com foco em qualidade, governança, rastreabilidade e preparação dos dados para consumo por soluções de Inteligência Artificial.

---

🎯 Objetivo

Construir uma estrutura de dados confiável que permita responder perguntas de negócio relacionadas à operação aérea, como:

- Quantos voos foram realizados?
- Quais voos sofreram atrasos?
- Quais companhias apresentam determinados padrões de atraso?
- Quantos voos foram cancelados?
- Quais são as principais rotas?
- Como os atrasos se comportam ao longo do tempo?
- Qual a diferença entre operações domésticas e internacionais?

Além da análise, o projeto busca preparar os dados para que agentes de IA possam consultar e interpretar essas informações de maneira mais eficiente.

---

🏗️ Arquitetura

O pipeline segue uma arquitetura em camadas:

                 ┌──────────────────────┐
                 │      Dados ANAC      │
                 │   VRA + Referências  │
                 └──────────┬───────────┘
                            │
                            ▼
                 ┌──────────────────────┐
                 │       🥉 BRONZE      │
                 │    Dados brutos      │
                 │   + ingestão         │
                 └──────────┬───────────┘
                            │
                            ▼
                 ┌──────────────────────┐
                 │       🥈 SILVER      │
                 │ Qualidade e          │
                 │ governança           │
                 └──────────┬───────────┘
                            │
                  ┌─────────┴─────────┐
                  ▼                   ▼
          ┌───────────────┐   ┌────────────────┐
          │   Auditado    │   │   Quarentena   │
          │  Expectations │   │  Diagnóstico   │
          └───────────────┘   └────────────────┘
                            │
                            ▼
                 ┌──────────────────────┐
                 │        🥇 GOLD       │
                 │ Regras de negócio +  │
                 │ modelo analítico     │
                 └──────────┬───────────┘
                            │
                            ▼
                 ┌──────────────────────┐
                 │      IA / GENIE      │
                 │ Consulta dos dados  │
                 └──────────────────────┘

---

🥉 Bronze

A camada Bronze representa os dados de origem, mantendo os registros próximos ao formato original.

São utilizados dados da ANAC relacionados a:

- Voos realizados;
- Empresas aéreas;
- Aeroportos;
- Códigos de operação;
- Referências cadastrais.

Nesta etapa são realizadas principalmente atividades de ingestão e rastreabilidade, preservando os dados para que possam ser auditados posteriormente.

---

🥈 Silver

A camada Silver é responsável pela padronização, tipagem, qualidade e governança dos dados.

Entre os tratamentos realizados estão:

- Conversão de strings para "TIMESTAMP";
- Tratamento de valores "'null'";
- Padronização de campos;
- Criação de métricas de atraso;
- Cálculo de minutos recuperados;
- Validação de horários;
- Validação de situações de voo;
- Integridade referencial entre voos, aeroportos e empresas.

🔎 Contrato de dados

O projeto utiliza regras de qualidade para verificar a integridade dos dados.

Exemplos:

✓ Horários previstos preenchidos
✓ Situação do voo conhecida
✓ Chegada posterior à partida
✓ Atrasos dentro de faixas plausíveis
✓ Empresa existente no cadastro
✓ Aeroporto de origem existente
✓ Aeroporto de destino existente

As validações são utilizadas inicialmente em modo WARN, permitindo medir a qualidade sem simplesmente descartar os registros.

---

🚨 Quarentena

A quarentena funciona como uma camada de diagnóstico de qualidade.

Em vez de simplesmente excluir registros problemáticos, o pipeline identifica quais regras foram violadas e registra os respectivos motivos.

Exemplo:

horarios_previstos_presentes
aeroporto_origem_no_cadastro_anac
atraso_partida_plausivel

Isso permite investigar problemas na origem dos dados sem perder informações potencialmente relevantes para o negócio.

---

🥇 Gold

Na camada Gold são aplicadas as regras de negócio e os dados são preparados para consumo analítico.

Dimensão de aeroportos

A "dim_aeroporto" reúne informações dos aeroportos utilizados pelos voos, incluindo:

- ICAO;
- Nome;
- Município;
- UF;
- País;
- Existência no cadastro ANAC.

A dimensão também contempla aeroportos internacionais que não estão presentes no cadastro nacional.

Fato de voos

A "fato_voos" representa o principal conjunto analítico do projeto.

São disponibilizadas informações como:

- Companhia aérea;
- Número do voo;
- Origem e destino;
- Rota;
- Tipo de operação;
- Escopo doméstico/internacional;
- Horários previstos e reais;
- Atrasos;
- Pontualidade;
- Cancelamentos;
- Indicadores de qualidade.

Também são aplicadas regras de negócio, como a definição de pontualidade considerando 15 minutos.

---

🧠 OBT — One Big Table

O projeto também possui a tabela:

gold.obt_voos

A OBT (One Big Table) consolida as principais informações necessárias para consumo.

A ideia é reduzir a necessidade de múltiplos joins para quem utiliza os dados.

Em vez de disponibilizar somente códigos como:

SBSP
SBGR

a tabela também fornece informações semânticas:

São Paulo / Congonhas
São Paulo / Guarulhos

Isso é especialmente importante para aplicações de IA e agentes, pois modelos de linguagem trabalham melhor quando recebem dados com contexto e significado explícito.

---

🤖 Preparação para IA

Um dos objetivos do projeto é preparar os dados para que possam ser utilizados por agentes de IA, incluindo soluções como o Databricks Genie.

Para isso, a estrutura busca fornecer:

- Dados confiáveis;
- Metadados;
- Nomes semânticos;
- Descrições de códigos;
- Contexto sobre as colunas;
- Regras de qualidade;
- Métricas previamente calculadas;
- Estrutura simplificada para consulta.

A ideia é permitir que o usuário faça perguntas de negócio em linguagem natural sem precisar conhecer a estrutura interna do banco.

Por exemplo:

«"Quantos voos internacionais foram realizados?"»

ou:

«"Quais companhias tiveram maior quantidade de voos cancelados?"»

A camada Gold funciona como uma ponte entre os dados técnicos e o consumo por usuários e sistemas de IA.

---

🛠️ Tecnologias utilizadas

Tecnologia| Utilização
Databricks| Ambiente de processamento e engenharia de dados
Python| Manipulação e preparação dos dados
PySpark| Processamento distribuído
SQL| Transformações, validações e regras de negócio
Delta / Lakehouse| Armazenamento e organização dos dados
Databricks Pipelines| Orquestração e validação do pipeline
Genie / IA| Consumo dos dados através de linguagem natural

---

📂 Estrutura do projeto

.
├── Dados anac/
│   ├── referencias/
│   │   ├── AerodromosPublicos.csv
│   │   ├── pda_empresas_aereas_estrangeiros.csv
│   │   └── pda_empresas_aereas_nacionais.csv
│   │
│   └── vra/
│       ├── VRA_202510.csv
│       ├── VRA_202511.csv
│       ├── ...
│       └── VRA_20267.csv
│
├── notebooks/
│   ├── bronze_referencias.py
│   ├── bronze_vra.py
│   ├── silver.py
│   └── gold.py
│
├── pipelines/
│   ├── 01_vra.marcado.sql
│   ├── 02_vra_auditado.sql
│   └── 03_vra_quarentena.sql
│
├── sql/
│   └── gold/
│       ├── 01_dim_aeroporto.sql
│       ├── 02_fato_voos.sql
│       └── 03_obt_voos.sql
│
├── aula1.txt
├── aula2.txt
├── aula3.txt
└── aula4.txt

---

🔄 Fluxo de processamento

O pipeline segue o fluxo:

Dados ANAC
    ↓
Ingestão
    ↓
Bronze
    ↓
Tipagem e padronização
    ↓
Silver
    ↓
Marcação das regras de qualidade
    ↓
Auditoria
    ↓
Quarentena
    ↓
Aplicação das regras de negócio
    ↓
Gold
    ↓
OBT
    ↓
Análise / IA

---

📊 Principais conceitos aplicados

Durante o desenvolvimento foram aplicados conceitos de:

- Engenharia de Dados;
- Data Lakehouse;
- Arquitetura Medallion;
- ETL/ELT;
- Processamento distribuído;
- PySpark;
- SQL;
- Data Quality;
- Data Governance;
- Data Contracts;
- Integridade referencial;
- Metadados;
- Modelagem dimensional;
- One Big Table;
- Regras de negócio;
- Preparação de dados para IA;
- Agentes de IA.

---

📚 Fonte dos dados

Os dados utilizados no projeto são provenientes da Agência Nacional de Aviação Civil (ANAC).

Os arquivos VRA (Voo Regularmente Autorizado) são utilizados como principal fonte para as análises de operação, atrasos e cancelamentos.

Fonte: "ANAC — Dados Abertos" (https://www.gov.br/anac/pt-br/acesso-a-informacao/dados-abertos)

---

👩‍💻 Sobre o projeto

Este projeto foi desenvolvido como parte da Imersão de Engenharia de Dados com IA, com foco na aplicação prática de conceitos de engenharia de dados, qualidade e preparação de informações para soluções baseadas em Inteligência Artificial.

Tecnologias principais: "Databricks" "Python" "PySpark" "SQL" "Data Quality" "Data Governance" "IA"

---
