
## Using Postgresql DB from docker-compose.yml

```
services:
  postgres:
    image: postgres:16
    container_name: ccp-multi-ied-multi-plc-pg
    environment:
      POSTGRES_USER: root
      POSTGRES_PASSWORD: root
      POSTGRES_DB: powerplant
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data # persistent volume for DB data
      - ./postgres-sql:/docker-entrypoint-initdb.d # init scripts

volumes:
  postgres_data: # named volume definition
```

Got it ✅ — you’ve already defined your PostgreSQL service with user root, password root, and database powerplant.

Once you get a bash shell inside the container, you can access PostgreSQL using the psql client. Here’s how:

1. Get into the container’s shell
`docker exec -it ccp-multi-ied-multi-plc-pg bash`

To enter PostgreSQL:

`psql -U root -d powerplant`

will prompt for a password -> type `root` based on docker-compose.yml used

Once insde psql, you will see something like:

`powerplant=#`

Now you can run SQL commands:

```
\l           -- list databases
\c powerplant -- connect to powerplant DB
\dt          -- list tables
SELECT * FROM ref;
SELECT * FROM ccp_values;
```

