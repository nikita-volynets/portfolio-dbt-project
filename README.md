# portfolio-dbt-project

Dbt-showflake tutorial: [https://quickstarts.snowflake.com/guide/data_teams_with_dbt_cloud/#0](https://quickstarts.snowflake.com/guide/data_teams_with_dbt_core/index.html#0)

E-commerce dataset: https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce

### Additional Materials

Surfalytics dbt-snowflake git project: https://github.com/surfalytics/data-projects/tree/main/de-projects/02_dbt_core_snowflake </br>
Medium article dbt-snowflake project: https://medium.com/@dipan.saha/dbt-on-snowflake-a-comprehensive-guide-a849e893a2e </br>
Git actions for dbt by Oleg Agopov: https://dbtips.substack.com/p/run-dbt-with-github-actions
Maksim Kazartsev's dbt-snowflake project: https://medium.com/@kazarmax/data-modeling-using-dbt-core-and-snowflake-78b87ea3c660

### Start of the project

#### 1. Set-up free Snowflake account
#### 2. Create a database and service accounts for dbt

```sql
-------------------------------------------
-- dbt credentials
-------------------------------------------
USE ROLE securityadmin;
-- dbt roles
CREATE OR REPLACE ROLE dbt_dev_role;
CREATE OR REPLACE ROLE dbt_prod_role;
------------------------------------------- Please replace with your dbt user password
CREATE OR REPLACE USER dbt_user PASSWORD = "<mysecretpassword>";

GRANT ROLE dbt_dev_role,dbt_prod_role TO USER dbt_user;
GRANT ROLE dbt_dev_role,dbt_prod_role TO ROLE sysadmin;

-------------------------------------------
-- dbt objects
-------------------------------------------
USE ROLE sysadmin;

CREATE OR REPLACE WAREHOUSE dbt_dev_wh  WITH WAREHOUSE_SIZE = 'XSMALL' AUTO_SUSPEND = 60 AUTO_RESUME = TRUE MIN_CLUSTER_COUNT = 1 MAX_CLUSTER_COUNT = 1 INITIALLY_SUSPENDED = TRUE;
CREATE OR REPLACE WAREHOUSE dbt_dev_heavy_wh  WITH WAREHOUSE_SIZE = 'LARGE' AUTO_SUSPEND = 60 AUTO_RESUME = TRUE MIN_CLUSTER_COUNT = 1 MAX_CLUSTER_COUNT = 1 INITIALLY_SUSPENDED = TRUE;
CREATE OR REPLACE WAREHOUSE dbt_prod_wh WITH WAREHOUSE_SIZE = 'XSMALL' AUTO_SUSPEND = 60 AUTO_RESUME = TRUE MIN_CLUSTER_COUNT = 1 MAX_CLUSTER_COUNT = 1 INITIALLY_SUSPENDED = TRUE;
CREATE OR REPLACE WAREHOUSE dbt_prod_heavy_wh  WITH WAREHOUSE_SIZE = 'LARGE' AUTO_SUSPEND = 60 AUTO_RESUME = TRUE MIN_CLUSTER_COUNT = 1 MAX_CLUSTER_COUNT = 1 INITIALLY_SUSPENDED = TRUE;

GRANT ALL ON WAREHOUSE dbt_dev_wh  TO ROLE dbt_dev_role;
GRANT ALL ON WAREHOUSE dbt_dev_heavy_wh  TO ROLE dbt_dev_role;
GRANT ALL ON WAREHOUSE dbt_prod_wh TO ROLE dbt_prod_role;
GRANT ALL ON WAREHOUSE dbt_prod_heavy_wh  TO ROLE dbt_prod_role;

CREATE OR REPLACE DATABASE dbt_hol_dev; 
CREATE OR REPLACE DATABASE dbt_hol_prod; 
GRANT ALL ON DATABASE dbt_hol_dev  TO ROLE dbt_dev_role;
GRANT ALL ON DATABASE dbt_hol_prod TO ROLE dbt_prod_role;
GRANT ALL ON ALL SCHEMAS IN DATABASE dbt_hol_dev   TO ROLE dbt_dev_role;
GRANT ALL ON ALL SCHEMAS IN DATABASE dbt_hol_prod  TO ROLE dbt_prod_role;
```

#### 3. Create a new dbt project in any local folder by running the following commands:

```
$ dbt init dbt_hol
$ cd dbt_hol
```

#### 4. Open ~/.dbt/profiles.yml in text editor and add the following section
```
dbt_hol:
  target: dev
  outputs:
    dev:
      type: snowflake
      ######## Please replace with your Snowflake account name
      account: <your_snowflake_trial_account>
      
      user: dbt_user
      ######## Please replace with your Snowflake dbt user password
      password: <mysecretpassword>
      
      role: dbt_dev_role
      database: dbt_hol_dev
      warehouse: dbt_dev_wh
      schema: public
      threads: 200
    prod:
      type: snowflake
      ######## Please replace with your Snowflake account name
      account: <your_snowflake_trial_account>
      
      user: dbt_user
      ######## Please replace with your Snowflake dbt user password
      password: <mysecretpassword>
      
      role: dbt_prod_role
      database: dbt_hol_prod
      warehouse: dbt_prod_wh
      schema: public
      threads: 200
```
#### 5. Validate configuration - Run the following command (in dbt_hol folder):

```
dbt debug
```

#### 6. Finally, let's run the sample models that comes with dbt templates by default to validate everything is set up correctly. For this, please run the following command (in dbt_hol folder):

```
dbt run
```
#### 7. Data Upload

Let's use the following dataset: https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce

Let's create stage called 'my_stage' and upload there our tables.

```sql
USE DATABASE e_commerce_data;
USE SCHEMA sales;
CREATE STAGE my_stage;
```

![image](https://github.com/nikita-volynets/portfolio-dbt-project/assets/22579201/5529297e-fc51-40d8-b332-97f43aa9feed)

```sql
CREATE TABLE customers (
    customer_id VARCHAR(255) NOT NULL,
    customer_unique_id VARCHAR(255) NOT NULL,
    customer_zip_code_prefix INT,
    customer_city VARCHAR(255),
    customer_state VARCHAR(2),
    PRIMARY KEY (customer_id)
);

CREATE TABLE sellers (
    seller_id VARCHAR(255),
    seller_zip_code_prefix INT,
    seller_city VARCHAR(255),
    seller_state VARCHAR(2)
);

CREATE TABLE products (
    product_id VARCHAR(255),
    product_category_name VARCHAR(255),
    product_name_length INT,
    product_description_length INT,
    product_photos_qty INT,
    product_weight_g INT,
    product_length_cm INT,
    product_height_cm INT,
    product_width_cm INT
);

CREATE TABLE geolocation (
    geolocation_zip_code_prefix INT,
    geolocation_lat DECIMAL(9, 6),
    geolocation_lng DECIMAL(9, 6),
    geolocation_city VARCHAR(255),
    geolocation_state CHAR(2)
);

CREATE TABLE order_items (
    order_id VARCHAR(255),
    order_item_id INT,
    product_id VARCHAR(255),
    seller_id VARCHAR(255),
    shipping_limit_date TIMESTAMP,
    price DECIMAL(10, 2),
    freight_value DECIMAL(10, 2)
);

CREATE TABLE order_payments (
    order_id VARCHAR(255),
    payment_sequential INT,
    payment_type VARCHAR(255),
    payment_installments INT,
    payment_value DECIMAL(10, 2)
);

CREATE TABLE order_reviews (
    review_id VARCHAR(255),
    order_id VARCHAR(255),
    review_score INT,
    review_comment_title VARCHAR(255),
    review_comment_message TEXT,
    review_creation_date DATETIME,
    review_answer_timestamp DATETIME
);

CREATE TABLE orders (
    order_id VARCHAR(255),
    customer_id VARCHAR(255),
    order_status VARCHAR(255),
    order_purchase_timestamp DATETIME,
    order_approved_at DATETIME,
    order_delivered_carrier_date DATETIME,
    order_delivered_customer_date DATETIME,
    order_estimated_delivery_date DATETIME
);

CREATE TABLE product_translation (
    product_category_name VARCHAR(255),
    product_category_name_english VARCHAR(255)
);

```

#### To fill tables with data:
```sql

COPY INTO sellers
FROM @my_stage/sellers.csv
FILE_FORMAT = (
    TYPE = 'CSV'
    FIELD_DELIMITER = ','
    SKIP_HEADER = 1
    FIELD_OPTIONALLY_ENCLOSED_BY = '"' -- Add this line to handle quotes
);
```

### Building DBT data pipelines

```
mkdir models/staging
mkdir models/transform
mkdir models/mart
mkdir models/tests
```
#### Then let's open our dbt_project.yml and modify the section below to reflect the model structure.

```
models:
  dbt_hol:
    staging:
      schema: staging
      materialized: view
    transform:
      schema: transform
      materialized: view
    mart:
      schema: mart
      materialized: view
```

#### Let's create a file macros\call_me_anything_you_want.sql with the following content:

```
{% macro generate_schema_name(custom_schema_name, node) -%}
    {%- set default_schema = target.schema -%}
    {%- if custom_schema_name is none -%}
        {{ default_schema }}
    {%- else -%}
        {{ custom_schema_name | trim }}
    {%- endif -%}
{%- endmacro %}


{% macro set_query_tag() -%}
  {% set new_query_tag = model.name %} {# always use model name #}
  {% if new_query_tag %}
    {% set original_query_tag = get_current_query_tag() %}
    {{ log("Setting query_tag to '" ~ new_query_tag ~ "'. Will reset to '" ~ original_query_tag ~ "' after materialization.") }}
    {% do run_query("alter session set query_tag = '{}'".format(new_query_tag)) %}
    {{ return(original_query_tag)}}
  {% endif %}
  {{ return(none)}}
{% endmacro %}

```
#### Let's create a file called packages.yml in the root of your dbt project folder and add the following lines:

```
packages:
  - package: dbt-labs/dbt_utils
    version: 1.1.1
```

```
dbt deps
```
