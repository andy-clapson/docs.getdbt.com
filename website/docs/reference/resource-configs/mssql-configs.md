---
title: "Microsoft SQL Server configurations"
id: "mssql-configs"
---

## Materializations

Ephemeral materialization is not supported.

### Tables

Tables will, by default, be materialized as a columnstore tables.
This requires SQL Server 2017 or newer for on-premise instances or service tier S2 or higher for Azure.

This behaviour can be disabled by setting the `as_columnstore` configuration option to `False`.

<Tabs
defaultValue="model"
values={[
{label: 'Model config', value: 'model'},
{label: 'Project config', value: 'project'}
]}
>

<TabItem value="model">

<File name="models/example.sql">

```sql
{{
    config(
        as_columnstore='False'
        )
}}

select *
from ...
```

</File>

</TabItem>

<TabItem value="project">

<File name="dbt_project.yml">

```yaml
models:
  your_project_name:
    materialized: view
    staging:
      materialized: table
      as_columnstore: False
```

</File>

</TabItem>

</Tabs>

## Seeds

By default, `dbt-sqlserver` will attempt to insert seed files in batches of 400 rows.
If this exceeds SQL Server's 2100 parameter limit, the adapter will automatically limit to the highest safe value possible.

To set a different default seed value, you can set the variable `max_batch_size` in your project configuration.

<File name="dbt_project.yml">

```yaml
vars:
  max_batch_size: 200 # Any integer less than or equal to 2100 will do.
```

</File>

## Snapshots

Columns in source tables can not have any constraints.
If, for example, any column has a `NOT NULL` constraint, an error will be thrown.

## Indices

You can specify indices to be created for your table by specifying post-hooks calling purpose-built macros.

The following macros are available:

* `create_clustered_index(columns, unique=False)`: columns is a list of columns, unique is an optional boolean (defaults to False).
* `create_nonclustered_index(columns, includes=columns)`: columns is a list of columns, includes is an optional list of columns to include in the index.
* `drop_all_indexes_on_table()`: drops current indices on a table. Only meaningful if the model is incremental.`

Some examples:

<File name="models/example.sql">

```sql
{{
    config({
        "as_columnstore": false, 
        "materialized": 'table',
        "post-hook": [
            "{{ create_clustered_index(columns = ['row_id', 'row_id_complement'], unique=True) }}",
            "{{ create_nonclustered_index(columns = ['modified_date']) }}",
            "{{ create_nonclustered_index(columns = ['row_id'], includes = ['modified_date']) }}",
        ]
    })
    
}}

select *
from ...
```

</File>

## dbt-utils

Many [`dbt-utils`](https://hub.getdbt.com/dbt-labs/dbt_utils/latest/) are supported,
but require the installation of the [`tsql_utils`](https://hub.getdbt.com/dbt-msft/tsql_utils/latest/) dbt package.
