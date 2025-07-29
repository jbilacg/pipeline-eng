
Markdown

# Projeto de Engenharia de Dados - Processamento de JSON com Databricks

Este projeto demonstra a ingestão, modelagem e processamento de dados JSON utilizando o Databricks Free Edition. O objetivo é transformar um arquivo JSON aninhado (`ERP.json`) em um modelo relacional normalizado para facilitar análises e operações em um contexto de sistema ERP.

---

## Participante

* **Jéssica Bilac Gaspareto**

---

## Ferramentas Utilizadas

* **Databricks Free Edition**
* **PySpark** (para processamento de dados)

---

## Desafio 1: Modelagem e Criação de Tabelas a Partir de um JSON

### 1. Entendimento da Estrutura do JSON

Para compreender a estrutura do JSON fornecido (`ERP.json`), foram utilizadas as funções `printSchema()` e `display()` no Databricks. Isso permitiu visualizar o schema complexo do JSON, que inclui arrays e structs aninhadas.

**Código para Análise do Schema:**

```python
# Define o caminho do arquivo JSON
caminho_json = "/Volumes/workspace/default/data/ERP.json"

# Lê o arquivo JSON, tratando múltiplas linhas
df = spark.read.option("multiline", "true").json(caminho_json)

# Exibe o schema do DataFrame
df.printSchema()

# Exibe uma amostra dos dados para visualização
df.display()
```

## 2. Modelo Físico e Normalização
A normalização para a 3ª Forma Normal (3NF) foi adotada para o modelo relacional. Essa escolha foi motivada pela natureza de um sistema ERP, que geralmente lida com grandes volumes de dados e requer operações complexas, beneficiando-se da redução de redundância e da melhoria da integridade dos dados.

Diagrama de Entidade-Relacionamento (Conceitual em formato similar ao DBML):

```SQL

Table guest_check {
  guest_check_id BIGINT [pk]
  chk_num BIGINT
  chk_ttl DOUBLE
  bal_due_ttl STRING
  clsd_flag BOOLEAN
  clsd_bus_dt STRING
  clsd_lcl STRING
  clsd_utc STRING
  emp_num BIGINT
  gst_cnt BIGINT
  last_trans_lcl STRING
  last_trans_utc STRING
  last_updated_lcl STRING
  last_updated_utc STRING
  non_txbl_sls_ttl STRING
  num_chk_prntd BIGINT
  num_srvc_rd BIGINT
  oc_num STRING
  opn_bus_dt STRING
  opn_lcl STRING
  opn_utc STRING
  ot_num BIGINT
  pay_ttl DOUBLE
  rvc_num BIGINT
  sub_ttl DOUBLE
  tbl_name STRING
  tbl_num BIGINT
  loc_ref STRING
}

Table detail_line {
  dtl_id BIGINT [pk]
  guest_check_id BIGINT [ref: > guest_check.guest_check_id]
  agg_qty BIGINT
  agg_ttl DOUBLE
  bus_dt STRING
  detail_lcl STRING
  detail_utc STRING
  dsp_qty BIGINT
  dsp_ttl DOUBLE
  dtl_oc_num STRING
  dtl_ot_num BIGINT
  guest_check_line_item_id BIGINT
  last_update_lcl STRING
  last_update_utc STRING
  line_num BIGINT
  menu_item_id BIGINT [ref: > menu_item.menu_item_id]
}

Table menu_item {
  menu_item_id BIGINT [pk]
  active_taxes STRING
  incl_tax DOUBLE
  mi_num BIGINT
  mod_flag BOOLEAN
  prc_lvl BIGINT
  rvc_num BIGINT
  seat_num BIGINT
  svc_rnd_num BIGINT
  ws_num BIGINT
}

Table tax {
  tax_id BIGINT [pk]
  guest_check_id BIGINT [ref: > guest_check.guest_check_id]
  tax_coll_ttl DOUBLE
  tax_num BIGINT
  tax_rate DOUBLE
}

Table discount {
  detail_line_id BIGINT [pk, ref: > detail_line.dtl_id]
  amount DOUBLE
  reason STRING
}

Table service_charge {
  detail_line_id BIGINT [pk, ref: > detail_line.dtl_id]
  amount DOUBLE
  description STRING
}

Table tender_media {
  detail_line_id BIGINT [pk, ref: > detail_line.dtl_id]
  payment_type STRING
  amount DOUBLE
}

Table error_code {
  detail_line_id BIGINT [pk, ref: > detail_line.dtl_id]
  code STRING
  message STRING
}

```
## 3. Criação das Tabelas no Databricks
As tabelas foram criadas no Databricks utilizando PySpark, através de operações de explode para achatar as estruturas aninhadas e select para renomear e reorganizar as colunas conforme o modelo relacional. As tabelas são salvas no formato Delta Lake para garantir transacionalidade e versionamento.

Código para Criação e População das Tabelas:

