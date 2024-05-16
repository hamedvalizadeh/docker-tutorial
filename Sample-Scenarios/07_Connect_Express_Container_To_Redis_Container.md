# Subject

we want to:

1. run a mock REST API with `mockoon` application.
2. create a `network` named `net-node-redis` in docker.
3. run a `redis` container named `rc-t` and connect it to `net-node-redis` network.
4. create a web API with `nodejs` that cache mock API in `redis` container.
5. create Web API docker image tagged `img-node-redis`.
6. run a docker container named `nrc-t` from Web API app and connect it to `net-node-redis` network.



***

# Mock API

to have an API we use `mockoon`. go to the download page of the site   [](https://mockoon.com) and based on your OS and its architecture, download the proper installable file, and install it.

as our test host is Linux OS with `arm64` architecture, we download the `arm64 deb` package and navigate to the location of downloaded file in our host machine and run the following command:

```powershell
dpkg -i mockoon-8.1.1.arm64.deb
```

​    

after installation completed, open `mockoon` application and run an existing Mock API or create a one for yourself and run it. in our example we run an API with `GET` verb and `/template` as route. 



**Caution:** to find the architecture of your running OS you can execute the command `dpkg --print-architecture`.



then get the IP of your host by executing `ifconfig`; say our host machine IP is `192.168.56.102`. then be sure that you can call your you mock REST API from your local machine, with host IP. 

in our test machine the complete mock API URL is as follow:

- http://192.168.56.102:3000/template



***

# Create docker Network

execute following command to create a network of type `bridge` in docker, named `net-node-redis`.

```powershell
docker network create net-node-redis
```

 

to test if the network is created, run the following command:

```powershell
docker network ls
```



you should see your network named `net-node-redis` in the list.



***

# Run `redis` container

`redis` is a database used most of the times as cache database. we want to run a container of this tool and name it `rc-t` and connect it to network `net-node-redis` we created before.

to do so, execute following command:

```powershell
docker run -d --name rc-t --network net-node-redis -p 7000:6379 redis
```



each container that is connected to the network `net-node-redis`, could connect to this `redis` container by using its name `rc-t` as host address and `6379` as port number.



**HINT:** specifying port forwarding with `-p 7000:6379` will let you to connect to running container `rc-t` from out side of the network `net-node-redis` by using port `7000`. for example from host machine by using `redis-cli` as follow:

- ```powershell
  redis-cli -h localhost -p 7000
  ```

- ```powershell
  redis-cli -h 192.168.56.102 -p 7000
  ```



we do not need to connect to `redis` container named `rc-t` from out side of network `net-node-redis`, so you can remove port forwarding portion `-p 7000:6379` from the above command; 



**HINT:** if you remove port forwarding portion `-p 7000:6379` from the above command, you will not lose the ability to connect to this `redis` container by using its name `rc-t` as host address and `6379` as port number from inside network `net-node-redis`.



**HINT:** to be able to use `redis-cli` in you host machine, you need to install `redis-tools`.



***

# Create REST Web API

before you can be able to create the REST API web app, its recommended to install `nodejs` in your host machine. it is not needed to run the app, as we want to containerize it, but we need `npm` package manager to develop our app, this package manager is included in `nodejs`.



to create web API first create a directory named `dir-node-redis` and navigate to it by running following commands:

```powershell
mkdir dir-node-redis
cd dir-node-redis
```



now execute the following command:

```powershell
npm init -y
```



after your project initiated, run the following commands to add necessary packages to your project:

```powershell
npm install express
npm install redis
npm install axios
npm install circular-json
```

  

**HINT:** you do not need to install above packages locally if you don't want run REST API web app directly from you host machine. in this case just copy the content of following box to your  `package.json` file inside directory `dir-node-redis`.



after these commands have been executed successfully, inside directory `dir-node-redis`, you should have a file named `package.json` with following content:

```json
{
  "name": "rn-web-app",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "axios": "^1.6.8",
    "circular-json": "^0.5.9",
    "express": "^4.19.2",
    "redis": "^4.6.13"
  }
}
```

 

now create a file named `index.js` and copy following codes to it:

```javascript
const express = require('express');
const redis = require('redis');
const axios = require("axios");
const CircularJSON = require("circular-json");

const app = express();

const PORT = 5000;
const REDIS_IP = 'rc-t';
const REDIS_PORT = 6379;

let redisClient;
(async () => {
    const url = `redis://${REDIS_IP}:${REDIS_PORT}`;
    redisClient = redis.createClient({
        url
    });

  redisClient.on("error", (error) => console.error(`Error : ${error}`));

  await redisClient.connect();
})();

