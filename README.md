# 📦 pipelineDados - Análise de Mensagens do Telegram com AWS Athena

Este projeto tem como objetivo extrair, transformar, armazenar e analisar dados de mensagens provenientes da API do Telegram. Utilizando um pipeline de ETL (Extract, Transform, Load), os dados são limpos, estruturados e armazenados de forma que possam ser facilmente consultados via AWS Athena.

---

## 🚀 Objetivos do Projeto

- Automatizar a coleta de dados de mensagens via API do Telegram.
- Transformar os dados brutos em um formato tabular e estruturado.
- Armazenar os dados no formato **Parquet** em um bucket S3.
- Realizar consultas analíticas no AWS Athena para gerar **insights sobre o comportamento dos usuários** e o volume de mensagens.

---

## 🛠️ Tecnologias e Ferramentas Utilizadas

- Python
- API do Telegram
- AWS S3 (armazenamento)
- AWS Athena (consultas SQL)
- Pandas
- Parquet (via `pyarrow`)
- Jupyter Notebook

---

## 🧪 Pipeline de ETL

### 1. **Extração (Extract)**

- Os dados são coletados utilizando a [API do Telegram Bot](https://core.telegram.org/bots/api).
- A autenticação é feita via token seguro.
- O retorno da API em JSON é armazenado em arquivos locais.

### 2. **Transformação (Transform)**

- Os dados JSON são transformados com Pandas.
- Campos como `user_first_name`, `date`, `message_id` e `text` são extraídos.
- A coluna `date` é convertida em formato de data legível (`datetime`).
- Foi adicionada a coluna `context_date` para facilitar análises temporais.

### 3. **Carga (Load)**

- Os dados tratados são exportados para o formato `.parquet`.
- Os arquivos são enviados diretamente para um bucket no **Amazon S3**.
- A tabela é então disponibilizada no **AWS Athena** para consulta via SQL.

---

## 📊 Análises Realizadas no Athena

As análises foram feitas com base nos dados carregados no Athena. Abaixo estão alguns exemplos de queries executadas:

### 1. Volume de mensagens por dia

```sql
SELECT context_date, COUNT(1) AS message_amount
FROM telegram
GROUP BY context_date
ORDER BY context_date DESC;
```
## 👥 2. Quem são os usuários mais ativos?
Com essa análise, podemos identificar os principais contribuidores do grupo com base no número de mensagens enviadas.

```sql
SELECT
  user_first_name,
  count(1) AS "message_amount"
FROM "telegram"
GROUP BY user_first_name
ORDER BY message_amount DESC
LIMIT 10
```
---
## 📝 3. Qual o tamanho médio das mensagens por usuário?
Esta consulta permite observar quem envia mensagens mais longas (detalhadas) ou mais curtas (diretas).

``` sql
SELECT
  user_first_name,
  CAST(AVG(length(text)) AS INT) AS "average_message_length"
FROM "telegram"
WHERE text IS NOT NULL
GROUP BY user_first_name
ORDER BY average_message_length DESC
```
---
## ⏰ 4. Quais os horários e dias da semana com maior atividade?
Análise poderosa para entender padrões de engajamento e identificar horários de pico.

```sql

WITH parsed_date_cte AS (
    SELECT
        *,
        CAST(date_format(from_unixtime("date"),'%Y-%m-%d %H:%i:%s') AS timestamp) AS parsed_date
    FROM "telegram"
),
hour_week_cte AS (
    SELECT
        *,
        EXTRACT(hour FROM parsed_date) AS parsed_date_hour,
        CASE EXTRACT(dow FROM parsed_date)
            WHEN 1 THEN 'Segunda'
            WHEN 2 THEN 'Terça'
            WHEN 3 THEN 'Quarta'
            WHEN 4 THEN 'Quinta'
            WHEN 5 THEN 'Sexta'
            WHEN 6 THEN 'Sábado'
            WHEN 7 THEN 'Domingo'
        END AS parsed_date_weekday
    FROM parsed_date_cte
)
SELECT
    parsed_date_hour,
    parsed_date_weekday,
    count(1) AS "message_amount"
FROM hour_week_cte
GROUP BY parsed_date_hour, parsed_date_weekday
ORDER BY parsed_date_hour, parsed_date_weekday
```
