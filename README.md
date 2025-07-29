Desafio Engenharia de Dados - Coco Bambu 2025
Participante: Jéssica Bilac Gaspareto

Prazo de Entrega: 31/07/2025

Este repositório contém a solução para o Desafio de Engenharia de Dados proposto. Aqui você encontrará os scripts, a modelagem de dados e as respostas para as questões conceituais.

Ferramentas Utilizadas
Databricks Free Edition

Desafio 1: Processamento de Dados de ERP
O primeiro desafio consiste em analisar um arquivo JSON de um sistema ERP, propor uma modelagem de dados relacional e implementar um script para processar e carregar esses dados em tabelas.

1. Análise da Estrutura do JSON
Para iniciar, foi realizada a leitura e a análise do schema do arquivo ERP.json para entender sua estrutura aninhada.

Script de Análise:

# Usado para entendimento da estrutura do JSON fornecido:

caminho_json = "/Volumes/workspace/default/data/ERP.json"

df = spark.read.option("multiline", "true").json(caminho_json)

# Imprime o schema do DataFrame
df.printSchema()

# Exibe o conteúdo do DataFrame
df.display()

Schema Resultante:

root
 |-- curUTC: string (nullable = true)
 |-- guestChecks: array (nullable = true)
 |    |-- element: struct (containsNull = true)
 |    |    |-- balDueTtl: string (nullable = true)
 |    |    |-- chkNum: long (nullable = true)
 |    |    |-- chkTtl: double (nullable = true)
 |    |    |-- clsdBusDt: string (nullable = true)
 |    |    |-- clsdFlag: boolean (nullable = true)
 |    |    |-- clsdLcl: string (nullable = true)
 |    |    |-- clsdUTC: string (nullable = true)
 |    |    |-- detailLines: array (nullable = true)
 |    |    |    |-- element: struct (containsNull = true)
 |    |    |    |    |-- aggQty: long (nullable = true)
 |    |    |    |    |-- aggTtl: double (nullable = true)
 |    |    |    |    |-- busDt: string (nullable = true)
 |    |    |    |    |-- chkEmpId: long (nullable = true)
 |    |    |    |    |-- chkEmpNum: long (nullable = true)
 |    |    |    |    |-- detailLcl: string (nullable = true)
 |    |    |    |    |-- detailUTC: string (nullable = true)
 |    |    |    |    |-- dspQty: long (nullable = true)
 |    |    |    |    |-- dspTtl: double (nullable = true)
 |    |    |    |    |-- dtlId: long (nullable = true)
 |    |    |    |    |-- dtlOcNum: string (nullable = true)
 |    |    |    |    |-- dtlOtNum: long (nullable = true)
 |    |    |    |    |-- guestCheckLineItemId: long (nullable = true)
 |    |    |    |    |-- lastUpdateLcl: string (nullable = true)
 |    |    |    |    |-- lastUpdateUTC: string (nullable = true)
 |    |    |    |    |-- lineNum: long (nullable = true)
 |    |    |    |    |-- menuItem: struct (nullable = true)
 |    |    |    |    |    |-- activeTaxes: string (nullable = true)
 |    |    |    |    |    |-- inclTax: double (nullable = true)
 |    |    |    |    |    |-- miNum: long (nullable = true)
 |    |    |    |    |    |-- modFlag: boolean (nullable = true)
 |    |    |    |    |    |-- prcLvl: long (nullable = true)
 |    |    |    |    |-- rvcNum: long (nullable = true)
 |    |    |    |    |-- seatNum: long (nullable = true)
 |    |    |    |    |-- svcRndNum: long (nullable = true)
 |    |    |    |    |-- wsNum: long (nullable = true)
 |    |    |-- dscTtl: long (nullable = true)
 |    |    |-- empNum: long (nullable = true)
 |    |    |-- gstCnt: long (nullable = true)
 |    |    |-- guestCheckId: long (nullable = true)
 |    |    |-- lastTransLcl: string (nullable = true)
 |    |    |-- lastTransUTC: string (nullable = true)
 |    |    |-- lastUpdatedLcl: string (nullable = true)
 |    |    |-- lastUpdatedUTC: string (nullable = true)
 |    |    |-- nonTxblSlsTtl: string (nullable = true)
 |    |    |-- numChkPrntd: long (nullable = true)
 |    |    |-- numSrvcRd: long (nullable = true)
 |    |    |-- ocNum: string (nullable = true)
 |    |    |-- opnBusDt: string (nullable = true)
 |    |    |-- opnLcl: string (nullable = true)
 |    |    |-- opnUTC: string (nullable = true)
 |    |    |-- otNum: long (nullable = true)
 |    |    |-- payTtl: double (nullable = true)
 |    |    |-- rvcNum: long (nullable = true)
 |    |    |-- subTtl: double (nullable = true)
 |    |    |-- taxes: array (nullable = true)
 |    |    |    |-- element: struct (containsNull = true)
 |    |    |    |    |-- taxCollTtl: double (nullable = true)
 |    |    |    |    |-- taxNum: long (nullable = true)
 |    |    |    |    |-- taxRate: long (nullable = true)
 |    |    |    |    |-- txblSlsTtl: double (nullable = true)
 |    |    |    |    |-- type: long (nullable = true)
 |    |    |-- tblName: string (nullable = true)
 |    |    |-- tblNum: long (nullable = true)
 |-- locRef: string (nullable = true)

