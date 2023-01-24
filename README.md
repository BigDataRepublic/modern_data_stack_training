# Modern Data Stack Training

Setup instructions and exercises for the Modern Data Stack training session.

> NOTE: Cloud SQL will be powered down after each training session to prevent excessive/unnecessary cost accumulation.

> NOTE #2: 🚨 For several parts of this training it necessary to prefix certain
> application services/instances with a unique string to prevent conflicts or 
> accidental overwrites.
>
> Because of cost and technical limitations certain services need to be shared
> among all participants, e.g. database instances.
>
> So in the remainder of this training where ever `<YOUR_UNIQUE_PREFIX>` is
> mentioned you should replace that with your actual unique prefix string, e.g.
> using your name (i.e. `<YOUR_UNIQUE_PREFIX>_database` becomes 
> `john_database`)

## Getting Started

**Prerequisites:**
* Docker is installed and running
* You have the GCP credentials `.json` key file - See Slack
* You have the CloudSQL's postgres db password - See Slack

> ⚠️ The below shell commands are assuming a unix-like OS, e.g. macOS or Linux.

1. In the root of the repository create a directory `.config` (this directory is gitignored)
2. Put the `.json` GCP credentials key file in there as `creds.json`
    * NOTE: the cloud sql proxy defined in the docker compose yaml file will use this
3. Run docker-compose (from the root of the repository):
    * Make sure docker
    ```sh
    docker compose up
    ```
    This will create the following services:
        * CloudSQL Proxy - to connect to the postgres instance on GCP
        * pgAdmin - a webapp to navigate the postgres instance and execute commands on it
        * Airbyte - a (local) server and fronted (webapp) to load data from postgres to BigQuery (in our case)
4. 🙏 Hopefully docker compose instantiated all services successfully 🙏

## Create the source database - pgAdmin

We are going to use pgAdmin to connect to the CloudSQL instance on GCP, create a database and populate it.

1. Navigate to `http://localhost:8080`
2. Login with
    * username: `admin@admin.com`
    * password: `root`
3. Click on `Add New Server`
4. Give the server a name, e.g. `modern-data-stack-pg-server`
5. In the `Connection` tab configure the following:
    * Host name/address: `cloudsql-proxy`
    * Port: `5432`
    * MaintenanceDB: `postgres`
    * Username: `postgres`
    * Password: `<see slack for password>`

<img src="./assets/server_conn.png" width="350" />

6. Click `Save`
7. We should now be connected to the CloudSQL postgres instance on GCP
    * The sidebar should now contain this server
    * Expand this see its databases
8. Right click on the `postgres` database and click `Query Tool`

<img src="./assets/pg_qt.png" width="350" />

9. Now let's create our source database:
    > 🚨 Because we are working with only a single postgres instance every one should prefix a unique string, e.g. your name, to make the database name unique
    
    Copy-paste the following SQL into the Query Tool, replace `<YOUR_UNIQUE_PREFIX>` with your unique prefix and run the query (F5 - keyboard shortcut).

    ```sql
    CREATE DATABASE <YOUR_UNIQUE_PREFIX>_northwind
    WITH
    OWNER = postgres
    ENCODING = 'UTF8'
    CONNECTION LIMIT = -1
    IS_TEMPLATE = False;
    ```

10. In the sidebar right click on `Database` and click `Refresh`, your database should be visible there now

> With the source database created we can populate it with tables and data for the dummy retail store Northwind.
>
> ⚠️ To make sure subsequent queries will be run against your new database close the existing Query Tool which is connected to the `postgres` database

11. Open the `Query Tool` on the (newly created) source database, i.e. right click on the `<YOUR_UNIQUE_PREFIX>_northwind` database and click `Query Tool`
12. Copy-paste the SQL from `data_source/create_northwind.sql` into the Query Tool and run it
13. If the script ran successfully, in the sidebar, under your Databases -> `<YOUR_UNIQUE_PREFIX>_northwind` -> Schemas -> public -> Tables
    * If the tables do not show up a refresh of the database might be required
14. To double check the query worked, clear the opened Query tool and run:
    ```sql
    select * from customers;
    ```
    This should return all 91 customers.
    
## Load source data into BigQuery - Airbyte

> NOTE: after restart of airbyte, i.e. `docker compose down` -> `docker compose up`,
> if the docker volumes where **not** deleted then the anything configured, e.g.
> sources, destinations and connections, will still exist. So the setup for these
> components should only have to be done once.
 
1. Go to `http://localhost:8000`
2. Login with the following credentials
    * Username: `airbyte`
    * Password: `password`
3. To create a data source, click `Sources` in the sidebar and select source type `Postgres`
4. Set up Source:
    * Host: `localhost`
    * Port: `5432`
    * Database Name: `localhost`
    * Username: `postgres`
    * Password: `<see slack for password>`

<details>
<summary>Toggle to show image</summary>
<img src="./assets/ab_src.png"/>
</details>

5. Click `Set up source`
    * This will create the source and test the connection
    * Should anything be wrong with the connection details Airbyte should pick this up.
