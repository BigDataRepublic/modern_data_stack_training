# Modern Data Stack - Session 2

Continuing with dbt...

## Packages

[Docs](https://docs.getdbt.com/docs/build/packages)

[Packages Hub](https://hub.getdbt.com/)

* install dbt_utils
* dbt deps
* add test using `dbt_utils.<whatever>`


## Write advanced test

### Exercise: write custom singular test

[docs](https://docs.getdbt.com/docs/build/tests#singular-tests)

* create file `tests/aaa.sql`
* write test for model `X`

### Exercise: write custom generic test

[docs](https://docs.getdbt.com/guides/best-practices/writing-custom-generic-tests#generic-tests-with-default-config-values)

* create file `tests/generic/warn_if_odd.sql`
* ...
* Add test to model `X` column `Y`
    * warn if order_amount exceeds some value


## Macros

[docs](https://docs.getdbt.com/docs/build/jinja-macros)

### Exercise: Create formatting macro

* create file `macros/human_formatted_currency_amount.sql`
* create file `macros/_macros.yml`
* create macro human_formatted_currency_amount
* document the macro in `macros/_macros.yml`
    * when running `dbt docs generate` and `dbt docs serve` this will be visible in the docs
 

### Exercise: Use a dbt_utils macros in a model

* maybe


## Advanced materialization

Traditional data models run code against the entire dataset each time they are executed, overwriting previous results. Incremental models, on the other hand, run code against the entire dataset only once during the initial run and then only execute on new data in subsequent runs. 

Read about pros/cons of different materializations [here](https://docs.getdbt.com/docs/building-a-dbt-project/building-models/materializations#materializations)
### Exercise: Create incremental model

[docs](https://docs.getdbt.com/docs/build/incremental-models)

[when to use incremental models](https://docs.getdbt.com/docs/build/incremental-models#when-should-i-use-an-incremental-model)

1. Simulate adding additional rows by inserting few samples to the `orders` raw table. 
> NOTE: generally these rows would be added to the actual source
> (in our case Postgres), however as an example it is easier to add new rows
> directly to the raw table on BigQuery

* Execute the following insert statements from the Bigquery GCP console:
```sql
INSERT INTO `modern-data-stack-training.<YOUR_UNIQUE_PREFIX>_northwind_raw.orders` (freight, order_id, ship_via, ship_city, ship_name, order_date, customer_id, employee_id, ship_region, ship_address, ship_country, shipped_date, required_date, ship_postal_code, _airbyte_ab_id, _airbyte_emitted_at, _airbyte_normalized_at, _airbyte_orders_hashid)
VALUES (10.0, 15335, 1, 'New York', 'John Doe', '2022-01-01', 'CONSH', 1, 'North', '123 Main St', 'USA', '2022-02-01', '2022-03-01', '10001', 'ab_id_1', '2022-01-01 12:00:00', '2022-01-01 12:00:00', 'hash_id_1');

INSERT INTO `modern-data-stack-training.<YOUR_UNIQUE_PREFIX>_northwind_raw.orders` (freight, order_id, ship_via, ship_city, ship_name, order_date, customer_id, employee_id, ship_region, ship_address, ship_country, shipped_date, required_date, ship_postal_code, _airbyte_ab_id, _airbyte_emitted_at, _airbyte_normalized_at, _airbyte_orders_hashid)
VALUES (20.0, 15472, 2, 'Los Angeles', 'Jane Doe', '2022-02-01', 'NORTS', 2, 'West', '456 Main St', 'USA', '2022-03-01', '2022-04-01', '90001', 'ab_id_2', '2022-02-01 12:00:00', '2022-02-01 12:00:00', 'hash_id_2');

INSERT INTO orders (freight, order_id, ship_via, ship_city, ship_name, order_date, customer_id, employee_id, ship_region, ship_address, ship_country, shipped_date, required_date, ship_postal_code, _airbyte_ab_id, _airbyte_emitted_at, _airbyte_normalized_at, _airbyte_orders_hashid)
VALUES (15.0, 10462, 3, 'Chicago', 'James Smith', '2022-03-01', 'NORTS', 3, 'Midwest', '789 Main St', 'USA', '2022-04-01', '2022-05-01', '60001', 'ab_id_3', '2022-03-01 12:00:00', '2022-03-01 12:00:00', 'hash_id_3');

INSERT INTO orders (freight, order_id, ship_via, ship_city, ship_name, order_date, customer_id, employee_id, ship_region, ship_address, ship_country, shipped_date, required_date, ship_postal_code, _airbyte_ab_id, _airbyte_emitted_at, _airbyte_normalized_at, _airbyte_orders_hashid)
VALUES (12.0, 19472, 1, 'Seattle', 'Robert Johnson', '2022-04-01', 'DUMON', 4, 'West', '246 Main St', 'USA', '2022-05-01', '2022-06-01', '98001', 'ab_id_4', '2022-04-01 12:00:00', '2022-04-01 12:00:00', 'hash_id_4');

INSERT INTO orders (freight, order_id, ship_via, ship_city, ship_name, order_date, customer_id, employee_id, ship_region, ship_address, ship_country, shipped_date, required_date, ship_postal_code, _airbyte_ab_id, _airbyte_emitted_at, _airbyte_normalized_at, _airbyte_orders_hashid)
VALUES (8.0, 14450, 2, 'Houston', 'Jessica Brown', '2022-05-01', 'BLONP', 5, 'South', '369 Main St', 'USA', '2022-06-01', '2022-07-01', '77001', 'ab_id_5', '2022-05-01 12:00:00', '2022-05-01 12:00:00', 'hash_id_5');

```

* Change materialization on `stg_orders` model to `incremental`.
* Utilize `is_incremental()` macro to filter for new rows only.
* [Optional:] Configure materialization with a `unique_key='order_id'` to allow for modification of existing rows rather than only adding new ones.
* Run `dbt run -s stg_orders`
    * Only added order rows have been processed
    * Number of rows processed should be visible in logs
* [Optional:] run the same model with `--full-refresh` to force rebuilding the table from scratch.


> NOTE: about the complexity incremental models introduce
> see: https://www.getdbt.com/coalesce-2021/trials-and-tribulations-of-incremental-models/


## Snapshots
[DBT snapshots](https://docs.getdbt.com/docs/build/snapshots) record changes made to a mutable model over time, enabling point-in-time queries. This feature uses [Slowly Changing Dimensions type-2](https://www.sqlshack.com/implementing-slowly-changing-dimensions-scds-in-data-warehouses/) and tracks the validity of each row with "from" and "to" date columns.

### Exercise: Capture employees changes 

* Create file `snapshots/employees.sql`
* Update the content of `snapshots/employees.sql` file with the snapshot code `{% snapshot employees_snapshot %}`
    * Configure the snapshot
      * `target_schema='YOUR_UNIQUE_PREFIX>_dbt'`
      * `strategy='check'` (Read more about it [here](https://docs.getdbt.com/docs/build/snapshots#check-strategy))
      * `unique_key='employee_id'`
      * `check_cols=['address', 'postal_code', 'home_phone', 'title_of_courtesy']'`
    * In the body, select from the `employees` raw model 

* Run the command `dbt snapshot` to create the snapshot table
> Examine the created table, you will observe that dbt has incorporated the columns "dbt_valid_from" and "dbt_valid_to," with the latter set to null values. Future executions will modify this
* Update one of the employee existing records in the raw table
```sql
UPDATE employees
SET address = 'Reguliersdwarsstraat', title_of_courtesy = 'Mrs.' 
WHERE employee_id = 7;

```
* Re-run the `dbt snapshot` to capture the changes

## Create a Dashboard

### Looker Studio (previously named: Google Data Studio)

Go to https://lookerstudio.google.com/

* Choice one of
    * `category_sales`
    * `employee_sales`
    * `orders_per_month`
* Create a chart for it

## Keep learning

* [dbt discourse](https://discourse.getdbt.com/)
* [dbt best practices](https://docs.getdbt.com/guides/best-practices)


