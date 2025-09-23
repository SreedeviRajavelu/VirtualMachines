##  Matlab Data Sources - SQL vs PostgreSQL

#### PostgreSQL JDBC for macOS:

https://ww2.mathworks.cn/help/database/ug/postgresql-jdbc-for-macos.html

#### PostgreSQL JDBC for Windows:

https://ww2.mathworks.cn/help/database/ug/postgresql-jdbc-windows.html

**I have configured a JDBC data source on matlab using mysql-connector-j-9.3.0 as the driver , and connecting to a dockerized version of a mysql database, how can I change this to postgresql database instead with minimal changes?**

1. Replace JDBC Driver
MySQL and PostgreSQL require different JDBC drivers:
Current (MySQL): mysql-connector-j-9.3.0.jar
Required (Postgres): postgresql-<version>.jar (latest is 42.x)
👉 Download the .jar and place it in the same folder where MATLAB can access JDBC drivers (often under matlabroot/toolbox/local or a custom path you’ve set).

**Link to download JDBC Drivers**

https://ww2.mathworks.cn/en/products/database/driver-installation.html

- choose PostgreSQL JDBC driver

✅ Which JDBC Driver to Use
F
rom the PostgreSQL JDBC site:

You are on Postgres 16 → supported by all recent drivers.

You are likely running Java 8+ (MATLAB since R2020a bundles Java 8, and newer releases may bundle Java 11).

👉 So you should use the JDBC 4.2 driver (version 42.7.8, latest stable).
That means:
`postgresql-42.7.8.jar`
is the right driver for you.

- check version of Java on Matlab

- check version of PostgreSQL used as JDBC data source

🔍 Things to Check Before Connecting

Java Version in MATLAB

In MATLAB, check which JVM it’s using:

version -java

If it shows `Java 8 or higher`, you’re good with `postgresql-42.7.8.jar`.
If it shows Java 7 → use postgresql-42.2.29.jar.
If Java 6 (unlikely unless MATLAB is very old) → use postgresql-42.2.27.jar.


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
-  volumes:
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



## Steps to use PostgreSQL database with MATLAB:

1. Download JDBC driver from PostgreSQL JDBC site (check version of Java on MATLAB & Postgres version)
2. 