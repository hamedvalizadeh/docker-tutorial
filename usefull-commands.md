#### Find Docker Compose of a container

```shell
docker inspect [container-name] | grep -A 3 "com.docker.compose"
```

