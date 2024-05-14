# Subject

we want to:

1. run a Redis container in detach mode with persistent data.
2. test our Redis container is working properly.
3. create a NodeJS web app to connect to Redis and count visits to the site.
4. run the web app and test it.



***

# Run Container

in order to be able to run the Redis container we need to pull Redis image. to pull it we have 2 options at least, the first one it to use `pull` command, the second is to use `run` command. the `run` command will pull the image if it does not exist locally. so run the following command:

```dockerfile
docker run -d --name my-redis-stack -p 6379:6379 -v /Users/my-redis/:/data -e REDIS_ARGS="--requirepass 123 --appendonly yes" redis/redis-stack-server:latest	
```



- 123 is authentication password for Redis we choose for our Redis config.
- `my-redis-stack` is the name of our running container.
- path `/Users/my-redis/` is in host machine to save data in Redis, to use it for another instance of Redis container if the running container fails and stop working.
- `redis/redis-stack-server:latest` this is the image we want to run a container from.



***

# Test

to test if the Redis container is working properly we need to connect to the Linux in which the Redis container is based on, then inside that Linux we need to run the Redis CLI and run the command `ping`, if it says `PONG`, the running container is working as expected.

1. first list the docker running containers by executing command `docker ps` and memorize first three characters of the container ID related to container named `my-redis-stack`; say it is `ec4`.
2. then open a shell process in that container by executing command `docker exec -it sh ec4`. by running this command, a shell process inside the Linux of the container will be opened and your context is switched to it.
3. execute command `redis-cli` in Linux shell process opened for you, to open Redis CLI for you.
4. now as we setup our Redis container to force the user to authenticate with password, we need to execute command `auth 123`; if our `123` is a correct password which it is, it should say `ok` in response.
5. now execute command `ping`, in response if it says `PONG`, the running Redis container is working as expected.



***

# create a NodeJS web app

before beginning, you need to have NodeJS installed.

create a directory named `node-web-app` by executing following command and navigate to it:

```powershell
mkdir node-web-app
cd node-web-app
```



now execute the following command:

```powershell
npm init -y
```



the above command will create a file named `package.json` with some predefined values for you, that is not important for us now. run following commands:

```powershell
npm install express@4.17.2
npm install redis@3.1.2
```



create a file named `index.js` and paste following code in it:

```javascript
const express = require('express');
// Import Redis
const redis = require('redis');
const app = express();
// 'createClient()' usually takes an URL connection path, or the path of a host to connect to.
// In our case use the name of the service from 'docker-compose.yml', 'test-redis'.
// Redis server itself usually runs on Port '6379'
const redisClient = redis.createClient({
  host: 'localhost',
  port: 6379,
  password: '123'
});

// GET route
app.get('/', function(req, res) {
    redisClient.get('numVisits', function(err, numVisits) {
        numVisitsToDisplay = parseInt(numVisits) + 1;
        if (isNaN(numVisitsToDisplay)) {
            numVisitsToDisplay = 1;
        }
        res.send('How much you visited this site: ' + numVisitsToDisplay);
        numVisits++;
        redisClient.set('numVisits', numVisits);
    });
});

// Listen on Port 5000 in Docker Container (mapped to Local Machine Port 80)
app.listen(5000, function() {
    console.log('Web app is listening on port 5000');
});

```



now open a web browser and navigate to the address `http://localhost:5000`, you will see a message saying you the count of site visit.



**Caution:** the key to be able to connect from host machine to Redis docker container is port forwarding `-p 6379:6379` when execute the command `run`. 