6. To create a data destination, click on `Destinations` and select destination type `BigQuery`
7. Set up Destination:
    * Project ID: `modern-data-stack-training`
    * Dataset Location: `EU`
    * Default Dataset ID: `<YOUR_UNIQUE_PREFIX>_northwind_raw`
        * 🚨 A prefix is again required to avoid conflicts because we are working in the same BigQuery instance
        * Note also that the dataset does not have to exist yet, Airbyte will create a dataset in BigQuery by this name
    * Service Account Key JSON: copy-paste the contents of the `.json` credentials file into the field
        * The file should exist at `.config/creds.json`, as we've set it up in the beginning

<details>
<summary>Toggle to show image</summary>
<img src="./assets/ab_dest.png"/>
</details>

8. Click `Set up destination` 
    * Just like with the source Airbyte will verify the connection details.
9. Create a `Connection`: go to `Connections` (in the sidebar) and click `Create your first connection`
10. Select an existing source: `Postgres`
11. Click `Use existing source`
12. Select an existing destination: `BigQuery`
13. Click `Use existing destination`
14. A new connection setup should be shown. All fields can be left unchanged. At the bottom click `Set up Connection`
    * This create the connection and automatically start the initial sync process
    * Under `Connections`->`Postgres<>BigQuery`->`Status`->`Sync History` the sync should be running.
    * After the sync is done navigate to BigQuery `https://console.cloud.google.com/bigquery?project=modern-data-stack-training`
        * Under the `<YOUR_UNIQUE_PREFIX>_northwind_raw` some `_airbyte_*` tables should be created alongside the source tables, e.g. `customers`, `orders`, etc.
        * For each table Airbyte has added some metadata columns: `_airbyte_ab_id`, `_airbyte_emitted_at`, `_airbyte_normalized_at`, `_airbyte_products_hashid`.
        * The types for each table should be the same as in the source (postgres). However the nullability has not been carried over.

<details>
<summary>Toggle to show image (Connection Setup)</summary>
<img src="./assets/ab_conn_setup.png"/>
</details>

<details>
<summary>Toggle to show image (Connection Sync)</summary>
<img src="./assets/ab_conn_sync.png"/>
</details>

<details>
<summary>Toggle to show image (BigQuery Raw Data)</summary>
<img src="./assets/bq_raw_data.png"/>
</details>

## Transform raw data - dbt

Now that the we have loaded the raw data into our analytics database, BigQuery,
we can use dbt to define transformations (e.g. `join`s and aggregations) that
will help us reach our analytics goals, be that business intelligence or ML.

* [Install dbt](https://docs.getdbt.com/docs/get-started/pip-install)
    * `pip install dbt-bigquery` should suffice
        * Perhaps using a Python virtual environment if preferred.


### [Create a project](https://docs.getdbt.com/tutorial/create-a-project-dbt-cli#create-a-project)

The dbt CLI comes with a command to help you scaffold a dbt project.
To create your dbt project:

1. Ensure dbt is installed by running `dbt --version`:
    ```sh
    dbt --version
    ```

2. Initialize your dbt project, run:
    ```sh
    dbt init northwind
    ```
    Follow the instructions. You will be asked about:
        * Which database adapter to use: `BigQuery`
        * Authentication method: `service_account`
        * Authentication keyfile path: `/full/path/to/repo/.config/creds.json`
        * GCP Project ID: `modern-data-stack-training`
        * Your dataset name: `<YOUR_UNIQUE_PREFIX>`
            * NOTE: we haven't created this (BigQuery) dataset yet which is okay
        * Region: `EU`

<details>
<summary>Toggle to show image (BigQuery Raw Data)</summary>
<img src="./assets/dbt_init.png"/>
</details>

3. `cd` into your project:
    ```sh
    cd northwind
    ```

You can use `pwd` to confirm that you are in the right spot.

4. Open your project (i.e. the directory you just created) in a code editor like Atom or VSCode.
   You should see a directory structure with `.sql` and `.yml` files that were generated by the `init` command.


Test dbt's connection to BigQuery:

5. Run `dbt debug` to test the connection was setup correctly
    * If the connection test failed have a look at the `profiles.yaml` configuration
        * This is a file that dbt created during the init process and is saved at `~/.dbt/profiles.yaml` by default
        * It contains the connection information
    * `profiles.yaml` should look like this
    ```yaml
    northwind:
      outputs:
        dev:
          dataset: <YOUR_UNIQUE_PREFIX>
          job_execution_timeout_seconds: 300
          job_retries: 1
          keyfile: /full/path/to/repo/.config/creds.json
          location: EU
          method: service-account
          priority: interactive
          project: modern-data-stack-training
          threads: 2
          type: bigquery
      target: dev
    ```

### [Perform your first dbt run](https://docs.getdbt.com/tutorial/create-a-project-dbt-cli#perform-your-first-dbt-run)

Our freshly created project has some example models in it.
We're going to check that we can run them to confirm everything is in order.

Execute the `dbt run` command to build the example models.


### Structure your project

Now that we have initialized our dbt project we will start to structure it for
our use case.

* TODO

### Exercises

TODO (Show solutions in `<details>` blocks)
