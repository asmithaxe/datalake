# Open Data Stack - Data Lake
This project represents the basis of the Open Data Stack, providing a Data Lake for the storage and processing of data. This project implements the Data Lake using the following open source components:

- [MinIO](#minio) - S3 storage
- [Iceberg](#iceberg) - table format provider for relational tables on Trino
- [Trino](#trino) - SQL engine
- [PostgreSQL] - relational database and Gold tier implementation for Data Lakehouse
    - pgAdmin - admin tool for PostgreSQL

See [docker-compose.yml](./docker-compose.yml) as the starting point for stack configuration.

The Data Lakehouse follows the [Medallion architecture](https://databricks.com/fr/glossary/medallion-architecture).

## Acknowledgements
Thanks to the following public works for guidance:
- The [ngods-stocks](https://github.com/zsvoboda/ngods-stocks) project by [zsvoboda](https://medium.com/@zsvoboda).
- [Visualize parquet files with Apache Superset using Trino or PrestoSQL](https://sairamkrish.medium.com/visualize-parquet-files-with-apache-superset-using-trino-or-prestosql-511f18a37e3b) by [Sairam Krish](https://sairamkrish.medium.com)

### MinIO
[MinIO](https://min.io) is an S3-compatible object storage provider, implementing raw storage and the underlying storage layer for the Data Lake. The UI is available at http://localhost:9000 with the credentials `minio`/`minio123`. [MinIO Client](https://hub.docker.com/r/minio/mc) executes on start up to provide any initial configuration of the object store. Data is stored at `./data/minio`.

### Iceberg
[Apache Iceberg](https://iceberg.apache.org) provides the table format for the Data Lake. Iceberg tables are hosted by MinIO and accessed directly via the SQL engines in Trino (and Spark).

### Trino
[Trino](https://trino.io) is an SQL engine providing a standard and familiar interface to the data in the Data Lakehouse. Configuration is contained in the [./conf/trino](./conf/trino) folder, with Trino-specific data stored in the `./data/trino` folder. The UI is available at http://localhost:8060 using any credentials.

[Trino-cli](https://trino.io/docs/current/installation/cli.html) provides a command-line interface to the Trino SQL Engine, including an SQL client. The Trino command-line interface is included in the Trino Docker image, and can be  connected to via:
```
docker exec -it trino trino --server http://trino:8060
```

More information is available at: https://trino.io/docs/current/client/cli.html

Some background info on initialising a schema and tables can be found here:
https://sairamkrish.medium.com/visualize-parquet-files-with-apache-superset-using-trino-or-prestosql-511f18a37e3b

## Getting it running.

The Docker images can be built directly via the `build.sh` scripts in the sub-folders of the [build](/build/) folder. However the images will be built automatically if required when the stack is started.

Start the stack by executing the following from the root folder:
```
docker-compose -f docker-compose.yml up
```

### SQL to start

The following SQL command shows the Catalogs defined for the system. Catalogs are the equivalent of tiers in the Medallion architecture, so: `bronze`, `silver` and `gold`.

```
show catalogs;
```

The following SQL command shows the Schemas in a particular Catalog. By default, there are no Schemas set up.
```
show schemas in <catalog>;
```

The following SQL command shows the Tables in a particular Schema. By default, there are no Tables set up.
```
show tables in <catalog>.<schema>;
```

The following SQL commands create a Schema and a Table in the Bronze tier/Catalog.
```
create schema bronze.bronze_schema with (location = 's3a://bronze/bronze_schema');
create table bronze.bronze_schema.bronze_table (schema_name, table_name) AS VALUES ('bronze_schema', 'bronze_table');
```