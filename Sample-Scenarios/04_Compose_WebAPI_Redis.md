# Subject

we want to:

1. create a web API that count your call to an end-point, and save it to the Redis database as cache. 
2. create image of this API.
3. run the API container and Redis container in an isolated environment.



***

# Create API

create a directory named `compose-test-py-hello-api` and navigate to it:

```powershell
mkdir compose-test-py-hello-api
cd compose-test-py-hello-api
```



create a file named `app.py` and paste the following code in it:

```python
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```



as you can see, out app depends on two libraries, `redis`, and `flask`; so when we want to install this dependencies with `pip` command of linux, wen need to have following content inside a file named `requirements.txt` beside file `app.py`:

```python
flask
redis
```



***

# Create Image of API

to create image from our project, wen need to have `Dockerfile`, so create a file named `Dockerfile` in root directory of the project which in our example it is a folder named `compose-test-py-hello-api`, and copy the following content to it:

```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.10-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run", "--debug"]
```



in the first line of the above file, we mentioned that we want to create our image based on version `3.10-alpine` of `python` image. then inside our image we changed our directory to `code`, then created two environments and installed `gcc` and other dependencies, then copied `requirements.txt` file from directory `compose-test-py-hello-api` of the host to the directory `code` inside the image; in next layer we installed dependencies of our python API by using `pip` package manager. after that we exposed port `5000` of the container, copied all contents of directory `compose-test-py-hello-api` of our host to the `code` directory inside the image and finally specified a command to be run when starting container.



**Caution:** by running `docker build .` command inside directory `compose-test-py-hello-api` of the host machine the image will be build, but do not run it, because we want to it with `compose` and start our environment at once with a very simple command.



***

# Run Containers in Isolated Environment

  as it is mentioned in the `Subject` of this scenario, we have an API as a service that saves some data in Redis another service. so we have two interconnected services that have should interact with each other through docker networks. 

in order to have these services running correctly we have two options:

1. manual setup that consists of following steps:

   - to execute `build` command of the `docker` to create image of our API.

   - to create needed network manually.

   - to execute the command `run` with network specified in them separately for each of them to start containers.

2. automatic setup. that consists of one line of command and that is `docker compose up`. this command handles of three steps of manual setup. in order to be able to use `compose up` command, we need to have a file named `compose.yaml` inside directory `compose-test-py-hello-api` of our host machine. so create this file and paste following codes inside it:

   - ```dockerfile
     services:
       web:
         build: .
         ports:
           - "8000:5000"
       redis:
         image: "redis:alpine"		
     ```

     

in the `compose.yaml` file we specified two services named `web` that is a container of an image built from our `python` API that is exposed to port 5000 of container and port 8000 of host machine, and a container named `redis` which is based on image `redis:alpine`. the network needed for this 2 containers to communicate with each other is created by docker because of `compose` command.



to run the containers execute the following command:

```dockerfile
docker compose up
```



**Caution:** there are some conditions that after executing the above command, you need to change `Dockerfile`, and then re-call the above command. the problem is that, because the compose caches the files and re-calling this will not consider changes in `Dockerfile`, we need to use option `--build` to force the `compose` to build the image again. as follow:

```dockerfile
docker compose up --build	
```

  

now open a web browser and navigate to the URL `http://localhost:8000`. 



***

# HINTS

after running command `docker compose up` inside directory `compose-test-py-hello-api`, an image named `compose-test-py-hello-api-web` is created, that is the image of you Web API. 



to see all images related to the compose file you configured, first navigate to the folder `compose-test-py-hello-api` and execute command `docker compose images`. this command also shows you containers running based on these images. in our example this command will result in 2 images, one is named `compose-test-py-hello-api-web`, and the other one is `redis`.



to remove images and containers related to our compose environment, we could execute `docker compose down`.

 

in development environment it is boring to `down` the environment before any changes in the project and again `up` it manually. to confront this difficulty    we could set compose to watch our changes in project and update running containers automatically. 

to do so we need to change `composefile.yaml` as follow:

```dockerfile
services:
  web:
    build: .
    ports:
      - "8000:5000"
    develop:
      watch:
        - action: sync
          path: .
          target: /code
  redis:
    image: "redis:alpine"
```

now execute the command `docker compose watch`. 
