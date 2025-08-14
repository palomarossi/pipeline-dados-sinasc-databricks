# 🏥 Health Insights Brasil - Análise de Nascidos Vivos (SINASC)

![Databricks](https://img.shields.io/badge/Databricks-FF3621?style=for-the-badge&logo=databricks&logoColor=white)
![Delta Lake](https://img.shields.io/badge/Delta_Lake-01939A?style=for-the-badge&logo=delta&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-4479A1?style=for-the-badge&logo=postgresql&logoColor=white)

## 1. Visão Geral do Projeto
Este projeto implementa uma pipeline de dados completa para a "Health Insights Brasil", uma startup focada em otimizar a análise de dados de saúde pública. O objetivo é ingerir, processar e modelar dados brutos do Sistema de Informações sobre Nascidos Vivos (SINASC), disponibilizados pelo DataSUS, transformando-os em um modelo de dados limpo, performático e pronto para análises e geração de insights em dashboards.

A solução utiliza a plataforma Databricks para construir uma arquitetura Medallion (Bronze, Silver, Gold), garantindo governança, qualidade e escalabilidade dos dados.

## 2. Arquitetura da Solução
A pipeline foi construída seguindo a Arquitetura Medallion, que organiza os dados em camadas de qualidade crescente. O fluxo de dados segue a seguinte sequência:

- **Fonte de Dados:** CSV do SINASC
- **Landing Zone:** Armazena uma amostra dos dados brutos exatamente como foram baixados, garantindo uma fonte de dados rápida para o desenvolvimento.
- **Camada Bronze:** Contém os dados brutos estruturados em uma tabela Delta, com schema definido e metadados de ingestão. Serve como nossa "fonte única da verdade" histórica.
- **Camada Silver:** Onde os dados são limpos, tipos corrigidos, códigos decodificados para valores legíveis e as tabelas de dimensão criadas.
- **Camada Gold:** Camada final de apresentação, contendo a tabela fato e dimensões no modelo Star Schema, otimizada para consultas analíticas.

## 3. Modelo de Dados (Star Schema)
A camada Gold é estruturada como um **Star Schema**, otimizado para performance e facilidade de análise. O modelo consiste em uma tabela fato central conectada a várias tabelas de dimensão.

**Tabela Fato:**
- `fato_nascimento_sinasc`

**Tabelas Dimensão:**
- `dim_tempo_sinasc`
- `dim_localidade`
- `dim_mae_sinasc`
- `dim_parto_sinasc`

## 4. Como Rodar o Projeto
Para executar a pipeline completa do início ao fim, os notebooks devem ser executados manualmente na seguinte ordem:

1. **00_Setup_Diretorios**: Cria a estrutura de pastas no DBFS (uma única vez).
2. **01_Landing_Zone**: Baixa uma amostra dos dados do SINASC e armazena na Landing Zone.
3. **02_Bronze**: Lê a amostra, aplica schema rigoroso e cria `bronze_sinasc`.
4. **03_Silver**: Lê os dados da Bronze e cria as tabelas de dimensão (`dim_tempo_sinasc`, `dim_parto_sinasc`, `dim_mae_sinasc`, `dim_localidade`).
5. **04_Gold**: Constrói a tabela fato (`fato_nascimento_sinasc`) e finaliza o Star Schema.
6. **05_Analise_e_Insights**: Executa consultas e gera insights.

## 5. Orquestração e Automação
Para automatizar a execução desta pipeline em um ambiente de produção, a ferramenta ideal é o Databricks Workflows. Sua escolha se justifica pela integração nativa com o ambiente, o que simplifica a configuração sem a necessidade de infraestrutura externa como o Airflow. A plataforma gerencia as dependências entre as tarefas de forma confiável, garantindo que um notebook só execute após o sucesso do anterior, e oferece uma interface visual completa para monitoramento, verificação de logs e configuração de alertas de falha por e-mail. O plano de implementação consiste em criar um "Job" no Databricks Workflows, configurando uma tarefa para cada notebook da pipeline (de 01 a 04), com as devidas dependências sequenciais. Opcionalmente, uma tarefa final de validação pode ser adicionada para executar testes de qualidade na camada Gold. O job seria então agendado para execução diária, garantindo que os dados estejam sempre atualizados para os analistas.

## 6. Decisões de Design e Justificativas
- **Uso de Amostra de Dados:** Para agilizar o ciclo de desenvolvimento, o notebook de ingestão foi otimizado para baixar apenas uma amostra dos dados. Isso permite que toda a pipeline seja executada em segundos, em vez de minutos, facilitando a depuração e a iteração.
- **Schema Explícito na Bronze:** Em vez de depender da inferência de schema do Spark (que se mostrou frágil durante o desenvolvimento), optamos por definir um schema explícito e rigoroso. Isso torna a pipeline de ingestão muito mais robusta e à prova de falhas, garantindo que os dados sempre sejam carregados na estrutura correta.
- **Decodificação na Silver:** Os dados brutos do SINASC utilizam códigos numéricos para representar informações categóricas (ex: SEXO='1', PARTO='2'). Uma etapa crucial na camada Silver foi a utilização do dicionário de dados oficial do SINASC para "traduzir" esses códigos em valores textuais legíveis (ex: "Masculino", "Cesáreo"), agregando valor de negócio e tornando os dados prontos para análise.
- **Modelo Star Schema na Gold:** Foi escolhido por ser o padrão da indústria para data warehouses analíticos. Sua estrutura simples (fato central cercado por dimensões) oferece a melhor performance para consultas de BI e é muito mais intuitiva para os analistas de dados.
- **Tabelas Externas (Unmanaged):** Todas as tabelas são criadas com LOCATION explícito. Isso nos dá controle total sobre a organização dos arquivos no Data Lake e desacopla o metadados da tabela dos dados físicos, uma prática recomendada para a arquitetura Lakehouse.

## 7. Resultados e Insights Obtidos
**Insight 1: Relação entre Idade da Mãe e Peso do Recém-Nascido**  
A análise aprofundada da relação entre a idade materna e o peso do recém-nascido revela uma clara "curva em U invertido", um padrão clinicamente significativo. Observa-se que mães adolescentes (abaixo de 20 anos) tendem a ter bebês com peso médio inferior, que aumenta progressivamente até atingir um platô ideal na faixa dos 23 aos 38 anos. Acima dos 40 anos, o peso médio começa a decair e a apresentar maior variabilidade. Este insight é crucial para a saúde pública, pois permite identificar e direcionar o acompanhamento pré-natal para os grupos de maior risco nos extremos da idade reprodutiva.

**Insight 2: Distribuição de Peso por Tipo de Parto**  
A análise da distribuição de peso por tipo de parto, visualizada em um histograma, revela duas tendências importantes. Primeiro, a curva de nascimentos por cesárea é ligeiramente deslocada para a direita, indicando que, em média, os bebês de cesárea são um pouco mais pesados. Segundo, a maior altura da curva de cesáreas evidencia uma frequência superior deste procedimento na amostra. Este insight pode direcionar estudos sobre as razões para essa diferença de peso, como a prevalência de cesáreas eletivas em gestações a termo ou a indicação do procedimento para bebês com macrossomia.

**Insight 3: Cuidado Pré-Natal vs. Escolaridade da Mãe**  
A análise da relação entre escolaridade e cuidado pré-natal expõe uma nítida disparidade social. Os dados mostram uma forte correlação positiva: quanto maior o nível de instrução da mãe, maior a probabilidade de ela realizar o número ideal de consultas (7 ou mais). A análise percentual reforça que, embora o acesso ao pré-natal seja amplo, a adesão ao acompanhamento completo é significativamente maior entre mães com mais anos de estudo. Este é um insight poderoso para direcionar campanhas de conscientização e políticas de saúde para os grupos mais vulneráveis.

## 8. Inovações e Próximos Passos
**Inovação Implementada:** Para atender à necessidade de análise geográfica, a pipeline foi enriquecida com uma fonte de dados externa (lista de municípios do IBGE), criando uma dimensão de localidade que traduz códigos de municípios em nomes legíveis. 

**Próximos Passos:**
- Migrar transformações Silver e Gold para dbt.
- Automatizar pipeline com Databricks Workflows.
- Ingerir novos datasets (ex: SIM).
- Criar dashboards no Power BI.

## 9. Documentação (dbt docs)
Após a migração para dbt, será disponibilizado o link para a documentação gerada pelo `dbt docs`.
## 📁 Dataset e Documentação

### Dados Oficiais
| Recurso | Descrição | Link |
|---------|-----------|------|
| **Dataset Completo** | Dados brutos do SINASC 2024 | [Download CSV](https://s3.sa-east-1.amazonaws.com/ckan.saude.gov.br/SINASC/csv/SINASC_2024.csv) |
| **Dicionário de Dados** | Estrutura completa das variáveis | [Ver PDF](https://diaad.s3.sa-east-1.amazonaws.com/sinasc/SINASC+-+Estrutura.pdf) |
| **Amostra** | Subconjunto para desenvolvimento | [sample_100k.csv](./data/sample_100k.csv) |
