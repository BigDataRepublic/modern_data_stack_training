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

All models are created a `Views` currently.

TODO: Explain about ephemeral models

### Exercise: Create incremental model

[docs](https://docs.getdbt.com/docs/build/incremental-models)

[when to use incremental models](https://docs.getdbt.com/docs/build/incremental-models#when-should-i-use-an-incremental-model)

> NOTE: generally these rows would be added to the actual source,
> in our case Postgres, however as an example it is easier to add new rows
> directly to a raw table on BigQuery

* Add rows to raw table on BigQuery
```sql
insert into `<YOUR_UNIQUE_PREFIX>_northwind_raw.<TABLE>`
values (1,1,1,,1,1,1,1)
```

* change materialization on model `stg_orders` to `incremental`
* 
* add `unique_key='order_id'`
* run `dbt run`
    * only added order rows have been processed
    * number of rows processed should be visible in logs


> NOTE: about the complexity incremental models introduce
> see: https://www.getdbt.com/coalesce-2021/trials-and-tribulations-of-incremental-models/


## Snapshots

[docs](https://docs.getdbt.com/docs/build/snapshots)

### Exercise: Transform employees to snapshot

* create file `snapshots/employees.sql`
* create snapshot code `{% snapshot employees_snapshot %}`
    * copy paste stg model and add snapshot jinja 
    * configure snapshot
* rm stg_employees.sql and change all references of `stg_employees` to `employees_snapshot`
* dbt run to create the employee snapshot model
* update existing employee
```sql
update `<YOUR_UNIQUE_PREFIX>_northwind_raw.<TABLE>`
where employee_id = 1
set name = "John"
```

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