2. Modelagem de Dados
Foi proposta uma modelagem relacional baseada na 3ª Forma Normal (3FN) para garantir a integridade, reduzir a redundância e otimizar a performance para operações complexas, comuns em sistemas de ERP.

(Opcional: Adicione aqui o arquivo Modelo físico.png ao seu repositório e descomente a linha abaixo)

<!-- ![Modelo Físico](Modelo físico.png) -->

Código DBML (Database Markup Language) do Modelo:

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

3. Criação das Tabelas (Script ETL)
O script PySpark a seguir realiza o processo de Extração, Transformação e Carga (ETL), lendo o JSON, aplicando as transformações para normalizar os dados conforme o modelo e salvando o resultado em tabelas no formato Delta Lake.

from pyspark.sql.functions import explode, col, monotonically_increasing_id

# 1. Leitura do arquivo JSON
caminho_json = "/Volumes/workspace/default/data/ERP.json"
df = spark.read.option("multiline", "true").json(caminho_json)

# 2. Explode do array principal 'guestChecks' para obter uma linha por registro de conta
df_guest_checks = df.withColumn("guestCheck", explode("guestChecks")).select("guestCheck.*")

# 3. Criação do DataFrame principal 'guest_check'
df_guest_check = df_guest_checks.withColumn("guest_check_id", col("guestCheckId").cast("long"))

# 4. Criação do DataFrame 'detail_line' explodindo o array 'detailLines'
df_detail_lines = df_guest_check.withColumn("detailLine", explode("detailLines")).select(
    col("guest_check_id"),
    col("detailLine.*")
)

# 5. Criação do DataFrame 'menu_item' a partir dos detalhes
df_menu_item = df_detail_lines.select(
    col("dtlId").alias("detail_line_id"),
    col("menuItem.*")
).withColumnRenamed("miNum", "menu_item_id").dropDuplicates(["menu_item_id"])

# 6. Criação do DataFrame 'tax'
df_tax = df_guest_check.withColumn("tax", explode("taxes")).select(
    col("guest_check_id"),
    col("tax.taxNum"),
    col("tax.taxCollTtl"),
    col("tax.taxRate")
).withColumn("tax_id", monotonically_increasing_id())

# 7. Criação de DataFrames para campos aninhados (assumindo que podem existir)
# Nota: O JSON de exemplo não continha esses campos, mas a estrutura está preparada.
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