async function fetchApiData() {
    const apiResponse = await axios.get(
      `http://192.168.56.102:3000/template`
    );
    console.log("Request sent to the API");
    return apiResponse;
}

async function getSpeciesData_old(req, res) {
    let results;
  
    try {
      results = await fetchApiData();
      console.log('results', results);
      const str = CircularJSON.stringify(results);
      res.send({
        fromCache: false,
        data: JSON.parse(str),
      });
    } catch (error) {
      console.error(error);
      res.status(404).send("Data unavailable");
    }
 }

 async function getSpeciesData(req, res) {
    const species = 'users';
    let results;
    let isCached = false;
  
    try {
      const cacheResults = await redisClient.get(species);
      if (cacheResults) {
        isCached = true;
        results = JSON.parse(cacheResults);
      } else {
        results = await fetchApiData();
        if (results.length === 0) {
          throw "API returned an empty array";
       }
       await redisClient.set(species, CircularJSON.stringify(results));
      }
      const str = CircularJSON.stringify(results);
      res.send({
        fromCache: isCached,
        data: JSON.parse(str),
      });
    } catch (error) {
        console.error(error);
        res.status(404).send("Data unavailable");
    }
  }

app.get('/', getSpeciesData);

app.listen(PORT, () => {
    console.log(`App is running on port: ${PORT}`);
});
```



our REST web API is ready and could be run directly in your host machine. in order to run it locally, you need to change value of variable `REDIS_IP` to `localhost` or the IP of your host machine that in our running host machine is `192.168.56.102` and change the value of variable `REDIS_PORT` to `7000` and finally execute the command `node index.js`.

but as we want to containerize this REST API, we do not change these variables and will add some files to this project in next section, to be able to dockerize it.



***

# Create Web API docker Image

create file named `Dockerfile` inside directory `dir-node-redis` and copy following codes to it:

```dockerfile
# Fetching the minified node image on apline linux
FROM node:slim

# Declaring env
ENV NODE_ENV development

# Setting up the work directory
WORKDIR /express-docker

# Copying all the files in our project
COPY . .

# Installing dependencies
RUN npm install

# Starting our application
CMD [ "node", "index.js" ]

# Exposing server port
EXPOSE 5000
```



create file named `.dockerignore` inside directory `dir-node-redis` and copy following codes to it:

```dockerfile
node_modules
npm-debug.log
Dockerfile
.dockerignore
```



now everything is ready to create the image, so execute the following command to create the image tagged `img-node-redis`:

```powershell
docker build -t img-node-redis .
```



after the execution of the above command finished, run the following command to see if your image tagged `img-node-redis` is listed in the images list.



***

# Run Web API Container

to do so, execute following command:

```powershell
docker run -d --name nrc-t --network net-node-redis -p 5000:5000 img-node-redis
```



now open a web browser and navigate to one of the following addresses :

- http://localhost:5000
- http://192.168.56.102:5000



for first call, in response you will see that the value of property `fromCache` is `false`, because it fetched the data from mock API `http://192.168.56.102:3000/template`, but for the other calls the value of property `fromCache` is `true`, because the data is fetched from the `redis` database in `rc-t` container.