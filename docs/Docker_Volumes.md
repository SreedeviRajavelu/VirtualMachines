## What is a Volume in Docker

Think of Docker container as a small isolated computer - when you stop or delete it, everything inside disappears (its 'disk' is temporary).

A Docker volume is like pluggin in an external hard drive or shared folder to that container.

It lets you:

1. **Keep data** even after container is stopped or removed.
2. **Share files** between real machine (the host) and the container.

There are 2 main types of volume mounts:

- Bind mount (links a folder from your host machine to the container)
- Named volume (managed entirely by Docker - stored somewhere under `/var/lib/docker/volumes`)

Explanation for use of volumes:

- Left side (`./sql`) -> a folder on your computer at the same level as your docker-compose.yml file
- Right side (`/docker-entrypoint-initdb.d`) -> a special folder *inside the container* that MySQL checks when it first starts.

When MySQL starts for the first time, it looks in `/docker-entrypoint-initdb.d` and automatically runs any `.sql` files it finds there - this is how it initializes your database.

When Docker starts the MySQL container, it copies that folder into the container.

MySQL executes your CCP_data.sql automatically - creating the tables, inserting initial data, etc.

This mount does not store your database data (the rows & tables you later insert). It only gives MySQL initialization scripts.

Once MySQL is running, the database data is stored inside the container's temporary filesystem. If you delete the container, the data is lost.

This is called a **bind mount** (it directly links your folder `./sql` from the host to the continer).

Question:  does the special folder **docker-entrypoint-initdb.d** exist in every docker container ? if not where does it come from ?

No, the folder **/docker-entrypoint-initdb.d** does not exist in every Docker container.
It exists only in **certain official** database images (like MySQL, PostgreSQL, MongoDB, etc.) because the image creator chose to include it in the image’s startup logic. 

Each image defines **its own conventions** about where to put configuration or initialization files.

So when you use:
```
    volumes:
      - ./sql:/docker-entrypoint-initdb.d
```
you are not telling Docker anything special - you are telling the MySQL container:
When you run your startup script, here is the folder where you can find initialization files.



## Content of docker-compose.yml file to use postgresql database as docker container 

This version does not persist volume and data will vanish if you remove the container.


```
services:
  postgres:
    image: postgres:16
    container_name: ccp-multi-ied-multi-plc
    environment:
      POSTGRES_USER: root
      POSTGRES_PASSWORD: root
      POSTGRES_DB: powerplant
    ports:
      - "5432:5432"
    volumes:
    - ./postgres-sql:/docker-entrypoint-initdb.d

```

Use of 
**- volumes:**
  **- ./sql:/docker-entrypoint-initdb.d**

- This mounts your host directory ./sql into the container’s /docker-entrypoint-initdb.d directory.

- docker-entrypoint-initdb.d is a special folder: on first startup of the container, MySQL runs all .sql or .sh scripts there to initialize the DB.

- After initialization, new runs don’t automatically re-run scripts unless you remove the container and volume.

❗ This is not where live database data is stored. Live data is in /var/lib/mysql. Since you’re not mounting that, the data lives in a Docker-managed anonymous volume.

So: Changing this bind mount will not remove your MySQL data. It only affects which init scripts would run on container creation.



## Content of docker-compose.yml file to use postgresql database as docker container (with persistent volume)
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
      - postgres_data:/var/lib/postgresql/data   # <-- persistent volume for DB data
      - ./postgres-sql:/docker-entrypoint-initdb.d  # <-- init scripts

volumes:
  postgres_data:   # <-- named volume definition
```


### Use of Volumes inside PostgreSQL docker-compose.yml

```
volumes:
- postgres_data:/var/lib/postgresql/data # persistent volume for DB data
- ./postgres-sql:/docker-entrypoint-initdb.d # init scripts

volumes:
  postgres_data: # named volume definition 
```

Explanation of each line:

1. `-postgres_data:/var/lib/postgresql/data`

- left side (postgres_data) -> a named volume, managed by Docker

 - you don't see it as a folder on your host - Docker stores it internally (under /var/lib/docker/volumes/...)

- right side (/var/lib/postgresql/data) -> the main folder inside the container where the PostgreSQL stores all its databases.

This means your PostgreSQL data (tables, records, etc.) is stored in the `postgres_data` volume not inside the temporary cotainer filesystem.

Even if you:

- stop the container (docker compose down)

- rebuild it

- restart it

the data remains intact, because the volume is persistent.

If you ever want to delete the data, you explicitly run:
`docker compose down -v`

2. `- postgres-sql:/docker-entrypoint-initdb.d`

- it mounts your host folder `postgres-sql/` (next to your `docker-compose.yml`) into the container's `/docker-entrypoint-initdb.d` folder

- PostgreSQL automatically runs any `.sql` files there the first time the database is initialized (before any data exists).

So if you have `postgres-sql/CCP_data.sql`, it will create your tables and insert the initial values when the container is started for the first time.

3. section at the bottom:
```
volumes:
  postgres_data:
```

This **declares** the named volume used above, so Docker knows it should create and manage it.
