# RDI Quickstart Oracle

Docker image for testing RDI with Oracle 19c.

## Building the Image

- Clone the repo locally and cd into directory `rdi-quickstart-oracle`
- ```bash
  docker build -t sqlserver sqlserver-image
  ```

## Running a Container

- Copy file `env.oracle` to `.env`
- Adjust the passwords to your requirements
- Change permissions on directories:
  ```bash
  chmod 777 oradata
  ```
- ```bash
  docker run --name ora19c --env-file .env -v $PWD/oradata:/opt/oracle/oradata -p 1521:1521 -d sqlserver
  ```

## Connecting to the Chinook Database

Use a standard database client, such as DBeaver:

<img width="726" alt="image" src="https://github.com/user-attachments/assets/5f06a827-8a75-4d01-870d-39f814dc3c8d" />

- Host = `localhost` (or the FQDN of your machine)
- Port = `1433`
- Database = `Chinook`
- Username = `sa`
- Password = <value of `MSSQL_SA_PASSWORD` in file `.env`>

You should see 11 tables in schema `dbo` of database `Chinook`, as well as 11 corresponding tables in schema `cdc`:

<img width="301" alt="image" src="https://github.com/user-attachments/assets/892594ab-9a31-4b8a-9698-a1476b61c02e" />

You can also use the command line interface `sqlcmd` to execute queries directly in the container, for example:

```bash
docker exec -it sqlserver /opt/mssql-tools18/bin/sqlcmd -No -S localhost -U sa -P CompLex#987 -d Chinook -Q "select table_name from information_schema.tables where table_schema='dbo'"
```

Expected result:

```
table_name                                                                                                                      
--------------------------------------------------------------------------------------------------------------------------------
Album                                                                                                                           
Artist                                                                                                                          
Customer                                                                                                                        
Employee                                                                                                                        
Genre                                                                                                                           
Invoice                                                                                                                         
InvoiceLine                                                                                                                     
MediaType                                                                                                                       
Playlist                                                                                                                        
PlaylistTrack                                                                                                                   
Track                                                                                                                           
systranschemas                                                                                                                  

(12 rows affected)
```

## Connecting from RDI

The `source` section in file `config.yaml` needs to look like this:

```
sources:
  mssql:
    type: cdc
    logging:
      level: info
    connection:
      type: sqlserver
      host: <DB_HOST>
      port: 1433
      database: Chinook
      user: ${SOURCE_DB_USERNAME}
      password: ${SOURCE_DB_PASSWORD}
```

- <DB_HOST> = <FQDN of your machine (or `localhost` when running locally)\>
- ${SOURCE_DB_USERNAME} = <value of `DBZUSER` in file `.env`>
- ${SOURCE_DB_PASSWORD} = <value of `DBZUSER_PASSWORD` in file `.env`>

## Rebuilding the Database

Changing the username or password for the Debezium user requires rebuilding the database. Follow these steps:

- Stop and remove the container
- Delete the contents of directory `data`
- Start the container