```Python

from pyspark.sql.functions import explode, col, monotonically_increasing_id

caminho_json = "/Volumes/workspace/default/data/ERP.json"
df = spark.read.option("multiline", "true").json(caminho_json)

# Extrai e achata a array 'guestChecks'
df_guest_checks = df.withColumn("guestCheck", explode("guestChecks")).select("guestCheck.*")

# Cria a tabela guest_check com um ID explícito
df_guest_check = df_guest_checks.withColumn("guest_check_id", col("guestCheckId").cast("long"))

# Extrai e achata a array 'detailLines' e associa ao guest_check_id
df_detail_lines = df_guest_check.withColumn("detailLine", explode("detailLines")).select(
    col("guest_check_id"),
    col("detailLine.*")
)

# Cria a tabela menu_item, renomeando e removendo duplicatas
df_menu_item = df_detail_lines.select(
    col("dtlId").alias("detail_line_id"), # Usado para referência em detail_line, mas miNum é a PK
    col("menuItem.*")
).withColumnRenamed("miNum", "menu_item_id").dropDuplicates(["menu_item_id"])

# Cria a tabela tax, extraindo dados da array 'taxes' e gerando um ID único
df_tax = df_guest_check.withColumn("tax", explode("taxes")).select(
    col("guest_check_id"),
    col("tax.taxNum"),
    col("tax.taxCollTtl"),
    col("tax.taxRate")
).withColumn("tax_id", monotonically_increasing_id())

# Cria tabelas auxiliares para detalhes específicos, filtrando por valores não nulos
df_discount = df_detail_lines.filter(col("discount").isNotNull()).select(
    col("dtlId").alias("detail_line_id"),
    col("discount.*")
)

df_service_charge = df_detail_lines.filter(col("serviceCharge").isNotNull()).select(
    col("dtlId").alias("detail_line_id"),
    col("serviceCharge.*")
)

df_tender_media = df_detail_lines.filter(col("tenderMedia").isNotNull()).select(
    col("dtlId").alias("detail_line_id"),
    col("tenderMedia.*")
)

df_error_code = df_detail_lines.filter(col("errorCode").isNotNull()).select(
    col("dtlId").alias("detail_line_id"),
    col("errorCode.*")
)

# Exibição dos schemas e algumas linhas das tabelas criadas para verificação
print("guest_check schema")
df_guest_check.printSchema()
df_guest_check.show(5, truncate=False)

print("detail_line schema")
df_detail_lines.printSchema()
df_detail_lines.show(5, truncate=False)

print("menu_item schema")
df_menu_item.printSchema()
df_menu_item.show(5, truncate=False)

print("tax schema")
df_tax.printSchema()
df_tax.show(5, truncate=False)

# Salvando os DataFrames como tabelas Delta no Databricks
df_guest_check.write.format("delta").mode("overwrite").saveAsTable("guest_check")
df_detail_lines.write.format("delta").mode("overwrite").saveAsTable("detail_line")
df_menu_item.write.format("delta").mode("overwrite").saveAsTable("menu_item")
df_tax.write.format("delta").mode("overwrite").saveAsTable("tax")

df_discount.write.format("delta").mode("overwrite").saveAsTable("discount")
df_service_charge.write.format("delta").mode("overwrite").saveAsTable("service_charge")
df_tender_media.write.format("delta").mode("overwrite").saveAsTable("tender_media")
df_error_code.write.format("delta").mode("overwrite").saveAsTable("error_code")
```

# Desafio 2: Estratégias de Armazenamento e Impacto de Mudanças de Schema
## 1. Vantagens de Manter o Dado Original Bruto (Raw Data)
Manter o dado original bruto (raw data) é uma prática essencial para garantir a resiliência e a flexibilidade de um pipeline de dados. As principais vantagens incluem:

Auditabilidade e Reprocessamento: Permite refazer processos em caso de falhas ou erros, acompanhar as mudanças históricas e facilitar auditorias, garantindo a conformidade e a rastreabilidade dos dados.

Economia de Requisições: Evita chamadas repetitivas à API de origem, utilizando dados já salvos, o que pode reduzir custos e latência.

Flexibilidade para Novas Análises: Oferece o dado cru e sem transformações para outras equipes (ex: cientistas de dados) ou novas análises (ex: machine learning) que podem surgir no futuro, sem as restrições de um modelo pré-processado.

## 2. Estratégia de Armazenamento em Data Lake para Respostas de API
Para organizar as respostas de API em um Data Lake, a estratégia ideal é particionar os dados de forma lógica, facilitando a governança, o acesso e a otimização de custos. Sugere-se uma estrutura de pastas particionada por API, ano, mês, dia e ID da loja.

Exemplo de Caminho no Data Lake:

``` /api_responses/bi_getFiscalInvoice/year=2025/month=07/day=28/storeId=123/response_20250728120000.json```

Essa abordagem hierárquica permite:

Filtragem Eficiente: Consultar dados de períodos ou lojas específicas de forma rápida.

Gestão de Histórico: Manter um histórico completo e organizado de todas as respostas.

Integridade dos Dados: Salvar o JSON original garante que o dado esteja sempre íntegro.

Flexibilidade de Formato: Os dados brutos podem, posteriormente, ser transformados para formatos otimizados para análise, como Parquet ou Delta Lake, em camadas de processamento subsequentes.

## 3. Impacto da Mudança no Nome de um Campo de API

A alteração no nome de um campo em uma API (ex: de old_field_name para new_field_name) pode ter um impacto significativo no pipeline de dados:

Quebra do Processamento: O pipeline atual, que espera o nome antigo do campo, falhará ao tentar acessá-lo, resultando em erros ou perda de dados.

Necessidade de Atualização: O pipeline de ingestão e transformação precisará ser atualizado para reconhecer e lidar com o novo nome do campo. Isso pode envolver modificações no código PySpark, como em operações de select ou withColumnRenamed.

Lógica de Compatibilidade: Em alguns casos, pode ser necessário implementar lógica para suportar ambos os nomes (o antigo e o novo) durante um período de transição, especialmente em sistemas com dependências complexas ou quando a mudança não pode ser aplicada imediatamente em todos os consumidores.

Monitoramento e Comunicação: É crucial ter um processo robusto de monitoramento do schema das APIs e garantir uma comunicação proativa com as equipes consumidoras sobre quaisquer alterações. Ferramentas de governança de dados e contratos de API podem ajudar a mitigar esses riscos.
