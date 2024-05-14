# Subject

we want to:

1. run a Redis container in detach mode
2. test our Redis container is working properly
3. connect to it from host machine



***

# Run Container

in order to be able to run the Redis container we need to pull Redis image. to pull it we have 2 options at least, the first one it to use `pull` command, the second is to use `run` command. the `run` command will pull the image if it does not exist locally. so run thr following command:

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

# connect to it from host machine

to do so, first of all you need to have `redis-cli` installed in your host machine. after that run following command:

```powershell
redis-cli -h localhost -p 6379
```



now as we setup our Redis container to force the user to authenticate with password, we need to execute command `auth 123`; if our `123` is a correct password which it is, it should say `ok` in response.

now execute command `ping`, in response if it says `PONG`, the running Redis container is working as expected.