# 8. Salvando as tabelas no formato Delta
df_guest_check.write.format("delta").mode("overwrite").saveAsTable("guest_check")
df_detail_lines.write.format("delta").mode("overwrite").saveAsTable("detail_line")
df_menu_item.write.format("delta").mode("overwrite").saveAsTable("menu_item")
df_tax.write.format("delta").mode("overwrite").saveAsTable("tax")
df_discount.write.format("delta").mode("overwrite").saveAsTable("discount")
df_service_charge.write.format("delta").mode("overwrite").saveAsTable("service_charge")
df_tender_media.write.format("delta").mode("overwrite").saveAsTable("tender_media")
df_error_code.write.format("delta").mode("overwrite").saveAsTable("error_code")

print("Tabelas criadas com sucesso no formato Delta Lake.")

Desafio 2: Questões Conceituais
1. Por que manter uma cópia fiel dos dados crus (raw data) retornados pela API?
Manter uma cópia fiel dos dados originais (raw data) é uma prática fundamental em engenharia de dados por várias razões:

Rastreabilidade e Auditoria: Permite rastrear a linhagem dos dados (data lineage) e realizar auditorias, comparando os dados transformados com a fonte original a qualquer momento.

Reprocessamento: Se ocorrer um erro no pipeline de transformação, é possível reprocessar os dados a partir da cópia fiel sem a necessidade de chamar a API novamente, o que economiza recursos e evita sobrecarga nos sistemas de origem.

Flexibilidade Analítica: Os dados brutos contêm todas as informações, sem perdas. Outras equipes ou projetos futuros podem ter necessidades diferentes e utilizar os mesmos dados crus para novas análises, modelos de machine learning ou relatórios.

Análise Histórica: Permite analisar como os dados evoluíram ao longo do tempo, incluindo mudanças na estrutura da API.

2. Como você estruturaria o Data Lake para armazenar os dados JSON da API bi_getFiscalInvoice?
Eu estruturaria o Data Lake com um sistema de particionamento hierárquico para otimizar a consulta e a organização dos dados. A estrutura de diretórios seguiria o padrão:

/datalake/raw/bi_getFiscalInvoice/year=<YYYY>/month=<MM>/day=<DD>/storeId=<ID_DA_LOJA>/<timestamp>_response.json

Exemplo:
/datalake/raw/bi_getFiscalInvoice/year=2025/month=07/day=28/storeId=123/20250728120000_response.json

Justificativa:

Particionamento: Particionar por ano, mês, dia e storeId permite que as ferramentas de consulta (como Spark) filtrem os dados de forma eficiente, lendo apenas os diretórios necessários e evitando a varredura completa (full scan).

Organização: A estrutura é intuitiva e facilita a localização de dados para um período ou loja específica.

Formato: Manter o JSON original na camada raw garante a fidelidade dos dados. Para as camadas seguintes (ex: trusted ou refined), os dados seriam convertidos para um formato colunar otimizado como Parquet ou Delta Lake.

3. Qual o impacto da alteração do nome de um campo na origem dos dados (ex: valor_total para valorTotal)?
A alteração do nome de um campo na origem tem um impacto direto e crítico nos pipelines de dados:

Quebra do Pipeline: O pipeline de ETL que espera o nome antigo (valor_total) falhará ao tentar acessar um campo que não existe mais, causando a interrupção do processamento.

Perda de Dados: Se o pipeline não tiver um tratamento de erro robusto, as novas informações do campo renomeado (valorTotal) não serão processadas, resultando em dados nulos ou incorretos nas tabelas de destino.

Necessidade de Manutenção: Exige uma atualização imediata no código do pipeline para refletir o novo nome do campo. Uma abordagem robusta seria implementar uma lógica que possa lidar com ambas as versões do campo por um tempo (para garantir a compatibilidade com dados antigos) ou versionar o schema.

Governança de Dados: Reforça a necessidade de ter uma boa governança, incluindo monitoramento de schema (schema evolution/drift) e uma comunicação clara entre as equipes que mantêm as APIs e as que consomem os dados.
