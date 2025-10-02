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
- volumes:
  - ./sql:/docker-entrypoint-initdb.d

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

