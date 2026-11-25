# 🏛️ BI - Simulação de Federação Partidária & Inteligência Eleitoral

[![Power BI](https://img.shields.io/badge/Power_BI-PBIP_%7C_TMDL-yellow?logo=powerbi&logoColor=white)](https://powerbi.microsoft.com/)
[![Apache Parquet](https://img.shields.io/badge/Data_Format-Apache_Parquet-blue?logo=apache)](https://parquet.apache.org/)
[![DAX](https://img.shields.io/badge/Analytics-88_Medidas_DAX-orange)](#-modelagem-dax--inteligência-analítica)
[![TSE Open Data](https://img.shields.io/badge/Data_Source-TSE_Open_Data-green)](https://dadosabertos.tse.jus.br/)

Repositório analítico desenvolvido para **simulação de cenários de federação partidária no Brasil**, cruzando dados oficiais do Tribunal Superior Eleitoral (TSE). O projeto foi construído utilizando a arquitetura moderna do Power BI em código aberto (**Power BI Project - `.pbip` / TMDL**) com camada de dados desacoplada e otimizada em **Apache Parquet**, com parâmetro centralizado de configuração de dados (`CaminhoDados`), garantindo fácil replicação e portabilidade para qualquer desenvolvedor ou avaliador que clonar o repositório.

---

## 🎯 Contexto de Negócio & O que o Dashboard Analisa

Com a instituição das **Federações Partidárias** no Brasil (Emenda Constitucional nº 111/2021 e Lei nº 14.208/2021), os partidos políticos podem se unir em nível nacional por um período mínimo de 4 anos, atuando no parlamento e nas eleições como se fossem uma única agremiação.

Essa união gera grandes oportunidades estratégicas, mas também **desafios complexos de governança e montagem de chapas**. Este relatório foi concebido para responder perguntas críticas de inteligência eleitoral:

1. **Simulação Dinâmica de Federações (What-If):**
   * Seleção interativa de qualquer combinação de partidos para simular a criação de uma nova federação.
   * Cálculo imediato da soma de forças: bancada de Deputados Federais, Deputados Estaduais, Senadores, Governadores, Prefeitos e Vereadores.

2. **Tempo de Rádio e TV:**
   * Cálculo da nova fatia proporcional de tempo no Horário Eleitoral Gratuito da federação simulada, baseado no resultado da última eleição para a Câmara dos Deputados.

3. **Recursos de Campanha (FEFC & Fundo Partidário):**
   * Projeção do volume financeiro combinado do Fundo Especial de Financiamento de Campanha (FEFC) e cotas do Fundo Partidário.

4. **Superação da Cláusula de Barreira:**
   * Avaliação do cumprimento dos critérios progressivos da cláusula de desempenho (1%, 1.5%, 2%, 2.5% e 3% dos votos válidos distribuídos em múltiplos estados), assegurando o acesso a recursos e estrutura parlamentar.

5. **Mapeamento de Conflitos Municipais:**
   * Identificação de cidades onde dois ou mais partidos da federação pretendem lançar candidaturas próprias ou já possuem prefeitos/mandatários concorrentes, apontando previamente zonas de atrito político.

6. **Geografia e Presença Partidária:**
   * Mapeamento da densidade de filiados, órgãos partidários vigentes (diretórios municipais e comissões provisórias) e distribuição espacial de votos em mapas de calor.

---

## 🏗️ Arquitetura de Dados & Engenharia de Dados

O projeto aplica padrões modernos de Engenharia de Analytics:

```
simulacao-federacao/
│
├── data/                               # Camada de Dados em Apache Parquet (17 tabelas)
│   ├── d_cor_raca.parquet
│   ├── d_estado.parquet
│   ├── d_genero.parquet
│   ├── d_local_municipio.parquet
│   ├── d_partidos.parquet
│   ├── f_candidatos_2022.parquet
│   ├── f_fefc.parquet
│   ├── f_filiados_2024.parquet
│   ├── f_fundo_partidario.parquet
│   ├── f_historico_eleicoes.parquet    (Histórico eleitoral otimizado com GZIP)
│   ├── f_investimento_partido_candidato.parquet
│   ├── f_mdt_mandatarios_atuais.parquet
│   ├── f_orgaos_vigentes.parquet
│   ├── f_partido_distribuicao_municipio_2024.parquet
│   ├── f_recursos_partidos_2022.parquet
│   ├── f_votos_partido_uf.parquet
│   └── Data de Atualização.parquet
│
├── dashboard/
│   ├── BI - Simulação Federação.pbip   # Arquivo mestre do Power BI Project
│   ├── BI - Simulação Federação.Report/ # Layout, páginas, visuais e tooltips (JSON)
│   └── BI - Simulação Federação.SemanticModel/ # Modelo de dados em TMDL
│       ├── definition/
│       │   ├── expressions.tmdl         # Parâmetro centralizado de dados (CaminhoDados)
│       │   ├── model.tmdl               # Metadados do modelo e ordenação
│       │   ├── relationships.tmdl       # Relacionamentos Star Schema
│       │   └── tables/                  # 17 tabelas de dados + tabelas DAX + medidas
│
├── .gitignore                          # Exclusão de caches (.abf, binários .pbix e logs)
└── README.md                           # Documentação técnica do projeto
```

### Principais Decisões Técnicas de Engenharia:
* **Formato Apache Parquet Colunar:** Alta compressão, leitura colunar eficiente, preservação de tipagem forte e redução de mais de 75% da volumetria comparada a CSVs brutos.
* **Tamanho Otimizado (<100 MB):** A tabela de histórico eleitoral (`f_historico_eleicoes`), contendo mais de **23 milhões de registros**, foi otimizada para apenas **70.4 MB**, permitindo versionamento nativo no GitHub sem estourar limites de arquivo.
* **Parâmetro Centralizado de Dados (`CaminhoDados`):** Em conformidade com a arquitetura do Power Query (onde conectores de arquivo local `File.Contents` exigem caminhos absolutos válidos), o projeto desacopla a localização dos arquivos através do parâmetro `CaminhoDados`. Todas as 17 tabelas consom esse parâmetro com normalização automática de barras (`\` ou `/`). Qualquer pessoa que clonar o repositório só precisa ajustar este parâmetro único com 2 cliques na interface do Power BI Desktop (**Página Inicial > Transformar Dados > Editar Parâmetros**), sem necessidade de alterar código M em nenhuma tabela.
* **Separação Clara de Responsabilidades (ETL vs. DAX):** As colunas analíticas que dependem de contexto dinâmico do modelo foram eliminadas da camada de ingestão no Power Query (`Table.RemoveColumns`) e mantidas estritamente como **colunas calculadas em DAX**, evidenciando maturidade em modelagem relacional.

---

## 📐 Modelagem DAX & Inteligência Analítica

O modelo semântico conta com **88 medidas DAX** organizadas e estruturadas:

* **Tabela Desconectada de Simulação (`s_partidos_federacao`):**
  Permite ao usuário escolher um partido fixo no relatório ou simular um bloco (`"SIMULAÇÃO"`) sem quebrar a integridade referencial das dimensões.
* **Cálculos de Desempenho Eleitoral e Cláusula de Barreira:**
  Medidas que avaliam regras complexas da legislação eleitoral brasileira, calculando votos válidos nominais, votos de legenda e percentual exigido por estado.
* **Fórmulas de Contexto Dinâmico:**
  Utilização avançada de `CALCULATE`, `FILTER`, `SELECTEDVALUE`, `ALLSELECTED` e tabelas virtuais para recalcular métricas de tempo de TV e repasses em tempo de execução.

---

## 🖥️ Páginas do Dashboard

1. **Visão Geral:** Painel executivo com os principais KPIs de bancada, comparativo antes vs. depois da federação, tempo de TV e total de verba pública simulada.
2. **Conflitos:** Matriz analítica para detectar sobreposições e disputas de protagonismo entre os partidos da federação em cada município brasileiro.
3. **Mapa Votos Candidato - Calor:** Análise geoespacial da concentração de votos por região, estado e município.
4. **Tooltips Contextuais Avançados:**
   * *ToolTip Tempo de TV*
   * *Tooltip Investimento Entre Simulação*
   * *Tooltip Mandatários Estaduais e Municipais*
   * *ToolTip Presença Partidária*

---

## 🚀 Como Executar o Projeto Localmente

1. **Pré-requisitos:**
   * [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (recomenda-se a versão mais recente com suporte a PBIP habilitado).
   * [Git](https://git-scm.com/).

2. **Clonar o Repositório:**
   ```bash
   git clone https://github.com/SEU_USUARIO/simulacao-federacao.git
   cd simulacao-federacao
   ```

3. **Abrir o Relatório:**
   * Abra diretamente o arquivo:
     `dashboard/BI - Simulação Federação.pbip`
   * O Power BI carregará automaticamente as definições do modelo (`.SemanticModel`) e o layout das telas (`.Report`).

4. **Configurar o Caminho dos Dados (Apenas no primeiro acesso após clonar):**
   * Como a função `File.Contents` do Power BI exige o caminho absoluto do sistema de arquivos local, o projeto gerencia a localização das 17 tabelas por meio do parâmetro centralizado **`CaminhoDados`**.
   * Na barra superior (guia **Página Inicial**), clique na setinha para baixo do botão **Transformar Dados** e selecione **Editar Parâmetros**.
   * No campo **`CaminhoDados`**, insira o caminho absoluto da pasta `data` clonada na sua máquina (por exemplo: `D:\Projetos\simulacao-federacao\data\` ou `C:\Users\SeuUsuario\simulacao-federacao\data\`).
   * Clique em **OK** e, na faixa amarela que surgir, clique em **Aplicar Alterações**. Todas as 17 tabelas serão carregadas simultaneamente sem necessidade de qualquer alteração manual no código M.

