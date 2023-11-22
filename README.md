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

Let's create stage called 'my_stage'

![image](https://github.com/nikita-volynets/portfolio-dbt-project/assets/22579201/5529297e-fc51-40d8-b332-97f43aa9feed)
