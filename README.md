# MAT - Migration Assistant Tool

Manage your database versions with pure SQL

MAT allows you to write your app migrations, rollbacks and data seeding scripts in pure sql, no matter which languages or frameworks your application is build on

## Supported databases

Currently following databases are supported:

- MySQL

Compatibility for following databases is in development

- SQLite
- Postgres
- SQL Server 

## Install

### Python PIP

You can use pip:
```bash
pip install mat
```
> MAT depends on python 3, in some systems you would want to use 'pip3' to install programs under python3 environment

### Usage through docker

You don't know pip/python? No problem, you can use 'mat' through docker.

I have created a wrapper script that will invoke `mat` inside docker whenever you need it, just download it inside your migrations directory and use it the same way you would do with `mat` command

Get script
```bash
wget https://raw.githubusercontent.com/DiganmeGiovanni/Mat/master/docker/matw
chmod +x matw # Grant exec permissions to wrapper
```

Then you can use the wrapper the same way as `mat` command
```bash
./matw status
./matw migrate -s 2
```

> Check 'Getting started' section to understand what above commands means

Behind scenes the script will:
1. Create a docker container ready to run 'mat'
2. Mount current path '$PWD' as a volume in /mat container path
3. Invoke 'mat' command inside container by forwarding arguments

## How it works?

* You write your migration scripts in multiple sql files (One per migration)
* You write rollback migration scripts (One per migration)
* You setup database connection and scripts location in a configuration file
* Run `mat` to apply/unapply/list your migrations

## Getting started

### Prepare migrations directory

Create a directory to hold all your database migrations

> Directory location is up to you, but usually you'll want to place it
> inside your main project or as an independent repository in cases where
> multiple applications will access same database

Your directory should have a structure like this:

```bash
|-migrations/
|- |- migrations.yml 
|- |- up/
|- |- - v1_create_tables.sql
|- |- - v2_create_credentials_table.sql
|- |- down/
|- |- - v1_undo_create_tables.sql
|- |- - v2_destroy_credentials_table.sql
```

Where:
- The `up/` directory will hold all your migrations scripts
- The `down/` directory will hold all your rollback scripts
- The `migrations.yml` will be used for setup to specify how to access database

> You can use whatever name you want for your migration directories as long as
> you specify the right paths in the config file


### Create your migrations

Place your first migration inside `up/` directory in a sql file, ensure to name the file like: `V<version_number>_<snake_case_name>.sql`

> Follow naming convention is essential in order to run migrations
> in appropriate order and shows right name to you
 
Let's say you have following file named `v1_create_tables.sql` (which will be parsed as: Version: `V1`, name: `Create tables`)

```sql
CREATE TABLE user(
    id INTEGER PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(255) NOT NULL,
    last_name VARCHAR(255) NOT NULL,
    email VARCHAR(500) NOT NULL
);

CREATE TABLE car(
    id INTEGER PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    user_id INTEGER NOT NULL,
    
    CONSTRAINT fk_user FOREIGN KEY (user_id)
        REFERENCES user(id)
)
```

### Create your rollbacks

Place your rollback inside `down/` directory in a sql file, following same name conventions as per migrations: `V<version_number>_snake_case_name>.sql`

> Version number must match exactly with version in migration file, however you can use
> whatever you want for the name part
 
Let's say you have: `v1_undo_create_tables.sql`:
 ```sql
DROP TABLE car;
DROP TABLE user;
 ```

### Setup database connection and settings

Place the database connection settings into `migrations.yml` file:

```yaml
datasource:
  host: 192.168.100.43
  username: developer
  password: d3v3lopm3nt
  database: cars

migrations_path:
  up: up/
  down: down/
```

And now let 'mat' do it's magic ;)

### Using mat

`cd` into migrations directory and run one of following commands

```bash
mat status        # Will list all your migrations

mat migrate       # Will apply your migrations
mat migrate -s 2  # Will apply only '2' non applied migrations

mat rollback      # Will run you rollback scripts
mat rollback -s 2 # Will run rollback scripts for latest '2' applied migrations
```
