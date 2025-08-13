

## Content of docker-compose.yml
For Powerplant Twin


`
version: '3.8'
services:
  database:
    container_name: mysql
    image: mysql/mysql-server:5.7
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_USER=root
      - MYSQL_PASSWORD=root
      - MYSQL_DATABASE=scadalts
    ports:
      - "3307:3306"
    volumes:
      - mysql-data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-uroot", "-proot"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 20s

  scadalts:
    container_name: scadalts
    image: scadalts/scadalts:latest
    depends_on:
      database:
        condition: service_healthy
    ports:
      - "8080:8080"
    expose:
      - "8000"

volumes:
  mysql-data:

`
