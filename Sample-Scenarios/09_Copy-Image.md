# Subject

we want to copy already built image from server A and paste it in server B. 



## Create Image on Server A

in server A  we want to create is a customized `mssql` image that when a container is run from it, a shell script is run after `sql` server is started. this shell script execute every `sql` scripts inside  directory `/sql-scripts` of the container.



the project structure is as follow:

```
.
├── docker-compose.yml
└── images
    └── mssql
        ├── Dockerfile
        └── init-sql.sh
```



content of `init-sql.sh` is as follow:

```shell
#!/bin/bash
set -e

# Start SQL Server in the background
/opt/mssql/bin/sqlservr &

# Wait for SQL Server to start up
echo "Waiting for SQL Server to start..."
sleep 20

# Get SA password from environment variable
SA_PASSWORD=${MSSQL_SA_PASSWORD}

# Run all SQL scripts inside /sql-scripts
for f in /sql-scripts/*.sql
do
  if [ -f "$f" ]; then
    echo "Running $f..."
    /opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P "$SA_PASSWORD" -i "$f" -N -C
  fi
done

# Keep container running
wait
***

#!/bin/bash
set -e

# Start SQL Server in the background
/opt/mssql/bin/sqlservr &

# Wait until SQL Server is ready
echo "Waiting for SQL Server to start..."
until /opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P "$MSSQL_SA_PASSWORD" -Q "SELECT 1" -b -N -C 2>/dev/null; do
  sleep 5
done

echo "SQL Server is up. Running initialization scripts..."

# Run all SQL scripts
for f in /sql-scripts/*.sql
do
  if [ -f "$f" ]; then
    echo "Running $f..."
    /opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P "$MSSQL_SA_PASSWORD" -i "$f" -N -C
  fi
done

# Keep the container running with sqlservr in the foreground
wait
```



`Dockerfile` content is as follow:

```dockerfile
FROM mcr.microsoft.com/mssql/server:2022-latest

USER root

COPY ./init-sql.sh /usr/src/app/init-sql.sh
RUN chmod +x /usr/src/app/init-sql.sh

USER mssql

# Use init-sql.sh as the main entrypoint
ENTRYPOINT ["/usr/src/app/init-sql.sh"]
```



docker compose file content is as follow:

```yaml
services:
  innovation.sqlserver:
    image: my-mssql
    container_name: innovation.sqlserver
    build:
      context: ./images/mssql/
      dockerfile: Dockerfile
    environment:
      ACCEPT_EULA: "Y"
      MSSQL_SA_PASSWORD: 'Innovation158@abv!'
    ports:
      - "1433:1433"
    volumes:
      - ./images/rest_api/dataBase/scripts/:/sql-scripts:ro
    restart: unless-stopped
```



by executing the following command in the root of the project, an image named `my-mssql` will be created and a container named `innovation.sqlserver` will run:

```shell
docker compose up -d 
```



to check if image is create or not, execute following command:

```shell
docker images
```



you should see following image:

```shell
REPOSITORY                        TAG           IMAGE ID       CREATED        SIZE
my-mssql                          latest        63b695605aa3   3 weeks ago    1.61GB
```



## Copy Image from Server A to Server B

now that we have created image `my-mssql` on server A, if we execute command `docker compose up -d` again, it will not create the image again and will use created one. 

so what if we want the project in server B and execute the command `docker compose up -d`? this will result in creating image again, but we do not want the image be created, and instead we want to reuse the created one in server A in server B.

to do so, we can copy the image in server A by executing following command:

```shell
docker save -o my-mssql.tar my-mssql:latest
```

 above command will create a file named `my-mssql.tar` where-ever the command is executed, that contains all layers and metadata.

now we can copy `my-mssql.tar` file from server A to server B with what-ever tools we know (for example by using `scp` program, or using `MobaXTerm`). after copying has been finished, we should add the image to our image list in docker. to do so execute the following command:

```shell
docker load -i /path/to/my-mssql:latest
```



to check if image is added or not, execute following command in server B:

```shell
docker images
```



you should see following image:

```shell
REPOSITORY                                     TAG                      IMAGE ID       CREATED        SIZE
my-mssql                                       latest                   63b695605aa3   3 weeks ago    1.61GB
```



now you can run a container from this image in server B, and executing command `docker compose up -d` will use it instead of building new one.  