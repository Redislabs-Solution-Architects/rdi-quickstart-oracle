# RDI Quickstart Oracle

Docker configuration for testing RDI with Oracle 19c.

## Building the Image

The method for building the Oracle 19c Docker image is described [here](https://github.com/oracle/docker-images/tree/main/OracleDatabase/SingleInstance)

## Running a Container

- Clone this repository:
  ```bash
  git clone https://github.com/Redislabs-Solution-Architects/rdi-quickstart-oracle.git
  cd rdi-quickstart-oracle
  ```
- Copy file `env.oracle` to `.env`
- Adjust the passwords to your requirements
- Set user and password for the Oracle LogMiner user script:
  ```bash
  source ./.env
  sed -e "s/<DBZUSER>/$DBZUSER/g" \
      -e "s/<DBZUSER_PASSWORD>/$DBZUSER_PASSWORD/g" templates/04-Logminer_User.template > sql/04-Logminer_User.sql
- Set directory permissions:
  ```bash
  chmod -R 777 oradata
  ```
- Create the container:
  ```bash
  docker run --name ora19c --env-file .env -v $PWD/oradata:/opt/oracle/oradata -v $PWD/sql:/docker-entrypoint-initdb.d/setup -p 1521:1521 -p 5500:5500 -d oracle/database:19.3.0-ee
  ```
- Check the status:
  ```bash
  docker logs ora19c --follow
  ```
  The first time you start the database, it will take 10 to 15 minutes for Oracle to be available. This is indicated by the following log output:
  ```
  #########################
  DATABASE IS READY TO USE!
  #########################
  ```

## Connecting to the Chinook Database

Use a standard database client, such as DBeaver:

<img width="696" alt="image" src="https://github.com/user-attachments/assets/c2a37838-6e5a-4a87-8b95-fcc3f1823944" />

- Host = `localhost` (or the FQDN of your machine)
- Port = `1521`
- Database = `ORCLPDB1`
- Username = `chinook`
- Password = `chinook`

In tab `Oracle properties` tick box `Show only connected user schema`:

<img width="808" alt="image" src="https://github.com/user-attachments/assets/69f66b4f-f8bf-4926-b2a1-53fd4746dd74" />

You should see 11 tables in schema `CHINOOK`:

<img width="290" alt="image" src="https://github.com/user-attachments/assets/a43af111-7229-47ee-90ff-a9e413441568" />

You can also use the command line interface `sqlplus` to execute queries directly in the container, for example:

```bash
docker exec -it ora19c sqlplus chinook/chinook@localhost:1521/ORCLPDB1
SQL> select table_name from user_tables;
```

Expected result:

```
TABLE_NAME
--------------------------------------------------------------------------------
GENRE
MEDIATYPE
ARTIST
ALBUM
TRACK
EMPLOYEE
CUSTOMER
INVOICE
INVOICELINE
PLAYLIST
PLAYLISTTRACK

11 rows selected.
```

## Connecting from RDI

The `source` section in file `config.yaml` needs to look like this:

```
sources:
  oracle:
    type: cdc
    logging:
      level: info
    connection:
      type: oracle
      host: <DB_HOST>
      port: 1521
      user: ${SOURCE_DB_USERNAME}
      password: ${SOURCE_DB_PASSWORD}
      database: ORCLCDB
    advanced:
      source:
        database.pdb.name: ORCLPDB1
```

- <DB_HOST> = <FQDN of your machine (or `localhost` when running locally)\>
- ${SOURCE_DB_USERNAME} = <value of `DBZUSER` in file `.env`>
- ${SOURCE_DB_PASSWORD} = <value of `DBZUSER_PASSWORD` in file `.env`>

## Rebuilding the Database

Changing the username or password for the Debezium user requires rebuilding the database. Follow these steps:

- Stop and remove the container
- Delete the contents of directory `oradata`, including the hidden file `.ORCLCDB.created`, but excluding directory `recovery_area`.
- Delete the contents of directory `oradata/recovery_area`
- Start the container
